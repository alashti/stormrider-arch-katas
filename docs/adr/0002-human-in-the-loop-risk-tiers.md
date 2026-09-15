# Human-in-the-loop, tiered by risk profile rather than one-size-fits-all

## Status

Accepted

## Context

All six AI capabilities produce an output that could, in principle, act on the world without a human ever seeing it: an animal-health alert could trigger automated dosing, a maintenance risk score could take a ride offline on its own, a pricing model could change ticket prices in real time, an offer-generation model could message thousands of visitors unsupervised. Each capability's level-2 diagram (`docs/diagrams/ai-*.md`) independently arrived at a human checkpoint of some kind, but at different strengths: a hard gate for animal/plant health, a double gate for predictive maintenance, a dashboard-only checkpoint for crowd analytics, a guardrail-plus-approval for pricing, a template-level review for offers, and a plain checkout tap for the concierge's booking flow. Left implicit, this variation could look arbitrary or inconsistent to a reader (and to judges scoring "architectural characteristics match" and "dealing with AI uncertainty") — as if some capabilities got a human-in-the-loop treatment and others didn't.

## Decision

Human-in-the-loop is deliberately **tiered by risk profile**, not applied uniformly, using this principle: the strength of the human checkpoint is proportional to what's actually at stake if the AI is wrong, in this order — **welfare/safety > trust/reputation/money > operational efficiency**.

| Tier | Risk being managed | Capability | Checkpoint shape |
|---|---|---|---|
| Hard gate (single) | Living-thing welfare | c. Animal health, d. Plant health | Vet / horticulturist must confirm before any care/medical action; model never acts alone. |
| Hard gate (double) | Visitor physical safety | e. Predictive maintenance | Staff sign-off required both to take a ride offline *and* to return it to service. |
| Guardrail + business approval | Trust / reputation / revenue | f2. Revenue & retention (pricing) | Bounded automatic guardrail (max daily price movement), plus a business approval step — a PR/trust decision, not a safety one. |
| Template/campaign-level review | Reputation at scale | f1. Personalized offers & campaigns | Marketing/business staff approve offer *content* once per template/campaign (not per message — operationally impossible at 15,000 visitors/day), then an automated guardrail (frequency/tone) checks each instantiated message. |
| Decision support (dashboard) | Operational efficiency, no independent ground truth | b. Crowd & popularity analytics | Heatmap/flow map informs staff decisions; nothing is auto-dispatched. |
| Customer's own action | Normal commercial transaction | a. Visitor concierge (booking) | The visitor's own "Confirm & Pay" tap — identical to any booking website's checkout, not a staff approval gate at all. |

## Consequences

- **Makes the risk-based reasoning explicit** rather than requiring a reader to infer it by comparing six separate diagrams — directly speaks to the judging criteria on architectural fit and handling of AI uncertainty.
- **Prevents the design from drifting toward whichever gate is easiest to build.** Any future capability added to this architecture should be placed in this table before its diagram is drawn, not after — the tier should drive the diagram, not the other way around.
- **Risk of looking over-engineered for the lightest tier.** The concierge's "checkpoint" is just a checkout button — calling that out as a formal tier could read as padding. It's kept explicit anyway because it was a genuine point of confusion earlier in this design (see `docs/diagrams/ai-visitor-concierge.md`'s "Why this shape" section) and because contrasting it with the heavier tiers is exactly what makes the tiering principle legible.
- **The table is a live document, not a one-time decision.** If a capability's diagram changes its human checkpoint, this ADR needs a follow-up (or a superseding ADR) — it is the single source of truth for "why does capability X have this much human oversight," and letting it drift out of sync with the diagrams would undermine its own purpose.
