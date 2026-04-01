---
description: "Deep dive into a single web article or essay. Fetches, extracts key passages, and produces an HTML slide presentation with analytical margin annotations."
argument-hint: "[URL to article or essay]"
---

# Digest Skill

You will create one output from the following web article URL: $ARGUMENTS
- An elegant **HTML slide presentation** (`[article-name]-digest.html`)

Follow the four phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[article-slug]/` (derive slug from the article title — lowercase, kebab-case, drop articles).

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

## Phase 1: Fetch & Extract

### Fetch the article:

1. Use WebFetch to retrieve the article content from the provided URL
2. If the page is behind a paywall or returns an error, inform the user and stop
3. Extract the article's **title**, **author**, **publication date** (if available), and **source/publication name**

### Parse the content:

- Strip navigation, ads, sidebars, and other non-content elements
- Preserve the article's structure: headings, subheadings, paragraphs, block quotes
- Note any images or diagrams that are central to the argument (describe them in annotations later)
- If the article is very long (5000+ words), mentally divide it into logical sections for systematic extraction

### Extract candidate passages:

Collect **8-20 candidate passages** depending on article length. For a typical essay (1500-4000 words), aim for 10-15 candidates.

For each candidate passage, capture:
- The **exact text**, preserving the author's wording precisely
- The **section or heading** where it appears (if any)
- A **one-line note** on why this passage matters

**What to look for — the five passage types:**

- **Thesis crystallizations**: Where the author states their core claim with maximum clarity. In essays, these often appear 2-3 paragraphs in, not in the opening line.
- **Key arguments**: Where the author makes a logical move the rest of the piece depends on — a distinction, an analogy, a reductio.
- **Striking examples**: Where an abstract idea is made concrete through a case, story, or analogy. In blog essays, the example often *is* the argument.
- **Rhetorical high points**: Where the prose reaches peak intensity — passages that are beautifully or powerfully said.
- **Provocations**: Where the author challenges assumptions or received wisdom. The passages that make you stop and think.

**What NOT to extract:**
- Opening hooks or throat-clearing ("I've been thinking about X lately...")
- Promotional content, calls to action, or self-references to other posts
- Repetition — if the same idea appears twice, take the stronger formulation
- Fragmentary sentences that depend on uncaptured context

### Checkpoint: Save `extraction.md` to `.readweave/[article-slug]/` with all candidate passages, article metadata, and section structure.

---

## Phase 2: Analyze & Select

### Map the argument:

1. Identify the article's **core thesis** in one sentence
2. Map its **structure** — how does the argument build? What is the logical progression?
3. Note what the author is **responding to** — what assumption, trend, or opposing view prompted this piece?

### Select excerpts:

From the candidate pool, select **8-15 excerpts** for the presentation.

**Two-pass selection:**

**Pass 1 — The essentials.** The thesis statement, the key argument, the defining example. These are the 4-6 passages you cannot leave out. If you removed them, the article's argument would collapse.

**Pass 2 — The texture.** Add passages that give the piece its distinctive voice — the memorable analogy, the sharp provocation, the honest qualification. These make the presentation feel alive rather than merely correct.

**Selection quality tests:**
1. **The idea test**: Can you state what this passage captures in one sentence?
2. **The stand-alone test**: Can someone who hasn't read the article understand this passage?
3. **The redundancy test**: Does another selected excerpt already capture this idea?

**Passage length:**
- Sweet spot: 40-150 words / 2-6 sentences. Articles are denser than books — excerpts should be tighter.
- Never truncate mid-thought. Use `<span class="elide">[…]</span>` for minimal trims.
- Preserve the author's exact wording.

### Checkpoint: Save `selection.md` to `.readweave/[article-slug]/` with the argument map, selected excerpts, and critical context.

---

## Phase 3: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**.

### Labels must be:
- Descriptive concepts: "The Selection Pressure", "Convexity Argument", "Status Game", "Founder Mode"
- Bad: "Key Point!", "Important!", "Note", "Interesting!"

### Note body must be:
- **Analytical, not summary** — never restate what the excerpt says
- **Contextual** — connect to broader ideas, other thinkers, the author's other work, real-world implications
- **Hidden implications** — surface what a casual reader might miss
- **Concrete** — use specific references, named examples, real events

### Prose voice:
- Write in a calm, literate register — the tone of a thoughtful reader's pencil notes in the margin, not a lecturer or enthusiast
- Favor observations that unfold naturally over declarations that announce themselves
- Avoid rhetorical questions used for emphasis, exclamation marks, and breathless discovery language ("This is the crucial move", "Here we see the key insight", "This changes everything")
- Let connections speak for themselves — state them plainly rather than dramatizing them
- The best margin notes sound like something a well-read friend might write in pencil — brief, specific, genuinely clarifying, never performing cleverness

This voice also applies to all commentary prose: overview slides and discussion questions.

### Quality test: Cover the excerpt text. Can someone learn something new from just the margin note? If not, rewrite it.

### Checkpoint: Save `annotations.md` to `.readweave/[article-slug]/` with all annotated excerpts, overview prose, and discussion questions.

---

## Phase 4: Generate HTML Slides

1. **Read the template**: Use the Glob tool to find `**/gist-template.html` within this plugin's installation directory, then read it with the Read tool. This file contains the full HTML/CSS/JS scaffold for the slide presentation.
2. **Read the design guide**: Use the Glob tool to find `**/read-design-guide.md` within this plugin's installation directory, then read it. It documents the HTML structure for each slide type.
3. **Replace placeholders**:
   - `{{BOOK_TITLE}}` → the article's title (used in both `<title>` and sidebar header)
   - `{{SIDEBAR_SUBTITLE}}` → "Author Name, Source/Date"
4. **Populate slides** inside the `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — article title, author, source, date, and an opening quotation from the piece. Include `begin-hint`.

- **Slide 1 — Overview**: `class="slide no-margin"` — the article's core argument in brief. Use a visual diagram (concept-cards, or flow diagram) plus commentary. Frame what follows: "The following pages present [Author]'s argument *in [their] own words*."

- **Slides 2 through N — Excerpts**: `class="slide"` — direct article excerpts with margin annotations. This is the bulk (8-15 slides). Each slide has:
  - `page-chapter` label with the section heading (if any) or a descriptive label
  - `page-divider`
  - One or more `.excerpt` paragraphs (first excerpt on most slides gets `.drop-cap`)
  - `page-source` citation (article title, author)
  - `.margin` div with 2-3 `.margin-note` elements

- **Slide N+1 — Discussion**: `class="slide no-margin"` — 4-6 thought-provoking questions using `.discussion-q` elements

- **Slide N+2 — Closing**: `class="slide no-margin"` — the article's thesis distilled + a final quotation + author/title/source attribution

### Formatting rules:
- Use `<span class="elide">[…]</span>` for elisions within excerpts
- Use literal Unicode: `—` for em-dashes, `'` / `'` for single quotes, `"` / `"` for double quotes (do NOT use HTML entities — they break in CSS `content:` properties)
- Apply `drop-cap` class to the first `.excerpt` on most excerpt slides
- Each slide needs a descriptive `data-title` attribute for the sidebar TOC

### Slide count guidance:
- Short article (<2000 words): 8-12 total slides
- Medium article (2000-5000 words): 12-18 total slides
- Long article (5000+ words): 18-25 total slides

Save as `[article-name]-digest.html` in the working directory. Derive `[article-name]` from the article title: lowercase, kebab-case, drop articles.

---

## Important Notes

- **Direct excerpts only**: The presentation shows the author's actual words. Margin notes provide the analysis.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **Preserve the aesthetic**: Reuse the book-page design from the template — parchment colors, Garamond fonts, Caveat margin notes, drop caps.
- **Attribute the source**: Always include the original URL, author name, and publication date in the title slide and closing slide.
- **The margin notes are the value-add**: Anyone can read an article. The margin notes — analytical, contextual, concrete — are what make this a *digest* rather than a bookmark.
