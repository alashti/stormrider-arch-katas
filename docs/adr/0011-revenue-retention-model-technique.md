# Revenue & retention: forecasting and churn model choice for capability f2

## Status

Accepted

## Context

`docs/diagrams/ai-revenue-retention.md` names two models (demand forecasting → pricing; churn prediction → segments) without committing to technique. Both models' outputs are gated by a human checkpoint before they affect anything real (pricing guardrail + business approval; segments handed to f1's own reviewed pipeline), which matters for the choice: the technique needs to produce a result a business owner can actually reason about and approve/reject, not just a number to trust blindly.

## Options considered — demand forecasting

| Option | Pros | Cons |
|---|---|---|
| **Classical time-series (SARIMA / Prophet-style)** | Well-understood, interpretable seasonal decomposition (weekday/weekend, seasonal park attendance patterns); works with relatively little history; easy to explain to business staff approving a price move | Struggles to incorporate exogenous signals (weather, local events, the crowd-analytics heatmap) as naturally as a feature-based model |
| **Gradient-boosted trees with exogenous features** (day-of-week, season, weather, crowd-analytics heatmap, local events) | Directly incorporates the heatmap/other signals already available from capability b; handles nonlinear interactions; strong accuracy-per-effort in practice | Less naturally "explains" a smooth seasonal trend than a dedicated time-series decomposition; still needs a reasonable history to train well |
| **Deep learning (temporal fusion transformer or similar)** | Highest ceiling on accuracy with enough data and enough exogenous signals | Needs the most data and MLOps maturity; least explainable — a poor fit for a guardrail+business-approval workflow that depends on staff being able to reason about *why* a price change is recommended |

## Options considered — churn prediction

| Option | Pros | Cons |
|---|---|---|
| **Logistic regression on visit-history features** | Fully interpretable coefficients (useful for f1's marketing staff reviewing why a segment was targeted); simple to validate against A/B outcomes | Misses nonlinear interaction effects between features |
| **Gradient-boosted trees (classification)** | Better accuracy on nonlinear patterns; feature importances still give some interpretability | Less directly interpretable than logistic regression's coefficients |
| **Survival analysis (time-to-churn)** | Answers "when" not just "if," useful for timing win-back campaigns | Adds complexity that may not be needed if f1 mainly needs "who," not precise timing |

## Decision

- **Demand forecasting**: **gradient-boosted trees with exogenous features**, explicitly wired to consume the crowd-analytics heatmap (`ai-crowd-analytics.md`) as an input feature — this reuses an output the design already produces rather than forecasting demand in a vacuum, and stays interpretable enough (via feature importances) for the business-approval step in `ai-revenue-retention.md`'s guardrail.
- **Churn prediction**: **gradient-boosted trees (classification)** as the primary model, with **logistic regression kept as a simpler, more interpretable secondary model specifically to justify segment membership to marketing staff** reviewing offer templates (per `ai-offers-campaigns.md`'s human-review gate) — accuracy from the GBM, explainability from the simpler model where a person needs to understand "why is this visitor in this segment."

## Consequences

- **Both models directly consume outputs the design already produces** (the heatmap for forecasting) rather than duplicating signal collection — consistent with ADR-0005's principle that `f2` and `b` shouldn't independently re-derive the same information.
- **Running two churn models (GBM + logistic regression) is deliberate redundancy, not waste** — the interpretable model exists specifically to serve the human-review step already required by ADR-0002, not as a backup in case the GBM fails.
- **Neither choice is the theoretical maximum-accuracy option (deep learning)** — the same trade-off as ADR-0009 (predictive maintenance): explainability for a human-in-the-loop, guardrail-gated decision is weighted above squeezing out marginal accuracy gains a business-approval workflow couldn't act on anyway.
- **A/B testing (already the validation loop in `ai-revenue-retention.md`) is what ultimately justifies either model choice** — feature importance and coefficients explain a recommendation, but actual revenue/return-visit lift from the A/B loop is the real evidence these techniques are working, not a backtest alone.
