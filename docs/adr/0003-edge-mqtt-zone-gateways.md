# Per-zone edge gateways with MQTT store-and-forward, not a single estate-wide gateway

## Status

Accepted

## Context

The estate's on-site wifi coverage is explicitly called out in the brief as patchy — the park is large and sprawling, and connectivity cannot be assumed to be continuous everywhere at all times. Four physical zones need to get sensor and camera data off-site reliably: rides (40), animal enclosures (55), gardens & the carnivorous plant collection, and ticket gates. Two of those zones (animal enclosures, gardens) are camera-heavy, and shipping raw video off-site over an already-unreliable link is not viable. A design that assumes a single, always-available uplink from the whole estate to the cloud would fail exactly where it matters most — a dead zone in one corner of the park would take down monitoring or alerting everywhere, not just locally.

## Decision

- **One edge gateway per physical zone**, not one gateway for the whole estate. Each zone gateway runs a local MQTT broker with a **persistent store-and-forward queue**: sensors always have somewhere local to publish to, connectivity or not, and nothing is lost — only delayed until the next sync window.
- **Edge pre-processing at the gateway for camera-derived signals.** Animal-enclosure and garden-zone cameras run lightweight CV inference locally (behavior flags, population counts, disease signals) and forward only the derived signal, never raw video.
- **Priority-tagged opportunistic uplink.** When a zone gateway does get a connectivity window, safety-relevant signals (ride anomalies, acute animal-health alerts) are tagged to sync ahead of routine analytics data (footfall counts, routine garden readings), so a short window doesn't get consumed by low-stakes traffic.
- **Ride telemetry is split into four categories at the edge** (structural/mechanical, safety-critical, usage/load, environmental) rather than one undifferentiated stream, so downstream capabilities (predictive maintenance, crowd analytics) can each be pointed at exactly the signals they need.

Full detail: `docs/diagrams/onsite-edge.md`.

## Consequences

- **Zones fail independently.** A connectivity outage in the garden zone has zero effect on ride-safety monitoring or vet alerts elsewhere — this is the direct, structural answer to "wifi coverage is patchy," not a mitigation bolted on after the fact.
- **Nothing is silently lost**, only delayed — store-and-forward means a sensor reading taken during an outage still arrives once connectivity returns, rather than being dropped.
- **Makes the camera-heavy capabilities survivable on this network at all.** Without edge CV pre-processing, animal/plant monitoring and crowd analytics would need to ship video continuously — not viable given the stated constraint. This is a hard dependency: those capabilities' whole design assumes derived signals, not raw footage, ever leave the zone.
- **Operational cost: N gateways instead of one.** Each zone gateway is a piece of physical infrastructure to install, power, and maintain — more moving parts than a single centralized box, in exchange for the fault-isolation above. Power/connectivity for gateways in genuinely remote corners of the estate (solar/battery backup) is a real open question this decision creates, not yet solved.
- **Priority-tagging only helps if it's actually enforced end-to-end.** If the uplink layer or the cloud ingestion side doesn't respect the safety-tag priority under real bandwidth pressure, this decision is cosmetic — the priority behavior needs to be validated under a simulated patchy-connectivity scenario, not just designed on paper.
- **The exact backhaul technology (site-wide wifi mesh vs. wired trunk vs. cellular failover) is deliberately left open** — this decision fixes the *shape* (per-zone, store-and-forward, priority-tagged) independent of the specific physical medium, so the medium can be chosen or changed later without revisiting this ADR.
