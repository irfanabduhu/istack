---
description: "Analyze a podcast series across multiple episodes. Discovers episodes from an RSS feed or multiple audio files, traces recurring themes and evolving ideas, and produces a unified HTML slide presentation following the show's intellectual arc."
argument-hint: "[podcast RSS feed URL or paths to multiple audio files]"
---

# Podcast Skill

You will create one output from the following podcast source: $ARGUMENTS
- A unified **HTML slide presentation** (`[podcast-name]-podcast.html`)

The presentation traces how ideas develop, recur, and deepen across a podcast series — treating the show as a body of work rather than isolated episodes.

Follow the six phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[podcast-slug]/` (derive slug from the podcast name — lowercase, kebab-case, drop articles).

**Read the checkpoint reference**: Use the Glob tool to find `**/checkpoints.md` within this plugin's installation directory, then read it for the full format specification.

**Resume logic** — check in reverse order:
1. If `annotations.md` exists → load it and skip to Phase 6 (HTML generation)
2. If `selection.md` exists → load it and skip to Phase 5 (annotations)
3. If `extraction-[episode-slug].md` files exist → load them and skip to Phase 3 (arc mapping). If only some episodes have extraction checkpoints, extract only the missing ones.
4. If no checkpoints → start from Phase 1

If resuming, report: "Found checkpoint for [phase]. Resuming from [next phase]..."

If the user's input contains `--fresh`, ignore all checkpoints and start from scratch.

**Save after each phase** — write the checkpoint file in the format specified by the checkpoint reference before proceeding to the next phase.

---

## Phase 1: Discover

### Determine source type and discover episodes:

**If the input is an RSS feed URL:**
1. Use WebFetch to retrieve the RSS feed
2. Parse the feed to extract:
   - **Podcast title**, **host(s)**, **description**
   - For each episode: **title**, **audio URL**, **publication date**, **description/show notes**, **duration**, **guest(s)** (parse from title or description)
3. Sort episodes chronologically
4. If the feed has more than 30 episodes, inform the user and present a selection strategy:
   - All episodes (may take significant time)
   - Most recent N episodes
   - A date range
   - User-specified episode selection

**If the input is a podcast page URL** (not RSS):
1. Use WebFetch to retrieve the page
2. Look for the RSS feed link in:
   - `<link rel="alternate" type="application/rss+xml">`
   - Common podcast platform patterns (Apple Podcasts, Spotify page → RSS)
3. If RSS found, follow the RSS path above
4. If no RSS found, extract episode links from the page directly

**If the input is multiple audio files:**
1. List all provided files
2. Attempt to determine episode order from filenames, metadata, or user-specified order
3. Extract any metadata from file tags (ID3 for mp3, etc.) using:
   ```bash
   ffprobe -v quiet -print_format json -show_format "$AUDIO_PATH" 2>/dev/null
   ```

### Present discovery to user:

Report: "Found N episodes of [Podcast Name]. The series spans [date range]."

If more than 15 episodes, suggest focusing on a subset and ask the user to confirm which episodes to include.

---

## Phase 2: Transcribe & Extract

### Transcribe all selected episodes in parallel:

Launch **parallel agents** (one per episode, or group 2-3 short episodes per agent). Each agent:

1. **Downloads the audio** (if URL): `curl -L -o /tmp/episode-XXXXXX.[ext] "[audio_url]"`

2. **Transcribes** using Whisper (try methods in order):
   ```bash
   whisper "$AUDIO_PATH" --model medium --language en --output_format txt --output_dir /tmp/whisper-out-XXXXXX
   ```
   If Whisper is unavailable, inform the user: "Whisper is required for audio transcription. Install with: `pip install openai-whisper`"

3. **Cleans the transcript**: Remove filler, fix speech-to-text errors, normalize punctuation. Apply the same cleanup rules as the watch/listen skills (false starts, stammer repetitions, filler chains, hedge piles, echo phrases).

4. **Detects speakers**: Identify host vs. guest(s) through conversational patterns and show notes.

5. **Divides into logical sections** based on topic shifts and conversation flow.

6. For each episode, extracts **5-10 candidate passages** with:
   - The **exact spoken words**, cleaned but faithful
   - The **speaker** who said it
   - The **approximate timestamp** or section
   - A **one-line note** on why it matters

7. Records for each episode:
   - The **core claim or central question** in one sentence
   - How it **relates to other episodes** — does it build on, echo, or contradict ideas from other episodes?
   - Any **recurring concepts** that appeared in earlier episodes
   - The **guest's unique contribution** (if applicable)

### What to look for:

Same seven passage types as the listen skill (thesis crystallizations, key arguments, striking examples, provocations, dialogue sparks, honest admissions, distillation moments), plus:

- **Cross-episode echoes**: Where the host returns to an idea from a previous episode — sometimes refining it with a new guest's perspective, sometimes contradicting the earlier framing
- **Recurring questions**: The questions the host keeps asking different guests — these reveal the show's central intellectual project
- **Evolving positions**: Where the host's own view shifts across episodes, often visible in how they frame questions differently over time
- **Guest disagreements**: Where two guests across different episodes take opposing positions on the same question — the host has (intentionally or not) built a dialogue across time

### What NOT to extract:
- Ad reads, sponsor segments, or promotional content
- Opening/closing boilerplate that's the same every episode
- Filler conversation without substance
- Content that depends on visuals or links the listener was expected to follow
- Redundant formulations — take the strongest version and note the pattern

### Checkpoint: Save `extraction-[episode-slug].md` per episode to `.readweave/[podcast-slug]/` with all candidate passages, episode metadata, speaker identification, and per-episode context.

---

## Phase 3: Map the Arc

### Trace the podcast's intellectual evolution:

1. **Map the episode sequence**: For each episode in chronological order, identify what it contributes that previous episodes did not. What question does it address, what concept does it introduce or deepen?

2. **Identify recurring themes**: Find **3-7 thematic threads** that run across multiple episodes:
   - **Thread name**: A descriptive phrase (e.g., "The Craft of Decision-Making", "Institutions vs. Individuals", "The Expertise Paradox")
   - **Thread description**: One sentence capturing the thread
   - **Episodes involved**: Which episodes participate
   - **How the thread develops**: Does understanding deepen? Do positions shift? Do different guests illuminate different facets?

3. **Identify the host's arc**: Over a series of conversations, hosts often develop their own evolving perspective. Trace this where visible.

4. **Note pivot points**: Episodes where the show shifts direction — new topic areas, format changes, or conversations that reframe what came before.

5. **Determine conceptual weight**: Not all episodes contribute equally. A landmark interview might get 6-8 slides; a lighter episode might get 2-3 slides.

### Present the arc to the user:

Show the discovered structure: the podcast's themes, recurring threads, and key episodes. Ask if they want to:
- Proceed with the full selection
- Focus on specific threads
- Adjust the weighting
- Suggest different groupings

Proceed with the user's selection.

---

## Phase 4: Curate & Select

### Select excerpts following the thematic threads:

1. **Select 3-8 excerpts per episode** from the candidate pool, scaled by conceptual weight. Favor passages that:
   - State the episode's core contribution to the show's arc
   - Show ideas building on or connecting to other episodes
   - Reveal the host-guest dynamic producing unique insight
   - Capture the speaker's distinctive voice on a recurring theme

2. **Order excerpts** within each thread by **conceptual progression** — start with the most foundational formulation and build toward the most mature or provocative.

3. **Plan cross-episode connections**: Identify moments where a passage in one episode echoes, extends, or challenges a passage from another. These become cross-reference margin notes.

4. **Plan cross-thread connections**: Identify 2-3 moments where threads intersect.

**Cleaning repetitive words from excerpts:**

Apply the same cleanup as the listen/watch skills: remove false starts, stammer repetitions, filler chains, hedge piles, echo phrases, auto-caption stutters, and confirming ticks. Keep rhythm words and rhetorical repetitions that serve emphasis.

### Selection quality tests:
1. **The thread test**: Does this excerpt advance its thread's intellectual arc?
2. **The stand-alone test**: Can someone who hasn't listened understand this?
3. **The evolution test**: Have you selected passages that show how ideas develop across episodes?
4. **The oral-to-written test**: Does this land on the page?
5. **The dialogue test**: For exchanges, is enough context included?

### Checkpoint: Save `selection.md` to `.readweave/[podcast-slug]/` with all selected excerpts, thread analysis, cross-episode connections, and conceptual weight allocation.

---

## Phase 5: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**.

### Labels must be:
- Descriptive concepts: "The Recurring Question", "The Host's Pivot", "Guest Disagreement", "Evolving Framework", "The Compound Insight"
- Bad: "Key Point!", "Great Quote!", "See Also"

### Note body must be:
- **Analytical, not summary** — never restate what the excerpt says
- **Cross-episode connections** — "This deepens the idea [Previous Guest] introduced in [Episode Title], but shifts the emphasis from..."
- **Thread-aware** — "This is the third time the host has asked this question — compare [Guest A]'s answer in [Episode] with [Guest B]'s framing here"
- **Speaker-aware** — note who is speaking and what perspective they bring
- **Contextual** — who is this guest? What are they known for? What prompted this conversation?
- **Hidden implications** — what follows from this that the speaker doesn't spell out?

### Prose voice:
- Write in a calm, literate register — the tone of a thoughtful listener's notes, not a lecturer or enthusiast
- Favor observations that unfold naturally over declarations that announce themselves
- Avoid rhetorical questions used for emphasis, exclamation marks, and breathless discovery language
- Let connections speak for themselves — state them plainly rather than dramatizing them
- The best margin notes sound like something a well-read friend might write in pencil — brief, specific, genuinely clarifying, never performing cleverness

This voice also applies to all commentary prose: overview slides, transition slides, synthesis slides, and discussion questions.

### At least 25% of margin notes must make cross-episode connections. This is what makes a podcast analysis more than a collection of episode summaries.

### Checkpoint: Save `annotations.md` to `.readweave/[podcast-slug]/` with all annotated excerpts (including cross-episode notes), overview prose, and discussion questions.

---

## Phase 6: Generate HTML Slides

### Chunked generation strategy

**This phase uses the Scaffold-Batch-Assemble pattern.** Read `**/chunked-html-generation.md` within this plugin's installation directory for the full specification.

**Podcast-specific batch planning:** One content batch per thread or episode cluster (max 8 slides each). Structural batches: opening (title, episode map, arc overview) and closing (discussion, closing).

### Setup:

1. **Read the template**: Use the Glob tool to find `**/study-template.html` within this plugin's installation directory, then read it with the Read tool.
2. **Read the design guide**: Use the Glob tool to find `**/study-design-guide.md` within this plugin's installation directory, then read it.
3. **Replace placeholders**:
   - `{{STUDY_TITLE}}` → a title capturing the show's intellectual arc (e.g., "Conversations on Craft" or the podcast's own title if strong)
   - `{{SIDEBAR_SUBTITLE}}` → "Host Name" or "Podcast Name"
   - `{{BOOKS_SUBTITLE}}` → episode count and thread count (e.g., "Twelve Episodes Across Four Threads")
4. **Populate slides** inside `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — the podcast title, host, episode count, date range, and a unifying quotation from the series. Uses `study-title-page` class. Includes `begin-hint`.

- **Slide 1 — The Episodes**: `class="slide no-margin"` — overview of all episodes in the study. Uses `.book-card` elements repurposed for episodes: each shows the episode title, guest (if any), date, and one-sentence description. Listed chronologically.

- **Slide 2 — The Threads**: `class="slide no-margin"` — the show's intellectual threads mapped visually. A diagram showing the themes, which episodes participate in each, and how threads interrelate.

- **Slides 3 through N — Content slides** (the bulk): A mix of:
  - **Excerpt slides** (`class="slide"`): Direct transcript excerpts with margin notes. The `page-chapter` label includes the episode and speaker: `"Episode Title &middot; [Guest Name] &middot; ~15:30"`. Use `data-book` attribute with the thread number for color coding.
  - **Evolution slides** (`class="slide evolution-slide"`): Side-by-side comparison of how different guests answer the same question, or how the host's framing of an idea shifts across episodes. Uses `.evolution-panel` elements.
  - **Thread transition slides** (`class="slide no-margin book-transition"`): Brief orienting slide when focus shifts to a new thread. Shows thread name, episode count, and a one-sentence framing.
  - **Synthesis slides** (`class="slide no-margin"`): Analysis of cross-episode patterns — how the host's understanding evolves, where guests agree or disagree, what the accumulated conversations reveal.

- **Slide N+1 — Discussion**: `class="slide no-margin"` — 6-8 questions, prioritizing questions that span episodes or trace the show's cumulative insight. Uses `.discussion-q` elements.

- **Slide N+2 — Closing**: `class="slide no-margin"` — the show's intellectual project distilled + a final quotation + attribution with podcast name, host, and episode list.

### Formatting rules:
- Same typographic rules as the read and study skills
- Each slide needs a descriptive `data-title` attribute
- First slide has `class="active"`
- `page-chapter` labels on excerpt slides MUST include the episode title and speaker for orientation
- `page-source` should include the episode title and date
- Use `data-book` attribute to map threads to the study template's color system
- Use `<span class="speaker">[Name]:</span>` for speaker attribution in dialogue excerpts

### Slide count guidance:
- **3-5 episodes**: 15-25 total slides
- **6-10 episodes**: 25-40 total slides
- **11-15 episodes**: 35-50 total slides
- **15+ episodes**: 45-65 total slides
- Roughly: 50% excerpt slides, 15% evolution/cross-reference, 15% transition/synthesis, 20% structural

Save as `[podcast-name]-podcast.html`. Derive the name from the podcast title: lowercase, kebab-case, drop articles.

---

## Important Notes

- **Threads, not episodes**: The presentation is organized by thematic threads, not episode-by-episode. Episodes are the raw material; threads are the structure.
- **Direct transcript excerpts only**: Show the speakers' actual words. Margin notes provide the analysis.
- **Attribution matters**: Every excerpt slide must identify which episode and speaker it came from.
- **Respect the oral medium**: Spoken language has a different rhythm than prose. Light cleanup is fine, but don't sanitize speakers' voices into essay prose.
- **The host is a character**: In a series, the host's evolving perspective is part of the story. Surface how their questions and framing change over time.
- **Conversations are not monologues**: Prioritize passages where the host-guest dynamic produces unique insight.
- **Not all episodes need equal weight**: A landmark interview might get 6-8 slides. A lighter episode might get 2-3.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **The margin notes are the value-add**: Cross-episode annotations — showing how a guest in Episode 3 unknowingly answers a question left open in Episode 1 — are what make this a *study* of a podcast rather than a listening log.
