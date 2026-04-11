---
description: "System design deep dive. Architectural thinking for a specific problem — constraints, trade-offs, and concrete implementation strategy."
argument-hint: "[system or problem description] [optional: --scale \"expected scale\"] [--constraints \"specific constraints\"]"
---

System design deep dive for: $ARGUMENTS

1. **Requirements** — clarify functional and non-functional requirements. Separate must-haves from nice-to-haves. Quantify scale: requests/sec, data volume, latency targets, consistency needs. If `--scale` or `--constraints` given, anchor to them.
2. **High-level architecture** — identify the 3-5 major components and their responsibilities. Show how data flows between them. Justify *why* these boundaries exist — what would go wrong if you merged or split differently?
3. **Deep dives** — for each critical component: data model (actual schema, not boxes), API contracts (actual endpoints/interfaces), storage choices with justification, and the one thing that will break first under load.
4. **Trade-off analysis** — identify the 2-3 fundamental tensions in this design (consistency vs. availability, latency vs. throughput, simplicity vs. flexibility). For each: explain the spectrum of options, where this design sits, and what would push you to a different point.
5. **Evolution path** — what does v1 look like (ship in a week)? What changes at 10x scale? At 100x? Which decisions are easy to reverse and which are load-bearing?

Output as `[system-kebab-case]-design.md`. Include concrete technology choices with reasoning, not abstract "use a cache layer." Show actual data models and API shapes. Prefer boring technology unless exotic choices are justified.
