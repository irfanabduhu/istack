---
description: "Distill a technical documentation site into a learning-path HTML slide presentation. Crawls hierarchical docs, identifies conceptual layers, and builds a progressive guide from foundations to advanced topics with analytical margin annotations."
argument-hint: "[URL to documentation site root or index page]"
---

# Docs Skill

You will create one output from the following documentation site: $ARGUMENTS
- A comprehensive **HTML slide presentation** (`[project-name]-docs.html`)

The presentation transforms reference documentation into a **learning path** — not a flat summary of every page, but a curated intellectual journey from core concepts through intermediate mastery to advanced patterns. Documentation is written for lookup; this presentation is designed for understanding.

Follow the seven phases below in order.

---

## Checkpoint System

Before starting, check for existing checkpoints in `.readweave/[docs-slug]/` (derive slug from the project or library name — e.g., `react-docs`, `python-asyncio-docs`).

**Read the checkpoint reference**: Use the Glob tool to find `**/checkpoints.md` within this plugin's installation directory, then read it for the full format specification.

**Resume logic** — check in reverse order:
1. If `annotations.md` exists → load it and skip to Phase 6 (HTML generation)
2. If `selection.md` exists → load it and skip to Phase 5 (annotations)
3. If `extraction-[section-slug].md` files exist → load them and skip to Phase 4 (curation). If only some sections have extraction checkpoints, extract only the missing ones.
4. If no checkpoints → start from Phase 1

If resuming, report: "Found checkpoint for [phase]. Resuming from [next phase]..."

If the user's input contains `--fresh`, ignore all checkpoints and start from scratch.

**Save after each phase** — write the checkpoint file in the format specified by the checkpoint reference before proceeding to the next phase.

---

## Phase 1: Discover & Map

### Fetch the documentation structure:

1. Use WebFetch to retrieve the root page or index
2. Extract the **site map / table of contents** — identify:
   - **Top-level sections** (e.g., Getting Started, Core Concepts, API Reference, Guides, Advanced)
   - **Page hierarchy** within each section
   - **Page titles and URLs** for every documentation page
   - **Navigation structure** — how the docs organize themselves (sidebar, breadcrumbs, section headers)
3. Identify the **project name**, **version** (if visible), and **primary language/framework**

### Handle different documentation formats:

- **Structured docs sites** (e.g., React docs, Rust book, Python docs): Follow the sidebar/TOC navigation
- **Single-page docs** (e.g., some library README-style docs): Treat sections as pages
- **Book-style docs** (e.g., Rust Book, SICP online): Follow chapter order — use the `read` skill's approach but fetch via URL
- **Wiki-style docs** (e.g., Arch Wiki): Identify the most important entry pages and follow internal links strategically
- **API reference only** (e.g., generated Javadoc/Doxygen): Focus on the conceptual overview pages and the most architecturally significant types/modules

### Classify pages into conceptual layers:

Sort all discovered pages into **4-6 layers** based on their role in understanding the project:

| Layer | Purpose | Typical pages |
|-------|---------|---------------|
| **Foundations** | What is this? Why does it exist? Core mental model | Introduction, Philosophy, Getting Started, Overview |
| **Core Concepts** | The essential abstractions and mechanisms | Core API, Key Concepts, Architecture, Data Model |
| **Patterns & Practice** | How to use it well — idioms, best practices | Guides, Tutorials, Common Patterns, Best Practices |
| **Deep Mechanics** | How it works internally — the implementation model | Internals, Advanced Concepts, Performance, Architecture Deep Dives |
| **Edge Cases & Escape Hatches** | Where the abstractions leak — gotchas, workarounds | Caveats, Migration Guides, Troubleshooting, FAQ |
| **Ecosystem & Integration** | How it connects to the broader world | Plugins, Extensions, Integration Guides, Comparison pages |

Not every docs site needs all 6 layers. Adapt to the project.

### Present discovery to user:

Show the user the discovered structure: total page count, layers, and which sections map where. Ask if they want to:
- Include all layers
- Focus on specific layers (e.g., "just Core Concepts and Deep Mechanics")
- Exclude API reference pages (often too granular for a learning path)
- Add specific pages they care about

Proceed with the user's selection.

---

## Phase 2: Prioritize & Scope

### Goal: Select 15-30 documentation pages for deep extraction.

Documentation sites can have hundreds of pages. You cannot and should not extract from all of them. Select the pages that carry the most conceptual weight.

### Prioritization criteria:

1. **Mental model pages** (highest priority): Pages that explain how to think about the system — the conceptual overview, the architecture guide, the "thinking in X" page. These are worth 3x a reference page.
2. **Core API pages**: The types, functions, or modules that every user interacts with. Not the full API surface — the essential 20% that covers 80% of use cases.
3. **"Why" pages**: Pages that explain design decisions — why the API looks the way it does, what trade-offs were made. These are rare and extremely valuable.
4. **Pattern pages**: Best practices, common patterns, and idiomatic usage guides. These teach judgment, not just mechanics.
5. **Gotcha pages**: Migration guides, breaking changes, common mistakes, FAQs. These encode hard-won experience.
6. **Comparison pages**: "X vs Y" pages that help the reader understand the design space. Valuable for context.

### Deprioritize:
- Exhaustive API reference for every method/property (too granular)
- Installation/setup guides (procedural, not conceptual)
- Changelog/release notes (historical record, not learning material)
- Contribution guides (project governance, not understanding)

### Output: A prioritized page list with layer assignments.

---

## Phase 3: Fetch & Extract

### Fetch all selected pages in parallel:

Launch **parallel agents** (one per layer, or one per 3-5 pages). Each agent:

1. Fetches the full content of each assigned page using WebFetch
2. Strips navigation, ads, sidebars, and boilerplate
3. Preserves: headings, code blocks, tables, diagrams (described in text), callouts/warnings

4. For each page, extracts **3-10 candidate passages** depending on page length and density:

   **For conceptual/overview pages (5-10 candidates):**
   - **Mental model passages**: Where the docs explain how to think about a concept — the sentence that makes the abstraction click
   - **Definition passages**: Where a key term or concept is precisely defined
   - **Motivation passages**: Why this feature exists, what problem it solves
   - **Design decision passages**: "We chose X over Y because..." — these are gold
   - **Warning/gotcha passages**: "A common mistake is..." or "Note that..."

   **For API/reference pages (3-6 candidates):**
   - **Signature + explanation**: The type signature or function signature paired with its most illuminating description
   - **Key constraint passages**: When the docs specify important invariants, preconditions, or guarantees
   - **Example code with explanation**: Code samples paired with the prose that explains them

   **For guide/tutorial pages (4-8 candidates):**
   - **Pattern descriptions**: Where the docs describe an idiomatic pattern and why it's preferred
   - **Before/after comparisons**: Where the docs show the wrong way and the right way
   - **Architectural insight passages**: Where a tutorial steps back to explain the bigger picture

5. For each candidate passage, capture:
   - The **exact text** (including code blocks where relevant)
   - The **page title and section heading** where it appears
   - The **page URL** for attribution
   - A **one-line note** on why this passage matters
   - The **passage type** (mental-model, definition, motivation, design-decision, warning, signature, constraint, example, pattern, architectural)

6. Records for each page:
   - The page's **core concept** in one sentence
   - How it **connects to other pages** — what does it depend on? What does it enable?
   - Any **prerequisite concepts** the reader needs

### What NOT to extract:
- Step-by-step installation or setup instructions
- Exhaustive parameter lists without conceptual content
- Boilerplate code that doesn't illuminate a concept
- Version-specific changelog entries
- Content duplicated across multiple pages — take the best formulation

### Checkpoint: Save `extraction-[section-slug].md` per conceptual layer to `.readweave/[docs-slug]/` with all candidate passages, page metadata, and layer assignments.

---

## Phase 4: Build the Learning Path

### Goal: Organize extracted content into a progressive learning arc.

### Step 1: Map concept dependencies

For each major concept in the documentation:
- What must the reader understand before this concept makes sense?
- What later concepts does this enable?
- Which pages explain this concept best?

Build a lightweight dependency graph. This determines presentation order.

### Step 2: Design the learning stages

Using the conceptual layers from Phase 1 and the dependency graph, design **4-6 learning stages**:

Each stage answers a question for the reader:
1. **"What is this?"** — Foundations: the problem space, the core mental model, why this project exists
2. **"How do I think about it?"** — Core Concepts: the essential abstractions, the type system, the data model
3. **"How do I use it?"** — Patterns: idiomatic usage, common patterns, best practices
4. **"How does it actually work?"** — Mechanics: internal architecture, performance model, implementation details
5. **"Where does it break?"** — Edge Cases: gotchas, limitations, escape hatches, migration paths
6. **"How does it fit?"** — Ecosystem: integrations, comparisons, the broader landscape

### Step 3: Select excerpts

From all candidates across all pages, select **40-60 excerpts** organized by learning stage.

**Three-pass selection:**

**Pass 1 — The mental model builders (15-20 excerpts).** Passages that explain how to think about the system. These are the slides a new user needs most. Prioritize: conceptual overviews, key definitions, architecture explanations, design decision rationales.

**Pass 2 — The practical builders (15-20 excerpts).** Passages that show how to use the system well. Key API explanations, pattern descriptions, idiomatic code examples with their prose context, before/after comparisons.

**Pass 3 — The depth builders (10-15 excerpts).** Passages that deepen understanding: internal mechanics, performance characteristics, gotchas, edge cases, comparison passages. These make the difference between a user who can follow tutorials and one who can debug problems and make architectural decisions.

**Selection quality tests:**
1. **The understanding test**: After reading this passage, does the reader understand something they didn't before?
2. **The stand-alone test**: Can someone understand this without reading the surrounding docs page?
3. **The redundancy test**: Does another excerpt already cover this concept? Keep the clearer one.
4. **The stage test**: Does this excerpt belong in the learning stage it's assigned to?
5. **The code test** (for code excerpts): Is this code illuminating a concept, or just showing syntax? Only keep code that teaches.

### Checkpoint: Save `selection.md` to `.readweave/[docs-slug]/` with all selected excerpts, learning stages, concept dependencies, and ordering rationale.

---

## Phase 5: Write Margin Annotations

For each selected excerpt, write **2-3 margin notes**.

### Labels must be:
- Descriptive concepts: "The Ownership Model", "Inversion of Control", "The Rendering Lifecycle", "Why Not Inheritance", "The Escape Hatch", "Implicit Contract", "The Cost of Abstraction"
- Bad: "Key Concept!", "Important!", "Note", "Remember This!"

### Note body — five types specialized for documentation:

1. **Mental model notes** — restate the concept in different terms or with an analogy. Documentation explains the API; your margin note explains the mental model behind the API. "Think of this as a pipeline where each stage owns its data — once you pass data downstream, you can't reach back and mutate it."

2. **Design space notes** — what alternatives exist and why the project chose this approach. "React chose a virtual DOM diffing approach; Svelte compiles away the framework entirely. The trade-off is runtime flexibility vs. bundle size and performance predictability."

3. **Gotcha surface notes** — practical warnings that the docs mention briefly but that cause real pain. "The docs note this is 'not recommended' — in practice, this will silently corrupt state in concurrent mode. See [issue/discussion] for war stories."

4. **Cross-concept notes** — connect this concept to others in the same documentation. "This is the complement to [Other Concept] — together they form the system's core invariant. You can't understand either in isolation."

5. **Ecosystem context notes** — connect to the broader ecosystem, prior art, or alternative approaches in other tools. "This pattern is the docs' answer to the 'prop drilling' problem. Vue solves this with `provide/inject`; Angular uses hierarchical dependency injection."

### Prose voice:
- Write in a calm, literate register — the tone of an experienced colleague explaining the docs, not a textbook author
- Be specific: name functions, types, patterns, and related projects
- Avoid enthusiasm markers ("elegant", "powerful", "amazing") — let the design speak for itself
- Surface trade-offs alongside features — honest annotation is more useful than promotional annotation
- The best margin notes sound like something a senior engineer would write during a documentation review: precise, practical, occasionally opinionated

This voice also applies to all commentary prose: overview slides, transition slides, and discussion questions.

### Quality test: Cover the excerpt text. Can someone learn something new from just the margin note? If not, rewrite it.

### Checkpoint: Save `annotations.md` to `.readweave/[docs-slug]/` with all annotated excerpts, overview prose, stage summaries, and discussion questions.

---

## Phase 6: Generate HTML Slides

### Chunked generation strategy

**This phase uses the Scaffold-Batch-Assemble pattern.** Read `**/chunked-html-generation.md` within this plugin's installation directory for the full specification.

**Docs-specific batch planning:** Organize content batches by learning stage (max 8 slides each). Structural batches: opening (title, concept map, learning path roadmap) and closing (cheat sheet, discussion, closing).

### Setup:

1. **Read the template**: Use the Glob tool to find `**/study-template.html` within this plugin's installation directory, then read it with the Read tool.
2. **Read the design guide**: Use the Glob tool to find `**/study-design-guide.md` within this plugin's installation directory, then read it.
3. **Replace placeholders**:
   - `{{STUDY_TITLE}}` → a title capturing the learning journey (e.g., "Understanding React: From Mental Model to Mastery")
   - `{{SIDEBAR_SUBTITLE}}` → the project/library name and version
   - `{{BOOKS_SUBTITLE}}` → source count and stage count (e.g., "25 Pages Across Five Learning Stages")
4. **Populate slides** inside `<div class="slides-wrapper">`:

### Slide structure:

- **Slide 0 — Title**: `class="slide active no-margin"` — the project name, version, learning path title, and a quotation from the docs that captures the project's philosophy. Uses `study-title-page` class. Includes `begin-hint`.

- **Slide 1 — Concept Map**: `class="slide no-margin"` — visual overview of the project's core concepts and how they relate. Uses a dependency diagram showing what builds on what. This is the reader's mental anchor.

- **Slide 2 — Learning Path**: `class="slide no-margin"` — the learning stages laid out with brief descriptions. Shows what the reader will understand at each stage. A map of the journey.

- **Slides 3 through N — Content slides** (the bulk): A mix of:
  - **Excerpt slides** (`class="slide"`): Direct documentation excerpts with margin notes. The `page-chapter` label includes the docs page: `"Core Concepts &middot; Components"`. Use `data-book` attribute with the **learning stage number** for color coding.
  - **Code slides** (`class="slide code-slide"`): Documentation code examples with surrounding context. Uses `<pre><code class="language-X">` with the project's primary language. Include `.code-context` prose before and after.
  - **Evolution slides** (`class="slide evolution-slide"`): Side-by-side comparison of concepts (e.g., "Class Components vs. Hooks", "Sync vs. Async", "Before/After" patterns). Uses `.evolution-panel` elements.
  - **Stage transition slides** (`class="slide no-margin book-transition"`): Brief orienting slide when focus shifts to a new learning stage. Shows stage name, page count, and what the reader will learn.

- **Slide N+1 — Cheat Sheet**: `class="slide no-margin"` — quick-reference summary of the most important concepts, APIs, and patterns. Dense, practical, designed for revisiting.

- **Slide N+2 — Discussion**: `class="slide no-margin"` — 6-8 questions that probe understanding of the project's design, trade-offs, and appropriate use cases. Uses `.discussion-q` elements.

- **Slide N+3 — Closing**: `class="slide no-margin"` — the project's core philosophy distilled + a quotation that captures its essence + attribution with link to the documentation site.

### Formatting rules:
- Same typographic rules as other skills (elisions, em-dashes, smart quotes, drop caps)
- Each slide needs a descriptive `data-title` attribute
- First slide has `class="active"`
- `page-chapter` labels MUST include the docs section and page title
- `page-source` should include the page URL
- Use `data-book` attribute to map **learning stages** to the study template's color system
- Prism.js language class on every `<code>` element — use the project's primary language
- For documentation that mixes prose and code: interleave excerpt slides and code slides

### Slide count guidance:
- **Small docs site (15-20 pages selected)**: 35-50 total slides
- **Medium docs site (20-25 pages)**: 45-60 total slides
- **Large docs site (25-30 pages)**: 55-70 total slides
- Roughly: 40% prose excerpt slides, 20% code slides, 10% evolution/comparison, 10% stage transitions, 20% structural (title, concept map, cheat sheet, discussion, closing)

Save as `[project-name]-docs.html`. Derive from the project name: lowercase, kebab-case.

---

## Phase 7: Refine

Review the output:

1. **Learning path check**: Does the presentation build understanding progressively? Can someone new to the project follow from foundations to advanced topics?
2. **Mental model check**: After reading the first 10 slides, does the reader have a working mental model of the project? If not, the Foundations stage needs work.
3. **Cross-concept check**: Are concept dependencies surfaced in margin notes? Does the reader know what builds on what?
4. **Code quality check**: Every code slide has prose context explaining what the code demonstrates. No orphan code blocks.
5. **Margin note audit**: Verify no margin note merely restates the documentation. Every note must add: a mental model, a design space comparison, a gotcha, a cross-concept link, or ecosystem context.
6. **Practical value check**: After reading the full presentation, could the reader make informed architectural decisions with this project? If it only teaches syntax but not judgment, the Patterns and Edge Cases stages need more depth.
7. **Stage balance check**: Do the learning stages flow naturally? Is any stage too thin or too heavy?
8. **Attribution check**: Every excerpt slide identifies its source docs page with URL.
9. **Technical check**: Well-formed HTML, all slides have `data-title`, first slide has `class="active"`, Prism.js language classes correct.

---

## Important Notes

- **Learning path, not reference**: The value is in transforming reference documentation into a progressive learning experience. A page-by-page summary is not a learning path.
- **Direct excerpts only**: Show the documentation's actual text and code. Margin notes provide the analytical layer.
- **Mental models over APIs**: Prioritize passages that explain how to think about the system over passages that list API signatures. Understanding enables lookup; lookup doesn't enable understanding.
- **Code serves concepts**: Include code slides when the code illuminates a concept, not when it merely shows syntax. Every code slide should teach something a reader couldn't get from just reading the type signature.
- **Design decisions are the highest-value content**: Pages where the docs explain why the project is designed the way it is are worth more than pages that explain how to use it. These teach judgment, not just mechanics.
- **Single-file HTML**: Completely self-contained. All CSS/JS inline. External dependencies: Google Fonts CDN and Prism.js CDN only.
- **The margin notes are the value-add**: Anyone can read the docs. The margin notes — mental models, design space comparisons, gotchas, cross-concept links — are what make this a *learning experience* rather than a docs mirror.
