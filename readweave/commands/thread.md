---
description: "Distill a discussion thread into an HTML slide presentation. Handles Twitter/X threads, Reddit posts with comments, Hacker News discussions, forum threads, and other multi-voice conversations. Captures the dialectic structure — claims, rebuttals, synthesis — that makes threads a unique medium."
argument-hint: "[URL to thread (Twitter/X, Reddit, HN, forum)]"
---

# Thread Skill

You will create one output from the following discussion thread: $ARGUMENTS
- An elegant **HTML slide presentation** (`[thread-name]-thread.html`)

Threads are a unique medium: ideas develop through **dialectic** — claim, challenge, refinement, synthesis — across multiple voices. This skill captures that structure rather than flattening it into a monologue.

Follow the four phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[thread-slug]/` (derive slug from the thread topic — lowercase, kebab-case, drop articles).

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

### Determine source type and fetch the thread:

**Twitter/X threads:**
1. Use WebFetch to retrieve the thread page
2. If the thread is a single author's thread: extract all tweets in order
3. If the page doesn't render fully, try alternative approaches:
   - Fetch via Nitter or similar mirror: try `nitter.net/[username]/status/[id]`
   - Use the thread reader service: try `threadreaderapp.com/thread/[id]`
4. Extract: **author**, **date**, **all tweets in sequence**, **engagement metrics** (likes, retweets — as signal of resonance, not authority)
5. For reply threads / quote tweet chains: follow the conversation structure

**Reddit posts with comments:**
1. Fetch the post and comments: `https://old.reddit.com[path].json` or use WebFetch on the URL
2. Extract: **subreddit**, **post title**, **post author**, **post body**, **date**
3. Extract top-level comments and notable reply chains, sorted by score (community signal)
4. For deep threads, follow reply chains that develop an argument (not just joke chains)

**Hacker News discussions:**
1. Fetch via API: `https://hacker-news.firebaseio.com/v0/item/[id].json` for the post, then fetch child comments
2. Or use WebFetch on the HN URL directly
3. Extract: **title**, **submitter**, **URL** (if link post), **points**, **comment tree**
4. HN threads often have exceptionally high-quality technical comments — prioritize comments with deep domain expertise

**Forum threads (Discourse, Stack Exchange, etc.):**
1. Use WebFetch on the thread URL
2. Extract: **thread title**, **original poster**, **date**, **all posts in order**
3. Preserve the reply structure — which posts respond to which

**Other platforms (Mastodon, Bluesky, etc.):**
1. Use WebFetch on the thread URL
2. Extract the conversation structure as best as possible
3. Follow reply chains that develop the argument

### Parse the conversation:

- **Map the conversation tree**: Identify which posts respond to which. Not all threads are linear — some branch into sub-conversations.
- **Identify key participants**: Who are the main voices? Note usernames, and where discoverable, their credentials or context (HN profiles often reveal expertise).
- **Detect the conversation type**:
  - **Solo thread**: One author developing an argument across multiple posts (Twitter threads, blog-post-length Reddit posts)
  - **Dialogue**: Two or three people going back and forth, developing an idea through exchange
  - **Community discussion**: Many voices contributing perspectives, often with clear "camps"
  - **Expert thread**: Technical discussion where domain experts weigh in (common on HN)
- **Preserve upvote/score data** where available — this is community signal about which contributions resonated

### Extract candidate passages:

Collect **10-30 candidate passages** depending on thread length and quality.

For each candidate passage, capture:
- The **exact text**, preserving the author's wording
- The **author/username**
- The **timestamp** or position in thread
- Whether it's a **top-level post**, **reply**, or **part of a chain**
- What it's **responding to** (if a reply — capture enough context that the response makes sense)
- A **one-line note** on why this passage matters

**What to look for — the eight passage types for threads:**

1. **Opening claims**: The thesis that starts the conversation. In solo threads, this is the first tweet/post. In community discussions, this is the submission or the first substantive comment.

2. **Strong rebuttals**: Where someone challenges the opening claim with evidence, logic, or a counter-example. The best rebuttals don't just disagree — they change the frame.

3. **Expert contributions**: Posts from people with direct experience or domain knowledge. "I worked on this system for 5 years and..." — these carry authority that upvotes alone can't confer.

4. **Synthesis moments**: Where someone integrates multiple previous contributions into a higher-level insight. "What both sides are missing is..." These are the thread's highest-value passages.

5. **Concrete evidence**: Data points, links to studies, specific examples from practice. These ground the discussion in reality.

6. **Reframes**: Where someone shifts the entire frame of the discussion. "The real question isn't X, it's Y." These often appear mid-thread and redirect everything that follows.

7. **Concessions and updates**: Where someone changes their mind or acknowledges a point. "I hadn't considered that — you're right that..." These are rare in online discourse and extremely valuable when they appear.

8. **Memorable formulations**: Phrases that compress the thread's insight into a sentence. The comment that gets quoted because it captured what everyone was trying to say.

**What NOT to extract:**
- Meta-commentary about the thread itself ("This thread is so good", "Why is this being downvoted")
- Low-effort agreement ("This.", "+1", "Exactly")
- Ad hominem attacks or flamewars that don't advance understanding
- Jokes and memes (unless they genuinely illuminate the topic)
- Repetitive points — take the strongest formulation
- Deleted or removed content

### Checkpoint: Save `extraction.md` to `.readweave/[thread-slug]/` with all candidate passages, thread metadata, conversation structure, and participant analysis.

---

## Phase 2: Analyze & Select

### Map the discussion:

1. **Core question**: What is this thread really about? State it in one sentence. (Often different from the original post's explicit topic — discussions drift toward the real question.)
2. **Argument structure**: Map the main positions or camps. Who argues what? What evidence do they cite?
3. **Dialectic arc**: How does the discussion develop? Does it reach resolution, productive disagreement, or fragmentation? Does understanding deepen or just multiply viewpoints?
4. **Key turns**: Identify the 2-3 moments where the conversation changes direction — a reframe, a piece of evidence, a concession.
5. **What the thread produces**: What understanding does the reader gain from the whole thread that no single post provides?

### Select excerpts:

From the candidate pool, select **8-20 excerpts** for the presentation.

**Three-pass selection:**

**Pass 1 — The dialectic skeleton (4-8 excerpts).** The opening claim, the strongest rebuttal, the key synthesis, and the resolution (or the most productive impasse). These passages alone should convey the thread's intellectual arc.

**Pass 2 — The evidence and expertise (3-6 excerpts).** The expert contributions, the concrete evidence, the data points that ground the discussion. These give the dialectic empirical weight.

**Pass 3 — The texture (2-4 excerpts).** The memorable formulation, the honest concession, the reframe that nobody saw coming. These make the thread come alive on the page.

**Selection quality tests:**
1. **The dialectic test**: Does this excerpt advance the conversation's argument, or is it just a well-written tangent?
2. **The stand-alone test**: Can someone understand this passage without reading the full thread? If it's a reply, is enough context included (or captured in a margin note)?
3. **The voice test**: Does this passage represent a genuinely distinct perspective, or does it repeat what another excerpt already says?
4. **The authority test**: For expert contributions, is the author's expertise evident from the passage or discoverable from context?

**Passage length:**
- Sweet spot: 30-200 words. Thread posts are often short and punchy — respect that format.
- For reply chains (claim → rebuttal → counter): capture enough of each turn to show the exchange. These may run to 300+ words across multiple speakers.
- Preserve the author's exact wording, including their formatting (bold, italics, lists).
- Use `<span class="elide">[…]</span>` for minimal trims.

### Checkpoint: Save `selection.md` to `.readweave/[thread-slug]/` with the dialectic map, selected excerpts, conversation analysis, and key turns.

---

## Phase 3: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**.

### Labels must be:
- Descriptive concepts: "The Reframe", "Survivorship Bias", "The Practitioner's Veto", "Hidden Assumption", "Steelman Attempt", "The Concession", "Evidence Quality"
- Bad: "Key Point!", "Great Reply!", "Important!", "Note"

### Note body must be:
- **Analytical, not summary** — never restate what the excerpt says
- **Dialectic-aware** — note what position this excerpt occupies in the conversation: "This is the strongest version of the counter-argument. Notice that it concedes [X] while challenging [Y]."
- **Context-filling** — for replies, explain what the author is responding to if it's not in the excerpt. "This responds to [username]'s claim that [paraphrase]. The key move is..."
- **Authority-noting** — where relevant, note the author's credentials or perspective. "As a former [role] at [company], this comment carries firsthand experience weight."
- **Hidden implications** — what follows from this that the author doesn't spell out?
- **Epistemic quality** — note the quality of evidence and reasoning. "This is an anecdote, not data — but it's the kind of anecdote that signals a real pattern."
- **Cross-excerpt connections** — show how this excerpt relates to others in the selection

### Prose voice:
- Write in a calm, literate register — the tone of someone annotating a debate transcript, not taking sides
- Note the quality of arguments on all sides — strong arguments, weak evidence, good intentions with flawed logic
- Avoid taking a position — let the annotated thread speak for itself. Your job is to illuminate the structure, not to referee.
- Avoid rhetorical questions used for emphasis, exclamation marks, and breathless discovery language
- The best margin notes for threads are like a philosophy seminar leader's marginalia — noting the logical structure, flagging assumptions, and connecting to broader contexts

This voice also applies to all commentary prose: overview slides and discussion questions.

### Quality test: Cover the excerpt text. Can someone learn something new from just the margin note? If not, rewrite it.

### Checkpoint: Save `annotations.md` to `.readweave/[thread-slug]/` with all annotated excerpts, overview prose, dialectic analysis, and discussion questions.

---

## Phase 4: Generate HTML Slides

1. **Read the template**: Use the Glob tool to find `**/gist-template.html` within this plugin's installation directory, then read it with the Read tool.
2. **Read the design guide**: Use the Glob tool to find `**/read-design-guide.md` within this plugin's installation directory, then read it.
3. **Replace placeholders**:
   - `{{BOOK_TITLE}}` → the thread's topic or original post title (used in both `<title>` and sidebar header)
   - `{{SIDEBAR_SUBTITLE}}` → "Platform &middot; Original Author &middot; Date"
4. **Populate slides** inside the `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — thread topic, platform, original author, date, participant count, and a quotation that captures the thread's central question. Include `begin-hint`.

- **Slide 1 — Overview**: `class="slide no-margin"` — the thread's core question and what the discussion produces. Use a visual diagram showing the dialectic structure (concept-cards: "Claim → Challenges → Synthesis" or "Position A vs. Position B → Resolution"). Note the platform, the community context, and what makes this thread worth preserving. Frame what follows: "The following pages present this conversation *in the participants' own words*."

- **Slides 2 through N — Excerpts**: `class="slide"` — direct thread excerpts with margin annotations. This is the bulk (8-20 slides). Each slide has:
  - `page-chapter` label with the conversation position and author: `"Opening Claim &middot; @username"` or `"Rebuttal &middot; u/username"` or `"Synthesis &middot; [username]"`
  - `page-divider`
  - One or more `.excerpt` paragraphs. For single-author posts, first excerpt gets `.drop-cap`. For exchanges, use `<span class="speaker">[@username]:</span>` before each speaker's contribution.
  - `page-source` citation (platform, thread URL)
  - `.margin` div with 2-3 `.margin-note` elements

  **Special handling for reply chains:**
  When an excerpt captures a claim-rebuttal exchange, present it as a dialogue:
  ```html
  <p class="excerpt"><span class="speaker">[@author1]:</span> [Their claim...]</p>
  <p class="excerpt"><span class="speaker">[@author2]:</span> [Their rebuttal...]</p>
  ```
  Do NOT use drop-cap on dialogue slides.

- **Slide N+1 — Discussion**: `class="slide no-margin"` — 4-6 thought-provoking questions that go beyond the thread's own conclusions. At least two should question assumptions shared by all participants. Uses `.discussion-q` elements.

- **Slide N+2 — Closing**: `class="slide no-margin"` — what the thread produces: the insight, the unresolved tension, or the productive disagreement. A final quotation (often the synthesis or the most memorable formulation) + attribution with link to the original thread.

### Formatting rules:
- Use `<span class="elide">[…]</span>` for elisions within excerpts
- Use literal Unicode: `—` for em-dashes, `'` / `'` for single quotes, `"` / `"` for double quotes (do NOT use HTML entities — they break in CSS `content:` properties)
- Apply `drop-cap` class to the first `.excerpt` on single-author excerpt slides (not dialogue slides)
- Each slide needs a descriptive `data-title` attribute for the sidebar TOC
- Use `<span class="speaker">[@username]:</span>` for speaker attribution
- Preserve the author's formatting (bold, italics, lists, code blocks) where it serves communication

### Slide count guidance:
- Short thread (<20 meaningful posts): 8-12 total slides
- Medium thread (20-50 posts): 12-18 total slides
- Long thread (50-100+ posts): 18-25 total slides
- Epic thread (100+ posts, major discussion): 25-35 total slides

Save as `[thread-name]-thread.html` in the working directory. Derive `[thread-name]` from the thread topic: lowercase, kebab-case, drop articles.

---

## Important Notes

- **Dialectic, not monologue**: The unique value of a thread is the interaction between voices. Prioritize exchanges, rebuttals, and syntheses over individual statements — a thread presentation that reads like a single-author essay has failed.
- **Direct excerpts only**: Show the participants' actual words. Margin notes provide the analytical layer.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. Only external dependency is Google Fonts CDN.
- **Preserve the aesthetic**: Reuse the book-page design from the template — parchment colors, Garamond fonts, Caveat margin notes, drop caps.
- **Respect the medium**: Thread posts are often informal, punchy, and conversational. Don't sanitize this into essay prose — the informality is part of the medium's value. Clean up obvious typos but preserve voice.
- **Attribution and context**: Always include the platform, original thread URL, key participants, and date. For Reddit/HN, note the subreddit or community context.
- **The annotations are neutral**: Thread annotations must not take sides in the debate. Illuminate the structure of the argument, note the quality of reasoning on all sides, and let the reader form their own view.
- **Signal vs. noise**: Most thread posts are noise. The skill's value is in finding the 10-20% that carry genuine signal — the expert contribution, the devastating rebuttal, the synthesis that nobody else could see. Be ruthless in curation.
- **The margin notes are the value-add**: Anyone can read a thread. The margin notes — dialectic structure, epistemic quality assessment, context-filling, cross-excerpt connections — are what make this a *study* rather than a bookmark.
