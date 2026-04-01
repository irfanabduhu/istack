---
description: "Deep dive into a single technical or academic paper. Extracts key claims, methodology, results, and produces an HTML slide presentation with analytical margin annotations connecting to the broader research landscape."
argument-hint: "[path to .pdf or paper file]"
---

# Paper Skill

You will create one output from the following technical paper: $ARGUMENTS
- An elegant **HTML slide presentation** (`[paper-slug]-paper.html`)

Follow the four phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[paper-slug]/` (derive slug from the paper title — lowercase, kebab-case, drop articles).

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

## Phase 1: Read & Extract

### Read the paper:

**For `.pdf` files:**

First, detect whether the PDF is scanned or has extractable text:

```bash
pdftotext "$PDF_PATH" - 2>/dev/null | head -c 500
```

- **If output is empty or near-empty** (fewer than ~50 characters of real text): the PDF is scanned. Run OCR:
  ```bash
  ocrmypdf --skip-text --output-type pdf "$PDF_PATH" "${PDF_PATH%.pdf}-ocr.pdf"
  ```
  Then use the OCR'd file for all subsequent reading. Inform the user that OCR was performed.

- **If output contains readable text**: proceed normally.

Read the full paper using the Read tool. For papers longer than 20 pages, use page ranges and launch **2-3 parallel Explore agents** covering different sections.

**For other file types** (`.txt`, `.tex`, etc.): read directly.

### Parse the structure:

Identify and record:
- **Title**, **authors**, **affiliations**, **publication venue/year**
- **Abstract** — capture verbatim
- **Section structure** — map all sections and subsections
- **Figures and tables** — note each with its caption, what it demonstrates, and the key takeaway. Figures and data tables carry the paper's empirical argument; describe them precisely enough that a reader who cannot see the image understands the result.
- **Key quantitative results** — extract headline numbers: accuracy, speedup, bounds, error rates, etc. Note which table or figure they come from.
- **Citation count of key references** — which prior works are cited most heavily? These anchor the research context.

### Extract candidate passages:

Collect **12-25 candidate passages**. Papers are dense — extract more than you think you need.

For each candidate passage, capture:
- The **exact text**, preserving wording, notation, and any inline citations precisely
- The **section** where it appears
- A **one-line note** on why this passage matters

**What to look for — the eight passage types for academic papers:**

1. **Core claims**: Where the paper's main contribution is stated most precisely. Usually mid-introduction or early discussion — not the abstract (which is compressed) and not the conclusion (which is hedged). Hunt for the strongest formulation.

2. **Methodology descriptions**: The key steps of the approach, algorithm, or experimental design. Capture the passage that would let a reader replicate the essential idea, not just the full procedural detail.

3. **Theorem/lemma/proposition statements**: Formal claims with their conditions. Capture the statement itself; for proofs, extract only the key insight or proof sketch, not every step.

4. **Result passages**: Where main experimental or theoretical results are presented and interpreted. The sentences around key tables/figures that explain what the numbers mean, not just the numbers themselves. **These are among the most important passages in any paper** — prioritize them heavily.

5. **Insight passages**: Where authors explain *why* something works, not just *that* it works. "The reason for this improvement is..." or "Intuitively, this holds because..." These are the highest-value passages in any paper.

6. **Limitation and scope passages**: Where authors explicitly acknowledge what their approach cannot do, when it fails, or what assumptions it requires. These are easy to skip but essential for honest understanding.

7. **Comparison passages**: Where the approach is contrasted with alternatives. "Unlike X, our method..." or "While previous work assumed Y, we..." These reveal the design space.

8. **Problem motivation passages**: Where the paper explains why the problem matters, what breaks without a solution, or what real-world impact is at stake. These anchor the technical work in purpose.

**What NOT to extract:**
- Boilerplate: "The rest of this paper is organized as follows..."
- Generic related work summaries that don't reveal the paper's positioning
- Notation definitions without conceptual content
- Repetition between abstract, introduction, and conclusion — take the strongest formulation only
- Acknowledgments, funding statements, or standard ethical disclaimers

### Checkpoint: Save `extraction.md` to `.readweave/[paper-slug]/` with all candidate passages, paper metadata, and section structure.

---

## Phase 2: Analyze & Select

### Map the paper's contribution:

1. **Core contribution** in one sentence — what does this paper add to the field that didn't exist before?
2. **Argument structure** — how does the paper build its case? What is the logical chain from problem → approach → validation → conclusion?
3. **Position in the literature** — what line of research does this extend? What does it challenge? What gap does it fill?
4. **Key assumptions** — what must be true for the results to hold? Are these stated explicitly or buried?

### Select excerpts:

From the candidate pool, select **10-18 excerpts** for the presentation.

**Two-pass selection:**

**Pass 1 — The skeleton (6-10 excerpts).** The core claim, the methodology essence, the main result, the key insight, and the most important limitation. These passages alone should let someone understand what the paper did and why it matters. If you removed all other excerpts, the presentation should still be coherent.

**Pass 2 — The depth (4-8 excerpts).** Add the problem motivation, the most revealing comparison, the strongest theorem statement, the illuminating proof sketch, the honest qualification. These make the difference between a superficial summary and genuine understanding.

**Selection quality tests:**
1. **The contribution test**: Does this passage illuminate what the paper actually contributes? If it's just context-setting, cut it unless the context is essential.
2. **The stand-alone test**: Can someone with general domain knowledge understand this passage without the surrounding paper?
3. **The redundancy test**: Does another excerpt already capture this idea? Keep only the sharper formulation.
4. **The density test** (unique to papers): Is there a more information-dense passage that captures the same point in fewer words? Papers often state ideas at multiple levels of detail — choose the level that balances precision with readability.

**Passage length:**
- Sweet spot: 40-180 words / 2-8 sentences. Papers are denser than books or articles.
- For theorem statements: capture the full statement including conditions, even if compact.
- For methodology: prefer the high-level description over procedural detail.
- Use `<span class="elide">[…]</span>` for minimal trims. Never elide conditions from a formal claim.
- Preserve the author's exact wording, including notation and inline citations.

### Checkpoint: Save `selection.md` to `.readweave/[paper-slug]/` with the contribution map, selected excerpts, and key assumptions.

---

## Phase 3: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**.

### Labels must be:
- Descriptive research concepts: "The Bottleneck Assumption", "Variance-Bias Trade-off", "Expressivity vs. Tractability", "The Identifiability Condition", "Computational Budget", "The Approximation Gap"
- Bad: "Key Result!", "Important!", "Note", "Interesting Finding"

### Note body — six types specialized for papers:

1. **Research landscape notes** — situate the claim within the broader field. Name specific prior works, competing approaches, or subsequent developments. "This extends [Author et al., Year]'s framework by relaxing the [specific] assumption, which had limited applicability to [specific domain]."

2. **Assumption surface notes** — make implicit assumptions explicit. "This result assumes i.i.d. samples — in the streaming setting where distribution shifts occur, the bound no longer holds. See [Author, Year] for the non-stationary extension."

3. **Methodology critique notes** — honest assessment of experimental or theoretical choices. "The comparison uses [Baseline] from 2018; more recent methods like [X] and [Y] have since closed much of this gap."

4. **Implication notes** — what follows from this result that the paper doesn't fully explore? "If this bound is tight, it implies that no polynomial-time algorithm can improve on [specific aspect] — an open question the authors flag but don't resolve."

5. **Connection notes** — link to related concepts, techniques, or results from other subfields. "The proof technique here — reducing to a known hard instance — is the same strategy used in [different area], suggesting a deeper structural connection."

6. **Translation notes** — restate a formal or notation-heavy passage in plain terms. Only for passages where the notation obscures the intuition. "In plain terms: the more data you have, the less your prior beliefs matter — but the rate of convergence depends on how 'wrong' your prior was."

### Prose voice:
- Write in a calm, literate register — the tone of a knowledgeable colleague's annotations in the margin, not a textbook or review article
- Be specific: name papers, methods, theorems, and researchers rather than waving at "prior work" or "the literature"
- Avoid breathless language ("groundbreaking", "remarkable", "elegant proof") — let the work speak for itself
- Surface tensions and limitations alongside contributions — honest annotation is more useful than enthusiastic annotation
- The best margin notes sound like something a senior researcher would write during a careful first read: precise, contextual, probing

This voice also applies to all commentary prose: overview slides and discussion questions.

### Quality test: Cover the excerpt text. Can someone learn something new from just the margin note? If not, rewrite it.

### Checkpoint: Save `annotations.md` to `.readweave/[paper-slug]/` with all annotated excerpts, overview prose, and discussion questions.

---

## Phase 4: Generate HTML Slides

1. **Read the template**: Use the Glob tool to find `**/gist-template.html` within this plugin's installation directory, then read it with the Read tool. This file contains the full HTML/CSS/JS scaffold for the slide presentation.
2. **Read the design guide**: Use the Glob tool to find `**/read-design-guide.md` within this plugin's installation directory, then read it. It documents the HTML structure for each slide type.
3. **Replace placeholders**:
   - `{{BOOK_TITLE}}` → the paper's title (used in both `<title>` and sidebar header)
   - `{{SIDEBAR_SUBTITLE}}` → "Authors, Venue Year"
4. **Populate slides** inside the `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — paper title, authors, venue, year, and a key sentence from the abstract or introduction. Include `begin-hint`.

- **Slide 1 — Overview**: `class="slide no-margin"` — the paper's contribution in brief. Use a visual diagram: position the paper's contribution relative to prior work (concept-cards showing "Problem → Prior Approach → This Paper's Advance → Result"). Include headline results. Frame what follows: "The following pages present [Authors]'s argument *in their own words*."

- **Slide 2 — Context & Key Results**: `class="slide no-margin"` — what problem does this paper address, why does it matter, and what are the headline results? Use a `.context-block` or bullet list to orient the reader in the research landscape. Name the key prior works this builds on. Summarize the most important quantitative results (from key tables/figures) so the reader has the punchline before diving into the argument.

- **Slides 3 through N — Excerpts**: `class="slide"` — direct paper excerpts with margin annotations. This is the bulk (10-18 slides). Each slide has:
  - `page-chapter` label with the section name: `"§3 &middot; Methodology"` or `"§5.2 &middot; Ablation Study"`
  - `page-divider`
  - One or more `.excerpt` paragraphs (first excerpt on most slides gets `.drop-cap`)
  - `page-source` citation (paper title, authors)
  - `.margin` div with 2-3 `.margin-note` elements

- **Slide N+1 — Discussion**: `class="slide no-margin"` — 4-6 questions that probe the paper's assumptions, limitations, and implications. At least two questions should connect to the broader research landscape.

- **Slide N+2 — Closing**: `class="slide no-margin"` — the paper's core contribution distilled + a final quotation (usually the paper's strongest claim or most honest limitation) + full citation (authors, title, venue, year, DOI/URL if available)

### Formatting rules:
- Use `<span class="elide">[…]</span>` for elisions within excerpts
- Use literal Unicode: `—` for em-dashes, `'` / `'` for single quotes, `"` / `"` for double quotes (do NOT use HTML entities — they break in CSS `content:` properties)
- Apply `drop-cap` class to the first `.excerpt` on most excerpt slides
- Each slide needs a descriptive `data-title` attribute for the sidebar TOC
- For inline math notation: use Unicode where possible (∀, ∃, ∈, ≤, →, α, β, etc.), or `<span class="math">...</span>` for short expressions. Avoid full LaTeX rendering — keep it readable.
- Preserve `[Author, Year]` citation format within excerpts

### Slide count guidance:
- Short paper (4-8 pages, workshop/short paper): 10-15 total slides
- Standard paper (8-12 pages, conference): 15-22 total slides
- Long paper (12-20+ pages, journal/arxiv): 20-28 total slides

Save as `[paper-slug]-paper.html` in the working directory. Derive `[paper-slug]` from the paper title: lowercase, kebab-case, drop articles.

---

## Important Notes

- **Direct excerpts only**: The presentation shows the authors' actual words. Margin notes provide the analysis and research context.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **Preserve the aesthetic**: Reuse the book-page design from the template — parchment colors, Garamond fonts, Caveat margin notes, drop caps.
- **Full citation**: Always include complete citation information (all authors, title, venue, year) in the title slide and closing slide.
- **The margin notes are the value-add**: Anyone can read a paper. The margin notes — research landscape context, surfaced assumptions, methodology critiques, cross-field connections — are what make this a *deep reading* rather than a skim.
- **Notation handling**: Papers are notation-heavy. Include notation in excerpts where essential, but always pair it with a translation note in the margin that states the intuition in plain language.
- **Figures, tables, and data are first-class**: You cannot embed images, but key visual and quantitative results must be surfaced. Describe figures precisely in margin notes: "Figure 3 shows [specific observation], demonstrating that [interpretation]." For tables, state the key comparisons and headline numbers. A paper presentation that ignores the empirical results is incomplete.
- **Data-driven margin notes**: When annotating result passages, include the specific numbers, not just prose interpretation. "Achieves 94.3% accuracy vs. the previous SOTA of 89.7% — a 4.6pp improvement driven primarily by the [specific component], as shown in the ablation (Table 4)."
