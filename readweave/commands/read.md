---
description: "Distill a single book into a reading guide and slide presentation. Supports epub, PDF (including scanned), and text files."
argument-hint: "[path to .epub or book file]"
---

# ReadWeave Skill

You will create two outputs from the following book file: $ARGUMENTS
1. A comprehensive **markdown reading guide** (`[book-name]-reading-guide.md`)
2. An elegant **HTML slide presentation** (`[book-name]-gist.html`)

Follow the five phases below in order. Phases 4a and 4b are independent and can run in parallel.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[book-slug]/` (derive slug from the book filename — lowercase, kebab-case, no extension, drop articles).

**Read the checkpoint reference**: Use the Glob tool to find `**/checkpoints.md` within this plugin's installation directory, then read it for the full format specification.

**Resume logic** — check in reverse order:
1. If `annotations.md` exists → load it and skip to Phase 4 (HTML generation)
2. If `selection.md` exists → load it and skip to Phase 3 (annotations)
3. If `extraction.md` exists → load it and skip to Phase 2 (selection)
4. If no checkpoints → start from Phase 1

If resuming, report: "Found checkpoint for [phase]. Resuming from [next phase]..."

If the user's input contains `--fresh`, ignore all checkpoints and start from scratch.

**Save after each phase** — write the checkpoint file in the format specified by the checkpoint reference before proceeding to the next phase.

---

## Phase 1: Extract & Read

### Detect the file type and extract content:

**For `.epub` files:**
1. Create a unique temp directory and unzip: `mktemp -d /tmp/book-extract-XXXXXX` then `unzip book.epub -d $TMPDIR`
2. Read the OPF/NCX files to identify chapter order and HTML file paths
3. Launch **3-5 parallel Explore agents**, each covering a range of chapters (e.g., agent 1: chapters 1-4, agent 2: chapters 5-8, etc.)
4. Give each agent the **detailed extraction brief** below

**For `.pdf` files:**

First, detect whether the PDF is scanned (image-based) or has extractable text:

```bash
# Test if PDF has extractable text (check first 5 pages)
pdftotext "$PDF_PATH" - 2>/dev/null | head -c 500
```

- **If output is empty or near-empty** (fewer than ~50 characters of real text): the PDF is scanned. Run OCR before proceeding:
  ```bash
  ocrmypdf --skip-text --output-type pdf "$PDF_PATH" "${PDF_PATH%.pdf}-ocr.pdf"
  ```
  Then use the OCR'd file (`*-ocr.pdf`) for all subsequent reading. Inform the user that OCR was performed and how long it took.

- **If output contains readable text**: proceed normally.

After OCR (if needed):
- Read the PDF directly using the Read tool with page ranges
- Launch parallel agents to cover different page ranges
- Give each agent the same extraction brief

**For plain text / `.txt` files:**
- Read the file directly, splitting into logical sections for parallel processing
- Give each agent the same extraction brief

### Critical: Parallel reading is the key performance optimization. Always use multiple agents reading simultaneously rather than sequential reading.

### Extraction Brief (include this in every agent prompt)

Each agent must read its assigned chapters **slowly and completely** — do not skim. For each chapter, extract:

**1. Candidate passages (the most important output)**

Collect **8-15 candidate passages per chapter**. This is deliberately more than you will use — over-extraction is essential. You cannot curate well from a thin pool.

For each candidate passage, capture:
- The **exact text**, preserving the author's wording precisely. Do not paraphrase, compress, or "clean up" the prose.
- **Full paragraphs**, not isolated sentences. A good excerpt is typically 3-8 sentences (60-200 words). If the idea needs two paragraphs to land, capture both. An excerpt that ends mid-thought is worse than one that runs slightly long.
- The **chapter number, chapter title, and section heading** (if any) where the passage appears, so it can be precisely cited later.
- A **one-line note** on why this passage matters — what idea it captures.

**2. What to look for — the seven passage types:**

Hunt specifically for these. Most chapters will contain 2-4 of these types:

- **Thesis crystallizations**: Where the author states a core claim with maximum clarity and force. These are often mid-chapter, NOT in introductions — authors warm up before making their sharpest formulations.
- **Defining passages**: Where a key concept is introduced, named, or given its clearest definition. These anchor the reader's understanding. Capture the full paragraph around the definition, not just the sentence with the term.
- **Pivotal arguments**: Where the author makes a logical move that the rest of the book depends on — an inference, a distinction, a reductio, a thought experiment. These are the structural load-bearing passages.
- **Striking examples and cases**: Where an abstract idea is made concrete through a historical case, an analogy, a narrative, or a thought experiment. Often the most memorable passages in a book. (Kuhn's duck-rabbit, Rawls's veil of ignorance, Arendt's Eichmann — the example *is* the idea.)
- **Rhetorical high points**: Where the author's prose reaches peak intensity — passages that are not just true but beautifully or powerfully said. These often come at chapter endings or in moments of polemic. The prose quality matters because it makes the idea stick.
- **Self-revisions and qualifications**: Where the author complicates, limits, or walks back a previous claim. These are easy to miss but essential — they are where intellectual honesty lives.
- **Provocation and challenge**: Where the author deliberately challenges the reader's assumptions or the received wisdom of their field. Passages that make you stop and argue back.

**3. What NOT to extract:**

- **Throat-clearing**: Introductory paragraphs that preview the chapter without making a claim ("In this chapter, I will argue that..."). Wait for the actual argument.
- **Literature review**: Passages that merely summarize other thinkers' positions without the author's own analytical voice. Exception: when the summary itself reveals the author's interpretive framework.
- **Methodological boilerplate**: Passages about the book's structure, organization, or scope — unless the author's method IS a key idea.
- **Repetition**: If the same idea is stated in three places, capture the strongest formulation and note the others exist.
- **Fragmentary sentences**: Never extract a sentence that begins with "This" or "It" where the referent is in the previous (uncaptured) paragraph. If you must include such a passage, extend the excerpt upward to include the antecedent.

**4. Chapter-level context:**

For each chapter also record:
- The chapter's **central claim** in one sentence
- How it **advances the book's argument** — what does this chapter add that the previous one didn't?
- Any **key terms introduced or redefined** in this chapter
- **Internal cross-references** — does the author refer back to earlier chapters or forward to later ones?

### Checkpoint: Save `extraction.md` to `.readweave/[book-slug]/` with all candidate passages, chapter context, and metadata.

---

## Phase 2: Analyze & Select

This phase is where raw extraction becomes a presentation. Work through it carefully.

### 2a. Map the Argument

Synthesize the findings from all agents into:

1. **Structural outline** of the book's argument — how it builds from premise to conclusion. Map the logical dependencies: which chapters depend on which? What is assumed vs. argued?
2. **The book's arc**: identify the trajectory from introduction through core argument, evidence, implications, to conclusion
3. **Critical perspectives**: note major critiques or responses from the broader intellectual community where relevant

### 2b. Select Excerpts (the critical step)

From the pool of candidates, select **25-35 best excerpts**. This is NOT a simple "pick the best" — selection must be strategic.

**The three-pass selection process:**

**Pass 1 — The non-negotiables.** Identify the passages that you *cannot* leave out — the ones where the core thesis is stated most powerfully, where the defining concept is introduced, where the pivotal argument is made. These are typically 8-12 passages. If you can't find them, your agents missed them — go back and re-read those chapters.

**Pass 2 — The arc builders.** Select passages that fill in the argument's progression. After Pass 1, you have the peaks; now add the connective tissue. Choose passages that show *how* the author gets from one peak to the next. These build the reader's understanding so the big moments land.

**Pass 3 — The texture.** Add passages that provide the distinctive flavor of the author's thinking — the memorable examples, the surprising asides, the rhetorical high points, the honest qualifications. These make the difference between a presentation that is *correct* and one that is *alive*.

**Selection quality tests (apply to every excerpt):**

1. **The idea test**: Can you state what idea this passage captures in one sentence? If you can't, the passage may be atmospheric but not substantive — cut it unless it's a rare rhetorical high point.
2. **The stand-alone test**: Can someone who hasn't read the book understand this passage? If it depends on context from the surrounding pages, either extend the excerpt or cut it.
3. **The substitution test**: Could a different passage from the same book capture the same idea equally well? If yes, you haven't found the *best* formulation yet — go back to the candidate pool.
4. **The redundancy test**: Does this passage capture an idea already represented by another selected excerpt? If so, keep only the stronger one.

**Passage length and integrity:**

- **Minimum**: 40 words / 2 sentences. Anything shorter is a quotation, not an excerpt.
- **Sweet spot**: 80-180 words / 3-8 sentences. Long enough to develop the idea, short enough for a slide.
- **Maximum**: ~250 words. If longer, use `<span class="elide">[…]</span>` to trim non-essential clauses — but never elide the argumentative core.
- **Never truncate mid-thought**. An excerpt must begin and end at natural boundaries — the start/end of a paragraph, or a clear sentence break where no antecedent is left dangling.
- **Preserve the author's exact wording**. Do not modernize spelling, "fix" grammar, or simplify vocabulary. If the author wrote "incommensurable" and not "incompatible," that word choice IS the idea.

### 2c. Verify Excerpts Against Source

After selection, **go back to the source material** and re-read each selected passage in its original context. Check:
- Is the wording exactly correct? (Agents sometimes introduce small errors — missing words, swapped phrases, modernized punctuation.)
- Does the excerpt still work when you read the sentences immediately before and after it? Or did you cut too aggressively?
- Is the chapter/section attribution correct?

Fix any errors before proceeding. An excerpt with a wrong word is worse than no excerpt at all — it attributes something to the author they didn't say.

### Checkpoint: Save `selection.md` to `.readweave/[book-slug]/` with the argument map, arc, critical context, and all selected excerpts.

---

## Phase 3: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**. Each note has a **label** and a **body**.

### Labels must be:
- Descriptive concepts, not performative exclamations
- Good: "Pessimistic Induction", "Semantic Rupture", "Immunological", "The Logical Fork", "Bootstrapping Problem"
- Bad: "Key Insight!", "Bombshell!", "Important!", "Mind-blowing!", "Note"

### Note body must be:
- **Analytical, not summary** — NEVER restate what the excerpt says. If your note could serve as a summary of the passage, delete it and write something that adds.
- **Cross-referential** — connect to other parts of the book, other thinkers, broader intellectual traditions
- **Hidden implications** — surface what the reader might miss on first reading
- **Concrete** — use specific examples, historical parallels, named thinkers, real events

### Prose voice:
- Write in a calm, literate register — the tone of a thoughtful reader's pencil notes in the margin, not a lecturer or enthusiast
- Favor observations that unfold naturally over declarations that announce themselves
- Avoid rhetorical questions used for emphasis, exclamation marks, and breathless discovery language ("This is the crucial move", "Here we see the key insight", "This changes everything")
- Let connections speak for themselves — state them plainly rather than dramatizing them
- The best margin notes sound like something a well-read friend might write in pencil — brief, specific, genuinely clarifying, never performing cleverness

This voice also applies to all commentary prose: overview slides, discussion questions, and the reading guide.

### Quality test: Cover the excerpt text. Can someone learn something new from just the margin note? If not, rewrite it.

### Checkpoint: Save `annotations.md` to `.readweave/[book-slug]/` with all annotated excerpts, overview prose, and discussion questions.

---

## Phase 4a: Generate Markdown Reading Guide

Generate a **lean** quick-reference companion (not a standalone document — the HTML slides carry the full argument). Keep it short and avoid repeating any passage more than once across the entire document.

Structure:

1. **Title and metadata** — book title, author, year, edition
2. **Overview** — 1-2 paragraphs: what the book argues, why it matters, who it's for
3. **Argument map** — a compact outline of the book's structure and how ideas connect (use nested bullet points, not full prose sections)
4. **Discussion questions** — 6-10 thought-provoking questions for group discussion
5. **Key quotations** — 10-15 of the most important passages, each with a one-line note on why it matters. No passage should appear elsewhere in this document.

Total target length: 2-4 pages when rendered. If a section feels padded, cut it.

Save as `[book-name]-reading-guide.md` in the working directory. Use kebab-case for the filename.

---

## Phase 4b: Generate HTML Slides

**This phase uses the Scaffold-Batch-Assemble pattern.** Read `**/chunked-html-generation.md` within this plugin's installation directory for the full specification. The steps below are read-specific.

### Step 1: Prepare

1. **Read the template**: Use the Glob tool to find `**/gist-template.html` within this plugin's installation directory, then read it with the Read tool.
2. **Read the design guide**: Use the Glob tool to find `**/read-design-guide.md` within this plugin's installation directory, then read it.
3. **Read the chunked generation reference**: Use the Glob tool to find `**/chunked-html-generation.md` within this plugin's installation directory, then read it.
4. **Replace placeholders**:
   - `{{BOOK_TITLE}}` → the book's title (used in both `<title>` and sidebar header)
   - `{{SIDEBAR_SUBTITLE}}` → "Author Name, Year"

### Step 2: Plan batches

Plan 3-5 batches based on the annotated excerpts from `annotations.md`:

| Batch | Slides | Content |
|-------|--------|---------|
| 1 | 2 | Title (slide 0, `active`), Overview |
| 2 | up to 8 | Excerpt slides, first half of chapters |
| 3 | up to 8 | Excerpt slides, second half of chapters |
| 4 | 2-3 | Discussion, Closing |

For books with 25+ excerpts, add a batch 5 to keep content batches at max 8 slides each.

### Step 3: Write scaffold

Write the HTML file with the template content and batch markers inside `<div class="slides-wrapper">`. Include the batch manifest comment. See the chunked generation reference for the exact format.

### Step 4: Launch batch agents

Launch **all batch agents in a single message** (parallel Agent tool calls). Each agent receives the prompt template from the chunked generation reference, with:
- The design guide path
- The scaffold file path
- Its batch marker text (exact old_string for the Edit)
- Only the EX-N entries assigned to its batch from `annotations.md`

### Step 5: Verify

After all agents complete, follow the verification checklist from the chunked generation reference.

### Slide types:
- **Slide 0 — Title**: `class="slide active no-margin"` — book title, author, year, opening quotation. Include `begin-hint`.
- **Slide 1 — Overview**: `class="slide no-margin"` — the book's core argument. Use a visual diagram (cycle-diagram for horizontal sequences, level-tower for hierarchies, concept-cards, or similar) plus brief commentary. Frame what follows: "The following pages present [Author]'s argument *in [their] own words*."
- **Slides 2 through N — Excerpts**: `class="slide"` — direct book excerpts with margin annotations. This is the bulk of the presentation (20-30 slides). Each slide has:
  - `page-chapter` label with chapter number and name
  - `page-divider`
  - One or more `.excerpt` paragraphs (first excerpt on most slides gets `.drop-cap`)
  - `page-source` citation
  - `.margin` div with 2-3 `.margin-note` elements
- **Slide N+1 — Discussion**: `class="slide no-margin"` — 6 thought-provoking questions using `.discussion-q` elements
- **Slide N+2 — Closing**: `class="slide no-margin"` — the book's thesis distilled into a phrase + a final quotation + author/title/year attribution

### Formatting rules:
- Use `<span class="elide">[…]</span>` for elisions within excerpts
- Use literal Unicode: `—` for em-dashes, `'` / `'` for single quotes, `"` / `"` for double quotes (do NOT use HTML entities — they break in CSS `content:` properties)
- Use `·` as separator in chapter labels and author lines
- Apply `drop-cap` class to the first `.excerpt` on most excerpt slides
- Use `<em>` for italics within excerpts (book titles, emphasis)
- Each slide needs a descriptive `data-title` attribute for the sidebar TOC

**Filename:** `[book-name]-gist.html`. Derive `[book-name]` from the book title: lowercase, kebab-case, drop articles (a, an, the). For example, *The Structure of Scientific Revolutions* becomes `structure-scientific-revolutions-gist.html`.

---

## Phase 5: Refine

Review both outputs:

1. **Flow check**: Read through the slides in order — does the argument build naturally? Are there gaps or redundancies?
2. **Margin note audit**: Verify NO margin note merely restates its excerpt. Every note must add analytical value.
3. **Consistency check**: Both the reading guide and slides should reference the same key excerpts and themes
4. **Technical check**: Verify the HTML is well-formed — all tags closed, all slides have `data-title`, first slide has `class="active"`
5. **Pacing**: Aim for 25-35 total slides. Too few loses depth; too many loses momentum.

---

## Important Notes

- **Direct excerpts only**: The presentation shows the author's actual words, not paraphrase. The margin notes provide the analysis.
- **Single-file HTML**: The output HTML must be completely self-contained. All CSS inline in `<style>`, all JS inline in `<script>`. Only external dependency is Google Fonts CDN.
- **Preserve the aesthetic**: The book-page design (parchment colors, Garamond fonts, Caveat margin notes, drop caps) is integral to the experience. Do not modify the CSS from the template.
- **The margin notes are the value-add**: Anyone can excerpt a book. The margin notes — analytical, cross-referential, concrete — are what make this a book *club* rather than a book *report*.
