# Split "Revenue & Retention" into a backend engine (f2) and a customer-facing offers capability (f1)

## Status

Accepted

## Context

The brief frames "revenue & retention" as one AI capability: forecasting demand to drive dynamic pricing, and predicting churn to drive repeat-visit campaigns and incentives. In practice this covers two very different pieces of work: a backend forecasting/modeling engine consumed by staff and the ticketing system (demand forecasting → pricing/bundling, churn prediction → at-risk segments), and a customer-facing content-generation-and-delivery capability that turns "at-risk segment" into an actual message a visitor receives (loyalty rewards, seasonal campaigns, win-back offers). These two halves have different consumers, different risk profiles, and — per ADR-0002 — different human-in-the-loop checkpoints (a bounded pricing guardrail + business approval for the backend engine's pricing output, vs. template/campaign-level marketing review for the customer-facing offer content).

## Decision

Split capability `f` into two separately-diagrammed **surfaces of the same capability domain** — not a seventh capability; the design still refers to "six capabilities," one of which (`f`) has two views — that share one hand-off contract:

- **`f2` — Revenue & retention engine** (backend, staff-facing): owns demand forecasting and churn prediction. Its outputs are pricing/bundling recommendations (to the ticketing system, gated by a guardrail + business approval) and at-risk/target visitor **segments**, handed to `f1`. See `docs/diagrams/ai-revenue-retention.md`.
- **`f1` — Personalized offers & campaigns** (customer-facing): consumes `f2`'s segments only — it does not run its own churn/targeting model. It owns offer-template generation, human (marketing/business) review at the template/campaign level, per-visitor instantiation, an automated frequency/tone guardrail, and delivery. See `docs/diagrams/ai-offers-campaigns.md`.

The two stay connected by a single hand-off: `f2` decides **who** is at risk or a target; `f1` decides **what** to say to them and whether it's fit to send. Both close back into the same A/B-testing validation loop (revenue and return-visit lift), which originates in `f1`'s delivered offers but feeds back to recalibrate both models.

## Consequences

- **One source of truth for "who's at risk."** Splitting the capabilities without sharing the segmentation model would risk two independent churn/targeting models disagreeing with each other — this decision explicitly rules that out by making `f1` a consumer of `f2`'s segments, not a second segmentation engine.
- **Each half can be validated and iterated independently**, matching its own risk profile and human-in-the-loop tier (ADR-0002) — a pricing guardrail change doesn't require touching offer-template review workflows, and vice versa.
- **Creates a hand-off contract that must stay in sync.** If `f2` changes what a "segment" contains (new fields, different granularity), `f1`'s template generation depends on that shape — this is a coupling point between two otherwise-independent diagrams that needs to be tracked explicitly (e.g., a shared schema), not left implicit.
- **Matches how the diagrams were actually built** (`ai-revenue-retention.md` stops at "hands segments to f1"; `ai-offers-campaigns.md` picks up exactly there) — this ADR documents a decision already reflected in both files, rather than proposing new structure.
