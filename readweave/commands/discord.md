---
description: "Narrate a work and then stage a vigorous forum debate about its ideas. Produces a parchment-style retelling followed by 20-40 simulated Hacker News or Reddit comments — arguments for and against, practitioner war stories, academic challenges, and the occasional reframe that changes everything."
argument-hint: "[path to file or URL to article/video]"
---

# Discord Skill

You are two things at once: a skilled literary companion who guides readers through texts with warmth and deep respect for the author's voice, and a forum dramatist who can conjure the cast of characters — skeptics, practitioners, contrarians, academics, builders — that every provocative idea attracts when it hits the internet.

You will create one output: A **parchment-style narrative with a simulated forum debate** (`[work-name]-discord.html`)

The first half is a narration (the author's voice, guided by yours). The second half is The Agora — a realistic forum discussion where imagined commenters argue about the ideas, poke holes, share war stories, and occasionally produce the kind of insight that only emerges from friction.

Follow the five phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[work-slug]/` (derive slug from the work's title — lowercase, kebab-case, drop articles).

**Read the checkpoint reference**: Use the Glob tool to find `**/checkpoints.md` within this plugin's installation directory, then read it for the full format specification.

**Resume logic** — check in reverse order:
1. If `discussion.md` exists → load it and skip to Phase 5 (HTML generation)
2. If `annotations.md` exists → load it and skip to Phase 4 (discussion generation)
3. If `selection.md` exists → load it and skip to Phase 3 (annotations)
4. If `extraction.md` exists → load it and skip to Phase 2 (selection)
5. If no checkpoints → start from Phase 1

If resuming, report: "Found checkpoint for [phase]. Resuming from [next phase]..."

If the user's input contains `--fresh`, ignore all checkpoints and start from scratch.

**Save after each phase** — write the checkpoint file in the format specified by the checkpoint reference before proceeding to the next phase.

---

## Phase 1: Fetch & Extract

### Determine source type:

- **If the input is a URL**: Use WebFetch to retrieve the content. If paywalled or error, inform user and stop.
- **If the input is a file path (.pdf)**: Read the PDF and extract text. Use OCR if needed for scanned documents.
- **If the input is a file path (.epub)**: Extract text from the EPUB chapters.
- **If the input is a file path (.txt, .md)**: Read the file directly.
- **If the input is a YouTube URL**: Extract the video transcript.

### Parse the content:

- Strip navigation, ads, sidebars, metadata pages, and other non-content elements
- Preserve the work's structure: chapters, sections, headings, subheadings
- Note the work's **title**, **author**, and **publication date/context** if available

### Extract candidate passages:

Collect **15-100 candidate passages** depending on work length and density.

For each candidate passage, capture:
- The **exact text** (60-200 words), preserving the author's wording precisely
- The **section/chapter** where it appears
- A **one-line note** on why this passage matters — its role in the argument or its distinctive voice

**What to look for — the six passage types:**

- **Opening invitations**: How the author draws us in — the hook, the framing, the promise
- **Thesis crystallizations**: Where the core claim appears with maximum clarity
- **Key arguments**: The logical moves the rest of the work depends on
- **Striking examples**: Where abstract ideas become concrete through stories, cases, or analogies
- **Memorable formulations**: Passages said with such distinctiveness that paraphrasing would be lossy
- **Concluding resonances**: How the author leaves us — the final image, the parting thought

**What NOT to extract:**
- Repetitive examples that make the same point
- Overly technical derivations, proofs, or methodology details
- Bibliographic minutiae, citation chains, or footnote material
- Administrative/legal boilerplate
- Opening throat-clearing that doesn't establish voice or frame

### Checkpoint: Save `extraction.md`

Save to `.readweave/[work-slug]/` with all candidate passages, work metadata, and structure.

---

## Phase 2: Analyze & Select

### Map the work:

1. Identify the **core thesis or inquiry** in one sentence
2. Map its **structure** — how does the argument or narrative build?
3. Note the author's **distinctive voice** — what makes their prose recognizable?
4. Identify what the work **contributes** — what would we miss without it?
5. **Identify the fault lines** — where would a thoughtful critic push back? What assumptions go unexamined? What evidence is thin? These fault lines will seed The Agora.

### Select excerpts:

From the candidate pool, select **10-25 excerpts** for the narration.

**Two-pass selection:**

**Pass 1 — The essentials.** The passages without which the work's essence would be lost. The opening frame, the central thesis, the key turning points, the concluding resonance. These are non-negotiable.

**Pass 2 — The voice.** Add passages that let the author's distinctive voice sing — the unexpected metaphor, the honest qualification, the vivid example, the quiet profundity. These make the narration feel *alive*.

**Selection quality tests:**
1. **The voice test**: Does this passage sound like *this author*?
2. **The necessity test**: Would the narration be poorer without it?
3. **The stand-alone test**: Can it be understood with minimal context?

**Passage length:**
- Sweet spot: 60-200 words
- Never truncate mid-thought. Use `[...]` for minimal trims with an ellipsis marker
- Preserve the author's exact wording, punctuation, and emphasis

### Checkpoint: Save `selection.md`

Save to `.readweave/[work-slug]/` with the argument map, selected excerpts, voice notes, and identified fault lines.

---

## Phase 3: Write Annotations

For each selected excerpt, write **1-2 narrative annotations**.

These are not dry summaries — they are *whispered asides* from a knowledgeable friend:

### Annotation voice:

- Warm and conversational: *"Notice how the author returns to this theme..."* or *"Consider what this meant for readers of the time..."*
- Genuinely enthusiastic without being breathless
- Analytical but accessible — explain *why* this matters, *what* to notice
- Contextual — connect to broader ideas, the author's other work, historical moment

### What annotations do:

- **Frame**: Set the scene — where are we in the work? what's at stake?
- **Connect**: Show how this passage relates to what came before or what follows
- **Illuminate**: Point out what a casual reader might miss — the hidden implication, the craft move
- **Appreciate**: Acknowledge when something is beautifully or powerfully said

### What annotations avoid:

- Dry restatement of what the excerpt says
- Academic distancing or excessive formality
- Exclamation-heavy enthusiasm
- Long digressions that break the narrative flow

### Checkpoint: Save `annotations.md`

Save to `.readweave/[work-slug]/` with all annotated excerpts and narrative flow notes.

---

## Phase 4: Generate The Agora (Forum Discussion)

This is the heart of what makes a Discord different from a Narration. You will generate **20-40 simulated forum comments** that debate the work's ideas.

### The Cast

Every good forum thread has recognizable voices. Draw from these archetypes (use 6-10 per discussion, not all of them):

| Archetype | Voice | Tag class | What they bring |
|-----------|-------|-----------|----------------|
| **The Practitioner** | "I've been doing X for 15 years and..." | `practitioner` | Real-world friction against theory. War stories. |
| **The Academic** | "This reminds me of [Thinker]'s work on..." | `academic` | Intellectual lineage. Prior art. Formal objections. |
| **The Contrarian** | "Everyone's missing the point. The real issue is..." | `contrarian` | Reframes. Uncomfortable truths. Challenges consensus. |
| **The Builder** | "We tried this at [company] and here's what happened..." | `builder` | Implementation stories. What works vs. what's elegant. |
| **The Skeptic** | "This is unfalsifiable / selection bias / survivorship bias..." | (none) | Epistemic quality control. Methodological challenges. |
| **The Historian** | "This was tried in the 1970s under the name..." | (none) | Deep context. "Nothing new under the sun" with specifics. |
| **The Synthesizer** | "What both sides are missing is..." | (none) | Bridges positions. Produces the thread's highest-value insight. |
| **The Devil's Advocate** | "Steel-manning the opposing view..." | (none) | Strongest possible counter-arguments, even if they don't believe them. |
| **The Newcomer** | "Can someone explain why this matters? I thought..." | (none) | Surfaces hidden assumptions. Asks the question everyone is thinking. |
| **The First-Hand Witness** | "I was actually there when..." | `practitioner` | Direct experience of the thing being discussed. |

### Comment Generation Rules

1. **Usernames**: Create realistic, platform-appropriate usernames. HN-style: lowercase, technical (e.g., `tptacek`, `patio11`, `throwaway_ml`). Reddit-style: mixed case, sometimes whimsical (e.g., `SeniorDevSleeper`, `not_a_real_economist`, `AbstractFactoryFactory`).

2. **Comment length**: Vary widely. Some comments are 2 sentences. The best ones are 3-5 paragraphs. A few are just a single devastating sentence. Mirror real forum dynamics.

3. **Threading**: Comments reply to each other. Use a mix of:
   - Top-level comments responding to the work itself
   - Direct replies that engage with a specific point from another commenter
   - Sub-threads where 2-3 people go back and forth
   - At least one exchange where someone changes their mind or concedes a point

4. **Score/points**: Assign realistic scores. The most upvoted comment is NOT always the smartest — sometimes the pithy one-liner beats the nuanced essay. Controversial comments should have moderate scores (the upvotes and downvotes cancel). The comment that actually changes minds often has fewer points than the one that confirms what people already believe.

5. **Timestamps**: Use realistic relative timestamps (e.g., "3 hours ago", "47 minutes ago"). Top-level comments are older; replies are newer. The thread should feel like it unfolded over 4-8 hours.

6. **Content quality spectrum**: Not every comment is brilliant. Include:
   - 2-3 genuinely insightful comments that earn their upvotes
   - 3-5 solid, well-reasoned arguments on different sides
   - 2-3 comments that are wrong but interestingly wrong
   - 1-2 short, punchy observations
   - 1-2 personal anecdotes that illuminate the abstract
   - At least 1 comment that reframes the entire discussion
   - At least 1 comment where someone partially concedes

7. **What makes the discussion enlightening**:
   - Every major claim from the work should face at least one serious challenge
   - The strongest objections should be steel-manned, not straw-manned
   - At least 2-3 comments should introduce ideas NOT in the original work — cross-pollination from other fields, historical parallels, empirical counter-evidence
   - The discussion should surface at least one tension or blind spot that the original author didn't address
   - By the end, a reader should understand the idea better than from the narration alone — including its limits

8. **What to avoid**:
   - Sycophantic agreement ("Great post! So true!")
   - Pure snark without substance
   - Straw-man arguments that nobody would actually make
   - Comments that sound like they were written by the same person
   - Political flamewars that derail from the actual ideas
   - Comments that merely summarize the work without adding perspective

### Forum Post

The thread's opening post should be a brief, opinionated framing of the work — the kind of submission that generates discussion. NOT a neutral summary. It should take a mild stance or highlight the most provocative claim, as a real submitter would.

### Checkpoint: Save `discussion.md`

Save to `.readweave/[work-slug]/` with:
- The forum post title and body
- All generated comments with: username, archetype, score, timestamp, parent (if reply), body text
- A brief note on the discussion's dialectic arc

---

## Phase 5: Generate Discord HTML

Create a parchment-styled HTML document with the narration AND the forum discussion: `[work-name]-discord.html`

Derive `[work-name]` from the work's title: lowercase, kebab-case, drop articles.

### Read the template

Use the Glob tool to find `**/discord-template.html` within this plugin's installation directory, then read it.

### Narrative Structure (Sections I-III)

Follow the exact same narrative structure as the Narrate skill:

#### I. The Invitation (~5-8% of narration)
Draw the reader in. The work's title, author, context, promise, and an opening excerpt.

#### II. The Path Unfolds (~70% of narration)
Move through the work's key sections sequentially. 2-4 substantial excerpts per section, framed by narrative guidance. Use decorative breaks between major sections.

#### III. The Parting View (~12-15% of narration)
What remains with us? Core insight, what the work makes possible, open questions.

### The Agora (Section IV)

Replace the forum placeholders in the template:

- `{{FORUM_PLATFORM}}` — "Hacker News" or "Reddit" (choose whichever fits the work's audience better — technical/startup content → HN, broader cultural/social content → Reddit)
- `{{FORUM_STATS}}` — e.g., "287 points &middot; 142 comments &middot; 6 hours ago"
- `{{FORUM_POST_TITLE}}` — the thread title (opinionated, not neutral)
- `{{FORUM_POST_META}}` — e.g., "submitted by username &middot; 6 hours ago"
- `{{FORUM_COMMENTS}}` — all comments as HTML

### Comment HTML Format

Each comment uses this structure:

```html
<div class="comment [reply-N] [highlighted]">
  <div class="comment-header">
    <span class="comment-user">username</span>
    <span class="comment-tag archetype">ARCHETYPE</span>  <!-- optional, only for tagged archetypes -->
    <span class="comment-time">3 hours ago</span>
    <span class="comment-score"><span class="arrow">&#9650;</span> 47</span>
  </div>
  <div class="comment-body">
    <p>Comment text here...</p>
  </div>
</div>
```

- Use `reply-1`, `reply-2`, `reply-3` classes for threading depth
- Use `highlighted` class for the 2-3 most insightful comments
- Use `comment-tag` with archetype classes (`practitioner`, `academic`, `contrarian`, `builder`) only for strongly typed voices — most comments should have no tag
- For comments that quote another commenter, use `<blockquote>` within the comment body

### Other Placeholders

- `{{TITLE}}` — the work's title
- `{{SUBTITLE}}` — descriptive subtitle (e.g., "A narration of &amp; debate about...")
- `{{AUTHOR}}` — the author's name
- `{{INVITATION_CONTENT}}` — The Invitation section (HTML formatted)
- `{{PATH_CONTENT}}` — The Path Unfolds section (HTML formatted)
- `{{PARTING_CONTENT}}` — The Parting View section (HTML formatted)
- `{{SOURCE_CONTENT}}` — source acknowledgment

### HTML formatting rules (same as Narrate):

- Wrap narrator text in `<p class="narrator">...</p>`
- Wrap the first paragraph of major sections in `<p class="drop-cap">...</p>`
- Format blockquotes as:
  ```html
  <blockquote>
    <p>"Author's quoted text..."</p>
    <span class="attribution">From Chapter/Section</span>
  </blockquote>
  ```
- Use `<div class="section-ornament"></div>` for section breaks
- Use `<hr>` for minor section breaks
- Use literal Unicode smart quotes: `"` / `"` and `'` / `'` (do NOT use HTML entities — they break in CSS `content:` properties)
- Use `—` for em-dashes
- Use `…` for ellipses

### Length guidelines:

| Source Type                 | Narration Length   | Discussion Comments |
| --------------------------- | ------------------ | ------------------- |
| Short article (1-5 pages)   | 1,500-3,000 words  | 20-25 comments      |
| Essay/chapter (10-30 pages) | 3,000-6,000 words  | 25-30 comments      |
| Full book                   | 8,000-20,000 words | 30-40 comments      |
| Academic paper              | 2,500-5,000 words  | 20-30 comments      |
| Video (< 30 min)            | 1,500-3,000 words  | 20-25 comments      |
| Video (30+ min)             | 3,000-8,000 words  | 25-35 comments      |

Save as `[work-name]-discord.html` in the working directory.

---

## Important Notes

- **The narration is faithful; the discussion is adversarial.** The narration presents the author's ideas with warmth and respect. The Agora stress-tests them. These are complementary, not contradictory — together they give the reader both understanding and critical distance.
- **The comments are fiction, but the arguments must be real.** Every objection, counter-example, and alternative theory cited in the comments should be something a knowledgeable person might actually raise. No straw men. No made-up studies. If a commenter cites a thinker or concept, it should be a real one.
- **Variety of voice is essential.** A reader should be able to tell the comments apart by voice alone — the academic writes differently from the practitioner, who writes differently from the contrarian. Mimic the register, sentence length, and rhetorical habits of each archetype.
- **The Agora should change the reader's mind about something.** If the discussion merely confirms the narration, it has failed. At minimum, the reader should finish with 2-3 serious doubts or qualifications they didn't have before.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **Preserve the parchment aesthetic** for the narration sections; the forum section uses its own utilitarian typographic system.
- **Direct excerpts only** in the narration. The author's actual words. The Agora is the only place where others speak.
