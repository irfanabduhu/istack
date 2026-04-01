---
description: "Master a CS or technical book — extracts code + prose, builds active recall cards, concept dependency map, typed challenges, and produces a mastery guide + slide presentation. Supports epub, PDF (including scanned), and text files."
argument-hint: "[path to .epub or book file]"
---

# ReadWeave Master Skill

You will create two outputs from the following technical book: $ARGUMENTS
1. A comprehensive **markdown mastery guide** (`[book-slug]-mastery-guide.md`)
2. An elegant **HTML master presentation** (`[book-slug]-master.html`)

Follow the phases below in order. Phases 4a and 4b are independent and can run in parallel.

---

## Phase 0: Checkpoint Check

Derive the slug: lowercase, kebab-case, drop articles (a, an, the), no extension.
Example: *The C Programming Language* → `c-programming-language`

**Read the checkpoint reference**: Use the Glob tool to find `**/checkpoints.md` within this plugin's installation directory, then read it for the full format specification.

**Resume logic** — check `.readweave/[slug]/` in reverse order:
1. If `annotations.md` exists → load it and skip to Phase 4 (output generation)
2. If `selection.md` exists → load it and skip to Phase 3 (annotations)
3. If `extraction.md` exists → load it and skip to Phase 2 (selection)
4. Nothing found → start from Phase 1

If resuming, report: "Found checkpoint for [phase]. Resuming from [next phase]..."

If the user's input contains `--fresh`, ignore all checkpoints and start from scratch. Still save new checkpoints as you go.

---

## Phase 1: Extract

### Detect the file type and extract content:

**For `.epub` files:**
1. Create a unique temp directory and unzip: `mktemp -d /tmp/book-extract-XXXXXX` then `unzip book.epub -d $TMPDIR`
2. Read the OPF/NCX files to identify chapter order and HTML file paths
3. Launch **3-5 parallel Explore agents**, each covering a range of chapters (e.g., agent 1: chapters 1-4, agent 2: chapters 5-8, etc.)
4. Give each agent the extraction brief below

**For `.pdf` files:**

First, detect whether the PDF is scanned (image-based) or has extractable text:

```bash
pdftotext "$PDF_PATH" - 2>/dev/null | head -c 500
```

- **If output is empty or near-empty** (fewer than ~50 characters of real text): the PDF is scanned. Run OCR before proceeding:
  ```bash
  ocrmypdf --skip-text --output-type pdf "$PDF_PATH" "${PDF_PATH%.pdf}-ocr.pdf"
  ```
  Then use the OCR'd file for all subsequent reading. Inform the user that OCR was performed.

- **If output contains readable text**: proceed normally.

After OCR (if needed):
- Read the PDF directly using the Read tool with page ranges
- Launch parallel agents to cover different page ranges

**For plain text / `.txt` files:**
- Read the file directly, splitting into logical sections for parallel processing

### Critical: Parallel reading is the key performance optimization. Always use multiple agents reading simultaneously rather than sequentially.

### Extraction Brief (include in every agent prompt)

Each agent must read its assigned chapters **slowly and completely** — do not skim. For each chapter, collect **two passage pools**.

---

**Pool A — Prose Candidates (5-10 per chapter)**

Collect the same seven passage types as the `read` skill:
- **Thesis crystallizations**: core claims stated with maximum clarity. Often mid-chapter, not in introductions.
- **Defining passages**: where a key concept is introduced or given its clearest definition. Capture the full paragraph around the definition.
- **Pivotal arguments**: logical moves the rest of the book depends on — inferences, distinctions, thought experiments.
- **Striking examples**: where abstract ideas are made concrete. The example often *is* the idea.
- **Rhetorical high points**: where prose reaches peak intensity. These make ideas stick.
- **Self-revisions and qualifications**: where the author complicates or limits a previous claim. Intellectual honesty lives here.
- **Provocation and challenge**: where the author deliberately challenges received wisdom.

Prose candidate guidelines:
- Capture **exact text**, preserving the author's wording precisely. Do not paraphrase.
- **Full paragraphs**, not isolated sentences. Sweet spot: 3-8 sentences (60-200 words).
- Record: chapter number, chapter title, section heading, and a one-line note on why it matters.
- Do NOT extract: throat-clearing ("In this chapter, I will argue..."), literature review, methodological boilerplate, repetition (capture the strongest formulation), fragmentary sentences with dangling antecedents.

---

**Pool B — Technical Candidates (3-8 per chapter)**

Seven technical passage types:

1. **Illuminating code examples** — code whose removal would leave the mechanism incomprehensible. Capture the code PLUS surrounding prose context (1-2 paragraphs). Record: language, whether the sample is complete or a fragment, and the concept demonstrated.

2. **Worked examples** — full problem-to-solution walkthroughs. Never truncate intermediate steps. The middle steps are where understanding lives.

3. **Algorithm/process descriptions** — textual step-by-step explanations of mechanisms. The prose that precedes and explains the code.

4. **Design decision passages** — "we could have done X but chose Y because..." These teach judgment, not just mechanics. Mark these explicitly — they are rare and high-value.

5. **Pitfall warnings** — "a common mistake is...", "do not confuse...", "beware of..." Hard-won experience encoded in text. Even experienced programmers need these.

6. **"Aha" moment passages** — where the author explicitly reframes how the reader sees something. Signaled by: "The crucial insight is...", "Notice that...", "What this means is...", "The key observation is..."

7. **Exercise passages** — problems posed by the author, with any framing commentary about what skill they test.

Technical candidate guidelines:
- For code examples: preserve every character of code exactly. Never paraphrase, summarize, or "clean up" code.
- For worked examples: capture ALL steps, not just the setup and answer.
- For code samples exceeding ~40 lines: capture the most pedagogically critical segment and note in a separate `omission-note` field what was cut and why.
- Prose context for code: capture 1-2 paragraphs before and after the code block (60-200 words total).

---

**Chapter-level context (for every chapter):**
- **Central claim**: one sentence
- **Advances the book by**: what does this chapter add that the previous one didn't?
- **Key terms introduced or redefined**: list
- **Internal cross-references**: references to other chapters
- **Prerequisites**: what must the reader understand before this chapter? (Used in Phase 3.5 for the concept dependency map)

---

### Checkpoint: Save `extraction.md`

Save to `.readweave/[book-slug]/extraction.md`. Use the standard checkpoint format from the checkpoint reference, extended with these fields per candidate:

```markdown
### CP-N
- **Location**: [Chapter N, "Section Title"]
- **Pool**: [A-prose | B-technical]
- **Type**: [thesis | defining | pivotal | example | rhetorical | self-revision | provocation | illuminating-code | worked-example | algorithm | design-decision | pitfall | aha-moment | exercise]
- **Language**: [python | c | java | pseudocode | etc. — for technical only]
- **Prose-context**: [yes/no and word count — for code passages]
- **Concept-demonstrated**: [one sentence — for technical only]
- **Why it matters**: [one line]
- **Text**:

> [Exact text. For code, use a fenced code block, not a blockquote.]
```

---

## Phase 2: Analyze & Select

### 2a. Map the Argument

Synthesize findings from all agents:

1. **Structural outline** — how the book builds from foundations to advanced topics. Map logical dependencies: which chapters require which earlier chapters?
2. **The pedagogical arc**: introduction → core mechanisms → complexity/edge cases → applications → advanced topics
3. **Rough concept dependency map**: sketch which concepts enable which other concepts. This will be formalized in Phase 3.5.

### 2b. Select Prose Excerpts: 20-25

Use the three-pass process:

**Pass 1 — Non-negotiables (8-12):** Core thesis statements, defining concept introductions, pivotal arguments the book depends on. If you can't find 8 genuinely non-negotiable passages, your agents under-extracted — go back.

**Pass 2 — Arc builders:** Connective tissue. After Pass 1 you have the peaks; now add passages that show how the author gets from one peak to the next.

**Pass 3 — Texture (3-5):** Memorable examples, honest qualifications, rhetorical high points. These make the presentation *alive*, not merely correct.

**Four quality tests (apply to every prose excerpt):**
1. **Idea test**: Can you state what idea this passage captures in one sentence? If not, cut it unless it's a rare rhetorical high point.
2. **Stand-alone test**: Can someone who hasn't read the book understand this passage? If it depends on surrounding context, either extend the excerpt or cut it.
3. **Substitution test**: Could a different passage capture the same idea equally well? If yes, you haven't found the best formulation — go back to the candidate pool.
4. **Redundancy test**: Does this capture an idea already represented by another selected excerpt? If so, keep only the stronger one.

**Excerpt length rules:** minimum 40 words / 2 sentences. Sweet spot: 80-180 words. Maximum ~250 words; use `<span class="elide">[…]</span>` to trim non-essential clauses — never elide the argumentative core. Preserve exact wording.

### 2c. Select Technical Passages: 15-20

Parallel three-pass process:

**Pass 1 — Canonical (5-8):** Code examples or worked examples that cannot be omitted — they introduce core algorithms, data structures, or techniques. These are the passages where a reader who skipped them would be lost in subsequent chapters.

**Pass 2 — Pedagogical coverage (5-7):** Show the arc from simple to complex. Include **at least one** design-decision passage and **at least one** pitfall warning. These two types are the highest-value technical content and must be represented.

**Pass 3 — Teaching quality (3-5):** The clearest worked examples, the most memorable aha moments. Ask: which passages would you use if you were teaching this material?

**Four technical quality tests:**
1. **Teachability test**: Could you teach the concept from this example alone (given general programming knowledge)?
2. **Implementation test**: After reading this passage plus its prose context, could a competent programmer implement this independently?
3. **Misconception test**: Does this passage correct a misconception, or does it risk creating one without adequate context? If the latter, note the risk in the `concept-demonstrated` field.
4. **Redundancy test**: Same as prose redundancy test.

**Code excerpt rules:** Preserve code exactly — no elisions within code. If a sample exceeds ~40 lines, capture the most pedagogically critical segment and add an `omission-note` field explaining what was cut. Prose context (what surrounds the code): 60-200 words.

### 2d. Verify Excerpts Against Source

Go back to the source and re-read each selected passage in its original context:
- Is wording exactly correct? (Parallel agents sometimes introduce small errors.)
- Does the excerpt still work when you read the sentences immediately before and after?
- Is the chapter/section attribution correct?
- For code: does the indentation and formatting match the source?

Fix errors before proceeding. An excerpt with a wrong word is worse than no excerpt.

### Checkpoint: Save `selection.md`

Use the standard checkpoint format, extended with:

```markdown
### EX-N
- **Pool**: [A-prose | B-technical]
- **Technical-type**: [illuminating-code | worked-example | algorithm | design-decision | pitfall | aha-moment | exercise — for Pool B only]
- **Language**: [programming language — for code passages]
- **Prose-context**: [the surrounding prose, 60-200 words]
- **Concept-demonstrated**: [one sentence]
- **Selection pass**: [non-negotiable | arc-builder | texture | canonical | pedagogical | teaching-quality]
```

The argument map in `selection.md` gets a `concept-dependencies` subsection:

```markdown
## Concept Dependencies (draft)
[Rough list: "Chapter 3 hash tables require modular arithmetic from Chapter 1"]
[Will be formalized in Phase 3.5]
```

---

## Phase 3: Annotate

For each selected excerpt (both prose and technical), write **2-3 margin notes**. Each note has a **label** and a **body**.

### Prose voice (identical to `read` skill):

Write in a calm, literate register — the tone of a thoughtful reader's pencil notes in the margin. Favor observations that unfold naturally over declarations that announce themselves. Avoid rhetorical questions used for emphasis, exclamation marks, and breathless discovery language. Let connections speak for themselves. The best notes sound like something a well-read friend would write in pencil: brief, specific, genuinely clarifying, never performing cleverness.

### For prose excerpts — same note quality as `read`:
- **Analytical, not summary**: never restate what the excerpt says
- **Cross-referential**: connect to other parts of the book, other thinkers, broader traditions
- **Hidden implications**: surface what the reader might miss on first reading
- **Concrete**: use specific examples, historical parallels, named thinkers, real events

### For technical passages — 5 specialized note types:

1. **Mechanism notes** — why does this work at a level deeper than the excerpt reaches? Explain the underlying cause, not a definition. Example: "The O(1) average case requires that the hash function distribute keys uniformly — birthday paradox math explains why ~70% load factor is the practical limit."

2. **Misconception surface notes** — name the specific wrong mental model a reader typically brings. Be specific: not "this is confusing" but "readers often conflate amortized cost with worst-case cost; the amortized analysis is a statement about a *sequence* of operations, not any single one."

3. **CS theory connection notes** — connect to a broader theorem, principle, or research area. Examples: birthday paradox for hashing, space-time tradeoff as a design principle, Rice's theorem for decidability questions, CAP theorem for distributed systems choices.

4. **Cross-chapter dependency notes** — "This technique reappears in Chapter 9 where it becomes..." or "This works because of the invariant established in Chapter 3." Make the book's internal structure visible.

5. **Design space notes** — what other approaches exist in the ecosystem that the author doesn't cover? Example: "The author presents separate chaining; open addressing (linear probing, Robin Hood hashing) achieves better cache locality at the cost of more complex deletion."

### Good label examples for technical notes:
"Copy Semantics vs. Reference", "The Invariant Being Maintained", "Why O(log n) Not O(n)", "Stack Discipline Assumption", "Cache Locality at Work", "Hidden Mutation Risk", "The Threshold Condition", "Amortized vs. Worst-Case", "What The Compiler Does Here", "The Load Factor Trade-off", "Pointer vs. Value Semantics", "Tail Call Condition", "When This Breaks", "The Implicit Assumption"

### Good label examples for prose notes (from `read` skill):
"Pessimistic Induction", "Semantic Rupture", "Bootstrapping Problem", "The Logical Fork", "Vocabulary Trap", "Immunological"

### Quality test: Cover the excerpt. Can someone learn something new from just the margin note? If not, rewrite it.

### Checkpoint: Save `annotations.md`

Use the standard checkpoint format. Add a `type` field to each margin note:

```markdown
- **Margin Notes**:
  1. **[Label]** *(type: mechanism | misconception | cs-theory | cross-chapter | design-space | analytical | cross-referential | hidden-implication)*: [Note body]
```

The four active learning artifact sections (3.5a–3.5d) will be appended to this same file in Phase 3.5.

---

## Phase 3.5: Build Active Learning Materials

No equivalent in the `read` skill. Produces 4 artifacts appended to `annotations.md` as new top-level sections.

### 3.5a. Active Recall Prompts (30-50 Q&A pairs)

Five question categories:

**1. Concept questions (8-12):** Test understanding of definitions and properties, but specifically enough to distinguish a reader who understood from one who skimmed. Not "What is a hash table?" but "What property must a hash function have to guarantee expected O(1) lookup, and why does this property matter?"

**2. Implementation questions (8-12):** Draw from code examples and worked examples. "How would you implement X?" — answers should include key invariants and edge cases, not just code. Example: "How would you implement dynamic array resizing such that the amortized cost of append remains O(1)? What invariant must hold after each resize?"

**3. Comparison questions (5-8):** Draw from design-decision passages. "When would you use X instead of Y?" These test judgment, not just recall. Example: "When would you choose open addressing over separate chaining, and what workload characteristics make that trade-off favorable?"

**4. Application questions (5-8):** Transfer tests. Reframe a book example in a novel context. These are the hardest to write — the scenario should be genuinely different from the book's examples while testing the same concept.

**5. Feynman questions (4-6):** The hardest concepts in simplest terms. "Explain amortized analysis without using the word 'amortized'." "Explain why quicksort's expected case is O(n log n) to someone who has never heard of expected value." These reveal whether the reader truly understands or just recognizes terms.

**Format for each Q&A pair:**
```markdown
### AR-N
- **q**: [Question text — specific enough that a reader who skimmed can't answer it]
- **a**: [Key insight first, then elaboration, then caveat if any. Lead with the conceptual core, not with "Well, it depends..."]
- **category**: [concept | implementation | comparison | application | feynman]
- **source**: [EX-N — the excerpt this question tests]
- **difficulty**: [foundational | intermediate | advanced]
```

### 3.5b. Concept Dependency Map (8-15 concepts)

For each major concept in the book:
```markdown
### CDM: [Concept Name]
- **definition**: [One sentence — what IS this concept?]
- **prerequisites**: [background: X], [background: Y], [book: Chapter N concept] — use "background:" for prerequisite knowledge outside the book, "book:" for earlier book concepts
- **enables**: [Concept A], [Concept B], [Concept C] — what later concepts in the book depend on this?
- **introduced**: Chapter N
- **reused-in**: Chapter M, Chapter P (if applicable)
```

Build 8-15 of these, covering the book's conceptual spine. Focus on concepts that appear in multiple chapters or that are prerequisites for other concepts.

### 3.5c. "Type It Yourself" Challenges (5-10)

For the book's most important code examples: challenges that ask the reader to implement from specification rather than copy from the book. The act of typing and debugging from first principles builds different (deeper) understanding than reading.

**Format for each challenge:**
```markdown
### TY-N
- **title**: [Imperative verb phrase — "Implement a Hash Table with Linear Probing"]
- **context**: [1-2 sentences: why this implementation matters, what skill it builds]
- **specification**:
  - Interface: [function signatures or class interface]
  - Test case 1: [concrete input → expected output]
  - Test case 2: [edge case]
  - Test case 3: [stress test or property]
- **hint**: [The key insight, stated without giving away the implementation. Guides without spoiling. "The tricky part is handling deletion — consider why you can't simply remove an entry and what that implies about search."]
- **source**: [EX-N — the passage this challenge is based on]
```

**Challenge selection criteria:**
- Choose examples where the spec is simple but the implementation requires understanding the mechanism
- Avoid challenges where the book's own code is the only reasonable implementation
- Include at least one challenge from a design-decision passage (implementing the alternative the author rejected)

### 3.5d. Feynman Simplifications (3-5)

For the book's 3-5 hardest concepts: a plain-language explanation that would work for a smart non-programmer (or a programmer who has never encountered this concept).

```markdown
### FS-N
- **concept**: [Concept name]
- **target**: [What makes this hard to understand at first? What does the naive mental model get wrong?]
- **simplification**: [3-6 sentences in plain language. Use analogies to everyday objects or processes. Do not use technical jargon — if you must name a concept, immediately explain it in plain terms. The test: would a smart high-schooler understand this?]
- **where-this-breaks**: [One sentence: where the analogy fails. Be honest about the simplification's limits.]
- **source**: [EX-N or chapter reference]
```

---

## Phase 4a: Generate Markdown Mastery Guide

Generate a **lean** quick-reference companion (not a standalone document — the HTML slides carry the full argument and annotations). Keep it short and avoid repeating any passage more than once across the entire document.

**Filename:** `[book-slug]-mastery-guide.md`

**Structure (in order):**

1. **Title and metadata** — book title, author, year, edition
2. **What this book is** — 1-2 paragraphs: significance, target reader, why this book over alternatives
3. **Prerequisites** — what to know before reading, grouped by type (math, programming, CS concepts). Bullet points only.
4. **Concept dependency map** — the full CDM from Phase 3.5b, rendered in markdown
5. **Active recall section** — all Q&A pairs from Phase 3.5a, organized by category
6. **Type it yourself challenges** — all challenges from Phase 3.5c
7. **Key passages** — 10-15 of the most important passages (prose and code), each with a one-line note. No passage should appear elsewhere in this document.
8. **Next steps** — bullet list of what to read/do after this book

Total target length: varies with active recall and challenges, but prose sections should be minimal. If a section feels padded, cut it.

---

## Phase 4b: Generate HTML Master Presentation

**This phase uses the Scaffold-Batch-Assemble pattern.** Read `**/chunked-html-generation.md` within this plugin's installation directory for the full specification. The steps below are master-specific.

### Step 1: Prepare

1. **Read the template**: Use the Glob tool to find `**/master-template.html` within this plugin's installation directory, then read it with the Read tool.
2. **Read the design guide**: Use the Glob tool to find `**/read-design-guide.md` within this plugin's installation directory, then read it.
3. **Read the chunked generation reference**: Use the Glob tool to find `**/chunked-html-generation.md` within this plugin's installation directory, then read it.
4. **Replace placeholders**: `{{BOOK_TITLE}}` and `{{SIDEBAR_SUBTITLE}}`

### Step 2: Plan batches

Plan 5-7 batches based on the annotated excerpts from `annotations.md`:

| Batch | Slides | Content |
|-------|--------|---------|
| 1 | 3 | Title (slide 0, `active`), Overview (concept dependency diagram), Prerequisites |
| 2 | 7-8 | Prose excerpts + code slides, early chapters |
| 3 | 7-8 | Prose excerpts + code slides, middle chapters |
| 4 | 7-8 | Prose excerpts + code slides, late chapters |
| 5 | up to 8 | Remaining content slides (overflow from above) |
| 6 | 3-5 | Recall slides (2-3), Challenge slides (1-2) |
| 7 | 2-3 | Discussion, Closing |

Distribute prose excerpts and code slides **interleaved by chapter order** — not all prose then all code. If the book has fewer chapters, merge batches 4-5 into one.

### Step 3: Write scaffold

Write the HTML file with the template content and batch markers inside `<div class="slides-wrapper">`. Include the batch manifest comment. See the chunked generation reference for the exact format.

### Step 4: Launch batch agents

Launch **all batch agents in a single message** (parallel Agent tool calls). Each agent receives the prompt template from the chunked generation reference, with:
- The design guide path
- The scaffold file path
- Its batch marker text (exact old_string for the Edit)
- Only the EX-N entries assigned to its batch from `annotations.md`
- For batch 6: the recall prompts (AR-N) and challenges (TY-N) from `annotations.md`

### Step 5: Verify

After all agents complete, follow the verification checklist from the chunked generation reference.

### Slide types (~40-50 slides total):

| Slide | Type | Class |
|-------|------|-------|
| 0 | Title | `slide active no-margin` — book title, author/year, opening quote, begin-hint |
| 1 | Overview | `slide no-margin` — core argument + concept dependency diagram (CSS boxes with arrows) |
| 2 | Prerequisites | `slide no-margin` — what to know before reading; `.prereq-item` elements |
| 3–N | Prose excerpt slides (20-25) | `slide` — chapter label, divider, excerpt with drop-cap, page-source, margin div with 2-3 margin-notes |
| — | Code slides (15-20) | `slide code-slide` — chapter label, `<pre><code class="language-X">`, `.code-context`, margin notes |
| — | Recall slides (2-3) | `slide recall-slide no-margin` — 6-8 `.recall-card` elements (click to reveal answer) |
| — | Challenge slides (1-2) | `slide challenge-slide no-margin` — 3-4 `.challenge-item` elements |
| N+1 | Discussion | `slide no-margin` — 6-8 `.discussion-q` elements |
| N+2 | Closing | `slide no-margin` — thesis distilled, final quote, attribution |

**Slide distribution:** ~50% excerpt/code slides, ~20% structural (overview/prereq), ~15% recall/challenge, ~15% discussion/closing

### Code slide HTML pattern:
```html
<div class="slide code-slide" data-title="Chapter N · Concept Name">
  <div class="page-spread">
    <div class="page">
      <div class="page-content">
        <span class="page-chapter">Chapter N &middot; Section Title</span>
        <div class="page-divider"></div>
        <div class="code-context"><p>[prose before the code — what problem is this solving?]</p></div>
        <pre><code class="language-python">[exact code from book]</code></pre>
        <div class="code-context code-context--after"><p>[prose after the code — what does this demonstrate?]</p></div>
        <p class="page-source">Ch. N — Section Name</p>
      </div>
    </div>
    <div class="margin">
      <div class="margin-note">
        <span class="note-label">The Invariant Being Maintained</span>
        <div class="note-connector"></div>
        <p>[Margin note that goes deeper than the code itself]</p>
      </div>
      <div class="margin-note">
        <span class="note-label">Why This Choice</span>
        <div class="note-connector"></div>
        <p>[Design space or mechanism note]</p>
      </div>
    </div>
  </div>
</div>
```

### Recall card slide HTML pattern:
```html
<div class="slide recall-slide no-margin" data-title="Active Recall · [Category]">
  <div class="page-spread">
    <div class="page">
      <div class="page-content">
        <span class="page-chapter">Active Recall</span>
        <h2>[Category Name]</h2>
        <div class="page-divider"></div>
        <div class="recall-card" onclick="this.classList.toggle('revealed')">
          <div class="recall-q">How would you implement X such that Y invariant holds?</div>
          <div class="recall-a">The key insight is Z. A correct implementation must first establish P, then...</div>
        </div>
        <!-- 6-8 recall cards per slide -->
      </div>
    </div>
  </div>
</div>
```

### Challenge slide HTML pattern:
```html
<div class="slide challenge-slide no-margin" data-title="Type It Yourself · [Topic]">
  <div class="page-spread">
    <div class="page">
      <div class="page-content">
        <span class="page-chapter">Type It Yourself</span>
        <h2>Implementation Challenges</h2>
        <div class="page-divider"></div>
        <div class="challenge-item">
          <div class="challenge-title">Implement a Hash Table with Linear Probing</div>
          <div class="challenge-context">Build this from specification to understand why the probing sequence matters for cache performance.</div>
          <div class="challenge-spec">
            <div class="spec-line"><span class="spec-label">Interface</span> insert(k, v), lookup(k) → v|None, delete(k)</div>
            <div class="spec-line"><span class="spec-label">Test 1</span> insert 100 items, lookup all → O(1) average</div>
          </div>
          <button class="hint-toggle" onclick="this.nextElementSibling.classList.toggle('visible')">Show Hint</button>
          <div class="challenge-hint">The tricky part is deletion — you can't simply remove an entry. Think about what happens to the probe sequence.</div>
        </div>
      </div>
    </div>
  </div>
</div>
```

### Formatting rules (reused from `read`):
- Use `<span class="elide">[…]</span>` for elisions within prose excerpts
- Use literal Unicode: `—` for em-dashes, `'` / `'` for single quotes, `"` / `"` for double quotes (do NOT use HTML entities — they break in CSS `content:` properties)
- Use `·` as separator in chapter labels and author lines
- Apply `drop-cap` class to the first `.excerpt` on most prose excerpt slides
- Each slide needs a descriptive `data-title` attribute for the sidebar TOC
- Prism.js language class on every `<code>` element (`language-python`, `language-c`, `language-java`, `language-javascript`, `language-go`, `language-rust`, `language-pseudocode` for algorithm descriptions)

**Filename:** `[book-slug]-master.html`

Derive `[book-slug]` from the book title: lowercase, kebab-case, drop articles. *The Art of Computer Programming* → `art-computer-programming-master.html`.

---

## Phase 5: Refine

Review both outputs:

1. **Flow check**: Read through the slides in order. Does the argument build? Are there gaps?

2. **Margin note audit**: Verify NO margin note merely restates its excerpt. Every note must add analytical or technical depth.

3. **Active recall audit**: Count Q&A pairs by category. At least 30% must be "implementation" or "feynman" categories — not just concept definitions.

4. **Code slide audit**: Every code slide must have ≥ 2 margin notes. None of those notes can merely describe what the code does — they must go deeper (mechanism, misconception, design space, theory connection).

5. **Challenge audit**: Each challenge has a hint that guides toward the key insight without giving it away. Test: can you solve the challenge from the hint alone? If yes, the hint is too strong.

6. **Dependency map consistency**: Every concept named in the CDM appears in at least one slide. Every concept named in a slide's margin notes as a prerequisite appears in the CDM.

7. **Technical check**: Prism.js highlight classes correct (language names match Prism's supported languages), all `data-title` attributes present, first slide has `class="active"`, all HTML tags closed.

8. **Pacing**: Aim for 40-50 total slides. Below 35 loses depth; above 55 loses momentum.

---

## Important Notes

- **Single-file HTML**: The output HTML must be completely self-contained. CSS inline in `<style>`, JS inline in `<script>`. External dependencies: Google Fonts CDN and Prism.js CDN only.
- **Preserve the aesthetic**: The book-page design (parchment colors, Garamond fonts, Caveat margin notes, drop caps) applies to prose slides. Code slides use a dark background (`#1E1E2E`) with light text. Both aesthetics must coexist.
- **The margin notes are the value-add**: The active recall cards, challenges, and dependency map are further value-adds. These make the difference between a reference and a learning tool.
- **Direct excerpts only**: Slides show the author's actual words for prose, and the author's actual code for technical slides. Margin notes provide the analysis.
