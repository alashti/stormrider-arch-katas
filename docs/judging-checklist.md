# Judging Checklist

Self-check against the brief before submitting. Not a deliverable itself — just a working checklist.

## Deliverables

- [ ] Overview narrative written (`docs/overview.md`) — short, AI-focused
- [ ] Diagrams present for each distinct AI use (`docs/diagrams/`)
- [ ] One key/legend provided if diagrams use shapes/colors to mean different things
- [ ] ADRs written for every AI-related architectural decision (`docs/adr/`), each with a trade-off analysis
- [ ] (Optional) Implementation details included, if applicable (`implementation/`)
- [ ] README updated with team name/members (and confirmed no other personal/employer info leaked into the repo)

## Judging criteria — does our solution address each one?

- [ ] **Innovative use of AI** — is AI doing something non-obvious/valuable, not just a chatbot bolted on?
- [ ] **Suitability given constraints** — patchy wifi, MQTT hardware budget, edge-to-cloud data flow all accounted for?
- [ ] **Appropriate level of detail** — not over- or under-specified; judges can't ask us follow-up questions
- [ ] **Dealing with AI uncertainty** — what happens if a model/provider changes price, degrades, or disappears?
- [ ] **Architectural fit** — do the AI components match the architectural characteristics (scalability, latency, cost, etc.) of the rest of the system?
- [ ] **Validation & verification of AI results** — how do we know the AI is working correctly, especially given non-deterministic outputs?

## Before submitting

- [ ] Solutions committed and pushed by **11:59pm ET, Wednesday, September 16, 2026**
- [ ] Repo README is a good 30-second read for a judge seeing this cold
