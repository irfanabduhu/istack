---
description: "Deliberate practice exercises for a specific skill. Progressive difficulty, immediate feedback, focused repetition."
argument-hint: "[skill or topic] [optional: --language \"language\"] [--level \"beginner|intermediate|advanced\"]"
---

Generate a deliberate practice set for: $ARGUMENTS

1. **Skill decomposition** — break the target skill into 3-5 sub-skills. Identify which sub-skills are prerequisites for others. If `--level` given, focus on sub-skills appropriate to that level.
2. **Warm-up (3 exercises)** — isolate the most fundamental sub-skill. Short exercises (5-15 min each) with unambiguous correct answers. Include expected output for self-checking.
3. **Core drills (5 exercises)** — combine sub-skills progressively. Each exercise should feel slightly uncomfortable — just beyond the current comfort zone. Include constraints that force specific techniques (e.g., "solve without mutation," "O(n) time required").
4. **Synthesis (2 exercises)** — integrate all sub-skills into realistic problems. These should resemble code you'd write at work, not puzzle tricks. One should have multiple valid approaches — ask the reader to implement two and compare.
5. **Stretch (1 exercise)** — one problem that's genuinely hard. Include hints (progressively more specific) rather than a full solution. The goal is productive struggle, not frustration.

Output as `[skill-kebab-case]-kata.md`. Every exercise: clear problem statement, constraints, expected behavior, and a reference solution (collapsed/spoiler-tagged). If `--language` specified, all code in that language; otherwise use the most natural language for the domain.
