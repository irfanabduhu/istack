---
description: "Deep dive into a single podcast episode or audio file. Transcribes audio, extracts key moments, and produces an HTML slide presentation with analytical margin annotations. Handles interviews and multi-speaker formats."
argument-hint: "[path to audio file or podcast episode URL]"
---

# Listen Skill

You will create one output from the following audio source: $ARGUMENTS
- An elegant **HTML slide presentation** (`[episode-name]-listen.html`)

Follow the four phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[episode-slug]/` (derive slug from the episode title — lowercase, kebab-case, drop articles).

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

## Phase 1: Transcribe & Extract

### Determine source type and obtain audio:

**If the input is a local file** (`.mp3`, `.wav`, `.m4a`, `.ogg`, `.flac`, `.aac`):
- Use the file directly for transcription.

**If the input is a URL** (podcast episode page, direct audio link, or RSS item):
1. Use WebFetch to retrieve the page
2. If it's a podcast episode page, look for the direct audio URL in:
   - `<audio>` or `<source>` tags
   - `<meta>` tags with `og:audio` or `twitter:player:stream`
   - Embedded player `<iframe>` attributes
   - JSON-LD structured data
3. If it's a direct audio file URL (`.mp3`, `.m4a`, etc.), use it directly
4. Download the audio file: `curl -L -o /tmp/episode-XXXXXX.[ext] "[audio_url]"`
5. Extract episode metadata: **title**, **show/podcast name**, **host(s)**, **guest(s)** (if any), **publication date**, **description/show notes**

### Transcribe the audio:

Use Whisper for transcription (try methods in order):

```bash
# Method 1: whisper CLI (OpenAI's Whisper)
whisper "$AUDIO_PATH" --model medium --language en --output_format txt --output_dir /tmp/whisper-out-XXXXXX

# Method 2: whisper-cpp (if whisper CLI unavailable)
whisper-cpp -m /path/to/model -f "$AUDIO_PATH" --output-txt

# Method 3: mlx-whisper (Apple Silicon optimized)
mlx_whisper "$AUDIO_PATH" --model mlx-community/whisper-medium-mlx
```

If no Whisper variant is available, inform the user:
> "Whisper is required for audio transcription. Install it with: `pip install openai-whisper` or `brew install whisper-cpp`"

### Parse the transcript:

- **Detect speaker changes**: Look for patterns in the transcript that indicate different speakers. If Whisper doesn't provide diarization, identify speakers by:
  - Conversational turn-taking patterns (question → answer)
  - Vocal style shifts noted in the transcript
  - Show notes that name the host and guest(s)
- **Label speakers** where possible: `[Host]`, `[Guest: Name]`, or `[Speaker 1]`, `[Speaker 2]` if names unknown
- **Clean transcript artifacts**: Remove filler words that don't carry meaning (um, uh, like), fix obvious speech-to-text errors, normalize punctuation
- **Remove repetitive verbal tics**: False starts, stammers, hedge piles, echo phrases, auto-caption stutters — same cleanup rules as the watch skill
- **Preserve natural speech rhythm**: Keep "well," "so," "now," "look" and emphasis repetitions that serve rhetorical purpose
- **Divide into logical sections** based on topic shifts, interviewer questions, or explicit transitions
- **Note timestamps** where available (even approximate ones)

### Extract candidate passages:

Collect **10-25 candidate passages** depending on episode length. For a typical episode (30-90 minutes), aim for 12-18 candidates.

For each candidate passage, capture:
- The **exact spoken words**, cleaned but faithful to the speaker's phrasing
- The **speaker** who said it (host, guest, or speaker label)
- The **approximate timestamp** or section
- A **one-line note** on why this passage matters

**What to look for — the seven passage types for audio:**

- **Thesis crystallizations**: Where a speaker states their core argument with maximum clarity. In conversations, these often come 10-20 minutes in, after the host has drawn the guest out past their rehearsed talking points.
- **Key arguments**: Where a speaker makes a logical move — a distinction, an analogy, a reductio — that the rest of the conversation depends on.
- **Striking examples**: Where an abstract idea becomes concrete through a story, anecdote, or case study. Podcast guests often think through stories more than through arguments — the story IS the insight.
- **Provocations**: Where a speaker challenges assumptions or conventional wisdom. Moments that would make a listener pause their walk.
- **Dialogue sparks**: Unique to conversations — where the interaction between speakers produces an insight neither would have reached alone. The host pushes back, the guest reconsiders, and something new emerges. These are the highest-value passages in interview formats.
- **Honest admissions**: Where a speaker acknowledges uncertainty, changes their mind mid-thought, or says "I used to think X but now I think Y." Podcasts surface these more than any other medium because the conversational format lowers the guard.
- **Distillation moments**: Where a speaker compresses a complex idea into a single memorable formulation — the "quotable quote" that captures an hour of nuance in two sentences.

**What NOT to extract:**
- Opening banter, ad reads, or sponsor segments
- Filler conversation ("That's a great question", "Thanks for having me")
- Content that only makes sense with context the listener has but the reader won't
- Repetition of the same point — take the sharpest formulation
- Self-promotion or calls to action ("Check out my book...", "Subscribe to...")
- Fragmentary statements that depend on uncaptured context

### Checkpoint: Save `extraction.md` to `.readweave/[episode-slug]/` with all candidate passages, episode metadata, speaker identification, and section structure.

---

## Phase 2: Analyze & Select

### Map the conversation:

1. Identify the episode's **core thesis or central question** in one sentence
2. Map its **structure** — how does the conversation develop? What is the arc from opening to conclusion?
3. Note the **conversation dynamics** — is this an interview, a debate, a co-exploration? Who drives the direction?
4. Identify what each speaker **uniquely contributes** — the host's framing, the guest's expertise, points of disagreement
5. Note what the speakers are **responding to** — what assumption, trend, or event prompted this conversation?

### Select excerpts:

From the candidate pool, select **10-18 excerpts** for the presentation.

**Two-pass selection:**

**Pass 1 — The essentials.** The core thesis, the key argument, the defining example, the most revealing exchange. These are the 5-8 passages you cannot leave out. If you removed them, the episode's intellectual content would collapse.

**Pass 2 — The texture.** Add passages that give the conversation its distinctive character — the spontaneous analogy, the honest admission, the push-back that changes direction, the moment of humor that carries a point. Conversations live or die by these moments more than written prose does.

**Selection quality tests:**
1. **The idea test**: Can you state what this passage captures in one sentence?
2. **The stand-alone test**: Can someone who hasn't listened understand this passage?
3. **The redundancy test**: Does another selected excerpt already capture this idea?
4. **The oral-to-written test**: Does this passage land on the page, or does it depend entirely on vocal delivery? If it only works spoken, cut it unless you can provide enough context in a margin note.
5. **The dialogue test** (unique to conversations): If this is an exchange between speakers, does the excerpt include enough of both sides for the exchange to make sense?

**Passage length:**
- Sweet spot: 40-180 words / 2-8 sentences. Spoken language is looser — trim filler but preserve the speaker's natural rhythm.
- For dialogue exchanges: capture enough turns to show the full exchange (may run to 200+ words).
- Never truncate mid-thought. Use `<span class="elide">[…]</span>` for minimal trims.
- Preserve the speaker's wording. Clean artifacts, but do not rewrite their sentences into "better" prose.

**Speaker attribution in excerpts:**
- For single-speaker passages: attribute in the `page-chapter` label
- For dialogue: use `<span class="speaker">[Speaker Name]:</span>` inline before each turn

### Checkpoint: Save `selection.md` to `.readweave/[episode-slug]/` with the conversation map, selected excerpts, speaker analysis, and dialogue dynamics.

---

## Phase 3: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**.

### Labels must be:
- Descriptive concepts: "The Experience Trap", "Survivorship Bias at Work", "The Interviewer's Reframe", "Implicit Hierarchy", "The Admission"
- Bad: "Key Point!", "Important!", "Great Quote!", "Interesting!"

### Note body must be:
- **Analytical, not summary** — never restate what the excerpt says
- **Speaker-aware** — acknowledge who is speaking and what perspective they bring. "As a practitioner who built X, [Guest] speaks from direct experience here — contrast this with [Researcher]'s more theoretical framing..."
- **Contextual** — connect to the speaker's other work, the broader discourse, real-world implications
- **Hidden implications** — surface what a casual listener might miss, especially in fast-moving dialogue
- **Dialogue dynamics** — for exchange passages, note what the interaction reveals: "Notice how the host's push-back forces the guest to move from anecdote to principle"
- **Concrete** — use specific references, named examples, real events

### Prose voice:
- Write in a calm, literate register — the tone of a thoughtful listener's notes, not a lecturer or enthusiast
- Favor observations that unfold naturally over declarations that announce themselves
- Avoid rhetorical questions used for emphasis, exclamation marks, and breathless discovery language ("This is the crucial move", "Here we see the key insight", "This changes everything")
- Let connections speak for themselves — state them plainly rather than dramatizing them
- The best margin notes sound like something a well-read friend might write in pencil — brief, specific, genuinely clarifying, never performing cleverness

This voice also applies to all commentary prose: overview slides and discussion questions.

### Quality test: Cover the excerpt text. Can someone learn something new from just the margin note? If not, rewrite it.

### Checkpoint: Save `annotations.md` to `.readweave/[episode-slug]/` with all annotated excerpts, overview prose, and discussion questions.

---

## Phase 4: Generate HTML Slides

1. **Read the template**: Use the Glob tool to find `**/gist-template.html` within this plugin's installation directory, then read it with the Read tool. This file contains the full HTML/CSS/JS scaffold for the slide presentation.
2. **Read the design guide**: Use the Glob tool to find `**/read-design-guide.md` within this plugin's installation directory, then read it. It documents the HTML structure for each slide type.
3. **Replace placeholders**:
   - `{{BOOK_TITLE}}` → the episode's title (used in both `<title>` and sidebar header)
   - `{{SIDEBAR_SUBTITLE}}` → "Host Name &middot; Podcast Name" or "Host Name with Guest Name"
4. **Populate slides** inside the `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — episode title, podcast name, host and guest names, date, and an opening quotation from the episode. Include `begin-hint`.

- **Slide 1 — Overview**: `class="slide no-margin"` — the episode's core question and what emerges. Use a visual diagram (concept-cards showing "The Question → Key Arguments → What Emerges") plus brief commentary. For interviews, note the speakers and what each brings. Frame what follows: "The following pages present [Speaker(s)]'s conversation *in their own words*."

- **Slides 2 through N — Excerpts**: `class="slide"` — direct transcript excerpts with margin annotations. This is the bulk (10-18 slides). Each slide has:
  - `page-chapter` label with the topic area and speaker(s): `"The Core Problem &middot; [Guest Name] &middot; ~12:30"` or `"Exchange &middot; ~25:00"` for dialogue
  - `page-divider`
  - One or more `.excerpt` paragraphs. For single-speaker passages, first excerpt on most slides gets `.drop-cap`. For dialogue, use `<span class="speaker">[Name]:</span>` before each turn.
  - `page-source` citation (episode title, podcast name)
  - `.margin` div with 2-3 `.margin-note` elements

- **Slide N+1 — Discussion**: `class="slide no-margin"` — 4-6 thought-provoking questions using `.discussion-q` elements

- **Slide N+2 — Closing**: `class="slide no-margin"` — the episode's thesis distilled + a final quotation + host/guest/podcast/date attribution

### Formatting rules:
- Use `<span class="elide">[…]</span>` for elisions within excerpts
- Use literal Unicode: `—` for em-dashes, `'` / `'` for single quotes, `"` / `"` for double quotes (do NOT use HTML entities — they break in CSS `content:` properties)
- Apply `drop-cap` class to the first `.excerpt` on most single-speaker excerpt slides (not dialogue slides)
- Each slide needs a descriptive `data-title` attribute for the sidebar TOC
- Use `<span class="speaker">[Name]:</span>` for speaker attribution in dialogue excerpts

### Slide count guidance:
- Short episode (<30 minutes): 8-12 total slides
- Medium episode (30-60 minutes): 12-18 total slides
- Long episode (60-120 minutes): 18-25 total slides
- Marathon episode (120+ minutes): 25-35 total slides

Save as `[episode-name]-listen.html` in the working directory. Derive `[episode-name]` from the episode title: lowercase, kebab-case, drop articles.

---

## Important Notes

- **Direct transcript excerpts only**: The presentation shows the speakers' actual words. Margin notes provide the analysis.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **Preserve the aesthetic**: Reuse the book-page design from the template — parchment colors, Garamond fonts, Caveat margin notes, drop caps.
- **Attribute the source**: Always include the podcast name, episode title, host, guest(s), and publication date in the title slide and closing slide.
- **Handle the oral-to-written gap**: Spoken language has a different rhythm than prose. Light cleanup of filler words and false starts is fine, but don't sanitize the speakers' voices into essay prose. The best transcript excerpts preserve the energy of speech while being readable on the page.
- **Conversations are not monologues**: Interview and dialogue formats have unique value — the interaction between minds. Prioritize passages where the exchange itself produces insight, not just passages where one speaker lectures.
- **Speaker balance**: In interview formats, the guest typically has more substantive content, but don't ignore the host's contributions — good hosts ask questions that reshape the conversation and their framing is often analytically sharp.
- **The margin notes are the value-add**: Anyone can listen to a podcast. The margin notes — analytical, contextual, speaker-aware — are what make this a *study* rather than a bookmark.
