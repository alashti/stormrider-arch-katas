# Overview

> Deliverable #1: a *short* narrative describing how the team used AI to solve the problems of the Von Digitalis Estates.

## The system, at a glance

A cloud-agnostic platform sitting on top of a patchy-wifi estate: four physical zones (rides, animal enclosures, gardens & the carnivorous plant collection, ticket gates), each with its own edge gateway that buffers locally and syncs opportunistically ([`onsite-edge.md`](diagrams/onsite-edge.md)). Everything that reaches the cloud lands in one platform — ingestion, storage, and a single **AI model gateway** every capability routes through, with a provider fallback chain and, over time, an owned model trained on the estate's own human-reviewed decisions ([`cloud-platform.md`](diagrams/cloud-platform.md)).

## Where AI shows up, and why there

Six distinct AI capabilities, split by who they serve ([`system-overview.md`](diagrams/system-overview.md)):

- **Customer-facing**: a **visitor concierge** (the one genuinely conversational capability — built agentic, with tool calls and RAG grounding, deliberately not the same technique as everything else), and **personalized offers & campaigns** (marketing-reviewed content, targeted using segments from the revenue engine).
- **Backend**: **crowd analytics** (sensor fusion + flow modeling — where's busy, and how do people move), **animal health** and **plant health** (anomaly detection + computer vision, same shape for both since they share a welfare risk profile, alerting a vet or horticulturist rather than acting alone), **predictive maintenance** (the highest-stakes capability — double staff sign-off before a ride goes offline or back into service), and the **revenue & retention engine** (demand forecasting and churn prediction, feeding pricing and the offers capability).

Each capability uses the AI technique that actually fits its problem — anomaly detection, CV, sensor fusion, forecasting, or an agentic LLM — rather than one technique applied everywhere (`docs/adr/0004`, `0007`–`0011`). And every capability's human checkpoint is sized to what's actually at risk if the AI is wrong: a hard gate for animal/plant welfare and ride safety, a guardrail-and-approval for pricing, a review gate for offer content, a dashboard for staffing decisions, and — for the visitor's own ticket purchase — nothing more than the same "Confirm & Pay" tap any booking site already requires (`docs/adr/0002`, `0006`).

## The single biggest win

**For the Countess**: turning "we have no real idea what's popular, or whether the animals are actually okay" into continuous, validated signal — crowd flow and per-attraction dwell time to guide staffing and investment, and health monitoring that never lets a model act alone but does mean a vet or horticulturist finds out about a problem before it becomes a crisis (or a lost animal, or a lost plant collection). Predictive maintenance turns the estate's biggest safety liability — 18th-century rides — into something proactively managed instead of reactively inspected.

**For visitors**: a concierge that actually knows the park (grounded, not guessing) and can book a ticket end-to-end with no more friction than any commercial booking site — plus offers that are actually relevant because they're built from real visit history, not blasted at everyone.

**The uncertainty answer, in one line**: no capability ever talks to an AI provider directly — everything goes through one gateway with a fallback chain, and every human-reviewed decision across all six capabilities doubles as training data for an owned model that reduces third-party dependency over time (`docs/adr/0001`). Concrete, per-capability confidence thresholds, drift triggers, release gates, rollback and fallback criteria, and escalation paths are fixed in `docs/adr/0013` — the direct answer to "how will you know if the AI starts misbehaving in production."

Ticket purchasing and family passes get their own standalone domain view ([`ticketing-gate.md`](diagrams/ticketing-gate.md)) beyond the concierge's booking tool calls — inventory, payment failure handling, refunds, issuance, and gate validation (including offline behavior during a connectivity gap). Rough capacity/scale assumptions for the 5,000 → 15,000 visitors/day growth target are in `docs/adr/0014`.

Full depth is in [`docs/diagrams/`](diagrams/) (comprehensive + per-capability views, including level-3 agent internals for the concierge, and the standalone ticketing/gate view) and [`docs/adr/`](adr/) (15 decision records with trade-off analysis).
