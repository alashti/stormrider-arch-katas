# Crowd analytics: fusion and flow-modeling technique for capability b

## Status

Accepted

## Context

`docs/diagrams/ai-crowd-analytics.md` names two models: a "sensor fusion model" (per-zone occupancy/dwell-time from footfall + queue + occupancy signals) and a "path/flow model" (common routes between zones/attractions, added in the Step 5 refinement). Neither commits to a specific technique. This capability has no independent ground truth stream (the diagram's own validation section notes this — spot-checks are manual headcounts), so the technique needs to be one whose outputs a human can sanity-check against a periodic manual count, not a black box that's only auditable in aggregate.

## Options considered — occupancy/dwell-time fusion

| Option | Pros | Cons |
|---|---|---|
| **Simple counting (sum of gate/beam in-minus-out events)** | Trivial to implement and explain | Drifts over time (double-counts, missed counts accumulate); no way to self-correct between spot-checks |
| **Kalman filter fusion** (treats "true occupancy" as a hidden state estimated from noisy counters) | Principled way to combine multiple noisy sensors into one estimate; self-corrects drift over time; well-understood, auditable math | Assumes roughly linear dynamics; needs sensor-noise parameters tuned per zone |
| **Bayesian occupancy grid** (probabilistic per-subzone occupancy, updated per sensor reading) | Finer-grained than one number per zone; naturally expresses uncertainty (useful for "how confident is this estimate") | More complex to implement and to explain to non-technical staff; likely more than this capability's decision-support use case needs |

## Options considered — path/flow modeling

| Option | Pros | Cons |
|---|---|---|
| **Rule-based zone-adjacency counting** (raw tallies of "entered zone B within N minutes of leaving zone A") | Simple, interpretable | Treats every transition independently; can't answer "what's the *most common overall route*," only pairwise adjacency |
| **Markov-chain transition model over zones/attractions** | Naturally models sequences of visits as a chain of transitions; common-route questions become straightforward (most probable paths); reasonably interpretable (a transition matrix) | Assumes the next move depends mainly on current location (short memory), which may miss longer patterns (e.g., "people who start at the gardens tend to end at the same three rides") |
| **Graph-based clustering/community detection over the visit graph** | Can surface non-obvious visitor-flow patterns and if attraction "communities" beyond simple adjacency | More analytical/exploratory than operational; harder to turn into the concierge's real-time route recommendation, which needs a fast, simple answer |

## Decision

- **Fusion**: a **Kalman filter** per zone — the right level of sophistication for "combine several noisy counters into one trustworthy occupancy estimate that self-corrects," without the complexity of a full occupancy grid this capability doesn't need at decision-support granularity.
- **Flow**: a **Markov-chain transition model** over zones/attractions — directly answers "what routes do people commonly take" and "how long do they dwell before moving on" in a form both staff and the visitor concierge's route-recommendation tool call (`docs/diagrams/ai-visitor-concierge.md`) can consume simply (a transition probability), without needing graph-analytics tooling.

## Consequences

- **Both choices stay auditable against the manual spot-check validation loop already designed** — a Kalman filter's occupancy estimate and a Markov chain's transition probabilities are both things a human can sanity-check against an actual headcount or observed walking pattern, unlike a more opaque model.
- **The Markov assumption (next move depends mainly on current location) may miss longer-range patterns** — acceptable for now since the concierge's route recommendation and staff's flow view both operate one step ahead, not full-itinerary planning; worth revisiting only if a longer-range flow question becomes a real product need.
- **Two distinct techniques for one capability's two models** is consistent with ADR-0004's principle (fit the technique to the sub-problem) rather than forcing fusion and flow into one model that would do neither well.
