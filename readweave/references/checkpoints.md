# Checkpoint System

ReadWeave saves intermediate results to `.readweave/[slug]/` in the working directory so that interrupted runs can resume and prose can be revised without redoing extraction.

## Directory Structure

### Single-source commands (read, watch, digest)

```
.readweave/
  [source-slug]/
    extraction.md     # After Phase 1: candidates + context
    selection.md      # After Phase 2: curated excerpts + analysis
    annotations.md    # After Phase 3: excerpts with margin notes
```

### Multi-source commands (study, crawl, playlist)

```
.readweave/
  [project-slug]/
    extraction-[source-slug].md   # Per-source extraction
    selection.md                   # Combined curated excerpts + analysis
    annotations.md                 # Combined excerpts with margin notes
```

### Discovery-based commands (research)

```
.readweave/
  [topic-slug]-research/
    discovery.md                   # After Phase 1: search queries + candidate URLs
    evaluation.md                  # After Phase 2: scored candidates + approved source library
    extraction-[source-slug].md   # Per-source extraction
    selection.md                   # Combined curated excerpts + analysis
    annotations.md                 # Combined excerpts with margin notes
```

## Slug Rules

- Derive from the source: book filename (without extension), video title, article title, study name
- Lowercase, kebab-case, drop articles (a, an, the)
- Examples: `structure-scientific-revolutions`, `lex-fridman-sam-altman`, `paul-graham-crawl`

## Resume Logic

At the start of each phase, check if the checkpoint file for that phase exists:

```
if .readweave/[slug]/annotations.md exists → skip to HTML generation
elif .readweave/[slug]/selection.md exists → skip to annotations
elif .readweave/[slug]/extraction.md exists → skip to selection
else → start from extraction
```

For multi-source commands, extraction checkpoints are per-source. If some sources have extraction checkpoints and others don't, only extract the missing ones.

When resuming, report to the user: "Found checkpoint for [phase]. Resuming from [next phase]..."

## `--fresh` Override

If the user's input contains the word `--fresh`, ignore all existing checkpoints and start from scratch. Still save new checkpoints as you go.

## Checkpoint Format

All checkpoints use markdown. The format is designed so Claude can read them back naturally and so the user can inspect or hand-edit them.

---

### extraction.md

```markdown
# Extraction: [Source Title]
<!-- source: [file path or URL] -->
<!-- date: [ISO timestamp] -->
<!-- command: [read|watch|digest|study|crawl|playlist] -->

## Metadata
- **Title**: ...
- **Author/Creator**: ...
- **Year/Date**: ...
- [any other relevant metadata: publisher, channel, duration, etc.]

## Structure
[Brief outline of the source's structure — chapters, sections, topics, timeline]

## Chapter/Section Context

### [Chapter/Section 1 Title]
- **Central claim**: [one sentence]
- **Advances the argument by**: [one sentence]
- **Key terms introduced**: [list]
- **Cross-references**: [internal references to other chapters/sections]

### [Chapter/Section 2 Title]
...

## Candidate Passages

### CP-1
- **Location**: [Chapter N, "Section Title" | Timestamp ~MM:SS | Article Title]
- **Type**: [thesis | defining | pivotal | example | rhetorical | self-revision | provocation]
- **Why it matters**: [one line]
- **Text**:

> [Exact text of the passage, preserving the author's wording precisely.
> Multi-paragraph passages use consecutive blockquote lines.]

### CP-2
...
```

---

### selection.md

```markdown
# Selection: [Source Title or Study Name]
<!-- source: [file path or URL, or list for multi-source] -->
<!-- date: [ISO timestamp] -->

## Argument Map
[Structural outline of the argument — how it builds from premise to conclusion.
For multi-source: per-source maps plus cross-source analysis.]

## The Arc
[The trajectory: introduction → core argument → evidence → implications → conclusion.
For multi-source: the intellectual evolution across sources.]

## Critical Context
[What the author was responding to, publication context, field state.]

## Selected Excerpts

### EX-1
- **Source**: [Book/Article/Video title]
- **Location**: [Chapter N, "Section Title" | ~MM:SS | Article Title, Year]
- **Selection pass**: [non-negotiable | arc-builder | texture]
- **Idea captured**: [one sentence]
- **Text**:

> [Exact text]

### EX-2
...

## Cross-Source Threads (multi-source only)

### Thread: [Thread Name]
- **Description**: [one sentence]
- **Sources involved**: [list]
- **Excerpts**: [EX-3, EX-7, EX-15]
- **How the idea transforms**: [brief narrative]
```

---

### annotations.md

```markdown
# Annotations: [Source Title or Study Name]
<!-- source: [file path or URL] -->
<!-- date: [ISO timestamp] -->

## Annotated Excerpts

### EX-1
- **Source**: [Book/Article/Video title]
- **Location**: [Chapter N, "Section Title"]
- **Text**:

> [Exact text]

- **Margin Notes**:
  1. **[Label]**: [Note body — analytical, cross-referential, concrete]
  2. **[Label]**: [Note body]
  3. **[Label]**: [Note body]

### EX-2
...

## Overview Prose
[The overview/introduction text for the presentation — written in the calm, literate voice.
Saved here so it can be revised without regenerating everything.]

## Discussion Questions
1. [Question]
2. [Question]
...
```

---

### discussion.md (discord command only)

```markdown
# Discussion: [Source Title]
<!-- source: [file path or URL] -->
<!-- date: [ISO timestamp] -->
<!-- command: discord -->

## Forum Post
- **Platform**: [Hacker News | Reddit]
- **Title**: [Thread title — opinionated, not neutral]
- **Submitted by**: [username]
- **Body**: [Optional post body text, 1-3 sentences]

## Dialectic Arc
[Brief narrative of how the discussion develops: opening positions, key turns, where consensus forms or productively fractures. 3-5 sentences.]

## Comments

### C-1
- **Username**: [username]
- **Archetype**: [practitioner | academic | contrarian | builder | skeptic | historian | synthesizer | devils-advocate | newcomer | witness]
- **Parent**: [top-level | C-N]
- **Score**: [integer]
- **Timestamp**: [relative, e.g., "6 hours ago"]
- **Highlighted**: [yes | no]
- **Tag**: [practitioner | academic | contrarian | builder | none]
- **Body**:

[Comment text, preserving paragraph breaks. May include blockquotes, code, emphasis.]

### C-2
...
```

---

## What Gets Cached vs. Not

**Always cache:**
- Extracted content, candidates, and chapter context (most expensive)
- Curated selections and argument analysis (moderately expensive)
- Margin annotations and prose (enables prose revision without full re-run)

**Never cache:**
- Final HTML/markdown output (that's the deliverable — always regenerated fresh)
- Refinement pass results (quick and should reflect latest state)

## Edge Cases

- **Source file changed**: Checkpoints don't track file hashes. If the user re-runs on a modified source, they should use `--fresh`. If in doubt, ask.
- **Partial extraction (multi-source)**: If a study has 3 books and only 2 have extraction checkpoints, extract only the missing book and merge.
- **User wants to revise only annotations**: If `annotations.md` exists but the user says something like "rewrite the margin notes" or "change the prose style", delete `annotations.md` and resume from `selection.md`.
