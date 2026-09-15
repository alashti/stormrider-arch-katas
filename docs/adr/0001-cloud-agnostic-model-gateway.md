# Cloud-agnostic AI model gateway, with a human-feedback flywheel toward an owned model

## Status

Accepted

## Context

Every one of the six AI capabilities (visitor concierge, crowd analytics, animal health, plant health, predictive maintenance, revenue & retention) needs access to AI/ML models, but the specific providers or model vendors available to us — and their pricing, availability, and terms — are likely to change over a multi-year deployment. Committing each capability to call a specific vendor's API directly means a provider price change, deprecation, or shutdown becomes six separate emergency rewrites instead of one.

Separately, every capability already produces a human-in-the-loop decision as part of its normal operation: a vet or horticulturist confirms or dismisses a health alert, maintenance staff sign off on taking a ride offline or back into service, business staff approve a pricing move, marketing staff approve an offer template. These decisions are currently used only to recalibrate that capability's own thresholds in place (see `docs/diagrams/ai-animal-health.md`, `ai-predictive-maintenance.md`, `ai-revenue-retention.md`, `ai-offers-campaigns.md`). That data has a second, unused use: as labeled training data.

## Decision

1. **No AI capability calls a model provider directly.** All six route through a single **AI model gateway** (`docs/diagrams/cloud-platform.md`) that owns a router with a fallback chain across generically-named providers ("Provider A/B/C" — deliberately not naming a real vendor at this stage of the design). If a provider is slow, errors, or exceeds a cost threshold, the router falls back to the next provider in the chain. Swapping, reordering, or dropping a provider is a change in one place (the router), not a change in six capabilities.
2. **Every human-in-the-loop decision across all six capabilities is captured centrally as labeled training data** (a `TrainingData` store in `cloud-platform.md`), not just consumed locally to nudge a threshold. This feeds a periodic training/fine-tuning pipeline that produces an **owned model**, hosted on-premises or in the estate's own cloud account.
3. **The owned model becomes an additional candidate in the router's chain**, alongside Providers A/B/C — not a forced replacement. It only gets promoted from "candidate" to something the router actually routes production traffic to once it clears the same validation bar every other provider is implicitly held to (accuracy/agreement with human decisions, latency, cost).

## Consequences

- **Directly and increasingly answers the "AI provider uncertainty" judging criterion.** On day one, this design tolerates a provider disappearing (fallback chain). Over time, it actively reduces dependency on any third-party provider at all, because the estate accumulates its own domain-specific model trained on its own vets', horticulturists', and staff's actual decisions — data no external vendor has access to and can't replicate.
- **Reuses data the design already produces.** The training-data flywheel doesn't require new sensors or new instrumentation — every confirm/dismiss and approval loop already drawn into the level-2 diagrams is the source.
- **New cost: an abstraction layer to build and maintain.** The gateway itself (routing, fallback logic, cost/latency tracking) is infrastructure that doesn't exist in a "just call the API" design — this is deliberate overhead in exchange for portability.
- **New cost: real MLOps and data-governance responsibility.** Training and hosting an owned model (on-prem or in an owned cloud account) means the estate now owns infrastructure, retraining cadence, and versioning it didn't own before — plus responsibility for how sensitive operational data (visitor history feeding `f2`/`f1`, animal-health records) is retained, access-controlled, and (where it touches visitor data) anonymized in that training store.
- **Cold-start period.** An owned model starts with no track record. It has to be evaluated against the same validation bar as any other candidate before the router trusts it with real traffic — "it's ours" is not a shortcut past that; this is a multi-month-or-longer trajectory, not something available at launch.
- **Observability has to cover four candidates, not three.** The drift/confidence monitoring and cost tracking already planned at the gateway level (per `cloud-platform.md`) now also has to evaluate the owned model on the same terms as the third-party providers, or the "candidate, once validated" promotion has no real gate behind it.
