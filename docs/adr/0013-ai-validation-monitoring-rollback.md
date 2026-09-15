# AI validation, monitoring, and rollback: concrete operating criteria

## Status

Accepted

## Context

The brief asks directly: "how will you know if your AI-driven functionality starts misbehaving once in production?" Every capability diagram already states a validation *mechanism* (vet/horticulturist confirm-dismiss, manual headcount spot-checks, A/B testing, maintenance-failure history), and `cloud-platform.md` already centralizes observability (logging, drift/confidence monitoring, cost tracking) and a fallback chain (ADR-0001). What none of that names is the **operational threshold**: what specific number, rate, or condition actually triggers an alert, a rollback, or a fallback — without that, "we monitor for drift" is a promise, not a mechanism a judge (or an on-call engineer) can evaluate. This ADR exists to close that gap with concrete, if intentionally conservative, starting criteria.

## Decision

Every capability behind the model gateway (`cloud-platform.md`) is subject to the same **operating criteria template**, instantiated per capability. The template itself, and its per-capability instantiation for the five highest-stakes capabilities:

| Criterion | Definition (generic) | c. Animal health | e. Predictive maintenance | a. Concierge |
|---|---|---|---|---|
| **Confidence threshold** | Minimum model confidence/score to act on a prediction at all; below it, treat as "no signal," not a weak signal | Anomaly score below threshold → no alert generated (vet not paged for noise) | Risk score below threshold → routine dashboard entry, not an active alert | Below a grounding-confidence threshold, the agent says "I'm not sure" rather than answering (ADR-0012) |
| **False positive / false negative targets** | Acceptable rates, reviewed against vet/staff confirm-dismiss outcomes | Target: false-negative rate near zero tolerated only with a correspondingly higher false-positive rate — a missed sick animal is worse than an extra vet check | Target: false-negative rate near zero (safety) even at a materially higher false-positive cost (unnecessary inspections) | False-positive "confidently wrong" statements tracked via post-hoc audit of a sampled reply log, not user-reported only |
| **Drift trigger** | Statistical shift in input distribution or output-confidence distribution vs. the training baseline, checked on a fixed cadence | Weekly check per enclosure-type against baseline telemetry distribution | Weekly check per ride against baseline vibration/usage distribution | Daily check of retrieval-hit-rate and tool-call-success-rate against baseline |
| **Model release gate** | What a candidate model (new provider, retrained owned model, threshold change) must clear before taking production traffic | Must match or beat the current model's confirm-rate (vet agreement) on a held-out recent period before promotion | Must match or beat current model's precision/recall against the last 6 months of actual maintenance/failure history | Must pass a fixed grounding-accuracy eval set before taking any real visitor traffic |
| **Rollback criteria** | Condition that reverts to the previous model/provider automatically or triggers a forced manual reversion | Vet dismiss-rate on alerts exceeds a set ceiling over a rolling week | Two consecutive missed/incorrect maintenance calls against real outcomes | Grounding-accuracy or tool-call-success-rate drops below the release-gate threshold on the live traffic sample |
| **Fallback activation criteria** | When the gateway's router moves off the primary provider (ADR-0001) | Provider error rate, latency, or cost exceeds its configured ceiling — same generic gateway-level trigger for every capability, not per-capability logic | (same, gateway-level) | (same, gateway-level) |
| **Alert escalation & notification** | Who gets paged, and how fast, when a threshold is breached | Vet dashboard alert immediately; on-call vet paged if unacknowledged within a defined window | Maintenance staff paged immediately for any safety-critical-signal-driven flag; routine risk-score flags go to the dashboard only | Engineering/on-call paged if tool-call failure rate or grounding-accuracy drift crosses its threshold; a single wrong answer does not page anyone |
| **Evaluation dataset & test cadence** | What the model is checked against, and how often | A held-out, continuously-growing set of vet-confirmed cases (true positives and true negatives) | A held-out set of actual maintenance/failure records, refreshed as new outcomes are recorded | A fixed park-knowledge Q&A eval set (facts, policies, live-status scenarios), re-run on every KB update and on a weekly cadence regardless |

The remaining three backend capabilities (plant health, crowd analytics, revenue & retention) follow the same template — mirroring animal health's (plant health) or the concierge's structure (revenue & retention's forecasting), so they are not separately tabled here to keep this ADR readable, per the shared-shape principle already established (ADR-0002, ADR-0004).

## Consequences

- **Turns a stated intention ("we monitor for drift") into something checkable.** A judge — or, in production, an on-call engineer — can now ask "what's the actual number" for any capability and get a real answer, not a restatement of the mechanism.
- **The specific numbers above are deliberately starting points, not final calibration.** Real thresholds (e.g., the exact drift-check statistical test, the exact confidence cutoff) need real production data to tune — this ADR fixes the *shape* of the criteria (what's measured, on what cadence, triggering what response) so that tuning is a parameter change, not a redesign.
- **Cost: this is real operational surface area to build and staff**, not just a diagram addition — held-out evaluation sets need to be built and maintained per capability, dashboards need the escalation logic wired up, and someone has to own responding to a page. This is the honest trade-off of taking "validation and verification" seriously rather than gesturing at it.
- **Gateway-level fallback activation is intentionally uniform across capabilities** (one trigger definition, not per-capability logic) — consistent with ADR-0001's whole point of centralizing provider risk in one place, while model-level rollback criteria are intentionally *not* uniform, because "is this model still good" has a genuinely different answer per capability's own ground truth (vet agreement vs. failure history vs. grounding accuracy).
