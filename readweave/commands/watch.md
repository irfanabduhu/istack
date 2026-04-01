---
description: "Deep dive into a single YouTube video. Extracts transcript, identifies key moments, and produces an HTML slide presentation with analytical margin annotations."
argument-hint: "[YouTube video URL]"
---

# Watch Skill

You will create one output from the following YouTube video: $ARGUMENTS
- An elegant **HTML slide presentation** (`[video-name]-watch.html`)

Follow the four phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[video-slug]/` (derive slug from the video title — lowercase, kebab-case, drop articles).

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

### Fetch the video metadata and transcript:

1. Use WebFetch to retrieve the YouTube video page from the provided URL
2. Extract the video's **title**, **channel/creator name**, **upload date**, and **description**
3. Extract the transcript using one of these methods (try in order):
   - Use `yt-dlp --write-auto-sub --sub-lang en --skip-download --print-to-file subtitle <output_path> <URL>` to extract subtitles
   - If `yt-dlp` is unavailable, use WebFetch to retrieve the transcript from a transcript service (e.g., append `&fmt=srv3` to the timed text URL found in the page source)
4. If no transcript is available in any form, inform the user and stop

### Parse the transcript:

- Clean auto-generated caption artifacts: remove duplicate phrases, fix obvious speech-to-text errors where meaning is clear, normalize punctuation
- **Remove repetitive words and verbal tics**: Edit out stammers, false starts, and habitual filler phrases that don't carry meaning
- Preserve the speaker's natural phrasing — do not "improve" their language
- Identify **timestamps** for key moments where possible (even approximate ones help)
- Divide the transcript into logical sections based on topic shifts, pauses, or explicit transitions ("Now let's talk about...", "The next thing is...")

### Extract candidate passages:

Collect **8-20 candidate passages** depending on video length. For a typical talk (15-60 minutes), aim for 10-15 candidates.

For each candidate passage, capture:
- The **exact spoken words**, cleaned of caption artifacts but faithful to the speaker's phrasing
- The **approximate timestamp** or section where it appears
- A **one-line note** on why this passage matters

**What to look for — the five passage types:**

- **Thesis crystallizations**: Where the speaker states their core argument with maximum clarity. In talks, these often come after a story or example, not in the opening — speakers build toward their sharpest formulations.
- **Key arguments**: Where the speaker makes a logical move the rest of the talk depends on — a distinction, an analogy, a reductio. In video, these are often marked by a change in pace or emphasis.
- **Striking examples**: Where an abstract idea is made concrete through a story, demonstration, case study, or analogy. In talks, the example often IS the argument — speakers think through stories.
- **Rhetorical high points**: Where the speaker reaches peak intensity — passages that are powerfully delivered, often with deliberate pacing or repetition.
- **Provocations**: Where the speaker challenges assumptions or received wisdom. Moments that would make the audience shift in their seats.

**What NOT to extract:**
- Opening pleasantries, thank-yous, or event-specific references ("It's great to be here at TED...")
- Filler, repetition, or verbal stumbles that don't carry meaning
- Q&A segments unless a response contains a genuinely revelatory formulation
- Self-promotion or calls to action ("Check out my book...", "Subscribe to...")
- Fragmentary statements that depend heavily on visual slides or demonstrations you cannot capture

### Checkpoint: Save `extraction.md` to `.readweave/[video-slug]/` with all candidate passages, video metadata, and section structure.

---

## Phase 2: Analyze & Select

### Map the argument:

1. Identify the video's **core thesis** in one sentence
2. Map its **structure** — how does the talk build? What is the arc from opening to conclusion?
3. Note what the speaker is **responding to** — what assumption, trend, or opposing view prompted this talk?
4. Note any **visual dependencies** — moments where the speaker relies heavily on slides, demos, or physical props. Flag these so margin notes can provide context.

### Select excerpts:

From the candidate pool, select **8-15 excerpts** for the presentation.

**Two-pass selection:**

**Pass 1 — The essentials.** The thesis statement, the key argument, the defining example. These are the 4-6 passages you cannot leave out. If you removed them, the talk's argument would collapse.

**Pass 2 — The texture.** Add passages that give the talk its distinctive voice — the memorable analogy, the sharp provocation, the honest qualification, the moment of humor that carries a point. Talks live or die by their texture more than articles do.

**Selection quality tests:**
1. **The idea test**: Can you state what this passage captures in one sentence?
2. **The stand-alone test**: Can someone who hasn't watched the video understand this passage?
3. **The redundancy test**: Does another selected excerpt already capture this idea?
4. **The oral-to-written test**: Does this passage still land on the page, or does it depend entirely on delivery? If it only works spoken, cut it unless you can provide enough context in a margin note.

**Passage length:**
- Sweet spot: 40-150 words / 2-6 sentences. Spoken language is looser than written — trim light filler but preserve the speaker's natural rhythm.
- Never truncate mid-thought. Use `<span class="elide">[…]</span>` for minimal trims.
- Preserve the speaker's wording. Clean caption artifacts, but do not rewrite their sentences into "better" prose.

**Cleaning repetitive words from excerpts:**

Video transcripts contain verbal patterns that read poorly on the page. Edit these out:

- **False starts and restarts**: "What I mean is, what I mean is..." → "What I mean is..."
- **Stammer repetitions**: "the the the key point" → "the key point"
- **Filler chains**: "um, um, um" or "you know, you know, you know" → remove entirely or reduce to one
- **Hedge piles**: "kind of sort of basically" → keep the strongest one or remove all if redundant
- **Echo phrases**: "it's important, it's important to understand" → "it's important to understand"
- **Auto-caption stutters**: "going to going to" or "I I I think" → "going to" / "I think"
- **Confirming ticks**: "right? right?" or "okay? okay?" → remove trailing confirmations

**Keep the speaker's voice**: rhythm words ("well," "so," "now," "look") and emphasis repetitions ("again and again," "over and over") that serve rhetorical purpose should stay. Only remove mechanical repetition that serves no communicative function.

### Checkpoint: Save `selection.md` to `.readweave/[video-slug]/` with the argument map, selected excerpts, and visual dependency notes.

---

## Phase 3: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**.

### Labels must be:
- Descriptive concepts: "The Selection Pressure", "Convexity Argument", "Status Game", "Founder Mode", "The Bottleneck"
- Bad: "Key Point!", "Important!", "Note", "Interesting!"

### Note body must be:
- **Analytical, not summary** — never restate what the excerpt says
- **Contextual** — connect to broader ideas, other thinkers, the speaker's other work, real-world implications
- **Hidden implications** — surface what a casual viewer might miss
- **Concrete** — use specific references, named examples, real events
- **Visual context where needed** — if the speaker was referencing a slide, demo, or visual aid at this moment, briefly describe what the audience was seeing, so the reader isn't lost

### Prose voice:
- Write in a calm, literate register — the tone of a thoughtful viewer's notes in the margin, not a lecturer or enthusiast
- Favor observations that unfold naturally over declarations that announce themselves
- Avoid rhetorical questions used for emphasis, exclamation marks, and breathless discovery language ("This is the crucial move", "Here we see the key insight", "This changes everything")
- Let connections speak for themselves — state them plainly rather than dramatizing them
- The best margin notes sound like something a well-read friend might write in pencil — brief, specific, genuinely clarifying, never performing cleverness

This voice also applies to all commentary prose: overview slides and discussion questions.

### Quality test: Cover the excerpt text. Can someone learn something new from just the margin note? If not, rewrite it.

### Checkpoint: Save `annotations.md` to `.readweave/[video-slug]/` with all annotated excerpts, overview prose, and discussion questions.

---

## Phase 4: Generate HTML Slides

1. **Read the template**: Use the Glob tool to find `**/gist-template.html` within this plugin's installation directory, then read it with the Read tool. This file contains the full HTML/CSS/JS scaffold for the slide presentation.
2. **Read the design guide**: Use the Glob tool to find `**/read-design-guide.md` within this plugin's installation directory, then read it. It documents the HTML structure for each slide type.
3. **Replace placeholders**:
   - `{{BOOK_TITLE}}` → the video's title (used in both `<title>` and sidebar header)
   - `{{SIDEBAR_SUBTITLE}}` → "Creator Name, Platform/Date"
4. **Populate slides** inside the `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — video title, creator/channel, date, and an opening quotation from the talk. Include `begin-hint`.

- **Slide 1 — Overview**: `class="slide no-margin"` — the video's core argument in brief. Use a visual diagram (concept-cards, or flow diagram) plus commentary. Note the talk's context: what event, what audience, what prompted it. Frame what follows: "The following pages present [Speaker]'s argument *in their own words*."

- **Slides 2 through N — Excerpts**: `class="slide"` — direct transcript excerpts with margin annotations. This is the bulk (8-15 slides). Each slide has:
  - `page-chapter` label with the section or topic area, and approximate timestamp if available (e.g., "The Core Problem &middot; ~12:30")
  - `page-divider`
  - One or more `.excerpt` paragraphs (first excerpt on most slides gets `.drop-cap`)
  - `page-source` citation (video title, creator)
  - `.margin` div with 2-3 `.margin-note` elements

- **Slide N+1 — Discussion**: `class="slide no-margin"` — 4-6 thought-provoking questions using `.discussion-q` elements

- **Slide N+2 — Closing**: `class="slide no-margin"` — the video's thesis distilled + a final quotation + creator/title/date attribution

### Formatting rules:
- Use `<span class="elide">[…]</span>` for elisions within excerpts
- Use literal Unicode: `—` for em-dashes, `'` / `'` for single quotes, `"` / `"` for double quotes (do NOT use HTML entities — they break in CSS `content:` properties)
- Apply `drop-cap` class to the first `.excerpt` on most excerpt slides
- Each slide needs a descriptive `data-title` attribute for the sidebar TOC

### Slide count guidance:
- Short video (<15 minutes): 8-12 total slides
- Medium video (15-45 minutes): 12-18 total slides
- Long video (45+ minutes): 18-25 total slides

Save as `[video-name]-watch.html` in the working directory. Derive `[video-name]` from the video title: lowercase, kebab-case, drop articles.

---

## Important Notes

- **Direct transcript excerpts only**: The presentation shows the speaker's actual words. Margin notes provide the analysis.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **Preserve the aesthetic**: Reuse the book-page design from the template — parchment colors, Garamond fonts, Caveat margin notes, drop caps.
- **Attribute the source**: Always include the original YouTube URL, creator name, channel, and upload date in the title slide and closing slide.
- **Handle the oral-to-written gap**: Spoken language has a different rhythm than prose. Light cleanup of filler words and false starts is fine, but don't sanitize the speaker's voice into essay prose. The best transcript excerpts preserve the energy of speech while being readable on the page.
- **The margin notes are the value-add**: Anyone can watch a video. The margin notes — analytical, contextual, concrete — are what make this a *study* rather than a bookmark.
