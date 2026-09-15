# Diagrams

Conventions for this folder, straight from the kata brief's own guidance:

- **Keep it simple.** Boxes, arrows, and words go a long way. Judges explicitly said they don't need complex diagrams — they need *understandable* ones.
- **Provide a key** if shapes, colors, or icons carry meaning that isn't self-evident.
- Aim for both a **comprehensive view** (the whole system) and **targeted views** (one per distinct AI use — e.g., ticketing recommendations, animal health monitoring, visitor-flow analytics).

## Format

Default to [Mermaid](https://mermaid.js.org/) diagrams in `.md` files — GitHub renders them natively, so no extra tooling or exported images are needed. Name files by what they show, e.g.:

- `system-overview.md` — the comprehensive view
- `ai-<use-case>.md` — one per targeted AI use case (e.g. `ai-visitor-analytics.md`, `ai-animal-monitoring.md`, `ai-ticketing.md`)

If a diagram is easier to draw as an image (e.g. exported from Excalidraw or a whiteboard photo), drop the image file alongside a short `.md` that embeds it and explains what it shows.

## Shared shape legend

Every diagram in this folder uses these shapes consistently — stated once here rather than repeated per file:

| Shape | Meaning |
|---|---|
| Rectangle (`["..."]`) | A process, model, store, or data flow — the default node |
| Rounded / stadium (`(["..."])`) | A human actor (visitor, vet, horticulturist, staff) or an external system |
| Hexagon (`{{"..."}}`) | A decision point, guardrail check, or gate — something that branches or blocks flow |
| Solid arrow | A direct read/write or call in the "happy path" |
| Dashed arrow | A calibration/feedback loop, a fallback path, or an otherwise secondary/conditional flow (each diagram's own key, where present, states the specific meaning) |

A few diagrams (`onsite-edge.md`, `cloud-platform.md`) additionally define one or two file-specific meanings inline, called out in their own "Key" line — those are refinements of this shared legend, not a different convention.
