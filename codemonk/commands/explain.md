---
description: "First-principles explanation of a CS concept. Build intuition from the ground up, then connect to practical applications."
argument-hint: "[concept name] [optional: --context \"specific application or language\"]"
---

First-principles explanation of: $ARGUMENTS

1. **The problem** — what real problem does this concept solve? Show a concrete scenario where the naive approach fails or becomes painful. Make the reader *feel* the need before introducing the solution.
2. **Core idea** — state the key insight in one paragraph. No jargon — if you need a term, define it first. Use analogies only if they're precise (most aren't — prefer examples over analogies).
3. **Build it up** — construct the concept incrementally. Start with the simplest possible version that works. Add complexity one layer at a time, each time showing *why* the previous version was insufficient. Include runnable code at each step.
4. **Formal definition** — now that intuition exists, give the precise definition. Connect it to the informal understanding. If there's relevant math or type theory, include it but make it optional reading.
5. **In practice** — show how this concept appears in real codebases and languages. Common patterns, common mistakes, and the difference between textbook usage and production usage. If `--context` specified, focus examples there.

Output as `[concept-kebab-case].md`. Optimize for the "aha moment" — the reader should understand not just *what* but *why it has to be this way*.
