---
description: "Generate a theatrical play with acts and scenes in a specific playwright's style."
argument-hint: "--author \"Playwright Name\" [premise] [--period \"time/setting\"] [--tone \"tone\"] [--style \"early/late/specific phase\"] [--characters \"char descriptions\"]"
---

Generate a play from: $ARGUMENTS

1. **Calibrate for drama** — dialogue architecture (sentence length, interruptions, silence/pauses, subtext density), stage direction style, dramatic structure, character-through-dialogue, thematic obsessions, audience relationship. Test 4–6 lines of dialogue; recalibrate until recognizable.
2. **Plan** — dramatis personae 3–8 (use `--characters` if provided), act/scene structure, central conflict, setting as character, 2–3 theatrical moments.
3. **Write** — dialogue does all the work. Each character sounds distinct. Stage directions match the playwright's convention. Silence is scripted. For 3+ acts, parallel agents can write different acts.
4. **Output** — save as `[title-kebab-case].md`. Format: `## Dramatis Personae` / `## Act I` / `### Scene 1` / stage directions in `*italics*` / character names in `**BOLD.**` / no quotation marks. Header/disclaimer as in tale.
