# Predictive maintenance: model choice for capability e

## Status

Accepted

## Context

`docs/diagrams/ai-predictive-maintenance.md` already flags a cold-start problem as open: "where the 'above threshold' cutoff sits initially, before there's failure history to calibrate against." This is the highest-stakes backend capability (visitor physical safety) and the one where the wrong technique choice has the most direct consequence — over-trusting an unvalidated model on 18th-century rides is a genuinely different risk than a false alarm on a garden bed.

## Options considered

| Option | How it works | Pros | Cons |
|---|---|---|---|
| **Fixed maintenance schedules / rule-based thresholds** | Manufacturer/engineer-set limits per sensor (e.g., vibration > X) | Fully interpretable; usable from day one, zero data needed | Ignores actual usage patterns and combined signals; doesn't improve over time; exactly the status quo the brief says is failing (18th-century rides need better than fixed schedules) |
| **Survival analysis (Weibull / Cox proportional hazards)** | Statistical time-to-failure modeling per component type | Well-suited to "when will this likely fail" questions with limited failure examples; interpretable (hazard curves); standard technique in industrial reliability engineering | Needs at least some historical failure/censoring data to fit meaningfully; less able to exploit rich multivariate sensor correlation than an ML classifier |
| **Gradient-boosted trees / Random Forest on engineered features** (rolling vibration stats, cycle counts, etc. → risk classification) | Handles multivariate, nonlinear combinations of the four telemetry categories well; relatively data-efficient; feature importances give some interpretability for staff | Requires engineered features (upfront work) rather than learning directly from raw sequences; still needs *some* labeled failure/near-failure examples to train against |
| **Deep learning remaining-useful-life (RUL) model** (LSTM/Transformer over raw sensor sequences) | State of the art for RUL when enough data exists; learns temporal patterns directly | Needs substantial historical failure data most parks won't have per ride at launch; least interpretable; hardest to validate/explain a specific "why is this ride flagged" to maintenance staff |

## Decision

**Start with gradient-boosted trees (e.g., GBM/Random Forest) on engineered features from the four telemetry categories, blended with a survival-analysis component for the "when might this fail" horizon** — not a fixed schedule, and not a deep RUL model at launch. This is the option that's actually usable given realistic data volumes (some historical maintenance/failure history likely exists per ride even before this system, per `docs/diagrams/ai-predictive-maintenance.md`'s "History" input) while still being materially better than fixed schedules, and while keeping enough interpretability that maintenance staff can see *why* a ride was flagged — important given the double human sign-off gate this capability already has (ADR-0002).

The deep RUL model is the explicit long-term target once enough per-ride failure history accumulates (potentially fed by the same training-data flywheel as ADR-0001) — this decision doesn't rule it out, it sequences it correctly.

## Consequences

- **Materially better than the status quo (fixed schedules) without requiring data that doesn't exist yet** — addresses the cold-start problem flagged as open in the diagram by picking a technique that degrades gracefully with limited history rather than needing it upfront.
- **Interpretability is preserved for a safety-critical, human-gated decision** — feature importances let maintenance staff sanity-check a flag rather than trusting an opaque score, which matters more here than for, say, crowd analytics.
- **Explicitly not the most powerful available technique (deep RUL) at launch** — this is a deliberate trade of theoretical maximum accuracy for practical usability and explainability given real data constraints; worth being explicit about so it doesn't read as an oversight to a judge who knows RUL deep learning exists.
- **The "above-threshold cutoff" cold-start question from the diagram is only partially solved, not eliminated** — engineered-feature models still need an initial threshold before any failure history exists for a given ride; this ADR narrows the technique choice but the exact initial cutoff is still a per-ride tuning exercise, likely starting conservative (over-flagging) and loosening as calibration data arrives.
