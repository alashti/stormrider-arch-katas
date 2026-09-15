# Fully automated ticket booking; the visitor's own checkout tap is the only manual step

## Status

Accepted

## Context

The brief's example diagram shows an agent that books flights/hotels, with "a human confirms payment" as a labeled step. An earlier draft of `docs/diagrams/ai-visitor-concierge.md` carried that same phrasing forward without clarifying what it meant here, which was ambiguous enough to be misread as "a staff member reviews and approves each booking" — directly contradicting the actual requirement that ticket booking be fully automated, like any commercial booking website (Expedia, Ticketmaster, etc.), with no added friction or staff bottleneck.

## Decision

Ticket search, availability checking, family-pass eligibility, pricing lookup (from `f2`), and cart assembly are **fully automated end-to-end** — no staff involvement anywhere in that path. The only manual step in the entire flow is the **visitor's own tap on "Confirm & Pay"** at checkout — functionally identical to the final step of any commercial booking website, and not a staff or approval gate of any kind. The agent is never permitted to charge a payment method without that explicit visitor action in the moment.

## Consequences

- **Matches visitor expectations from any booking site** — no unexpected friction, no "wait for approval" delay that a competitor's booking flow wouldn't have.
- **The checkout tap is a real precondition to enforce, not just a documented intention.** The booking/payment implementation must actually gate the charge behind that explicit visitor action — if this is only true in the diagram and not in how the payment flow is built, the design intent is violated silently. This is a concrete thing to check during any implementation or demo, not just a diagram label.
- **Removes a plausible source of judge confusion.** Since the brief's own example diagram uses similar "human confirms" language, our diagram and this ADR now spell out explicitly that this means the customer's own action, not a staff approval — precisely because that ambiguity already caused a real misreading once during this design process.
- **This is the lightest human-in-the-loop tier in the whole design** (see ADR-0002) — worth stating plainly rather than making it look, by omission, like booking has no human checkpoint at all when in fact it has the same one every checkout flow has.
