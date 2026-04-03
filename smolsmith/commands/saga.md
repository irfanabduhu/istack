---
description: "Generate a novella (10,000–30,000 words) in a specific author's literary style."
argument-hint: "--author \"Author Name\" [premise] [--period \"time/setting\"] [--tone \"tone\"] [--style \"early/late/specific phase\"] [--characters \"char descriptions\"]"
---

Generate a novella from: $ARGUMENTS

1. **Calibrate voice** — full author analysis with voice anchors for sustaining style over long stretches. Test sentence must be recognizable.
2. **Plan** — premise expansion, 8–15 section outline, character profiles (use `--characters` if provided), thematic throughline, pacing map. Report summary to user before proceeding.
3. **Draft** — parallel batches of 3–4 sections. Each agent gets voice profile, section outline, characters, and preceding summaries. No voice drift — re-anchor at each section's start.
4. **Revise** — check voice drift, continuity, pacing, arc coherence.
5. **Output** — save as `[title-kebab-case].md`. Chapters: `## Chapter N: Title`. Scene breaks: `* * *`. Header/disclaimer as in tale.
