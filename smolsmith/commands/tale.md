---
description: "Generate a single short story (1,000–5,000 words) in a specific author's literary style."
argument-hint: "--author \"Author Name\" [premise] [--period \"time/setting\"] [--tone \"tone\"] [--style \"early/late/specific phase\"] [--characters \"char descriptions\"]"
---

Generate a short story from: $ARGUMENTS

1. **Calibrate voice** — analyze the author's sentence architecture, diction, dialogue style, POV, thematic obsessions, and silences. If `--style` given, narrow to that phase. Compose an opening sentence; if it's not recognizable as this author, recalibrate.
2. **Plan** — opening, 1–3 characters (named in the author's style), arc in the author's structural terms, POV/tense per calibration. If `--characters` provided, use them.
3. **Write** — inhabit the voice from word one. Match the author's pacing, dialogue conventions, and endings. 1,000–5,000 words.
4. **Output** — save as `[title-kebab-case].md` with header: `# Title` / `*In the style of [Author]*` / `---` / body / `---` / disclaimer.
