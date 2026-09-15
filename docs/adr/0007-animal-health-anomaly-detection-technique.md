# Animal health: anomaly detection technique for capability c

## Status

Accepted

## Context

`docs/diagrams/ai-animal-health.md` names "anomaly detection over feeding/weight/water-quality telemetry" and "CV behavior classification" as the two model types feeding the per-enclosure risk score, but doesn't commit to *which* anomaly-detection technique. With 55 enclosures, each with multiple correlated sensor streams (feeder load, water quality, terrarium environment), and species-specific "normal" ranges that vary widely (a piranha tank's healthy water chemistry looks nothing like a terrarium's), the technique needs to handle multivariate, per-species baselines without requiring a hand-tuned rule set per enclosure.

## Options considered

| Option | How it works | Pros | Cons |
|---|---|---|---|
| **Static thresholds per species** | Fixed min/max bands per sensor, set by vet knowledge | Fully interpretable; zero training data needed at launch | Doesn't catch multivariate anomalies (each reading "in range" but the combination is wrong); brittle, needs manual retuning per species/season |
| **Isolation Forest / One-Class SVM** | Unsupervised model learns a "normal" envelope per enclosure from historical multivariate telemetry, flags points far from it | Handles multivariate correlation; no labeled failure data needed to start; relatively cheap to (re)train per enclosure | Less interpretable than thresholds; needs a reasonable baseline history before it's reliable (cold start) |
| **Autoencoder reconstruction error** | Neural net learns to reconstruct normal telemetry; high reconstruction error = anomaly | Captures complex nonlinear relationships across sensors; improves as more data accumulates | Needs more data and compute than isolation forest; harder to explain a specific flag to a vet ("why did it fire?") |
| **Temporal model (LSTM/Transformer) over sensor sequences** | Learns normal *trends over time*, not just point-in-time values | Catches slow-building issues (e.g., gradually declining water quality) that point-in-time models might miss until late | Highest data and compute requirement; longest to become trustworthy; overkill relative to the alert latency actually needed here |

## Decision

Start with **Isolation Forest (or an equivalent one-class model) per enclosure-type, seeded with static per-species threshold bands as a fallback for enclosures with insufficient history.** This is deliberately the middle option: it handles the multivariate case that thresholds alone miss, doesn't require labeled failure examples (which won't exist at launch — animals don't reliably get sick on a training schedule), and is cheap enough to retrain frequently as the vet's confirm/dismiss feedback (`docs/diagrams/ai-animal-health.md`) accumulates.

A temporal model is explicitly **not** ruled out — it's the natural upgrade path once enough calibrated history exists per enclosure (feeding the training-data flywheel from ADR-0001), particularly for slow-building issues the point-in-time model would catch late.

## Consequences

- **Fast to stand up without historical failure data**, at the cost of some sensitivity to slow-onset issues until enough per-enclosure history accumulates.
- **The vet's confirm/dismiss loop is the calibration mechanism** (as already designed) — but with an unsupervised model, "confirm" and "dismiss" feedback tunes the anomaly *threshold*, not the model's learned notion of "normal" directly; a periodic retrain (weekly/monthly, not real-time) is needed to actually incorporate new normal/abnormal examples into the model itself.
- **Species diversity means this isn't one model — it's one model per enclosure-type/species family**, which is more moving parts to maintain than a single global model, but is necessary given how different a healthy piranha tank looks from a healthy terrarium.
- **Leaves an explicit upgrade path** (temporal modeling) rather than closing it off, so this decision doesn't need to be revisited from scratch once more data exists — only extended.
