# ReadWeave Slide Design Guide

## Color Palette

```css
--cream: #FAF6F0      /* background behind page */
--page: #FFFDF8       /* page surface */
--ink: #2C2321        /* primary text */
--ink-light: #4A3C36  /* commentary text */
--ink-lighter: #8B7468 /* metadata, labels */
--accent: #8B3A3A     /* drop caps, highlights */
--gold: #B8963E       /* chapter labels, ornaments */
--divider: #D4C5B5    /* rules, separators */
--note-ink: #6B5D52   /* margin note text */
--note-accent: #9B6B4A /* margin note labels */
```

Body background: `#E8E0D4`

## Typography

| Element | Font | Weight | Size |
|---------|------|--------|------|
| Headings (h1, h2) | Cormorant Garamond | 300-400 | 2.6rem / 1.7rem |
| Body / Excerpts | EB Garamond | 400 | 1.2rem, line-height 1.9 |
| Margin notes | Caveat (cursive) | 400 | 0.95rem, line-height 1.45 |
| Labels / metadata | Inter | 500-700 | 0.45-0.65rem, uppercase, letter-spacing 0.15-0.2em |
| Commentary | EB Garamond | 400 | 1.15rem, line-height 1.8 |

## Layout

- **Page spread** = `.page` (660px) + `.margin` (220px)
- Page padding: 48px 52px 44px
- Margin padding: 48px 20px 44px 24px
- Page min-height: 480px, max-height: calc(100vh - 80px)
- Responsive: margin hidden below 900px; reduced padding below 768px

## Slide Types

### Title Slide
```html
<div class="slide active no-margin" data-title="Title">
  <div class="page-spread">
    <div class="page title-page">
      <div class="page-content">
        <div class="title-ornament">&loz; &loz; &loz;</div>
        <h1>Book Title</h1>
        <div class="author">AUTHOR NAME &middot; YEAR</div>
        <div class="page-divider" style="margin:0 auto 1.5rem"></div>
        <p class="tagline">"Opening quotation from the book."</p>
        <p class="begin-hint">Press &rarr; to begin</p>
      </div>
    </div>
  </div>
</div>
```

### Overview Slide
```html
<div class="slide no-margin" data-title="Overview Title">
  <div class="page-spread">
    <div class="page">
      <div class="page-content">
        <span class="page-chapter">The Argument</span>
        <h2>The Big Idea</h2>
        <div class="page-divider"></div>
        <p class="commentary">Summary of the book's core argument with <strong>key terms bold</strong>.</p>
        <!-- Optional: cycle-diagram for horizontal sequences, level-tower for hierarchies -->
        <p class="commentary">Additional commentary framing what follows.</p>
      </div>
    </div>
  </div>
</div>
```

### Excerpt + Margin Notes (the bulk of slides)
```html
<div class="slide" data-title="Short Descriptive Title">
  <div class="page-spread">
    <div class="page">
      <div class="page-content">
        <span class="page-chapter">Chapter N &middot; Chapter Title</span>
        <div class="page-divider"></div>
        <p class="excerpt drop-cap">First excerpt paragraph with drop cap...</p>
        <p class="excerpt">Continuation paragraph (auto-indented via CSS)...</p>
        <p class="page-source">Ch. N &mdash; Section Name</p>
      </div>
    </div>
    <div class="margin">
      <div class="margin-note">
        <span class="note-label">Descriptive Concept Label</span>
        <div class="note-connector"></div>
        <p>Analytical note that connects, contextualizes, or surfaces hidden implications. Never restates the excerpt.</p>
      </div>
      <!-- 2-3 margin notes per slide -->
    </div>
  </div>
</div>
```

### Discussion Questions
```html
<div class="slide no-margin" data-title="Discussion">
  <div class="page-spread">
    <div class="page">
      <div class="page-content">
        <span class="page-chapter">For Discussion</span>
        <h2>Questions</h2>
        <div class="page-divider"></div>
        <div class="discussion-q">
          <span class="qn">1.</span>
          <span>Thought-provoking question text?</span>
        </div>
        <!-- 6 questions -->
      </div>
    </div>
  </div>
</div>
```

### Closing Slide
```html
<div class="slide no-margin" data-title="Closing">
  <div class="page-spread">
    <div class="page title-page">
      <div class="page-content">
        <h2>Thesis distilled in a phrase</h2>
        <div class="page-divider" style="margin:1rem auto"></div>
        <p class="excerpt" style="text-align:center;text-indent:0;font-style:italic;font-size:1.15rem;max-width:480px;">Final quotation from the book.</p>
        <div class="page-ornament" style="margin-top:2rem">&bull; &bull; &bull;</div>
        <p style="font-family:'Inter',sans-serif;font-size:0.6rem;...">Author &middot; Title &middot; Year</p>
      </div>
    </div>
  </div>
</div>
```

## Prose Voice

All generated prose — margin notes, commentary, overview text, discussion questions — should be written in a calm, literate register. The tone is that of a thoughtful reader writing in pencil, not a lecturer or enthusiast. Favor observations that unfold naturally over declarations that announce themselves. Avoid rhetorical questions for emphasis, exclamation marks, and breathless discovery language. Let connections speak for themselves rather than dramatizing them. The best notes sound like something a well-read friend might pencil in the margin: brief, specific, genuinely clarifying, never performing cleverness.

## Margin Note Quality Criteria

### DO:
- **Analyze** — what does this passage imply that a casual reader would miss?
- **Connect** — link to other parts of the book, other thinkers, broader traditions
- **Historicize** — place the argument in its intellectual context with named figures
- **Surface tension** — identify where the author's argument is weakest or most radical
- **Use concrete examples** — specific historical parallels, named scholars, real events

### DON'T:
- Restate or summarize what the excerpt already says
- Use performative labels ("Bombshell!", "Mind-blowing!", "Key Point!")
- Write generic observations that could apply to any text
- Add notes that are just definitions of terms used in the excerpt

### Good label examples:
"Pessimistic Induction", "Semantic Rupture", "Immunological", "Quine's Web", "The Logical Fork", "Vocabulary Trap", "Bootstrapping Problem"

### Bad label examples:
"Key Insight", "Important Point", "Note", "Interesting", "Wow"

## Excerpt Selection Criteria

- Self-contained: readable without surrounding context
- Argumentatively significant: advances or crystallizes a key claim
- Memorable: distinctive prose, striking metaphors, or powerful logical moves
- Representative: covers the arc of the book, not just the highlights
- Varied: mix of theoretical claims, historical examples, methodological reflections

## Elisions

Use `<span class="elide">[...]</span>` to trim non-essential clauses. Keep elisions minimal and preserve the grammatical integrity of the sentence.
