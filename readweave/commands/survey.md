---
description: "Trace a research thread across multiple related papers. Analyzes how methods, results, and ideas evolve, maps the citation landscape, and produces a unified HTML slide presentation with cross-paper analysis."
argument-hint: "[paths to multiple .pdf or paper files]"
---

# Paper Survey Skill

You will create two outputs from the following collection of related papers: $ARGUMENTS
1. A comprehensive **markdown survey guide** (`[survey-slug]-survey-guide.md`)
2. A unified **HTML slide presentation** (`[survey-slug]-survey.html`)

The presentation traces how a research thread develops across papers — not just summarizing each paper individually, but showing how methods evolve, results build on each other, and the field's understanding deepens.

Follow the seven phases below. Phases 6a and 6b are independent and can run in parallel.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[survey-slug]/` (derive slug from the survey theme — e.g., `transformer-attention-mechanisms`).

**Read the checkpoint reference**: Use the Glob tool to find `**/checkpoints.md` within this plugin's installation directory, then read it for the full format specification.

**Resume logic** — check in reverse order:
1. If `annotations.md` exists → load it and skip to Phase 6 (output generation)
2. If `selection.md` exists → load it and skip to Phase 5 (annotations)
3. If `extraction-[paper-slug].md` files exist → load them and skip to Phase 2 (curation). If only some papers have extraction checkpoints, extract only the missing ones.
4. If no checkpoints → start from Phase 1

If resuming, report: "Found checkpoint for [phase]. Resuming from [next phase]..."

If the user's input contains `--fresh`, ignore all checkpoints and start from scratch.

**Save after each phase** — write the checkpoint file in the format specified by the checkpoint reference before proceeding to the next phase.

---

## Phase 1: Read & Extract All Papers

### For each paper file, detect type and extract content in parallel:

**For `.pdf` files:**

First, detect whether each PDF is scanned or has extractable text:

```bash
pdftotext "$PDF_PATH" - 2>/dev/null | head -c 500
```

- **If output is empty or near-empty**: the PDF is scanned. Run OCR:
  ```bash
  ocrmypdf --skip-text --output-type pdf "$PDF_PATH" "${PDF_PATH%.pdf}-ocr.pdf"
  ```
  Then use the OCR'd file for all subsequent reading. Inform the user that OCR was performed.

- **If output contains readable text**: proceed normally.

**Important**: Run OCR for all scanned PDFs in parallel before launching extraction agents.

After OCR (if needed):
- Read each paper using the Read tool with page ranges
- For papers longer than 15 pages, launch parallel agents per page range

**For other file types** (`.txt`, `.tex`, etc.): read directly.

### Critical: Launch extraction for ALL papers simultaneously. If you have 5 papers, you should have 5-10 agents running at once. This is the key performance optimization.

### Extraction Brief (include in every agent prompt)

Each agent must read its assigned paper **carefully and completely**. For each paper, record:

**1. Paper metadata:**
- Title, authors, affiliations, venue, year
- Abstract (verbatim)
- Section structure

**2. Figures, tables, and key data:**
- List each figure and table with its caption and key takeaway
- Extract headline quantitative results (accuracy, bounds, speedups, error rates) with the table/figure they come from
- Note which results are directly comparable across papers in the collection

**3. Citation analysis:**
- Which papers from the collection does this paper cite? (Direct links in the citation graph)
- Which other frequently-cited works anchor this paper's positioning?
- How does the paper position itself relative to prior work? ("We extend...", "Unlike...", "Building on...")

**4. Candidate passages (12-25 per paper):**

Use the same eight passage types as the `paper` skill:

- **Core claims**: The paper's main contribution stated most precisely.
- **Methodology descriptions**: Key steps of the approach or algorithm.
- **Theorem/lemma statements**: Formal claims with conditions.
- **Result passages**: Where main results are presented and interpreted.
- **Insight passages**: Where authors explain *why* something works.
- **Limitation and scope passages**: What the approach cannot do or assumes.
- **Comparison passages**: Where the approach is contrasted with alternatives.
- **Problem motivation passages**: Why the problem matters.

For each candidate passage, capture:
- The **exact text**, preserving wording, notation, and inline citations
- The **section** where it appears
- A **one-line note** on why this passage matters

**What NOT to extract:**
- Boilerplate organization previews
- Generic related work that doesn't reveal positioning
- Notation definitions without conceptual content
- Repetition between abstract, introduction, and conclusion — take the strongest formulation

### Checkpoint: Save `extraction-[paper-slug].md` per paper to `.readweave/[survey-slug]/` with all candidate passages, metadata, and citation analysis.

---

## Phase 2: Curate & Analyze

### 2a. Map Each Paper's Contribution

For each paper, synthesize:
1. **Core contribution** — what does this paper add that didn't exist before? One sentence.
2. **Approach summary** — the method in 2-3 sentences. Specific enough to distinguish from alternatives.
3. **Key results** — the headline numbers or theorems. What claims does this paper make?
4. **Assumptions and scope** — what must be true for the results to hold?
5. **Position in the collection** — how does this paper relate to the others? Does it extend, challenge, refine, or compete with them?

### 2b. Build the Citation Graph

From the extraction data, construct the citation relationships among the papers in the collection:
- Which papers cite which other papers in the set?
- What is the chronological order?
- Are there clear "lineage" chains (A → B → C)?
- Are there competing branches (A → B and A → C pursue different approaches)?
- Are there synthesis papers that unify or compare previous approaches?

This graph drives the presentation's narrative structure.

### 2c. Select Excerpts

From the candidate pool across all papers, select **8-15 excerpts per paper**. Selection must be strategic — not just "pick the best from each paper" but curated to tell the story of how the research thread develops.

**Three-pass selection:**

**Pass 1 — The non-negotiables (4-7 per paper).** The core claim, the methodology essence, the main result, and the most important limitation. These are the passages that would be lost if the paper disappeared from the collection.

**Pass 2 — The evolution markers (2-4 per paper).** Passages that explicitly connect to other papers in the collection: "Unlike [Paper X], we...", "Building on [Paper Y]'s framework...", "Our result improves on [Paper Z] by..." These are where the cross-paper story lives. Prioritize these heavily.

**Pass 3 — The texture (2-4 per paper).** The clearest insight passage, the most honest limitation, the strongest motivation. These keep the presentation alive and prevent it from reading like an annotated bibliography.

**Selection quality tests:**
1. **The contribution test**: Does this illuminate what the paper adds to the field?
2. **The stand-alone test**: Can someone with domain knowledge understand this without reading the full paper?
3. **The redundancy test**: Does another excerpt capture this idea? Keep the sharper formulation.
4. **The cross-paper test** (critical for surveys): Does this excerpt participate in a cross-paper thread? Prioritize passages that connect to other papers in the collection.
5. **The evolution test** (unique to surveys): Does this passage show how a method, idea, or result has changed since an earlier paper? If yes, strongly prioritize it.

**Passage length:**
- Sweet spot: 40-180 words. Papers are dense; excerpts should be tight.
- Preserve inline citations — they are part of the cross-paper story.
- Use `<span class="elide">[…]</span>` for minimal trims.

### 2d. Verify Excerpts Against Source

Go back to each paper and re-read selected passages in context. Check:
- Is the wording exactly correct?
- Does the excerpt work in isolation?
- Are section attributions correct?
- Are inline citations preserved accurately?

Fix errors before proceeding.

### Checkpoint: Save `selection.md` to `.readweave/[survey-slug]/` with per-paper contribution maps, selected excerpts, citation graph, and cross-paper threads.

---

## Phase 3: Cross-Paper Analysis

This is the phase that distinguishes a survey from a stack of paper summaries.

### 3a. Identify Research Threads

Find **conceptual threads** that run across multiple papers:

- **Methodology evolution**: How the approach to the core problem changes. Paper A used technique X; Paper B replaced it with Y; Paper C combined X and Y with modification Z. Trace the *why* of each change.
- **Result progression**: How performance, bounds, or guarantees improve. Not just "numbers went up" but what insight or technique drove the improvement.
- **Assumption relaxation**: Early papers may assume idealized conditions; later papers relax these. Trace which assumptions were dropped, what new challenges that created, and how they were addressed.
- **Problem reframing**: Sometimes the field's understanding of what the problem *is* shifts. The question changes, not just the answer.
- **Competing approaches**: Parallel lines of work that take fundamentally different strategies. Where do they converge? Where do their trade-offs diverge?
- **Open questions**: What each paper identifies as future work, and whether subsequent papers address it.

For each thread, note:
- Which papers participate
- How the thread develops at each stage
- The most revealing excerpt from each paper showing the progression

### 3b. Plan the Narrative Arc

Design the presentation's flow. The presentation should NOT be "Paper 1 slides, then Paper 2 slides, then Paper 3 slides." Instead, organize by **research thread**:

- Start with the problem motivation (whichever paper frames it best)
- **Organize by research thread, not by paper** — interleave papers as threads demand
- Follow each thread as it develops across papers
- Use **cross-reference slides** at key moments to show methodological evolution
- Use **paper transition slides** to orient the reader when shifting focus
- Build toward the most mature result or the frontier of open questions

---

## Phase 4: Determine Representation Weight

Not all papers in a survey deserve equal representation. Allocate slide count based on:

1. **Foundational importance** — papers that introduced the core framework or problem formulation get more space
2. **Methodological novelty** — papers with genuinely new techniques deserve emphasis over incremental improvements
3. **Result significance** — papers with breakthrough results (substantial improvement, impossibility results, surprising findings)
4. **Pedagogical value** — papers that explain concepts most clearly, regardless of chronological priority
5. **Position in threads** — papers at thread junctions (where approaches converge or diverge) deserve emphasis

A minor follow-up paper might get 2-3 excerpt slides; a seminal paper might get 6-10.

---

## Phase 5: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**. Follow the same quality standards as the `paper` skill, with critical additions for cross-paper annotation.

### Cross-paper annotations are the unique value-add:

- **Forward references**: "This assumption will be relaxed in [Later Paper], where [specific technique] allows..."
- **Backward references**: "Compare [Earlier Paper]'s approach, which required [stronger assumption]. The key difference is..."
- **Methodology evolution markers**: "Notice how the loss function has changed from [Paper A]'s [formulation] to [Paper B]'s [formulation] — the shift reflects the field's move toward..."
- **Result progression markers**: "This improves on [Paper]'s bound by a factor of [specific], achieved by [specific technique]. The gap between these bounds and the lower bound in [Paper X] is still [specific]."
- **Tension surfacing**: "This claim is in direct tension with [Other Paper]'s finding that [specific]. The discrepancy may stem from [specific difference in setup]."

### Labels must be:
- Descriptive research concepts: "The Relaxed Assumption", "Methodology Shift", "The Gap Narrowing", "Competing Objective", "The Scalability Question", "Convergence vs. Expressivity"
- Bad: "Connection!", "Compare!", "Cross-Reference", "See Also"

### Note body rules:
- **Analytical, not summary** — NEVER restate what the excerpt says
- **Cross-referential** — connect across papers, to the broader field, to named researchers and results
- **Specific** — name papers, methods, theorems, exact results. "Prior work" is never acceptable when you can name the work.
- **Honest** — surface limitations, tensions, and open questions alongside contributions

### Prose voice:
- Write in a calm, literate register — the tone of a knowledgeable colleague annotating a reading list, not a textbook author or conference reviewer
- Be precise: name papers, methods, and specific results rather than gesturing at "the literature"
- Avoid enthusiasm markers ("groundbreaking", "elegant", "remarkable") — let the work speak for itself
- State tensions plainly rather than dramatizing them
- The best margin notes sound like something a senior researcher would write during a reading group: precise, contextual, probing, occasionally skeptical

This voice also applies to all commentary prose: overview slides, transition slides, synthesis slides, and the survey guide.

### Quality test: Cover the excerpt text. Can someone learn something new from just the margin note? If not, rewrite it.

### At least 30% of margin notes must make cross-paper connections. This is the survey's core value proposition.

### Checkpoint: Save `annotations.md` to `.readweave/[survey-slug]/` with all annotated excerpts (including cross-paper notes), overview prose, thread summaries, and discussion questions.

---

## Phase 6a: Generate Markdown Survey Guide

Generate a **lean** quick-reference companion (not a standalone document — the HTML slides carry the full analysis). Keep it short and avoid repeating any passage more than once across the entire document.

Structure:

1. **Title and metadata** — survey title, papers covered (author, title, venue, year for each)
2. **Overview** — 1-2 paragraphs: the research thread, why these papers matter together, what story they tell
3. **Citation graph** — text description of how papers relate: which cites which, where lineages branch and converge
4. **Per-paper summaries** — for each paper, 2-3 bullet points: core contribution, key result, position in the thread. No quotations here.
5. **Research threads** — brief description of each identified thread (2-3 sentences each) with the papers involved
6. **Key results & figures** — summary of the most important quantitative results, key figures/tables, and what they demonstrate. Reference specific figure/table numbers.
7. **Open questions** — 4-6 questions the collection leaves unanswered, with specific pointers to which papers surface them
8. **Discussion questions** — 8-12 questions, at least half requiring knowledge of multiple papers
9. **Key quotations** — 10-15 passages organized by thread, each with a one-line note. No passage should appear elsewhere in this document.

Total target length: 3-5 pages when rendered. If a section feels padded, cut it.

Save as `[survey-slug]-survey-guide.md`.

---

## Phase 6b: Generate HTML Slides

### Chunked generation strategy

**This phase uses the Scaffold-Batch-Assemble pattern.** Read `**/chunked-html-generation.md` within this plugin's installation directory for the full specification.

**Survey-specific batch planning:** Organize content batches by research thread (2-3 papers per batch, max 8 slides each). Structural batches: opening (title, landscape, roadmap) and closing (open questions, discussion, closing).

### Setup

1. **Read the template**: Use the Glob tool to find `**/study-template.html` within this plugin's installation directory, then read it with the Read tool. This file contains the full HTML/CSS/JS scaffold for the multi-source slide presentation.
2. **Read the design guide**: Use the Glob tool to find `**/study-design-guide.md` within this plugin's installation directory, then read it. It documents the HTML structure for multi-source slide types.
3. **Replace placeholders**:
   - `{{STUDY_TITLE}}` → the survey's title (e.g., "The Evolution of Attention Mechanisms")
   - `{{SIDEBAR_SUBTITLE}}` → primary author(s) or "Multiple Authors"
   - `{{BOOKS_SUBTITLE}}` → number of papers (e.g., "A Survey Across Five Papers")
4. **Populate slides** inside `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — survey title, scope, paper count, and a unifying quotation from the most impactful paper. Uses `study-title-page` class. Includes `begin-hint`.

- **Slide 1 — Landscape**: `class="slide no-margin"` — overview of all papers in the survey. Uses `.book-card` elements (repurposed for papers) showing each paper's title, authors, venue, year, and one-line contribution. Listed in recommended reading order.

- **Slide 2 — Roadmap**: `class="slide no-margin"` — the research thread in brief. Uses a `.timeline` or `.evolution-diagram` showing how the core problem has been approached. Frame what follows.

- **Slides 3 through N — Content slides** (the bulk): A mix of:
  - **Excerpt slides** (`class="slide"`): Direct paper excerpts with margin notes. The `page-chapter` label includes the paper: `"[Short Paper Name] &middot; §3 Methodology"`.
  - **Evolution slides** (`class="slide evolution-slide"`): Side-by-side comparison of how a method, result, or assumption appears in different papers. Uses `.evolution-panel` elements.
  - **Paper transition slides** (`class="slide no-margin book-transition"`): Brief orienting slide when focus shifts to a new paper. Shows paper title, authors, venue, year, and a one-sentence framing of its contribution.
  - **Thread synthesis slides** (`class="slide no-margin"`): Step back and analyze a cross-paper pattern — how a technique evolved, why results improved, what assumptions were relaxed.

- **Slide N+1 — Key Results**: `class="slide no-margin"` — summary of the most important quantitative results across the collection. Use a comparison table or `.evolution-panel` to show how key metrics improve across papers. Reference specific figures/tables from each paper.

- **Slide N+2 — Open Questions**: `class="slide no-margin"` — 3-5 open research questions surfaced by the collection. Uses `.discussion-q` elements. Each question should reference which papers motivate it.

- **Slide N+3 — Discussion**: `class="slide no-margin"` — 6-8 thought-provoking questions, prioritizing cross-paper questions about methodology choices, trade-offs, and future directions.

- **Slide N+4 — Closing**: `class="slide no-margin"` — the research thread's current state distilled + a forward-looking quotation + full citations for all papers.

### Formatting rules:
- Same typographic rules as other skills (elisions, em-dashes, smart quotes, drop caps)
- Each slide needs a descriptive `data-title` attribute
- First slide has `class="active"`
- Use `data-book` attribute on excerpt slides to identify which paper (enables visual differentiation)
- `page-chapter` labels on excerpt slides MUST include a short paper identifier for orientation
- For inline math notation: use Unicode where possible, or `<span class="math">...</span>` for short expressions
- Preserve `[Author, Year]` citation format within excerpts

### Slide count guidance:
- **2-3 papers**: 20-35 total slides
- **4-5 papers**: 30-45 total slides
- **6-8 papers**: 40-55 total slides
- **9+ papers**: 50-65 total slides
- Roughly 40% excerpt slides, 25% evolution/cross-reference slides, 15% thread/synthesis slides, 20% structural slides (title, landscape, transitions, discussion, closing)

Save as `[survey-slug]-survey.html`. Use the same naming convention as the survey guide.

---

## Phase 7: Refine

Review both outputs:

1. **Thread check**: Does the presentation tell a coherent story of research development? Can a reader unfamiliar with any of the papers follow the progression from problem to current frontier?
2. **Cross-reference audit**: Are there enough evolution slides and cross-paper margin notes? At least 30% of margin notes must make cross-paper connections. The presentation should feel like a *survey*, not a playlist of paper summaries.
3. **Balance check**: Is any paper over- or under-represented relative to its importance in the thread?
4. **Margin note audit**: Verify no margin note merely restates its excerpt.
5. **Flow check**: Do paper transitions feel natural? Is the reader oriented about which paper they're reading from at all times?
6. **Citation consistency**: Are inline citations within excerpts preserved accurately? Are all papers fully cited in the closing slide?
7. **Figures and results check**: Are key figures, tables, and quantitative results described in margin notes or synthesis slides? A survey without data is an opinion piece.
8. **Open questions check**: Do the open questions genuinely emerge from the collection, or are they generic? Each should reference specific papers.
9. **Consistency check**: Both the survey guide and slides should reference the same threads and key excerpts.
10. **Technical check**: Well-formed HTML, all slides have `data-title`, first slide has `class="active"`.

---

## Important Notes

- **This is a survey, not a collection**: The value is in tracing how methods, results, and understanding evolve across papers. If you removed the cross-paper analysis, you should lose at least 30% of the content.
- **Direct excerpts only**: Show the authors' actual words. Let the margin notes and synthesis slides provide the analysis.
- **Respect the research timeline**: Don't flatten evolution into consistency. If later papers invalidate earlier claims, show that. If competing approaches coexist, surface the trade-offs.
- **The citation graph is structure**: Use the papers' own citations to each other as the backbone of the narrative. This is a richer source of connection than abstract thematic similarity.
- **The margin notes are the value-add**: Cross-paper annotations — showing how a method from Paper A was modified in Paper B and why Paper C abandoned it entirely — are what make this a *survey* rather than a reading list.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **Figures, tables, and data matter**: Papers live and die by their results. Describe key figures and tables in margin notes and synthesis slides. Quantitative results should be surfaced, compared across papers, and interpreted — not ignored because they can't be embedded as images.
