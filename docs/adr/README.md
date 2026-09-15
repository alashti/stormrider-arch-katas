# Architecture Decision Records

The brief calls for ADRs on **AI-related implementations, including trade-off analysis**. One file per decision.

## Conventions

- Number sequentially: `0001-<short-title>.md`, `0002-<short-title>.md`, etc.
- Copy `0000-template.md` to start a new one.
- Keep each ADR short (1–2 pages) — title, status, context, decision, consequences.
- The **Consequences** section should carry the trade-off analysis the judges are looking for — don't skip it.

## Suggested decisions to cover

Not prescriptive, but likely candidates given the brief's emphasis on AI uncertainty and validation:

- Choice of AI approach per use case (e.g., LLM vs. classical ML vs. rules for animal health anomaly detection)
- Model/provider selection, and the fallback strategy if pricing/availability changes
- How AI outputs are validated/verified before acting on them (especially anything customer-facing or safety-related)
- Edge vs. cloud placement for AI inference, given patchy wifi and MQTT hardware
