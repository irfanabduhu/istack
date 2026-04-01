---
description: "Research a topic from scratch. Discovers high-quality sources across the web — articles, blogs, papers, videos, book chapters — and weaves them into a comprehensive HTML slide presentation tracing a curated learning path from foundations to frontiers."
argument-hint: "[topic to research]"
---

# Research Skill

You will create one output from the following topic: $ARGUMENTS
- A comprehensive **HTML slide presentation** (`[topic-slug]-research.html`)

The presentation builds a **curated learning path** — not a link dump or topic overview, but a guided intellectual journey from solid foundations through intermediate mastery to the research frontier. You will discover your own sources, evaluate them for quality and signal density, and weave the best into a unified presentation.

**What makes a "gem"**: High signal-to-noise ratio, genuine insight, clear writing or speaking, and a perspective that earns its place in the learning path. A 1968 Dijkstra paper can be as much a gem as a blog post from last week. Quantity is not a goal — quality and coverage are.

Follow the eight phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[topic-slug]-research/` (derive slug from the topic — e.g., `distributed-consensus-research`).

**Read the checkpoint reference**: Use the Glob tool to find `**/checkpoints.md` within this plugin's installation directory, then read it for the full format specification.

**Resume logic** — check in reverse order:
1. If `annotations.md` exists → load it and skip to Phase 7 (HTML generation)
2. If `selection.md` exists → load it and skip to Phase 6 (annotations)
3. If `extraction-[source-slug].md` files exist → load them and skip to Phase 4 (Map & Organize). If only some sources have extraction checkpoints, extract only the missing ones.
4. If `evaluation.md` exists → load it and skip to Phase 3 (Fetch & Extract)
5. If `discovery.md` exists → load it and skip to Phase 2 (Evaluate)
6. If no checkpoints → start from Phase 0

If resuming, report: "Found checkpoint for [phase]. Resuming from [next phase]..."

If the user's input contains `--fresh`, ignore all checkpoints and start from scratch.

**Save after each phase** — write the checkpoint file in the format specified by the checkpoint reference before proceeding to the next phase.

---

## Phase 1: Scout — Discover Candidate Sources

### Goal: Cast a wide net across 4 search tiers to find 40-80 candidate URLs spanning the full temporal range.

Research is only as good as what you find. Use **10-15 WebSearch queries** organized across four tiers:

### Tier 1 — Foundational & Classic (3-4 queries)
Searches designed to surface seminal works, original papers, and historically important content:
- `"[topic] seminal paper"`
- `"[topic] original" OR "[topic] first proposed"`
- `"[topic] history" OR "origins of [topic]"`
- `"[topic] foundational" OR "[topic] classic"`

### Tier 2 — Depth & Technical (3-4 queries)
Searches for the most technically thorough and insightful treatments:
- `"[topic] tutorial" OR "[topic] explained"`
- `"[topic] deep dive" OR "[topic] in depth"`
- `"[topic] internals" OR "how [topic] works"`
- `"[topic] implementation" OR "[topic] from scratch"`

### Tier 3 — Recency & Frontier (2-3 queries)
Searches for the latest developments and cutting-edge work:
- `"[topic] 2025" OR "[topic] 2026" OR "[topic] latest"`
- `"[topic] recent advances" OR "[topic] state of the art"`
- `"[topic] new research" OR "[topic] breakthrough"`

### Tier 4 — Breadth & Meta (2-3 queries)
Searches that leverage others' curation and surface non-obvious angles:
- `"best [topic] resources" OR "best [topic] articles"`
- `"[topic] reading list" OR "[topic] curriculum"`
- `"[topic] misconceptions" OR "[topic] debate" OR "[topic] criticism"`

### Query adaptation:
- Adapt the query templates to the specific topic. For a narrow topic like "Raft consensus algorithm," the queries will be more specific. For a broad topic like "distributed systems," add sub-topic queries.
- If early queries reveal important sub-topics or related concepts, add 1-2 bonus queries to cover them.
- If the topic has a strong video/talk culture (e.g., conference talks), include `"[topic] talk" OR "[topic] conference"` in Tier 2.

### For each result, record:
- **URL**
- **Title**
- **Source type** (article, blog post, academic paper, video/talk, book chapter, documentation, tutorial)
- **Author/creator** (if visible)
- **Date** (if visible — even approximate decade helps)
- **Snippet** (the search result description)
- **Discovery tier** (which tier found it)

### Deduplication:
- Remove exact URL duplicates
- Merge results that point to the same content on different domains (e.g., a paper on arxiv and the same paper on the author's site — keep the more accessible version)

### Checkpoint: Save `discovery.md` to `.readweave/[topic-slug]-research/` with all candidate URLs, metadata, and which tier discovered them.

#### discovery.md format:

```markdown
# Discovery: [Topic]
<!-- date: [ISO timestamp] -->
<!-- command: research -->
<!-- query-count: [N] -->

## Search Queries Executed

### Tier 1 — Foundational & Classic
1. "[exact query]" → [N results]
2. "[exact query]" → [N results]
...

### Tier 2 — Depth & Technical
...

### Tier 3 — Recency & Frontier
...

### Tier 4 — Breadth & Meta
...

## Candidate Sources

### CS-1
- **URL**: [url]
- **Title**: [title]
- **Type**: [article | blog | paper | video | book-chapter | docs | tutorial]
- **Author**: [author or "Unknown"]
- **Date**: [date or approximate era]
- **Tier**: [1-4]
- **Snippet**: [search result description]

### CS-2
...
```

---

## Phase 2: Evaluate — Select the Gems

### Goal: Lightweight probe of candidates → select 12-20 "gems" for deep extraction.

### Step 1: Quick probe each candidate

For each candidate URL, use WebFetch to retrieve the page. Read enough to assess:
- **Signal density**: Is this mostly insight, or mostly filler/preamble/marketing?
- **Depth**: Does it go beyond surface-level explanation?
- **Clarity**: Is it well-written/well-presented?
- **Uniqueness**: Does it offer a perspective not covered by other candidates?
- **Temporal value**: Does it fill a gap in the timeline (classic foundational work vs. cutting-edge)?

You do NOT need to read each source fully — skim the first few paragraphs, section headings, and conclusion. The goal is triage, not extraction.

For sources that are clearly low-quality (thin content, clickbait, broken pages), mark them as rejected immediately.

### Step 2: Score and rank

For each viable candidate, assign:
- **Quality score** (1-5): Signal density + depth + clarity
- **Coverage role**: What part of the learning path does this serve? (foundation, core concept, technique, application, frontier, perspective, meta/overview)
- **Temporal era**: Classic (pre-2000), Established (2000-2015), Recent (2015-2023), Cutting-edge (2024+)
- **Redundancy flag**: Does another candidate cover the same ground better?

### Step 3: Select 12-20 gems

Select sources that together form a **complete learning path**:

**Selection criteria:**
1. **Temporal diversity** — the collection MUST span eras. A set that only has content from the last 3 years is missing foundations. A set that only has classics is missing the frontier. Aim for at least 2-3 sources per era that has relevant content.
2. **Type diversity** — mix articles, papers, videos, and tutorials. Don't let one type dominate unless the topic's literature is genuinely concentrated.
3. **Perspective diversity** — include practitioners, researchers, educators, and critics where possible.
4. **Coverage completeness** — every stage of the learning path (foundations → frontiers) should have at least 2 sources.
5. **No redundancy** — if two sources cover the same ground, keep the better one.

### Step 4: Present to user for approval

Present the curated source list to the user:

```
## Proposed Source Library — [Topic]

I've evaluated [N] candidates and selected [M] gems for deep extraction:

### Foundations
1. **[Title]** by [Author] ([Year]) — [Type]
   [One sentence on why this is a gem and what it contributes to the learning path]

### Core Concepts
2. **[Title]** by [Author] ([Year]) — [Type]
   ...

### Depth & Technique
...

### Frontiers
...

### Perspectives
...

---

Would you like to:
- Proceed with these sources
- Add or remove specific sources
- Suggest additional sources you know about
- Adjust the focus (e.g., more theory, more practice)
```

Wait for user approval before proceeding. If the user suggests changes, incorporate them and re-save `evaluation.md` with the updated Approved Source Library and modifications recorded in the User Modifications section before proceeding to Phase 3.

### Checkpoint: Save `evaluation.md` to `.readweave/[topic-slug]-research/` with all scored candidates and the final approved source list.

#### evaluation.md format:

```markdown
# Evaluation: [Topic]
<!-- date: [ISO timestamp] -->
<!-- command: research -->
<!-- candidates-evaluated: [N] -->
<!-- gems-selected: [M] -->

## Evaluation Summary

### Rejected ([N] sources)
| # | Title | Reason |
|---|-------|--------|
| CS-3 | [Title] | [Thin content / duplicate of CS-7 / broken page / ...] |
...

### Scored Candidates
| # | Title | Quality | Coverage Role | Era | Selected? |
|---|-------|---------|---------------|-----|-----------|
| CS-1 | [Title] | 4/5 | foundation | Classic | Yes |
| CS-2 | [Title] | 3/5 | core concept | Recent | No (redundant with CS-8) |
...

## Approved Source Library

### SRC-1
- **Title**: [title]
- **URL**: [url]
- **Author**: [author]
- **Date**: [date]
- **Type**: [article | blog | paper | video | ...]
- **Learning stage**: [Foundations | Core Concepts | Techniques | Deep Dives | Frontiers | Perspectives]
- **Why selected**: [One sentence on what makes this a gem]

### SRC-2
...

## User Modifications
[Record any changes the user requested: additions, removals, focus adjustments]
```

---

## Phase 3: Fetch & Extract

### Goal: Deep extraction of all approved sources, type-aware strategy per source.

Launch **parallel agents per source** (one agent per source, or group 2-3 short sources per agent). Each agent extracts based on source type:

### For articles and blog posts (digest-style extraction):
1. Fetch full text using WebFetch
2. Extract **5-10 candidate passages** per source
3. Record: exact text, section/heading, one-line note on why it matters
4. Record: core claim in one sentence, publication context

### For videos and talks (watch-style extraction):
1. Fetch transcript using yt-dlp or WebFetch
2. Clean auto-generated caption artifacts
3. Extract **5-10 candidate passages** — the speaker's actual words, cleaned but faithful
4. Record: approximate timestamp, topic section, one-line note

### For academic papers (paper-style extraction):
1. Fetch PDF or HTML version using WebFetch
2. Extract **8-15 candidate passages** covering: core claims, methodology, key results, insight passages, limitations
3. Record: section, passage type, one-line note
4. Record: abstract summary, key figures/tables described

### For tutorials and documentation:
1. Fetch full content using WebFetch
2. Extract **3-8 candidate passages** — focus on explanations that illuminate, not step-by-step instructions
3. Record: section, one-line note

### For all source types, also record:
- **Core contribution**: What does this source add to the learning path? One sentence.
- **Key concepts introduced**: Terms or ideas that appear here first (or are best explained here)
- **Connections to other sources**: Where this source's ideas connect to, extend, or challenge other sources in the library

### What to look for across all types:
- **Thesis crystallizations**: Where the core idea is stated most clearly
- **Key arguments with evidence**: The reasoning that supports the claims
- **Striking examples**: Concrete cases that make abstract ideas tangible
- **Insight moments**: Where understanding suddenly deepens
- **Provocations and challenges**: Ideas that question assumptions
- **Historical anchors**: Moments that ground the topic in its development

### What NOT to extract:
- Filler, preamble, or "in this article we will discuss" framing
- Content that only makes sense in the original context
- Promotional or self-referential content
- Step-by-step instructions without conceptual insight
- Redundant passages across sources — if two sources say the same thing, note the connection but extract only the stronger formulation

### Checkpoint: Save `extraction-[source-slug].md` per source to `.readweave/[topic-slug]-research/` with all candidate passages, metadata, and per-source context.

---

## Phase 4: Map & Organize — Build the Learning Path

### Goal: Organize all extracted content into a pedagogical arc with 4-6 learning stages.

### Step 1: Define the learning stages

Design **4-6 stages** that form a natural progression from beginner understanding to advanced mastery. The exact stages depend on the topic, but the general pattern is:

| Stage | Purpose | Typical content |
|-------|---------|-----------------|
| **Foundations** | Essential concepts, history, motivation | Classic papers, foundational articles, "why this matters" |
| **Core Concepts** | The central ideas and mechanisms | Technical articles, key tutorials, seminal explanations |
| **Techniques & Practice** | How it's applied, built, or implemented | Implementation guides, practitioner perspectives, case studies |
| **Deep Dives** | Advanced topics, nuances, edge cases | In-depth technical articles, research papers, expert analyses |
| **Frontiers** | Current state, open problems, recent advances | Latest research, cutting-edge blog posts, conference talks |
| **Perspectives** | Criticism, debate, alternative viewpoints | Opinion pieces, critiques, meta-analyses, retrospectives |

Not every topic needs all 6 stages. A narrow technical topic might have 4 stages; a broad interdisciplinary topic might need 6.

### Step 2: Assign sources to stages

Each source maps to its **primary stage** — the stage where it contributes most to the learning path. A source can contribute passages to adjacent stages, but it has one home.

### Step 3: Plan the narrative arc within each stage

For each stage:
1. What question does this stage answer for the learner?
2. What sources contribute to answering it?
3. What order should the sources appear within the stage?
4. What are the key transitions between sources within the stage?

### Step 4: Identify cross-stage connections

Find moments where:
- A foundation concept is refined or challenged in a later stage
- A frontier development connects back to a historical origin
- Two sources from different stages illuminate the same idea from different angles
- A technique stage shows the practical consequence of a theory stage

Record these connections — they become cross-reference margin notes and evolution slides.

---

## Phase 5: Curate & Select

### Goal: Select 55-70 excerpts across all sources, organized by learning stage.

### Three-pass selection:

**Pass 1 — The non-negotiables (3-5 per source).** The core insight, the key argument, the defining example. These are the passages that earn this source its place in the library.

**Pass 2 — The learning arc builders (2-3 per source).** Passages that connect to other sources or bridge learning stages. Where the author extends, challenges, or refines ideas from elsewhere. These are where the cross-source value lives.

**Pass 3 — The texture (1-2 per source).** The striking example, the provocative aside, the beautifully clear explanation. These keep the presentation alive.

### Selection quality tests:
1. **The learning test**: Does this excerpt advance the reader's understanding of the topic?
2. **The stand-alone test**: Can someone understand this passage without reading the full source?
3. **The redundancy test**: Does another excerpt capture this idea better? Keep the sharper formulation.
4. **The stage test**: Does this excerpt belong in the learning stage it's assigned to?
5. **The diversity test**: Does the selection represent the full range of sources, not just the 2-3 best ones?

### Passage length:
- Sweet spot: 40-200 words. Short enough to read on a slide, long enough to carry meaning.
- Use `<span class="elide">[…]</span>` for minimal trims.
- Preserve inline citations and references where they add context.

### Representation balance:
- Not all sources deserve equal representation. A foundational paper that shaped the field might contribute 6-8 excerpts; a short blog post with one key insight might contribute 2-3.
- Every approved source MUST contribute at least 2 excerpts. If a source can't contribute 2 worthy excerpts, it shouldn't have been selected as a gem.

### Checkpoint: Save `selection.md` to `.readweave/[topic-slug]-research/` with all selected excerpts, learning stage assignments, cross-stage connections, and ordering rationale.

---

## Phase 6: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**.

### Labels must be:
- Descriptive concepts: "The Impossibility Boundary", "Practical vs. Theoretical", "The Naming Problem", "Historical Echo", "The Simplicity Trap", "Convergent Discovery"
- Bad: "Key Point!", "Important!", "See Also", "Cross-Reference"

### Note body must be:
- **Analytical, not summary** — never restate what the excerpt says
- **Cross-source connections** — "This refines the idea first articulated in [Source Title] ([Year])..."
- **Cross-stage connections** — "The foundations laid here will be challenged in [Stage Name], where [Source] shows..."
- **Historical context** — when was this written? What was the state of the field?
- **Hidden implications** — what follows from this that the author doesn't spell out?
- **Practical significance** — for technique passages, what does this mean for practitioners?

### Prose voice:
- Write in a calm, literate register — the tone of a knowledgeable guide, not a lecturer or enthusiast
- Be specific: name sources, authors, methods, and results rather than gesturing at "the literature"
- Avoid enthusiasm markers ("groundbreaking", "elegant", "remarkable") — let the work speak for itself
- State connections plainly rather than dramatizing them
- The best margin notes sound like something a well-read colleague would write: precise, contextual, genuinely clarifying

This voice also applies to all commentary prose: overview slides, transition slides, synthesis slides, and discussion questions.

### At least 30% of margin notes must make cross-source connections. This is what makes a research presentation more than a reading list.

### Checkpoint: Save `annotations.md` to `.readweave/[topic-slug]-research/` with all annotated excerpts (including cross-source notes), overview prose, stage summaries, and discussion questions.

---

## Phase 7: Generate HTML Slides

### Chunked generation strategy

**This phase uses the Scaffold-Batch-Assemble pattern.** Read `**/chunked-html-generation.md` within this plugin's installation directory for the full specification.

**Research-specific batch planning:** Organize content batches by learning stage (max 8 slides each). Structural batches: opening (title, source library, learning path roadmap) and closing (bibliography, discussion, closing).

### Setup:

1. **Read the template**: Use the Glob tool to find `**/study-template.html` within this plugin's installation directory, then read it with the Read tool.
2. **Read the design guide**: Use the Glob tool to find `**/study-design-guide.md` within this plugin's installation directory, then read it.
3. **Replace placeholders**:
   - `{{STUDY_TITLE}}` → a title capturing the learning journey (e.g., "Distributed Consensus: From Impossibility to Practice")
   - `{{SIDEBAR_SUBTITLE}}` → the topic name
   - `{{BOOKS_SUBTITLE}}` → source count and stage count (e.g., "15 Sources Across Five Stages")
4. **Populate slides** inside `<div class="slides-wrapper">`:

### Slide structure (70-85 total):

- **Slide 0 — Title**: `class="slide active no-margin"` — the research title, topic, source count, and a unifying quotation from the most impactful source. Uses `study-title-page` class. Includes `begin-hint`.

- **Slide 1 — Source Library**: `class="slide no-margin"` — overview of all sources in the collection. Uses `.book-card` elements (repurposed for sources): each shows the source title, author, year, type, and one-sentence contribution. Grouped by learning stage.

- **Slide 2 — Learning Path Roadmap**: `class="slide no-margin"` — visual overview of the learning path. Shows the stages, their relationships, and what the reader will learn at each stage. A map of the journey ahead.

- **Slides 3 through N — Content slides** (the bulk): A mix of:
  - **Excerpt slides** (`class="slide"`): Direct source excerpts with margin notes. The `page-chapter` label includes the source: `"Source Title &middot; Author (Year)"`. Use `data-book` attribute with the **learning stage number** for color coding (stage 1 = `data-book="1"`, stage 2 = `data-book="2"`, etc.).
  - **Evolution slides** (`class="slide evolution-slide"`): Side-by-side comparison of how an idea appears in different sources across time or perspective. Uses `.evolution-panel` elements.
  - **Stage transition slides** (`class="slide no-margin book-transition"`): Brief orienting slide when focus shifts to a new learning stage. Shows stage name, source count, and a one-sentence framing of what this stage covers. Uses the stage's assigned color via `data-book`.
  - **Synthesis slides** (`class="slide no-margin"`): Analysis of cross-source patterns — how understanding of a concept has evolved, where sources agree or disagree, what the synthesis reveals.

- **Slide N+1 — Bibliography**: `class="slide no-margin"` — full annotated bibliography of all 12-20 sources. Each entry includes: title, author(s), year, source type, URL, and a one-sentence annotation on its contribution to the learning path. Organized by learning stage.

- **Slide N+2 — Discussion**: `class="slide no-margin"` — 8-10 questions, prioritizing questions that span learning stages or require synthesizing insights from multiple sources. Uses `.discussion-q` elements.

- **Slide N+3 — Closing**: `class="slide no-margin"` — the learning journey distilled + a forward-looking quotation + the topic's current state in one paragraph + attribution.

### Formatting rules:
- Same typographic rules as other skills (elisions, em-dashes, smart quotes, drop caps)
- Each slide needs a descriptive `data-title` attribute
- First slide has `class="active"`
- `page-chapter` labels on excerpt slides MUST include the source title and author for orientation
- `page-source` should include the source URL for reference
- Use `data-book` attribute to map **learning stages** (not individual sources) to the study template's book color system
- For inline math notation: use Unicode where possible, or `<span class="math">...</span>` for short expressions

### Slide count guidance:
- **12-14 sources**: 55-70 total slides
- **15-17 sources**: 65-80 total slides
- **18-20 sources**: 75-85 total slides
- Roughly: 50% excerpt slides (35-45), 10% evolution/comparison slides (4-6), 7% stage transition slides (4-6), 5% synthesis slides (3-4), 28% structural slides (title, library, roadmap, bibliography, discussion, closing)

### Batch planning for research:

| Batch | Content |
|-------|---------|
| 1 | Title, Source Library, Learning Path Roadmap (structural opening) |
| 2-N | One batch per learning stage (max 8 slides each, including stage transition + excerpts + any evolution slides for that stage) |
| N+1 | Synthesis slides (cross-stage analysis) |
| N+2 | Bibliography, Discussion, Closing (structural closing) |

Save as `[topic-slug]-research.html`.

---

## Phase 8: Refine

Review the output:

1. **Learning path check**: Does the presentation build understanding progressively? Can someone unfamiliar with the topic follow the journey from foundations to frontier?
2. **Cross-source audit**: Are there enough evolution slides and cross-source margin notes? At least 30% of margin notes must make cross-source connections. The presentation should feel like a *curated learning path*, not a playlist of article summaries.
3. **Temporal balance check**: Does the presentation span eras? Are foundational works represented alongside cutting-edge content?
4. **Source balance check**: Is any source over- or under-represented? Every source should contribute at least 2 excerpts; no single source should dominate more than 15% of the slides.
5. **Stage balance check**: Do the learning stages flow naturally? Is any stage too thin or too heavy?
6. **Margin note audit**: Verify no margin note merely restates its excerpt.
7. **Flow check**: Do stage transitions orient the reader? Is the reader always clear about which source they're reading from?
8. **Attribution check**: Every excerpt slide must identify its source (title, author, year). The bibliography must list all sources with URLs.
9. **Bibliography completeness**: All 12-20 sources appear in the bibliography with accurate URLs and annotations.
10. **Technical check**: Well-formed HTML, all slides have `data-title`, first slide has `class="active"`.

---

## Important Notes

- **This is a learning path, not a link dump**: The value is in the curation, organization, and cross-source connections. A chronological list of excerpts is not a research presentation.
- **Direct excerpts only**: Show the authors' actual words. Margin notes and synthesis slides provide the analysis.
- **Temporal diversity is mandatory**: The collection must span the topic's history. A research presentation on distributed systems that starts in 2020 is missing the foundations; one that stops in 2010 is missing the frontier.
- **The margin notes are the value-add**: Cross-source annotations — showing how an idea from a 1985 paper connects to a 2024 blog post, or how a practitioner's experience validates (or contradicts) a theorist's prediction — are what make this a *research presentation* rather than a reading list.
- **Learning stages, not source clusters**: Organize by pedagogical progression, not by grouping similar sources. The reader should feel they are building understanding, not browsing categories.
- **Colors map to stages, not sources**: The `data-book` attribute maps to learning stages (Foundations = 1, Core = 2, etc.), providing visual continuity as the reader progresses through the learning path.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **No content limit**: If the topic warrants 85 slides, use 85 slides. Quality and coverage determine length, not an arbitrary cap. But every slide must earn its place.
- **The bibliography is a slide, not a footnote**: With 12-20 sources, a proper annotated bibliography deserves its own slide(s). This is the reader's roadmap for further exploration.
