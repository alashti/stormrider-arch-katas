# Von Digitalis Estates — Architectural Katas 2026

Our team's submission for O'Reilly's **Architectural Katas 2026: AI-Assisted Software Architecture**.

> ⚠️ **TODO**: add team name and members here before final submission (per past years' rules, do not include personal names, employers, or other identifying info in this public repo).

## The problem, in a nutshell

The 72nd Countess Von Digitalis has inherited a sprawling estate — an 18th-century amusement park (40 rides) and a newly-public exotic/poisonous animal collection (200+ animals, 55 enclosures). ~5,000 visitors/day today, with a goal of 15,000/day within three years. She needs: ticketing (with family passes), analytics on which parts of the estate are popular, animal health/feeding/population monitoring, and ways to grow visitors and profitability — all with patchy on-site wifi but budget for MQTT-capable IoT hardware.

We're asked to design a modern architecture for the estate **with AI woven in deliberately** — for both the business and its visitors — not bolted on as an afterthought.

Full brief: [`docs/problem-statement.md`](docs/problem-statement.md)

## Repo guide (for judges)

| Path | What's there |
|---|---|
| [`docs/overview.md`](docs/overview.md) | Short narrative: how we used AI to solve the estate's problems |
| [`docs/problem-statement.md`](docs/problem-statement.md) | The original kata brief, captured in full |
| [`docs/diagrams/`](docs/diagrams/) | Architecture diagrams — comprehensive + targeted views per AI use |
| [`docs/adr/`](docs/adr/) | Architecture Decision Records for AI-related choices, with trade-off analysis |
| [`implementation/`](implementation/) | (Optional) supporting code/prototypes, if we build any |
| [`docs/judging-checklist.md`](docs/judging-checklist.md) | Our own checklist against the deliverables & judging criteria |

## Judging criteria (at a glance)

- Innovative use of AI in the solution(s)
- Suitability of the solution given the constraints
- Appropriate level of detail
- Dealing with uncertainty in AI technology (model/provider churn, pricing, outages)
- Architectural characteristics of the AI additions match the existing architecture
- Validation and verification of AI results

## Key dates

- Solutions due in this repo: **11:59pm ET, Wednesday, September 16, 2026**
- Semifinalists announced: Monday, October 5, 2026
- Semifinalist video due: 11:59pm ET, Monday, October 12, 2026
- Winners announced: Wednesday, October 21, 2026
