---
description: "Analyze a YouTube playlist as a curated sequence. Extracts transcripts, traces how ideas build across videos, and produces a unified HTML slide presentation following the curator's intended arc."
argument-hint: "[YouTube playlist URL]"
---

# Playlist Skill

You will create one output from the following YouTube playlist: $ARGUMENTS
- A unified **HTML slide presentation** (`[playlist-name]-playlist.html`)

The presentation follows the playlist's intended sequence — treating it as a curated intellectual path — and traces how ideas develop, recur, and deepen across the series.

Follow the six phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[playlist-slug]/` (derive slug from the playlist title — lowercase, kebab-case, drop articles).

**Read the checkpoint reference**: Use the Glob tool to find `**/checkpoints.md` within this plugin's installation directory, then read it for the full format specification.

**Resume logic** — check in reverse order:
1. If `annotations.md` exists → load it and skip to Phase 6 (HTML generation)
2. If `selection.md` exists → load it and skip to Phase 5 (annotations)
3. If `extraction-[video-slug].md` files exist → load them and skip to Phase 3 (arc mapping). If only some videos have extraction checkpoints, extract only the missing ones.
4. If no checkpoints → start from Phase 1

If resuming, report: "Found checkpoint for [phase]. Resuming from [next phase]..."

If the user's input contains `--fresh`, ignore all checkpoints and start from scratch.

**Save after each phase** — write the checkpoint file in the format specified by the checkpoint reference before proceeding to the next phase.

---

## Phase 1: Discover

### Fetch the playlist metadata:

1. Use WebFetch to retrieve the YouTube playlist page from the provided URL
2. Extract the playlist's **title**, **creator/curator**, **description**, and **video count**
3. For each video in the playlist, extract:
   - **Title**
   - **URL** (or video ID)
   - **Channel/speaker name** (may differ from playlist curator if it's a curated collection)
   - **Duration**
   - **Position** in the playlist (the curator's intended order)
4. If the playlist has more than 30 videos, inform the user and ask whether to proceed with all or select a subset

### Handle different playlist types:

- **Single-creator series** (e.g., a lecture course, a video essay series): All videos by the same person, building sequentially
- **Curated collection** (e.g., "Best talks on X", a conference playlist): Videos by different speakers, grouped by theme
- **Mixed format**: Talks, interviews, panels, tutorials — note the format of each video

Report to the user: "Found N videos in [Playlist Title]. Fetching transcripts in the curator's sequence..."

---

## Phase 2: Fetch & Extract

### Fetch transcripts for all videos in parallel:

Launch **parallel agents** (one per video, or group 2-3 short videos per agent). Each agent:

1. Extracts the transcript using one of these methods (try in order):
   - Use `yt-dlp --write-auto-sub --sub-lang en --skip-download --print-to-file subtitle <output_path> <URL>` to extract subtitles
   - If `yt-dlp` is unavailable, use WebFetch to retrieve the transcript from a transcript service
2. If no transcript is available for a video, note it and skip — do not block the entire playlist
3. Cleans the transcript: remove duplicate phrases and repetitive words, fix obvious speech-to-text errors, normalize punctuation
4. Divides the transcript into logical sections based on topic shifts
5. For each video, extracts **5-10 candidate passages** (fewer than books — talks are shorter and denser)
6. Records for each passage:
   - The **exact spoken words**, cleaned of caption artifacts but faithful to the speaker's phrasing
   - The **video title** and **approximate timestamp**
   - A **one-line note** on why it matters
7. Records for each video:
   - The **core claim** in one sentence
   - How it **relates to adjacent videos** in the playlist — does it build on, extend, contrast with, or deepen the previous video?
   - Any **recurring concepts** that appeared in earlier videos

### What to look for:

Same five passage types as the watch skill (thesis crystallizations, key arguments, striking examples, rhetorical high points, provocations), plus:

- **Callbacks and continuations**: Where a speaker references or builds on an idea from an earlier video in the playlist — the curator placed them in this order for a reason
- **Deepening moments**: Where a concept introduced briefly in an earlier video gets its full treatment
- **Tension across videos**: Where two videos in the playlist offer different or competing perspectives on the same question

### What NOT to extract:
- Opening pleasantries, event-specific references, or self-promotion
- Content that only makes sense with the video's visual aids and cannot be contextualized in a margin note
- Filler, verbal stumbles, or Q&A unless a response is genuinely revelatory
- Redundant formulations — if the same idea appears across multiple videos, take the strongest version and note the pattern

### Checkpoint: Save `extraction-[video-slug].md` per video to `.readweave/[playlist-slug]/` with all candidate passages, video metadata, and per-video context.

---

## Phase 3: Map the Arc

### Trace the playlist's intellectual progression:

Unlike the crawl skill (which discovers threads from scratch), this skill **respects the curator's sequence** as the primary structure. The playlist order is treated as an intentional curriculum.

1. **Map the sequence**: For each video in order, identify what it contributes that the previous videos did not. What concept does it introduce, deepen, or complicate?
2. **Identify the arc**: What is the trajectory from the first video to the last? Does the playlist build from foundations to advanced ideas? From problem to solution? From theory to practice?
3. **Find recurring motifs**: Concepts, metaphors, or questions that surface across multiple videos. These become the connective tissue of the presentation.
4. **Note the pivot points**: Videos where the playlist shifts direction — from one topic area to another, from one speaker to another, from one perspective to its counterpoint.
5. **Determine conceptual weight**: Not all videos contribute equally. A 5-minute introduction might get 1 slide; a 60-minute keynote might get 6-8 slides.

### Present the arc to the user:
Show the user the discovered structure: the playlist's arc, recurring motifs, and pivot points. Ask if they want to:
- Proceed with the full playlist
- Focus on a subset of videos
- Adjust the weighting

Proceed with the user's selection.

---

## Phase 4: Curate & Select

### Select excerpts following the playlist's sequence:

1. **Select 5-10 excerpts per video** from the candidate pool, scaled by the video's conceptual weight. Favor passages that:
   - State the video's core contribution to the playlist's arc
   - Show ideas building on or connecting to earlier videos in the sequence
   - Provide the memorable examples that make ideas stick
   - Reveal the speaker's distinctive angle on a shared theme

2. **Order excerpts within each video** by their natural progression within the talk — follow the speaker's own arc.

3. **Plan cross-video connections**: Identify moments where a passage in one video echoes, extends, or challenges a passage from another video. These will become cross-reference margin notes.

**Cleaning repetitive words from excerpts:**

Apply the same cleanup as the watch skill: remove false starts ("what I mean is, what I mean is"), stammer repetitions ("the the the"), filler chains ("um, um" / "you know, you know"), hedge piles ("kind of sort of basically"), echo phrases, auto-caption stutters ("I I I think"), and confirming ticks ("right? right?"). Keep rhythm words and rhetorical repetitions that serve emphasis. Make the excerpt readable on the page while preserving the speaker's natural voice.

### Selection quality tests:
1. **The arc test**: Does this excerpt advance the playlist's intellectual progression, or is it just a good passage that happens to be in this video?
2. **The stand-alone test**: Can someone who hasn't watched the video understand this passage?
3. **The sequence test**: Does the placement of this excerpt make sense given what comes before and after it in the presentation?
4. **The oral-to-written test**: Does this passage still land on the page, or does it depend entirely on delivery?

### Checkpoint: Save `selection.md` to `.readweave/[playlist-slug]/` with all selected excerpts, arc analysis, cross-video connections, and conceptual weight allocation.

---

## Phase 5: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**.

### Labels must be:
- Descriptive concepts: "The Ratchet Effect", "Status vs. Substance", "Compounding Insight", "The Founder Filter", "Accumulated Context"
- Bad: "Key Point!", "Great Quote!", "See Also"

### Note body must be:
- **Analytical, not summary** — never restate what the excerpt says
- **Cross-video connections** — "This deepens the idea introduced in [Earlier Video Title], where [Speaker] described..."
- **Sequence-aware** — "The playlist places this immediately after [Previous Video], and the juxtaposition reveals..."
- **Contextual** — who is this speaker? What are they known for? What were they responding to?
- **Hidden implications** — what follows from this that the speaker doesn't spell out?
- **Visual context where needed** — if the speaker was referencing a slide, demo, or visual at this moment, briefly describe what the audience was seeing

### Prose voice:
- Write in a calm, literate register — the tone of a thoughtful viewer's notes in the margin, not a lecturer or enthusiast
- Favor observations that unfold naturally over declarations that announce themselves
- Avoid rhetorical questions used for emphasis, exclamation marks, and breathless discovery language ("This is the crucial move", "Here we see the key insight", "This changes everything")
- Let connections speak for themselves — state them plainly rather than dramatizing them
- The best margin notes sound like something a well-read friend might write in pencil — brief, specific, genuinely clarifying, never performing cleverness

This voice also applies to all commentary prose: overview slides, transition slides, synthesis slides, and discussion questions.

### At least 25% of margin notes should make cross-video connections. This is what makes a playlist analysis more than a collection of individual video summaries.

### Checkpoint: Save `annotations.md` to `.readweave/[playlist-slug]/` with all annotated excerpts (including cross-video notes), overview prose, and discussion questions.

---

## Phase 6: Generate HTML Slides

### Chunked generation strategy

**This phase uses the Scaffold-Batch-Assemble pattern.** Read `**/chunked-html-generation.md` within this plugin's installation directory for the full specification.

**Playlist-specific batch planning:** One content batch per video cluster (max 8 slides each). Structural batches: opening (title, arc overview) and closing (discussion, closing).

### Setup:

1. **Read the template**: Use the Glob tool to find `**/study-template.html` within this plugin's installation directory, then read it with the Read tool. This file contains the full HTML/CSS/JS scaffold for the multi-section slide presentation.
2. **Read the design guide**: Use the Glob tool to find `**/study-design-guide.md` within this plugin's installation directory, then read it. It documents the HTML structure for multi-section slide types.
3. **Replace placeholders**:
   - `{{STUDY_TITLE}}` → a title capturing the playlist's intellectual arc (e.g., "Building Systems That Scale" or use the playlist's own title if it's strong)
   - `{{SIDEBAR_SUBTITLE}}` → Creator/Curator name, or "Various Speakers" for curated collections
   - `{{BOOKS_SUBTITLE}}` → number of videos (e.g., "A Sequence in Twelve Talks")
4. **Populate slides** inside `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — the playlist title, curator, video count, a unifying quotation from the series. Uses `study-title-page` class. Includes `begin-hint`.

- **Slide 1 — The Sequence**: `class="slide no-margin"` — overview of all videos in playlist order. Uses `.book-card` elements repurposed for videos: each shows the video title, speaker (if different from curator), duration, and one-sentence description of its contribution. Videos listed in playlist order.

- **Slide 2 — Roadmap**: `class="slide no-margin"` — the playlist's intellectual arc in brief. A visual diagram showing how the sequence builds — what each stage of the playlist contributes.

- **Slides 3 through N — Content slides** (the bulk): A mix of:
  - **Excerpt slides** (`class="slide"`): Direct transcript excerpts with margin notes. The `page-chapter` label includes the video title and approximate timestamp: `"Video Title &middot; ~12:30"`. Use `data-book` attribute with the video's position number for color coding.
  - **Evolution slides** (`class="slide evolution-slide"`): Side-by-side comparison of how an idea appears in different videos. Uses `.evolution-panel` elements. Especially useful when the playlist shows an idea being introduced simply and then developed rigorously.
  - **Video transition slides** (`class="slide no-margin book-transition"`): Brief orienting slide when focus shifts to a new video. Shows video title, speaker, duration, and a one-sentence framing of what this video contributes to the arc. Uses the video's assigned color.
  - **Synthesis slides** (`class="slide no-margin"`): Slides that step back and analyze a pattern across multiple videos in the sequence.

- **Slide N+1 — Discussion**: `class="slide no-margin"` — 6-8 questions, prioritizing questions that span multiple videos or trace the playlist's cumulative argument.

- **Slide N+2 — Closing**: `class="slide no-margin"` — the playlist's intellectual arc distilled + a final quotation + attribution with link to the original playlist.

### Formatting rules:
- Same typographic rules as the read and study skills
- Each slide needs a descriptive `data-title` attribute
- First slide has `class="active"`
- `page-chapter` labels on excerpt slides MUST include the video title for orientation
- `page-source` should include the video URL and approximate timestamp for reference
- Use `data-book` attribute to map videos to the study template's color system. For playlists with more than 6 videos, cycle the colors or group related videos under the same color.

### Slide count guidance:
- **3-5 videos**: 15-25 total slides
- **6-10 videos**: 25-40 total slides
- **11-20 videos**: 35-55 total slides
- **20+ videos**: 45-65 total slides (consider grouping minor videos)
- Roughly: 50% excerpt slides, 15% evolution/cross-reference, 15% transition/synthesis, 20% structural

Save as `[playlist-name]-playlist.html`. Derive the name from the playlist title or unifying theme: e.g., `ycombinator-startup-school-playlist.html`.

---

## Important Notes

- **Sequence, not threads**: Unlike the crawl skill, this skill respects the curator's ordering. The playlist is a path, not a web. Present it as the intended journey.
- **Direct transcript excerpts only**: Show the speakers' actual words. Margin notes provide the analysis.
- **Attribution matters**: Every excerpt slide must identify which video it came from. Include the video title, speaker, and timestamp in the `page-chapter` label and `page-source` citation.
- **Handle multi-speaker playlists**: If the playlist features different speakers, the margin notes should help readers understand each speaker's perspective and how it fits the curator's arc.
- **Respect the oral medium**: Spoken language has a different rhythm than prose. Light cleanup is fine, but don't sanitize speakers' voices into essay prose. The best transcript excerpts preserve the energy of speech while being readable on the page.
- **Not all videos need equal weight**: A 3-minute intro video might get 1 slide. A 45-minute deep dive might get 6-8 slides. Weight by conceptual contribution, not duration alone.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **The margin notes are the value-add**: Cross-video annotations — showing how an idea from Video 3 deepens in Video 9 — are what make this a *study* of a playlist rather than a collection of video notes.
