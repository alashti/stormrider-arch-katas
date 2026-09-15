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
