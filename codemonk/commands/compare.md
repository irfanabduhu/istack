---
description: "Structured comparison of two or more technologies. Same problem, different solutions — trade-offs made explicit."
argument-hint: "[tech A] vs [tech B] [optional: for specific use case]"
---

Structured technical comparison: $ARGUMENTS

1. **Frame the decision** — what problem does this choice solve? What are the actual selection criteria that matter (not marketing bullet points)? If a use case is specified, anchor everything to it.
2. **Mental models** — for each technology, explain its core philosophy in 2-3 sentences. What does it optimize for? What does it deliberately sacrifice? These philosophies predict behavior in edge cases.
3. **Head-to-head** — compare on dimensions that actually differ. Skip features both handle equally well. For each dimension: show concrete code/config examples side-by-side, explain the trade-off, state who wins and why.
4. **The uncomfortable truths** — what does each technology's community downplay? Where does each one break down? What are the migration/lock-in costs?
5. **Decision framework** — don't say "it depends." Give concrete criteria: "Choose A when [specific conditions]. Choose B when [specific conditions]. Choose neither when [conditions]."

Output as `[techA]-vs-[techB].md`. Include runnable code examples for the same task in each technology. No false balance — if one is clearly better for the stated use case, say so.
