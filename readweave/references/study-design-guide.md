# Book Study Slide Design Guide

This extends the read skill design system with slide types for multi-book analysis.

## Color Palette

Inherits the full read skill palette, plus book-specific accent colors for visual differentiation:

```css
/* Base palette (same as read skill) */
--cream: #FAF6F0;
--page: #FFFDF8;
--ink: #2C2321;
--ink-light: #4A3C36;
--ink-lighter: #8B7468;
--accent: #8B3A3A;
--gold: #B8963E;
--gold-soft: rgba(184, 150, 62, 0.10);
--divider: #D4C5B5;
--note-ink: #6B5D52;
--note-accent: #9B6B4A;

/* Book accent colors — assigned via data-book attribute */
--book-1: #8B3A3A;   /* deep red (default accent) */
--book-2: #3A5A8B;   /* slate blue */
--book-3: #5A7A3A;   /* olive green */
--book-4: #7A3A7A;   /* plum */
--book-5: #8B6B3A;   /* amber */
--book-6: #3A7A7A;   /* teal */
```

Body background: `#E8E0D4`

## Typography

Same as read skill. See the read skill design guide for the full type scale.

## New Slide Types

### Study Title Slide
```html
<div class="slide active no-margin" data-title="Title">
  <div class="page-spread">
    <div class="page study-title-page">
      <div class="page-content">
        <div class="title-ornament">&loz; &loz; &loz;</div>
        <h1>Study Title</h1>
        <div class="author">AUTHOR NAME</div>
        <div class="books-subtitle">A Study in Three Books</div>
        <div class="page-divider" style="margin:0 auto 1.5rem"></div>
        <p class="tagline">"A unifying quotation that captures the intellectual project."</p>
        <p class="begin-hint">Press &rarr; to begin</p>
      </div>
    </div>
  </div>
</div>
```

### Library Slide (Book Overview)
```html
<div class="slide no-margin" data-title="The Books">
  <div class="page-spread">
    <div class="page">
      <div class="page-content">
        <span class="page-chapter">The Corpus</span>
        <h2>Books in This Study</h2>
        <div class="page-divider"></div>
        <div class="book-card" data-book="1">
          <div class="book-card-marker"></div>
          <div class="book-card-content">
            <span class="book-card-year">1962</span>
            <span class="book-card-title">The Structure of Scientific Revolutions</span>
            <p class="book-card-thesis">One-sentence thesis of this book.</p>
          </div>
        </div>
        <!-- Repeat for each book -->
      </div>
    </div>
  </div>
</div>
```

### Book Transition Slide
```html
<div class="slide no-margin book-transition" data-title="Book Title" data-book="2">
  <div class="page-spread">
    <div class="page">
      <div class="page-content book-transition-content">
        <div class="book-transition-marker"></div>
        <span class="page-chapter">Book Two &middot; 1977</span>
        <h2>The Essential Tension</h2>
        <div class="page-divider"></div>
        <p class="commentary">One-paragraph framing: what this book contributes to the intellectual project and why we turn to it now.</p>
      </div>
    </div>
  </div>
</div>
```

### Excerpt Slide (Extended for Multi-Book)
```html
<div class="slide" data-title="Short Title" data-book="1">
  <div class="page-spread">
    <div class="page">
      <div class="page-content">
        <span class="page-chapter">The Structure of Scientific Revolutions &middot; Chapter V</span>
        <div class="page-divider"></div>
        <p class="excerpt drop-cap">Direct excerpt from the book...</p>
        <p class="page-source">Ch. 5 &mdash; The Priority of Paradigms</p>
      </div>
    </div>
    <div class="margin">
      <div class="margin-note">
        <span class="note-label">Descriptive Concept</span>
        <div class="note-connector"></div>
        <p>Analytical note. Cross-book notes can reference other works in the study.</p>
      </div>
    </div>
  </div>
</div>
```

Note: The `data-book` attribute applies a subtle left-border color to the page, matching the book's assigned color. This helps orient the reader.

### Evolution Slide
```html
<div class="slide evolution-slide no-margin" data-title="Concept Evolution">
  <div class="page-spread">
    <div class="page">
      <div class="page-content">
        <span class="page-chapter">Evolution &middot; Concept Name</span>
        <h2>How the Concept Transforms</h2>
        <div class="page-divider"></div>
        <div class="evolution-panel" data-book="1">
          <div class="evolution-panel-header">
            <span class="evolution-book">The Structure of Scientific Revolutions</span>
            <span class="evolution-year">1962</span>
          </div>
          <p class="excerpt">Direct quotation showing the early formulation...</p>
          <p class="page-source">Ch. N &mdash; Section</p>
        </div>
        <div class="evolution-arrow">&darr;</div>
        <div class="evolution-panel" data-book="2">
          <div class="evolution-panel-header">
            <span class="evolution-book">The Essential Tension</span>
            <span class="evolution-year">1977</span>
          </div>
          <p class="excerpt">Direct quotation showing the revised formulation...</p>
          <p class="page-source">Ch. N &mdash; Section</p>
        </div>
        <p class="commentary" style="margin-top:1.2rem">Brief analytical commentary on what changed and why.</p>
      </div>
    </div>
  </div>
</div>
```

### Reading Order Slide
```html
<div class="slide no-margin" data-title="Reading Order">
  <div class="page-spread">
    <div class="page">
      <div class="page-content">
        <span class="page-chapter">Recommended Path</span>
        <h2>Reading Order</h2>
        <div class="page-divider"></div>
        <div class="reading-order-item" data-book="1">
          <div class="reading-order-number">1</div>
          <div class="reading-order-content">
            <span class="reading-order-title">Book Title</span>
            <span class="reading-order-year">1962</span>
            <p class="reading-order-rationale">Why to read this first: introduces foundational concepts...</p>
          </div>
        </div>
        <!-- Repeat for each book -->
      </div>
    </div>
  </div>
</div>
```

### Discussion Slide (same structure as read skill)
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
          <span>Cross-book question requiring knowledge of multiple works?</span>
        </div>
        <!-- 6-8 questions -->
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
        <h2>The intellectual project distilled</h2>
        <div class="page-divider" style="margin:1rem auto"></div>
        <p class="excerpt" style="text-align:center;text-indent:0;font-style:italic;font-size:1.15rem;max-width:480px;">Final quotation that captures the arc of the whole study.</p>
        <div class="page-ornament" style="margin-top:2rem">&bull; &bull; &bull;</div>
        <p style="font-family:'Inter',sans-serif;font-size:0.6rem;font-weight:500;letter-spacing:0.15em;text-transform:uppercase;color:var(--ink-lighter);margin-top:0.8rem;">Author &middot; Study Title</p>
      </div>
    </div>
  </div>
</div>
```

## Book Color Assignment

Assign colors in the order books appear in the recommended reading order:
- Book 1: `--book-1` (deep red `#8B3A3A`)
- Book 2: `--book-2` (slate blue `#3A5A8B`)
- Book 3: `--book-3` (olive green `#5A7A3A`)
- Book 4: `--book-4` (plum `#7A3A7A`)
- Book 5: `--book-5` (amber `#8B6B3A`)
- Book 6: `--book-6` (teal `#3A7A7A`)

The `data-book` attribute controls the subtle left-border color on excerpt slide pages and the marker color on book cards, transition slides, and evolution panels.

## Prose Voice

Inherits the read skill's prose voice guidance. All generated prose — margin notes, commentary, transition framing, evolution analysis, discussion questions — should be written in a calm, literate register. The tone is that of a thoughtful reader writing in pencil, not a lecturer or enthusiast. Let connections and observations speak for themselves rather than dramatizing them.

## Margin Note Quality Criteria

Inherits all criteria from read skill, plus:

### Cross-Book Annotations (the unique value-add):
- **Forward references**: "This definition will be narrowed in [Later Book]..."
- **Backward references**: "Compare the more tentative formulation in [Earlier Book]..."
- **Evolution markers**: "Notice how [concept] has shifted from [meaning A] to [meaning B]..."
- **Tension surfacing**: "This claim sits uneasily with the argument in [Other Book]..."

### Good cross-book labels:
"Semantic Drift", "The Revised Criterion", "Abandoned Framework", "Conceptual Continuity", "The Fork", "Return to Origins", "Silent Retraction"

### Bad cross-book labels:
"Connection!", "Cross-Reference", "See Also", "Compare", "Link"

## Excerpt Selection for Multi-Book Studies

Beyond the standard criteria (self-contained, argumentatively significant, memorable):
- **Favor passages that show evolution**: If a concept changes across books, select the passage where the change is most visible
- **Select "before and after" pairs**: Excerpts that work together across books to show transformation
- **Include self-corrections**: Passages where the author explicitly revises earlier claims
- **Don't neglect minor works**: Even a short essay collection may contain the clearest statement of a revised position
