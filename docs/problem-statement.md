# Problem Statement — Architectural Katas 2026: AI-Assisted Software Architecture

Captured from the official kata brief PDF so it's durable in-repo (not dependent on external links).

## What we're asked to do

> You'll be presented with a realistic example of a company who want you to define an architecture for them. We want you to come back with your proposal for how the system should be built. We want you to pay special attention to how the use of AI tooling could help enhance the solution, both for the company, and also for the company's customers.

## Background

After a bizarre gardening accident, the 204th in line to the Von Digitalis estates has suddenly found themselves the newly appointed 72nd Countess Von Digitalis.

The estates are large and sprawling, and are in need of some help to make them profitable. The family's previous business of making unfortunately highly explosive garden gnomes has proved to no longer be viable. The 72nd Countess Von Digitalis is looking to bring some digital solutions to help her monetize parts of the estate.

There is an extensive, historically important collection of 18th century amusement park rides on the estates too. The rides have recently passed safety inspections after all the asbestos, broken glass and garden gnomes were removed.

The previously private exotic and poisonous animal collection is also going to be opened to the public. There is a mix of aquatic and land-based animals.

## Context

- The Von Digitalis Estates get on average 5,000 visitors a day, with expectation (and hope!) that this will grow to at least 15,000 within three years, or else the family may be forced to sell their carnivorous plant collection.
- There are 40 rides in the amusement park area, and over 200 animals in the exotic animal collection split across 55 different displays and enclosures.

## What we want you to come up with

A new, comprehensive, modern architecture for the Von Digitalis estates.

## What we need

- A system that provides the ability for people to buy tickets, including family passes, to access the estates.
- Understanding of how popular different parts of the park are, so the estate can understand where to improve things.
- The exotic animal collection requires careful monitoring — ways of tracking animal health, how much/well they are eating, and (for the jumping piranha collection) checking population levels.
- A way to grow the number of visitors while also making the estates more profitable, or it's back to the garden gnome business.

## Technical context/constraints

- Wifi coverage on the park is patchy.
- Cloud services can be used, but there needs to be a way of getting information from the estate to the cloud.
- Assume there is a budget for MQTT-capable hardware devices which can be installed throughout the park.

## Biggest business challenges

- No real idea of what parts of the estates are most popular, making it difficult to know where to invest and deploy staff.
- Looking after the animals is costly, even more so if they get sick — healthy and happy animals are the goal.
- Want more returning visitors, but aren't sure how.

**We want you to place a focus on how AI could be used to solve the problems of the 72nd Countess Von Digitalis.**

## Deliverables

- **Overview**: a *short* narrative describing how the team used AI to solve the problems of the Von Digitalis Estates.
- **Diagrams**: comprehensive and targeted views for each use of AI.
- **ADRs** for AI-related implementations, including trade-off analysis.
- *(optional)* Pertinent implementation details.
- *(for semi-final teams)* Five-minute video describing the team's approach.

## Judges' criteria

- Innovative use of AI in the solution(s).
- Suitability of the solution given the constraints.
- Appropriate levels of detail.
- Dealing with uncertainty in the world of AI technology.
- Do the architectural characteristics of the additions match the existing architecture?
- Validation and verification of AI results.

Notable framing from the brief:

> Our judges can't speak to you individually… so you need to communicate with them via your deliverables.

> AI is changing FAST — how are you going to deal with the fact that the best models or providers today might not be the best tomorrow? How would you handle your model provider changing prices on you? What might happen if the provider you used suddenly shut down?

> Verifying functionality of deterministic solutions is easy — but GenAI functionality tends to be non-deterministic! How will you know if your AI-driven functionality starts misbehaving once in production?

## How to submit

- Create a GitHub repo for the problem (this repo).
- Include all documentation and visuals in the repo.
- Include a simple README so judges can navigate what's been shared.

## Schedule

- Event 1 (kickoff): Tuesday, September 1, 2026
- Team submission closed: 12pm ET, Wednesday, September 2, 2026
- **Solutions due in this GitHub repo: 11:59pm ET, Wednesday, September 16, 2026**
- Judges review: September 17 – October 1, 2026
- Semifinalists announced: Event 2, Monday, October 5, 2026
- Semifinalist video presentations due: 11:59pm ET, Monday, October 12, 2026
- Judges review videos: October 13 – October 20, 2026
- Winners announced: Event 3, Wednesday, October 21, 2026
