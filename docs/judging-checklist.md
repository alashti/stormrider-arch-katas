# Judging Checklist

Self-check against the brief before submitting. Not a deliverable itself — just a working checklist.

## Deliverables

- [x] Overview narrative written (`docs/overview.md`) — short, AI-focused
- [x] Diagrams present for each distinct AI use (`docs/diagrams/`) — 13 files: system overview, on-site/edge, cloud platform, one per capability (a, b, c, d, e, f1, f2), the concierge's level-3 agent internals, and a standalone ticketing/gate-validation domain view
- [x] One key/legend provided if diagrams use shapes/colors — a shared shape legend now lives in `docs/diagrams/README.md` (rectangle/rounded/hexagon meanings, solid vs. dashed arrows), with `onsite-edge.md`/`cloud-platform.md` layering one or two file-specific meanings on top
- [x] ADRs written for every AI-related architectural decision (`docs/adr/`), each with a trade-off analysis — 15 ADRs: 6 cross-cutting (`0001`–`0006`), 6 per-capability technique choices (`0007`–`0012`), plus `0013` (AI validation/monitoring/rollback/fallback operating criteria), `0014` (capacity/scale assumptions), `0015` (offer-content generation technique)
- [x] (Optional) Implementation details — left empty, per brief this is optional
- [x] README updated with team name/members — done (Amin Heydari Alashti, Kateryna Shylina, William Lee, Connor Spear); confirmed no other personal/employer info or secrets in the repo (checked README, docs/, implementation/)

## Judging criteria — does our solution address each one?

- [x] **Innovative use of AI** — agentic LLM + tool-calling + RAG specifically for the concierge, not everywhere; distinct techniques per backend capability (isolation forest, few-shot image matching, GBM+survival analysis, Kalman+Markov flow, GBM+logistic regression) chosen to fit the actual problem, not one hammer (`0004`, `0007`–`0011`)
- [x] **Suitability given constraints** — patchy wifi handled by per-zone MQTT gateways with store-and-forward + edge CV pre-processing (`onsite-edge.md`, `0003`); MQTT hardware budget reflected directly in the edge design
- [x] **Appropriate level of detail** — 3 levels deep (system → module → agent internals) plus 15 ADRs and a standalone ticketing/gate view; diagrams stay diagrams (no premature implementation), narrative stays short and links out rather than repeating diagram content
- [x] **Dealing with AI uncertainty** — every capability routes through one model gateway with a provider fallback chain, plus a human-feedback flywheel training an owned model to reduce third-party dependency over time (`0001`); concrete fallback-activation and rollback criteria are fixed in `0013`, not just asserted
- [x] **Architectural fit** — human-in-the-loop explicitly tiered by risk profile (welfare/safety > trust/reputation > operational) rather than copy-pasted, stated as one named principle (`0002`) and checked against every capability; capacity/scale assumptions for 15,000 visitors/day are stated explicitly (`0014`), not left implicit
- [x] **Validation & verification of AI results** — every capability's diagram states its own validation loop (vet/horticulturist confirm-dismiss, maintenance failure history, manual headcount spot-checks, A/B testing), and `0013` fixes concrete confidence thresholds, drift triggers, release gates, and escalation paths per capability — not just the mechanism, but the actual trigger

## Before submitting

- [ ] Solutions committed and pushed by **11:59pm ET, Wednesday, September 16, 2026** — `0fee7cc` is pushed; this round of fixes (arrow/terminology corrections, ADRs `0013`–`0015`, `ticketing-gate.md`, animal-health metrics, shared legend) is not yet committed
- [x] Repo README is a good 30-second read for a judge seeing this cold — problem summary, repo nav table, judging criteria, key dates, team members
