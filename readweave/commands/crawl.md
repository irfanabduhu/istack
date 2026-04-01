---
description: "Crawl a blog or writings page, group articles into thematic threads, and weave them into a unified HTML slide presentation tracing the author's ideas."
argument-hint: "[URL to blog index or writings page]"
---

# Crawl Skill

You will create one output from the following blog or writings index URL: $ARGUMENTS
- A unified **HTML slide presentation** (`[author-or-theme]-crawl.html`)

The presentation weaves articles into thematic **threads** — clusters of articles that develop a shared idea — and presents them as a coherent intellectual map rather than a chronological list.

Follow the six phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[crawl-slug]/` (derive slug from the author or theme — e.g., `paul-graham-crawl`).

**Read the checkpoint reference**: Use the Glob tool to find `**/checkpoints.md` within this plugin's installation directory, then read it for the full format specification.

**Resume logic** — check in reverse order:
1. If `annotations.md` exists → load it and skip to Phase 6 (HTML generation)
2. If `selection.md` exists → load it and skip to Phase 5 (annotations)
3. If `extraction-[thread-slug].md` files exist → load them and skip to Phase 4 (curation). If only some threads have extraction checkpoints, extract only the missing ones.
4. If no checkpoints → start from Phase 1

If resuming, report: "Found checkpoint for [phase]. Resuming from [next phase]..."

If the user's input contains `--fresh`, ignore all checkpoints and start from scratch.

**Save after each phase** — write the checkpoint file in the format specified by the checkpoint reference before proceeding to the next phase.

---

## Phase 1: Discover

### Fetch the index page:

1. Use WebFetch to retrieve the blog index, articles page, or writings collection
2. Extract every article link with its:
   - **Title**
   - **URL**
   - **Date** (if available)
   - **Description or snippet** (if available — some index pages have brief summaries)
3. If the index page is paginated, follow pagination to discover all articles (up to a reasonable limit — ~200 articles max)
4. If links point to external publications (e.g., papers hosted elsewhere), note these but still include them

### Handle different index formats:

- **Simple link list** (e.g., paulgraham.com/articles.html): Extract `<a>` tags with surrounding text
- **Blog with posts** (e.g., geohot.github.io/blog): Extract post titles, dates, and URLs from the blog structure
- **Academic writings page** (e.g., papert.org/works.html): Extract paper titles, years, and links, noting co-authors where listed
- **Mixed format**: Some pages mix essays, talks, interviews — categorize by type if possible

Report to the user: "Found N articles on [Author]'s page. Grouping into thematic threads..."

---

## Phase 2: Map & Group into Threads

### Classify articles into threads from titles and descriptions alone:

Using ONLY the titles, dates, and descriptions (do NOT fetch full articles yet — that comes later):

1. **Identify recurring themes** — look for clusters of articles that share vocabulary, concepts, or subject matter
2. **Create 3-7 threads**, each with:
   - A **thread name** — a descriptive phrase capturing the intellectual thread (e.g., "The Startup Founder's Mind", "Programming as Thinking", "Wealth and Ambition")
   - A **one-sentence description** of what connects the articles in this thread
   - The **list of articles** belonging to this thread (by title and URL)
3. **Allow overlap** — an article can appear in multiple threads if it genuinely bridges themes
4. **Create an "Uncategorized" thread** for articles that don't fit any clear pattern — but try to minimize this

### Thread quality tests:
- Each thread should have **at least 3 articles**. Merge smaller clusters.
- Each thread should have a **coherent intellectual thread**, not just a shared topic. "Essays about X" is a topic; "How the author's thinking about X evolved from A to B" is a thread.
- The thread name should be evocative, not generic. "Technology" is bad; "Tools Shape the Thinker" is good.

### Present threads to user:
Show the user the discovered threads with article counts and descriptions. Ask if they want to:
- Include all threads
- Select specific threads
- Merge or split any threads
- Suggest different groupings

Proceed with the user's selection.

---

## Phase 3: Fetch & Extract

### For each selected thread, fetch the full articles:

Launch **parallel agents per thread** (one agent per thread, or split large threads across multiple agents). Each agent:

1. Fetches the full text of each article in its thread using WebFetch
2. For each article, extracts **3-8 candidate passages** (fewer than book chapters — articles are shorter and denser)
3. Records for each passage:
   - The **exact text**
   - The **article title** and **URL** it came from
   - A **one-line note** on why it matters
4. Records for each article:
   - The **core claim** in one sentence
   - How it **relates to other articles** in the thread — does it build on, contradict, refine, or echo ideas from other pieces?
   - Its **date** (for tracking how ideas evolved)

### What to look for:

Same five passage types as the digest skill (thesis crystallizations, key arguments, striking examples, rhetorical high points, provocations), plus:

- **Cross-article echoes**: Where the author reuses a concept, metaphor, or argument from another article — sometimes refining it, sometimes contradicting their earlier self
- **Origin moments**: The article where an idea first appears, before being developed in later pieces

### What NOT to extract:
- Content that only makes sense within the original article's context
- Dated references that don't serve the intellectual thread (e.g., "last week's news about X")
- Promotional or meta-blogging content ("Sorry for not posting lately")

### Checkpoint: Save `extraction-[thread-slug].md` per thread to `.readweave/[crawl-slug]/` with all candidate passages, article metadata, and per-article context.

---

## Phase 4: Curate & Weave

### For each thread, select excerpts and plan the narrative:

1. **Select 5-12 excerpts per thread** from the candidate pool. Favor passages that:
   - State the core idea of the thread most powerfully
   - Show the idea evolving across articles (earlier formulation → later refinement)
   - Provide the memorable examples that make the idea stick
   - Reveal tensions or self-corrections within the author's thinking

2. **Order excerpts within each thread** by **conceptual progression**, not chronology. Start with the most accessible or foundational formulation, build toward the most mature or provocative. Use dates to show evolution, but don't let chronology dictate flow.

3. **Plan cross-thread connections**: Identify 2-3 moments where a passage in one thread illuminates or connects to a passage in another thread. These will become cross-reference margin notes.

### Selection quality tests:
1. **The thread test**: Does this excerpt advance the thread's intellectual arc, or is it just a good passage that happens to be in this thread?
2. **The stand-alone test**: Can someone unfamiliar with the article understand this passage?
3. **The evolution test**: If this author wrote about this idea multiple times, have you selected the passages that best show how the idea developed?

### Checkpoint: Save `selection.md` to `.readweave/[crawl-slug]/` with all selected excerpts, thread analysis, cross-thread connections, and ordering rationale.

---

## Phase 5: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**.

### Labels must be:
- Descriptive concepts: "The Ratchet Effect", "Status vs. Substance", "Compounding Insight", "The Founder Filter"
- Bad: "Key Point!", "Great Quote!", "See Also"

### Note body must be:
- **Analytical, not summary** — never restate what the excerpt says
- **Cross-article connections** — "This refines an idea first sketched in [Article Title] (Year)..."
- **Cross-thread connections** — "This connects to the [Thread Name] thread — the same instinct expressed differently"
- **Contextual** — what was happening when this was written? What was the author responding to?
- **Hidden implications** — what follows from this that the author doesn't spell out?

### Prose voice:
- Write in a calm, literate register — the tone of a thoughtful reader's pencil notes in the margin, not a lecturer or enthusiast
- Favor observations that unfold naturally over declarations that announce themselves
- Avoid rhetorical questions used for emphasis, exclamation marks, and breathless discovery language ("This is the crucial move", "Here we see the key insight", "This changes everything")
- Let connections speak for themselves — state them plainly rather than dramatizing them
- The best margin notes sound like something a well-read friend might write in pencil — brief, specific, genuinely clarifying, never performing cleverness

This voice also applies to all commentary prose: overview slides, transition slides, synthesis slides, and discussion questions.

### At least 25% of margin notes should make cross-article or cross-thread connections. This is what makes a crawl more than a reading list.

### Checkpoint: Save `annotations.md` to `.readweave/[crawl-slug]/` with all annotated excerpts (including cross-thread notes), overview prose, and discussion questions.

---

## Phase 6: Generate HTML Slides

### Chunked generation strategy

**This phase uses the Scaffold-Batch-Assemble pattern.** Read `**/chunked-html-generation.md` within this plugin's installation directory for the full specification.

**Crawl-specific batch planning:** One content batch per thread (max 8 slides each). Structural batches: opening (title, map) and closing (discussion, closing).

### Setup:

1. **Read the template**: Use the Glob tool to find `**/study-template.html` within this plugin's installation directory, then read it with the Read tool. This file contains the full HTML/CSS/JS scaffold for the multi-section slide presentation.
2. **Read the design guide**: Use the Glob tool to find `**/study-design-guide.md` within this plugin's installation directory, then read it. It documents the HTML structure for multi-section slide types.
3. **Replace placeholders**:
   - `{{STUDY_TITLE}}` → a title capturing the author's intellectual project (e.g., "Paul Graham on Thinking and Making")
   - `{{SIDEBAR_SUBTITLE}}` → "Author Name"
   - `{{BOOKS_SUBTITLE}}` → number of threads (e.g., "A Web in Five Threads")
4. **Populate slides** inside `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — the crawl title, author name, number of threads, a unifying quotation from the writings. Uses `study-title-page` class. Includes `begin-hint`.

- **Slide 1 — The Map**: `class="slide no-margin"` — overview of all threads in the crawl. Uses `.book-card` elements repurposed for threads: each shows the thread name, article count, and one-sentence description. Threads listed in the presentation order.

- **Slide 2 — Roadmap**: `class="slide no-margin"` — the author's intellectual landscape in brief. A visual diagram showing how the threads relate to each other.

- **Slides 3 through N — Content slides** (the bulk): A mix of:
  - **Excerpt slides** (`class="slide"`): Direct article excerpts with margin notes. The `page-chapter` label includes the article title: `"Article Title &middot; Year"`. Use `data-book` attribute with the thread number for color coding.
  - **Evolution slides** (`class="slide evolution-slide"`): Side-by-side comparison of how an idea appears in different articles. Uses `.evolution-panel` elements.
  - **Thread transition slides** (`class="slide no-margin book-transition"`): Brief orienting slide when focus shifts to a new thread. Shows thread name, article count, and a one-sentence framing. Uses the thread's assigned color.
  - **Synthesis slides** (`class="slide no-margin"`): Synthesis slides that analyze a cross-thread pattern.

- **Slide N+1 — Discussion**: `class="slide no-margin"` — 6-8 questions, prioritizing questions that span threads or trace the author's evolution.

- **Slide N+2 — Closing**: `class="slide no-margin"` — the author's intellectual project distilled + a final quotation + attribution with link to the original writings page.

### Formatting rules:
- Same typographic rules as the read and study skills
- Each slide needs a descriptive `data-title` attribute
- First slide has `class="active"`
- `page-chapter` labels on excerpt slides MUST include the article title for orientation
- `page-source` should include the article URL for reference
- Use `data-book` attribute to map threads to the study template's book color system (thread 1 = `data-book="1"`, etc.)

### Slide count guidance:
- **2-3 threads**: 20-35 total slides
- **4-5 threads**: 30-45 total slides
- **6-7 threads**: 40-55 total slides
- Roughly: 45% excerpt slides, 20% evolution/cross-reference, 15% thread/synthesis, 20% structural

Save as `[author-or-theme]-crawl.html`. Derive the name from the author or the unifying theme: e.g., `paul-graham-crawl.html`.

---

## Important Notes

- **Threads, not lists**: The value is in the thematic grouping and the connections between articles. A chronological list of excerpts is not a crawl.
- **Direct excerpts only**: Show the author's actual words. Margin notes provide the analysis.
- **Attribution matters**: Every excerpt slide must identify which article it came from. Include the article URL in the `page-source` citation.
- **Respect evolution**: If the author changed their mind across articles, show that. Don't flatten contradiction into false consistency.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **The margin notes are the value-add**: Cross-article and cross-thread margin annotations — showing how an idea from a 2005 essay connects to a 2019 essay — are what make this a *crawl* rather than a reading list.
- **Not all threads need equal weight**: A thread with 3 articles on a minor theme might get 3-4 excerpt slides. A thread with 15 articles on the author's central obsession might get 10-15.
