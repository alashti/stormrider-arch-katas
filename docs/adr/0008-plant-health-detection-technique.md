# Plant health: detection technique for capability d

## Status

Accepted

## Context

`docs/diagrams/ai-plant-health.md` deliberately mirrors animal health's shape (telemetry anomaly + CV detection → risk score → horticulturist gate), but the plant collection has a distinct wrinkle: the carnivorous plant collection is explicitly named in the brief as a rare, valuable asset, which likely means **more species diversity and fewer labeled examples per species** than the 55 animal enclosures have. A CV disease/pest model here can't assume large labeled training sets per species the way a generic garden-center model might.

## Options considered — imagery (disease/pest detection)

| Option | Pros | Cons |
|---|---|---|
| **Rule-based image heuristics** (color/texture thresholds for wilting, discoloration) | No training data needed; fully interpretable | Misses subtle or novel disease patterns; brittle across species with naturally unusual coloring (many carnivorous plants aren't uniformly green) |
| **CNN classifier trained per species** | Good accuracy once trained | Needs meaningful labeled data *per species* — exactly what's scarce for a rare/diverse collection; doesn't generalize to a species it hasn't seen |
| **Fine-tuned general plant-disease model** (transfer learning from a public plant-disease dataset) | Reuses external labeled data; less data-hungry than training from scratch | Public datasets skew toward common agricultural crops, not carnivorous/exotic species — domain gap risk |
| **Few-shot / foundation vision model** (embedding-based similarity to a handful of horticulturist-labeled reference images per species) | Works with only a handful of examples per species — fits the rare/diverse-collection reality; new species addable without full retraining | Less mature/harder to validate than a standard classifier; may need a foundation-model API call (interacts with the model-gateway/fallback design in ADR-0001) rather than running fully at the edge |

## Options considered — soil/environment telemetry anomaly detection

Same shape as ADR-0007 (isolation-forest-style unsupervised anomaly detection, seeded with threshold fallbacks) — reused here rather than re-litigated, since the underlying problem (multivariate sensor telemetry, no labeled failure data at launch) is the same.

## Decision

- **Telemetry anomaly detection**: same technique as animal health (ADR-0007) — isolation-forest-per-bed/collection-type, threshold fallback for cold-start.
- **Imagery**: **few-shot/embedding-based matching against horticulturist-labeled reference images per species**, rather than a per-species CNN classifier. This is the option that actually fits the stated constraint (rare, diverse collection, limited labeled data per species) — it trades some model maturity/validation rigor for being usable at all given realistic data volumes, and lets a new species be added to the collection without a full retraining cycle.

## Consequences

- **Directly suited to the collection's actual shape** (diverse, rare, not a monoculture) rather than assuming crop-scale labeled datasets exist.
- **Likely means an external embedding/foundation-model call rather than a fully local edge model** — this ties the plant-health imagery path to the model gateway (ADR-0001) more tightly than a from-scratch CNN would, inheriting that ADR's fallback/uncertainty handling instead of needing its own.
- **Harder to validate upfront than a standard classifier** — "how confident is a similarity match" is a fuzzier signal than a classifier's class probability, so the horticulturist's confirm/dismiss loop carries more of the validation weight here than it does for animal health's more conventional anomaly-detection path.
- **A handful of reference images per species is still a real cold-start cost** — someone (the horticulturist) has to label at least a few images per species before this works at all; this needs to happen before the capability can go live, not something the model bootstraps itself.
