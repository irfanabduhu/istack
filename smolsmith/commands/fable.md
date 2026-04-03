---
description: "Generate a fable or parable (500–2,000 words) in a specific author's style. Allegorical narrative that teaches through story."
argument-hint: "--author \"Author Name\" [premise or moral] [--tone \"tone\"] [--style \"early/late/specific phase\"] [--characters \"char descriptions\"]"
---

Generate a fable from: $ARGUMENTS

1. **Brief voice calibration** — sentence rhythm, diction/register, moral explicitness, thematic obsessions. Adapt the fable form to the author (Aesop: animal characters + stated moral; Kafka: parable as trap; Borges: thought experiment; Le Guin: anthropological field note; Orwell: political allegory; Wilde: aesthetic provocation).
2. **Write** — single pass, 500–2,000 words. Must work on two levels: literal story AND parable. Characters are types with texture. If `--characters` provided, use them. Economy — every sentence advances plot or theme. Moral explicit only if the style demands it.
3. **Output** — save as `[title-kebab-case].md`. `**Moral:**` line after `* * *` only for authors who'd use it. Header/disclaimer as in tale.
