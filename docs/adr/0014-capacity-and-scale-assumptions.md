# Capacity and scale assumptions for 5,000 → 15,000 visitors/day

## Status

Accepted

## Context

The brief states current volume (~5,000 visitors/day) and a three-year growth target (15,000/day), but the design so far — `onsite-edge.md`, `cloud-platform.md`, and the capability diagrams — describes shape (zones, gateways, stores, models) without stating the numbers that shape needs to survive. Without at least rough sizing, "the architecture scales" is an assertion, not something a judge (or a future implementer) can check.

## Decision

Fix a working set of capacity assumptions, sized off 15,000 visitors/day as the design target (current 5,000/day is comfortably inside these numbers) — deliberately rough, order-of-magnitude figures, not a formal capacity-planning exercise:

- **Sensor/telemetry event rate**: ~40 rides × ~10 telemetry channels (per the categories in `onsite-edge.md`) at a 1–10s sampling interval, plus 55 enclosures × ~5 channels at a 10–60s interval (biological signals change more slowly than ride vibration) → on the order of low tens of events/second sustained at the edge, well within MQTT broker and per-zone gateway capacity; this is a bandwidth-light problem, the actual constraint is connectivity *reliability*, not volume (ADR-0003).
- **Camera-derived event rate**: edge CV pre-processing (ADR-0003) means only derived signals — behavior flags, counts, disease flags — leave a zone, at roughly one derived event per camera per few seconds, not raw frames; this is what keeps the patchy-wifi uplink viable regardless of visitor volume, since camera *inference* runs locally and scales with zones/cameras, not with visitor count.
- **Zone-gateway buffer capacity**: sized to hold at minimum 24 hours of that zone's full telemetry at the rates above during a connectivity outage — a small fraction of typical commodity edge-device storage; this is the concrete number behind `onsite-edge.md`'s store-and-forward design, not just "buffers locally."
- **Cloud ingestion throughput**: at 15,000 visitors/day, gate/footfall events alone are on the order of one to a few events per second at peak arrival times (mornings, weekends) — combined with the sensor/CV rates above, total sustained ingestion stays in the low hundreds of events/second at worst, not a big-data-scale stream; the event-stream/data-lake design (`cloud-platform.md`) is sized generously relative to this, not the bottleneck.
- **Concurrent concierge sessions**: assume a peak concurrency on the order of a few hundred simultaneous chat sessions at 15,000 visitors/day (not all visitors are mid-conversation at once) — this is the number that actually stresses the model gateway's LLM-call cost/latency (ADR-0001, ADR-0004), not the backend capabilities, and is the figure the gateway's rate-limiting/queueing behavior needs to be sized against.
- **Campaign delivery volume**: `ai-offers-campaigns.md`'s explicit reason for template-level (not per-message) human review is this exact number — up to 15,000 potential recipients/day for a live campaign; delivery infrastructure (push/email) needs to handle that volume, which is a solved problem at commodity notification-service scale, not a novel constraint here.
- **Availability targets**: ticketing/booking and gate validation (per the standalone ticketing view, `ticketing-gate.md`) are the two paths where downtime directly stops revenue or entry — these get the highest availability target in the system; the backend AI capabilities (crowd analytics, health monitoring) can tolerate materially more downtime/staleness (minutes to low hours) before real-world harm results, since a human is always the deciding actor downstream of them (ADR-0002).
- **Recovery objectives**: for the edge layer, recovery point is "whatever accumulated in the zone gateway's buffer" (bounded by its 24h+ capacity above) — nothing is lost, only delayed (ADR-0003); for the cloud platform, standard event-stream/data-lake replay and point-in-time recovery practices apply, not a bespoke design.

## Consequences

- **Converts "this should scale" into checkable numbers**, even at this rough a grain — a judge or implementer can now ask "does the gateway's rate-limiting handle a few hundred concurrent LLM sessions" as a concrete question, not an open one.
- **Confirms the actual bottleneck is connectivity reliability and LLM-call cost/latency, not raw data volume** — the sensor/telemetry/CV numbers above are all modest; this reframes where engineering effort should actually go (edge reliability, gateway cost management) rather than over-building for a "big data" scale problem the estate doesn't actually have.
- **These are placeholder-quality numbers, not a load-tested capacity plan** — real per-ride/per-enclosure sensor counts and channel rates need confirmation against actual hardware once selected; this ADR fixes the assumptions to design against, not a guarantee they're precisely correct.
- **Ties directly to the standalone ticketing view's availability requirement** (`ticketing-gate.md`) — this is the one place in the whole design where "the AI got it wrong" is not the primary risk; "the system was down" is, which is a conventional systems-reliability problem this ADR explicitly separates from the AI-specific concerns the rest of the ADRs address.
