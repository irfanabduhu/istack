---
description: "Trace intellectual evolution across multiple books. Analyzes how ideas develop and transform, suggests reading order, and produces a unified HTML slide presentation with cross-book analysis."
argument-hint: "[paths to multiple .epub, .pdf, or book files]"
---

# Book Study Skill

You will create two outputs from the following multiple book files: $ARGUMENTS
1. A comprehensive **markdown study guide** (`[study-name]-study-guide.md`)
2. A unified **HTML slide presentation** (`[study-name]-study.html`)

The presentation traces how ideas develop, transform, and interact across a body of work — not just summarizing each book individually.

Follow the seven phases below. Phases 5a and 5b are independent and can run in parallel.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[study-slug]/` (derive slug from the study name — e.g., `kuhn-incommensurability`).

**Read the checkpoint reference**: Use the Glob tool to find `**/checkpoints.md` within this plugin's installation directory, then read it for the full format specification.

**Resume logic** — check in reverse order:
1. If `annotations.md` exists → load it and skip to Phase 5 (output generation)
2. If `selection.md` exists → load it and skip to Phase 4 (annotations)
3. If `extraction-[book-slug].md` files exist → load them and skip to Phase 2 (curation). If only some books have extraction checkpoints, extract only the missing ones.
4. If no checkpoints → start from Phase 1

If resuming, report: "Found checkpoint for [phase]. Resuming from [next phase]..."

If the user's input contains `--fresh`, ignore all checkpoints and start from scratch.

**Save after each phase** — write the checkpoint file in the format specified by the checkpoint reference before proceeding to the next phase.

---

## Phase 1: Extract & Read All Books

### For each book file, detect type and extract content in parallel:

**For `.epub` files:**
1. Create a unique temp directory per book: `mktemp -d /tmp/book-extract-XXXXXX` then `unzip book.epub -d $TMPDIR`
2. Read OPF/NCX to identify chapter order
3. Launch **3-5 parallel Explore agents per book**, each covering a chapter range
4. Give each agent the **detailed extraction brief** below

**For `.pdf` files:**

First, detect whether each PDF is scanned (image-based) or has extractable text:

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

**Important**: Run OCR for all scanned PDFs in parallel before launching extraction agents. OCR can take 1-5 minutes per book depending on length.

After OCR (if needed):
- Read with page ranges, launch parallel agents per page range
- Give each agent the same extraction brief

**For plain text / `.txt` files:**
- Split into logical sections for parallel processing
- Give each agent the same extraction brief

### Critical: Launch extraction for ALL books simultaneously. If you have 3 books, you should have 9-15 agents running at once. This is the key performance optimization.

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
- **Self-revisions and qualifications**: Where the author complicates, limits, or walks back a previous claim. These are easy to miss but essential in a multi-book study — they are where intellectual honesty lives.
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

### Checkpoint: Save `extraction-[book-slug].md` per book to `.readweave/[study-slug]/` with all candidate passages, chapter context, and metadata.

---

## Phase 2: Curate & Analyze

This phase is where raw extraction becomes a study. Work through it carefully.

### 2a. Build Each Book's Argument Map

For each book, synthesize the agents' findings into:
1. **Core thesis** — the book's central argument in 2-3 sentences. This should be specific enough that someone who disagrees with the author could articulate what they disagree with.
2. **Argument structure** — how the book builds from premises to conclusion. Map the logical dependencies: which chapters depend on which? What is assumed vs. argued? Where is the argument strongest and weakest?
3. **Key concepts** — what new vocabulary, frameworks, or ideas does this book contribute? For each concept, note where it is introduced and where it does the most analytical work.
4. **Publication context** — when was it written? What was the author responding to? What was the state of the field? What events or debates prompted this book?

### 2b. Select Excerpts (the critical step)

From the pool of candidates across all books, select **15-25 excerpts per book**. This is NOT a simple "pick the best" — selection must be strategic.

**The three-pass selection process:**

**Pass 1 — The non-negotiables.** For each book, identify the passages that you *cannot* leave out — the ones where the core thesis is stated most powerfully, where the defining concept is introduced, where the pivotal argument is made. These are typically 5-8 per book. If you can't find them, your agents missed them — go back and re-read those chapters.

**Pass 2 — The arc builders.** Select passages that fill in the argument's progression. After Pass 1, you have the peaks; now add the connective tissue. Choose passages that show *how* the author gets from one peak to the next. These build the reader's understanding so the big moments land.

**Pass 3 — The texture.** Add passages that provide the distinctive flavor of the author's thinking — the memorable examples, the surprising asides, the rhetorical high points, the honest qualifications. These make the difference between a study that is *correct* and one that is *alive*.

**Selection quality tests (apply to every excerpt):**

1. **The idea test**: Can you state what idea this passage captures in one sentence? If you can't, the passage may be atmospheric but not substantive — cut it unless it's a rare rhetorical high point.
2. **The stand-alone test**: Can someone who hasn't read the book understand this passage? If it depends on context from the surrounding pages, either extend the excerpt or cut it.
3. **The substitution test**: Could a different passage from the same book capture the same idea equally well? If yes, you haven't found the *best* formulation yet — go back to the candidate pool.
4. **The redundancy test**: Does this passage capture an idea already represented by another selected excerpt? If so, keep only the stronger one — unless both are needed to show evolution within the book.
5. **The cross-book test** (unique to studies): Does this passage connect to or illuminate a passage you've selected from another book? Prioritize excerpts that participate in cross-book threads — these are where the study earns its value over individual book presentations.

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

### Checkpoint: Save `selection.md` to `.readweave/[study-slug]/` with per-book argument maps, all selected excerpts, cross-book threads, and reading order analysis.

---

## Phase 3: Cross-Book Analysis & Reading Order

This is the phase that distinguishes a study from a collection of book reports.

### 3a. Identify Cross-Book Threads

Find **conceptual threads** that run across multiple books:
- **Evolving concepts**: Ideas that the author refines, revises, or abandons across works (e.g., Kuhn's "incommensurability" shifting from perceptual to linguistic)
- **Recurring tensions**: Contradictions or unresolved questions the author keeps returning to
- **Building blocks**: Concepts from earlier works that later works presuppose or extend
- **Reversals**: Places where the author explicitly changes position
- **Vocabulary shifts**: When the same word means different things in different works, or when new terminology replaces old

For each thread, note:
- Which books participate
- How the idea transforms at each stage
- The most revealing excerpt from each book showing the transformation

### 3b. Determine Reading Order

Analyze the corpus and determine **two orderings**:

1. **Chronological order**: By publication date — the order the author actually wrote them
2. **Recommended study order**: Based on conceptual dependencies. Consider:
   - Which books introduce foundational concepts others build on?
   - Which books are more accessible entry points?
   - Are there works that only make sense after reading earlier ones?
   - Does reading order affect how you interpret the argument?

If chronological and recommended orders differ, explain why. If the user has specified a preferred order, respect it.

### 3c. Plan the Narrative Arc

Design the presentation's flow. The presentation should NOT be "Book 1 slides, then Book 2 slides, then Book 3 slides." Instead, organize by **conceptual progression**:

- Start with the foundational ideas (whichever book introduces them)
- Follow each thread as it develops across books
- Use **cross-reference slides** at key moments to show evolution
- Use **book transition slides** to orient the reader when shifting focus
- Build toward the most mature or provocative formulation of the core ideas

---

## Phase 4: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**. Follow the same quality standards as the read skill, with one addition:

### Cross-book annotations are the unique value-add:
- **Forward references**: "This definition will be significantly narrowed in [Later Book], where..."
- **Backward references**: "Compare the more tentative formulation in [Earlier Book]: '...'"
- **Evolution markers**: "Notice how [concept] has shifted from [meaning A] to [meaning B] since [Book]"
- **Tension surfacing**: "This claim sits uneasily with [Author]'s own argument in [Other Book] that..."

### Labels must be:
- Descriptive concepts: "Semantic Drift", "The Revised Criterion", "Abandoned Framework", "Conceptual Continuity"
- Bad: "Connection!", "Cross-Reference", "See Also", "Compare"

### Note body rules (same as the read skill):
- **Analytical, not summary** — NEVER restate what the excerpt says
- **Cross-referential** — connect across books, to other thinkers, to broader traditions
- **Hidden implications** — surface what the reader might miss
- **Concrete** — use specific examples, named thinkers, real events

### Prose voice:
- Write in a calm, literate register — the tone of a thoughtful reader's pencil notes in the margin, not a lecturer or enthusiast
- Favor observations that unfold naturally over declarations that announce themselves
- Avoid rhetorical questions used for emphasis, exclamation marks, and breathless discovery language ("This is the crucial move", "Here we see the key insight", "This changes everything")
- Let connections speak for themselves — state them plainly rather than dramatizing them
- The best margin notes sound like something a well-read friend might write in pencil — brief, specific, genuinely clarifying, never performing cleverness

This voice also applies to all commentary prose: overview slides, transition slides, synthesis slides, and the study guide.

### Checkpoint: Save `annotations.md` to `.readweave/[study-slug]/` with all annotated excerpts (including cross-book notes), overview prose, and discussion questions.

---

## Phase 5a: Generate Markdown Study Guide

Generate a **lean** quick-reference companion (not a standalone document — the HTML slides carry the full analysis). Keep it short and avoid repeating any passage more than once across the entire document.

Structure:

1. **Title and metadata** — study title, author(s), books covered with publication years
2. **Overview** — 1-2 paragraphs: the intellectual arc across these books and why they matter together
3. **Reading order** — recommended study order with one sentence of rationale per book
4. **Per-book summaries** — for each book, 2-3 bullet points: core thesis, unique contribution, position in the arc. No quotations here.
5. **Discussion questions** — 8-12 questions, at least half requiring knowledge of multiple books
6. **Key quotations** — 15-20 passages organized by theme, each with a one-line note on significance. No passage should appear elsewhere in this document.

Total target length: 3-5 pages when rendered. If a section feels padded, cut it.

Save as `[study-name]-study-guide.md`. Derive the name from the author or unifying theme: e.g., `kuhn-incommensurability-study-guide.md`.

---

## Phase 5b: Generate HTML Slides

### Chunked generation strategy

**This phase uses the Scaffold-Batch-Assemble pattern.** Read `**/chunked-html-generation.md` within this plugin's installation directory for the full specification.

**Study-specific batch planning:** Organize content batches by thread or book grouping (1-2 books per batch, max 8 slides each). Structural batches: opening (title, library, roadmap) and closing (reading order, discussion, closing).

### Setup

1. **Read the template**: Use the Glob tool to find `**/study-template.html` within this plugin's installation directory, then read it with the Read tool. This file contains the full HTML/CSS/JS scaffold for the multi-book slide presentation.
2. **Read the design guide**: Use the Glob tool to find `**/study-design-guide.md` within this plugin's installation directory, then read it. It documents the HTML structure for multi-book slide types.
3. **Replace placeholders**:
   - `{{STUDY_TITLE}}` → the study's title (e.g., "The Evolution of Incommensurability")
   - `{{SIDEBAR_SUBTITLE}}` → "Author Name" or "Author Name & Author Name"
   - `{{BOOKS_SUBTITLE}}` → number of books (e.g., "A Study in Three Books")
4. **Populate slides** inside `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — study title, author, book count, a unifying quotation. Uses `study-title-page` class. Includes `begin-hint`.

- **Slide 1 — Library**: `class="slide no-margin"` — overview of all books in the study. Uses `.book-card` elements showing each book's title, year, and one-line thesis. Books listed in recommended reading order.

- **Slide 2 — Roadmap**: `class="slide no-margin"` — the intellectual arc in brief. Uses a `.timeline` or `.evolution-diagram` showing how the central idea develops. Frame what follows.

- **Slides 3 through N — Content slides** (the bulk): A mix of:
  - **Excerpt slides** (`class="slide"`): Same as the read skill — direct excerpts with margin notes. The `page-chapter` label includes the book title: `"Book Title &middot; Chapter N"`.
  - **Evolution slides** (`class="slide evolution-slide"`): Side-by-side or sequential comparison of how a concept appears in different books. Uses `.evolution-panel` elements.
  - **Book transition slides** (`class="slide no-margin book-transition"`): Brief orienting slide when focus shifts to a new book. Shows book title, year, and a one-sentence framing of what this book contributes.
  - **Thread slides** (`class="slide no-margin"`): Synthesis slides that step back and analyze a cross-book pattern. Uses commentary prose, not excerpts.

- **Slide N+1 — Reading Order**: `class="slide no-margin"` — recommended reading order with rationale. Uses `.reading-order-item` elements.

- **Slide N+2 — Discussion**: `class="slide no-margin"` — 6-8 thought-provoking questions, prioritizing cross-book questions.

- **Slide N+3 — Closing**: `class="slide no-margin"` — the intellectual project distilled + a final quotation + attribution.

### Formatting rules:
- Same typographic rules as the read skill (elisions, em-dashes, smart quotes, drop caps)
- Each slide needs a descriptive `data-title` attribute
- First slide has `class="active"`
- Use `data-book` attribute on excerpt slides to identify which book (enables subtle visual differentiation)
- `page-chapter` labels on excerpt slides MUST include the book title for orientation

### Slide count guidance:
- **2-3 books**: 30-45 total slides
- **4-5 books**: 40-55 total slides
- **6+ books**: 50-65 total slides
- Roughly 40% excerpt slides, 25% evolution/cross-reference slides, 15% thread/synthesis slides, 20% structural slides (title, library, transitions, discussion, closing)

Save as `[study-name]-study.html`. Use the same naming convention as the study guide.

---

## Phase 6: Determine Conceptual Weight

Not all books in a study deserve equal representation. Allocate slide count based on:

1. **Conceptual density** — books that introduce more foundational ideas get more slides
2. **Pivotal moments** — books where the author makes major revisions deserve emphasis
3. **Accessibility** — the most approachable book might deserve slightly more space as the entry point
4. **Uniqueness** — if two books cover similar ground, favor the more developed treatment

A minor or transitional work might get 3-4 excerpt slides; a magnum opus might get 8-12.

---

## Phase 7: Refine

Review both outputs:

1. **Arc check**: Does the presentation tell a coherent story of intellectual development? Can a reader unfamiliar with any of the books follow the progression?
2. **Cross-reference audit**: Are there enough evolution slides and cross-book margin notes? The presentation should feel like a *study*, not a playlist of book summaries.
3. **Balance check**: Is any book over- or under-represented relative to its importance?
4. **Margin note audit**: Verify no margin note merely restates its excerpt. At least 30% of margin notes should make cross-book connections.
5. **Flow check**: Do book transitions feel natural? Is the reader oriented about which book they're reading from at all times?
6. **Reading order clarity**: Is the recommended reading order clearly justified?
7. **Consistency check**: Both the study guide and slides should reference the same threads and key excerpts
8. **Technical check**: Well-formed HTML, all slides have `data-title`, first slide has `class="active"`

---

## Important Notes

- **This is a study, not a collection**: The value is in tracing connections and evolution. If you removed the cross-book analysis, you should lose at least 30% of the content.
- **Direct excerpts only**: Show the author's actual words. Let the margin notes and synthesis slides provide the analysis.
- **Respect the author's development**: Don't flatten evolution into consistency. If the author changed their mind, show that. If they contradicted themselves, surface it.
- **The margin notes are the value-add**: Cross-book margin annotations — showing how an idea from page 47 of Book A connects to page 203 of Book C — are what make this a *study* rather than a bibliography.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **Reading order is optional but valuable**: Always provide it. Let the user decide whether to follow it.
