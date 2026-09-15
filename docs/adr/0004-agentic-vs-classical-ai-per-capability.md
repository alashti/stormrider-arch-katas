# Agentic LLM for the visitor concierge; classical ML everywhere else

## Status

Accepted

## Context

The brief's own example diagram shows an agent booking flights/hotels via MCP-style tool calls, with a human confirming payment — a strong signal that agentic, tool-calling LLM patterns are a technique the judges want to see used well, not just referenced. At the same time, judging explicitly rewards "innovative use of AI" and "architectural characteristics match [problem] characteristics," which rewards fitting the AI technique to the actual problem shape rather than applying one technique (an LLM) everywhere because it's fashionable, or avoiding LLMs everywhere because backend telemetry doesn't obviously need one.

The six capabilities have genuinely different shapes: five (crowd analytics, animal health, plant health, predictive maintenance, revenue & retention) are sensor/transaction-data-in, model-output-out pipelines with no natural-language interaction at all. One (the visitor concierge) is, by definition, a conversation — a visitor asking open-ended questions about wait times, routes, and bookings.

## Decision

- **The visitor concierge (`a`) is built agentic**: an LLM agent (via the model gateway from ADR-0001) with tool calls for wait-time lookup, route recommendation, and ticket search/selection/booking, grounded via RAG against a vector store/knowledge base (park info, live status) to reduce hallucination. See `docs/diagrams/ai-visitor-concierge.md`.
- **The five backend capabilities use classical ML techniques matched to their specific signal**: telemetry anomaly detection (animal health, plant health, predictive maintenance), CV classification/object detection (behavior monitoring, piranha population counting, plant disease detection), sensor fusion and flow modeling (crowd analytics), and forecasting/classification (demand forecasting, churn prediction in revenue & retention). None of these use an agentic LLM pattern — there's no conversation to have, and framing "is this animal's water quality trending badly" as an LLM prompt would trade a well-understood, auditable anomaly-detection model for a harder-to-validate and unnecessarily expensive one.

## Consequences

- **Demonstrates range for the "innovative use of AI" criterion**, rather than a single LLM/chatbot bolted onto every problem — the judges see five distinct techniques applied where each actually fits, plus one deliberately agentic pattern where a conversation is the real interface.
- **Two different validation stories exist side by side, on purpose.** The concierge's risk is "confidently wrong" (hallucination, stating incorrect park info as fact) — mitigated by RAG grounding, not by a confirm/dismiss loop. The backend capabilities' risk is a missed or false anomaly — mitigated by human confirm/dismiss loops that recalibrate thresholds (ADR-0002). These aren't the same validation mechanism, and that's intentional, not an inconsistency to paper over — but it does mean two different sets of monitoring/evaluation tooling need to exist behind the one shared model gateway, not one.
- **The agentic pattern doesn't generalize by design.** If a future capability is added to this architecture, "should this be agentic" should be answered by "is this fundamentally a conversation with open-ended intent," not "would an LLM be more impressive here" — this ADR exists partly to hold that line as the design grows.
- **Cost/latency profile differs sharply between the two groups.** LLM agent calls (concierge) are typically higher-latency and higher-cost per interaction than a classical anomaly-detection or forecasting inference call — the model gateway's cost/latency tracking (ADR-0001) needs to account for this difference rather than applying one budget assumption across all six capabilities.
