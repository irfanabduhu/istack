---
description: "Generate a collection of 3–7 short stories around a shared theme, in a specific author's literary style."
argument-hint: "--author \"Author Name\" [shared theme] [--count N] [--period \"time/setting\"] [--style \"early/late/specific phase\"] [--characters \"char descriptions\"]"
---

Generate a story collection from: $ARGUMENTS

1. **Calibrate voice** — full author analysis. Compose a test sentence; recalibrate until recognizable.
2. **Design the collection** — match the author's collection strategy (Joyce: progressive, Borges: encyclopedic variations, Carver: thematic echoes). Plan each story's premise, connection to theme, and the collection arc. If `--characters` provided, distribute across stories.
3. **Write** — generate each story (1,000–5,000 words) using parallel agents. Each gets the voice profile, its premise, and all other premises for echoes.
4. **Output** — save as `[title-kebab-case].md`. Stories separated by `---` + `## Story Title`. Header/disclaimer as in tale.
