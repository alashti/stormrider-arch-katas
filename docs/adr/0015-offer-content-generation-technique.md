# Offer content generation: template + instantiation, not fully-per-visitor generation

## Status

Accepted

## Context

`ai-offers-campaigns.md` describes offer content moving through generation, marketing/business review, and per-visitor instantiation, but — unlike the other five capabilities (ADRs `0007`–`0011`) — never had its own alternatives-considered pass on *how* the content itself is produced. Given human review is a hard requirement here (`docs/adr/0002`), the generation technique has to be chosen specifically to make that review workable at 15,000 visitors/day (`docs/adr/0014`), not just to maximize personalization.

## Options considered

| Option | How it works | Pros | Cons |
|---|---|---|---|
| **Fully generated per visitor** (an LLM call per visitor, per send, with that visitor's full context) | Maximum personalization; each message can reference a visitor's specific history | Human review becomes impossible at volume — reviewing thousands of individually-generated messages per campaign has no operational path, undermining the human-review requirement entirely |
| **Rule-based canned messages** (a fixed library of pre-written messages, selected by segment rules, zero generation) | Trivially reviewable (finite, static library); zero generation cost or risk | Least personalized; doesn't scale content variety with segment/situation diversity without a growing manual-writing backlog; wastes the model gateway's actual capability |
| **Template/variant generation + human approval + per-visitor variable instantiation** (a model generates a bounded set of templates/variants per campaign; a human approves the template; instantiation fills in *visitor-specific variables* — name, specific offer, relevant history reference — into the approved template, no new generation per send) | Human review stays at a workable scale (one template, not thousands of instances); personalization still happens via variable-filling (not fully static); the model gateway is used where it adds value (drafting variants) without being in the unreviewed hot path per visitor | Personalization is bounded by what the template's variables expose — can't reference something outside the template's designed variable set; requires enough templates/variants to avoid every visitor in a segment receiving visibly identical copy |
| **Fine-tuned/constrained per-visitor generation within an approved template's guardrails** (a model fills the template's free-text slots per visitor, constrained to the approved template's tone/structure, rather than simple variable substitution) | More natural-sounding personalization than plain variable substitution, while staying inside human-approved boundaries | Adds a second, smaller-scope generation step *after* approval — its own (lighter) validation question: does the constrained output ever drift outside what was actually approved |

## Decision

**Template/variant generation with human approval, then per-visitor instantiation via variable substitution** (the option already reflected in `ai-offers-campaigns.md`'s diagram) — not fully-per-visitor generation, and not a static rule-based library. This is the option that actually satisfies the human-review requirement at the stated visitor volume: a marketing/business reviewer looks at a bounded number of templates per campaign, not a per-visitor stream, while instantiation still varies the actual message per visitor (name, specific offer, relevant history) rather than sending identical copy to an entire segment.

The constrained-generation-within-guardrails option is explicitly **not** adopted at this stage — it reopens a (smaller) version of the same reviewability question this decision is meant to close, and plain variable substitution is sufficient personalization for a first version of this capability.

## Consequences

- **Human review stays operationally possible at 15,000 visitors/day** — the entire reason this option was chosen over full per-visitor generation, consistent with `docs/adr/0002`'s template-level review tier.
- **Personalization is real but bounded** — a visitor's name and specific offer vary, but the underlying copy structure doesn't; this is a deliberately lower ceiling on personalization than an LLM-per-message approach would offer, traded for reviewability.
- **Template/variant exhaustion is a real risk to monitor** — if a segment is large and long-lived enough, visitors within it may notice repeated phrasing across multiple sends; this is a signal for when a campaign needs new approved variants, not a flaw in the mechanism itself.
- **Reopens no new validation surface beyond what `docs/adr/0002` and `docs/adr/0013` already define** — because instantiation is substitution, not generation, it doesn't need its own confidence-threshold/drift-monitoring entry in the `0013` operating-criteria table; the generation step (template creation) does, and is covered there as part of the model-gateway-routed capabilities.
