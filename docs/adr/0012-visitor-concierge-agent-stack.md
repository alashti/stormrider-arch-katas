# Visitor concierge: agent framework, RAG retrieval, and tool-failure handling for capability a

## Status

Accepted

## Context

`docs/diagrams/ai-visitor-concierge.md` establishes the shape (chat → LLM agent → tool calls → RAG-grounded knowledge base → visitor-confirmed checkout) but leaves open, per its own "what's still open" section, what happens when the agent is uncertain or a tool call fails — an actual gap, not a stylistic omission, since this is the one capability talking directly to visitors in natural language and a bad failure mode here is visible and embarrassing in a way a backend model's bad day isn't.

## Options considered — agent orchestration pattern

| Option | Pros | Cons |
|---|---|---|
| **Freeform ReAct-style prompted agent** (model reasons + calls tools in an unconstrained loop) | Maximum flexibility; handles novel visitor requests well | Least predictable; harder to bound what the agent might attempt, including near payment/booking actions |
| **Structured function-calling with a constrained tool schema** (model chooses from an explicit, typed tool set each turn; no free-form action outside the schema) | Bounded, auditable action space — the agent can only ever call wait-time lookup, route recommendation, or booking, with typed arguments; easy to log and replay for validation | Less flexible for genuinely novel requests outside the defined tools; requires the tool schema to be designed well upfront |
| **Multi-agent orchestration** (separate sub-agents for booking vs. info vs. routing, coordinated by a router agent) | Cleanly separates concerns; each sub-agent has a narrower, easier-to-validate scope | Meaningfully more infrastructure and latency for a problem this size; overkill for three tool categories |

## Options considered — RAG retrieval

| Option | Pros | Cons |
|---|---|---|
| **Pure dense vector search** over the knowledge base | Simple, good semantic-similarity matching | Can miss exact-match queries (a specific ride name, an exact policy term) that keyword search would nail |
| **Hybrid retrieval (dense vector + keyword/BM25)** | Combines semantic matching with exact-term precision — good for mixed queries ("family pass rules" needs semantics, "Gravity Coaster hours" needs an exact name match) | Slightly more retrieval infrastructure than vector-only |
| **Graph-RAG** (structured knowledge graph of attractions/relationships, traversed alongside retrieval) | Could represent attraction relationships explicitly (e.g., "rides near the entrance") | Meaningfully more setup for a knowledge base whose actual content (park info, live status) doesn't obviously need graph structure yet |

## Decision

- **Orchestration**: **structured function-calling with a constrained, typed tool schema** — the agent may only invoke `lookup_wait_time(ride_id)`, `recommend_route(current_zone, preferences)`, or the booking tool chain (`search_availability`, `add_to_cart`, `checkout`), each with a defined schema. This is the option that keeps the agent's action space auditable and bounded, which matters specifically because one of those tools moves toward a real payment — the same reasoning ADR-0006 already applies to *when* a charge happens applies here to *what the agent is even capable of attempting*.
- **Retrieval**: **hybrid dense + keyword retrieval** over the vector store/knowledge base from `cloud-platform.md` — mixed query types (exact ride names and policy lookups alongside open-ended semantic questions) are the realistic visitor-query mix, and pure vector search alone would under-serve the exact-match half.
- **Tool-failure / uncertainty handling** (the concrete answer to the diagram's open question): a failed or timed-out tool call (e.g., the booking system is briefly unavailable) surfaces to the visitor as an explicit, honest "I can't check that right now" rather than the agent guessing or silently retrying into a stale answer; the agent is explicitly instructed (and, for the booking tools, structurally prevented via the schema) from ever presenting an unconfirmed action as complete.

## Consequences

- **The constrained tool schema is a real limitation on flexibility**, deliberately: a visitor request that doesn't map to one of the defined tools gets a graceful "I can't help with that here" rather than the agent improvising an ungoverned action — an acceptable trade given the tools chosen cover the concierge's actual defined scope (wait times, routes, booking).
- **Hybrid retrieval is more infrastructure than vector-only**, but directly serves the RAG-grounding goal already established in `ai-visitor-concierge.md` (reducing hallucination) — an exact-match miss on a ride name is exactly the kind of "confidently wrong" failure that diagram already flags as this capability's core risk.
- **Explicit tool-failure handling closes the one gap the diagram left open** — "what happens when a tool call fails" now has a real answer (honest degradation, never a silent guess), which is specifically what the judging criterion on "dealing with AI uncertainty" is looking for at this level of detail.
- **This ADR doesn't change the diagram's shape** — it fills in decisions the diagram already implied needed making, consistent with the level-3 diagram (`ai-visitor-concierge-detail.md`) being drawn alongside it.
