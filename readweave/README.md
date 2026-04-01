# ReadWeave Plugin

A unified plugin for both Claude Code and OpenCode that distills books, papers, web writings, and videos into elegant presentations and reading guides.

## Commands

### Books

| Command               | Scope            | Input                            | Output                                     |
| --------------------- | ---------------- | -------------------------------- | ------------------------------------------ |
| `/readweave:read`     | Single book      | `.epub`, `.pdf`, or `.txt` file  | HTML slides + markdown reading guide       |
| `/readweave:master`   | Technical book   | `.epub`, `.pdf`, or `.txt` file  | HTML slides + markdown mastery guide       |
| `/readweave:study`    | Multiple books   | Multiple book files              | Unified HTML slides + markdown study guide |
| `/readweave:narrate`  | Single book      | `.epub`, `.pdf`, `.txt`, or URL  | Parchment-style HTML narration             |

### Papers

| Command              | Scope            | Input                            | Output                                     |
| -------------------- | ---------------- | -------------------------------- | ------------------------------------------ |
| `/readweave:paper`   | Single paper     | `.pdf` or paper file             | HTML slides                                |
| `/readweave:survey`  | Multiple papers  | Multiple `.pdf` or paper files   | Unified HTML slides + markdown survey guide |

### Web

| Command              | Scope                 | Input                      | Output                                          |
| -------------------- | --------------------- | -------------------------- | ----------------------------------------------- |
| `/readweave:digest`  | Single web article    | URL to article or essay    | HTML slides                                     |
| `/readweave:crawl`   | Blog or writings page | URL to index/articles page | Unified HTML slides grouped by thematic threads |
| `/readweave:research`| Topic from scratch    | Topic description          | HTML slides tracing a curated learning path     |

### Video

| Command               | Scope             | Input                | Output             |
| --------------------- | ----------------- | -------------------- | ------------------ |
| `/readweave:watch`    | Single video      | YouTube video URL    | HTML slides        |
| `/readweave:playlist` | YouTube playlist  | YouTube playlist URL | Unified HTML slides |

### Audio / Podcasts

| Command                | Scope              | Input                                    | Output              |
| ---------------------- | ------------------ | ---------------------------------------- | ------------------- |
| `/readweave:listen`    | Single episode     | Audio file (mp3/wav/m4a) or episode URL  | HTML slides         |
| `/readweave:podcast`   | Podcast series     | RSS feed URL or multiple audio files     | Unified HTML slides |

### Documentation

| Command              | Scope                  | Input                         | Output      |
| -------------------- | ---------------------- | ----------------------------- | ----------- |
| `/readweave:docs`    | Documentation site     | URL to docs root or index     | HTML slides |

### Discussions

| Command               | Scope              | Input                                     | Output      |
| --------------------- | ------------------ | ----------------------------------------- | ----------- |
| `/readweave:thread`   | Discussion thread  | URL to thread (Twitter, Reddit, HN, etc.) | HTML slides |

## Prerequisites

For scanned PDFs (image-based), the plugin requires `ocrmypdf` and `poppler` (for `pdftotext`) to automatically detect and OCR them before extraction:

```bash
# macOS
brew install ocrmypdf poppler

# Ubuntu/Debian
sudo apt install ocrmypdf poppler-utils

# Fedora
sudo dnf install ocrmypdf poppler-utils
```

These are only needed if you work with scanned PDFs. Text-based PDFs, EPUBs, and plain text files work without any additional dependencies.

For podcast and audio file support (`/readweave:listen`, `/readweave:podcast`), the plugin requires [Whisper](https://github.com/openai/whisper) for audio transcription:

```bash
# OpenAI Whisper (Python)
pip install openai-whisper

# Or whisper-cpp (faster, C++ native)
brew install whisper-cpp

# Or mlx-whisper (Apple Silicon optimized)
pip install mlx-whisper
```

Audio commands will prompt you to install Whisper if it's not found.

## Installation

### For Claude Code

**From the marketplace (recommended):**

```bash
claude plugin marketplace add irfanabduhu/readweave
claude plugin install readweave@irfanabduhu
```

**Per-session (alternative):**

```bash
git clone https://github.com/irfanabduhu/readweave.git ~/readweave
claude --plugin-dir ~/readweave
```

### For Pi

**Add to your project:**

```bash
git clone https://github.com/irfanabduhu/readweave.git .pi/skills/readweave
```

**Install globally:**

```bash
git clone https://github.com/irfanabduhu/readweave.git ~/.pi/agent/skills/readweave
```

### For OpenCode

**Add to your project:**

Copy the `commands/` folder from this repository into your project's `.opencode` directory:

```bash
cp -r path/to/readweave/commands/. .opencode/commands/
```

**Install globally:**

To make these commands available everywhere, copy them to your global OpenCode config directory:

```bash
cp -r path/to/readweave/commands/. ~/.config/opencode/commands/
```

## Usage

```bash
# Distill a single book
/readweave:read path/to/book.epub

# Master a technical book (active recall cards, code slides, challenges)
/readweave:master path/to/sicp.pdf

# Study multiple books together
/readweave:study path/to/book1.pdf path/to/book2.epub path/to/book3.txt

# Create a parchment-style narration of a book or article
/readweave:narrate path/to/book.epub
/readweave:narrate https://paulgraham.com/greatwork.html

# Deep dive into an academic paper
/readweave:paper path/to/paper.pdf

# Survey a research thread across multiple papers
/readweave:survey path/to/paper1.pdf path/to/paper2.pdf path/to/paper3.pdf

# Deep dive into a web article
/readweave:digest https://paulgraham.com/greatwork.html

# Crawl a blog and weave articles into thematic threads
/readweave:crawl https://paulgraham.com/articles.html

# Research a topic from scratch (discovers its own sources)
/readweave:research "distributed consensus algorithms"

# Deep dive into a YouTube video
/readweave:watch https://www.youtube.com/watch?v=example

# Analyze a YouTube playlist as a curated sequence
/readweave:playlist https://www.youtube.com/playlist?list=example

# Distill a podcast episode (audio file or URL)
/readweave:listen path/to/episode.mp3
/readweave:listen https://podcast.example.com/episode-42

# Analyze a podcast series from an RSS feed
/readweave:podcast https://feeds.example.com/podcast.xml

# Distill a technical documentation site into a learning path
/readweave:docs https://react.dev/learn

# Distill a discussion thread (Twitter, Reddit, HN)
/readweave:thread https://news.ycombinator.com/item?id=example
/readweave:thread https://www.reddit.com/r/programming/comments/example
```

## How it works

### /readweave:read — Single book distillation
1. **Extract & Read** — Detects file type (auto-OCRs scanned PDFs), launches parallel agents to read chapters simultaneously
2. **Analyze & Select** — Synthesizes findings into 25-35 best excerpts across the book's arc
3. **Margin Annotations** — Writes analytical margin notes that connect, historicize, and surface hidden implications
4. **Generate Outputs** — Creates both the markdown reading guide and HTML slide presentation in parallel
5. **Refine** — Audits margin note quality, checks flow, verifies consistency between outputs

### /readweave:master — Technical book mastery
1. **Extract** — Parallel chapter extraction with code block detection
2. **Analyze & Select** — Identifies key concepts, code examples, and proof strategies
3. **Annotate** — Writes margin notes connecting theory to practice
4. **Build Active Learning Materials** — Creates recall cards, concept dependency maps, and typed challenges
5. **Generate Outputs** — Creates mastery guide and HTML presentation in parallel
6. **Refine** — Audits challenge quality, checks concept coverage

### /readweave:study — Multi-book intellectual tracing
1. **Extract & Read All Books** — Parallel extraction across all books simultaneously
2. **Curate & Analyze** — Builds argument maps per book, selects 15-25 excerpts each
3. **Cross-Book Analysis** — Identifies conceptual threads, determines reading order, plans narrative arc
4. **Margin Annotations** — Writes cross-book annotations showing how ideas evolve
5. **Generate Outputs** — Creates unified study guide and slide presentation (chunked generation for 4+ books)
6. **Refine** — Audits cross-references, checks balance, verifies arc coherence

### /readweave:narrate — Parchment-style narration
1. **Fetch & Extract** — Retrieves the content (file or URL), extracts 15-400 candidate passages depending on length
2. **Analyze & Select** — Maps the work's core thesis and structure, selects 10-25 excerpts that capture the author's voice
3. **Write Annotations** — Crafts narrative annotations as whispered asides from a knowledgeable friend
4. **Generate Parchment Narration** — Creates a flowing HTML narration with The Invitation, The Path Unfolds, The Parting View, and The Scroll's Edge
5. **Refine** — Audits flow, checks excerpt quality, verifies the narrative arc

### /readweave:paper — Single paper deep dive
1. **Read & Extract** — Reads paper, extracts key claims, methodology, and results
2. **Analyze & Select** — Maps the argument structure, selects key passages
3. **Margin Annotations** — Writes annotations connecting to the broader research landscape
4. **Generate HTML Slides** — Produces a single-file presentation

### /readweave:survey — Multi-paper research thread
1. **Read & Extract All Papers** — Parallel extraction across all papers
2. **Curate & Analyze** — Builds method/result maps per paper, selects key excerpts
3. **Cross-Paper Analysis** — Traces how methods evolve, results build, and understanding deepens
4. **Determine Representation Weight** — Balances coverage across papers
5. **Margin Annotations** — Writes cross-paper annotations showing field evolution
6. **Generate Outputs** — Creates survey guide and HTML presentation in parallel
7. **Refine** — Audits citation landscape, checks balance

### /readweave:digest — Single article deep dive
1. **Fetch & Extract** — Retrieves article, extracts 8-20 candidate passages
2. **Analyze & Select** — Maps the argument, selects 8-15 excerpts
3. **Margin Annotations** — Writes contextual, analytical margin notes
4. **Generate HTML Slides** — Produces a single-file presentation

### /readweave:crawl — Blog weaving
1. **Discover** — Fetches index page, extracts all article links and metadata
2. **Map & Group** — Classifies articles into 3-7 thematic threads from titles and descriptions
3. **Fetch & Extract** — Parallel agents fetch full articles per thread, extract candidate passages
4. **Curate & Weave** — Selects excerpts per thread, orders by conceptual progression, plans cross-thread connections
5. **Margin Annotations** — Writes cross-article and cross-thread annotations
6. **Generate HTML Slides** — Produces a unified presentation with thread transitions and evolution slides

### /readweave:research — Topic research from scratch
1. **Scout** — Discovers candidate sources across the web (articles, blogs, papers, videos)
2. **Evaluate** — Selects high-quality "gems" based on signal density and insight
3. **Fetch & Extract** — Retrieves and extracts key passages from selected sources
4. **Map & Organize** — Builds a learning path from foundations to frontiers
5. **Curate & Select** — Selects excerpts ordered by conceptual progression
6. **Margin Annotations** — Writes cross-source annotations
7. **Generate HTML Slides** — Produces a comprehensive presentation tracing the learning path
8. **Refine** — Audits coverage, checks path coherence

### /readweave:watch — YouTube video deep dive
1. **Fetch & Extract** — Retrieves transcript, identifies key moments and passages
2. **Analyze & Select** — Maps the argument, selects best segments
3. **Margin Annotations** — Writes contextual, analytical margin notes
4. **Generate HTML Slides** — Produces a single-file presentation

### /readweave:playlist — YouTube playlist analysis
1. **Discover** — Fetches playlist metadata, identifies all videos in sequence
2. **Fetch & Extract** — Parallel agents extract transcripts per video
3. **Map the Arc** — Traces how ideas build across the curator's intended sequence
4. **Curate & Select** — Selects excerpts showing conceptual progression across videos
5. **Margin Annotations** — Writes cross-video annotations showing idea development
6. **Generate HTML Slides** — Produces a unified presentation following the playlist arc

### /readweave:listen — Single podcast/audio episode
1. **Transcribe & Extract** — Transcribes audio via Whisper, detects speakers, cleans transcript, extracts key moments
2. **Analyze & Select** — Maps the conversation arc, selects 10-18 excerpts including dialogue sparks and honest admissions
3. **Margin Annotations** — Writes speaker-aware, dialogue-conscious analytical notes
4. **Generate HTML Slides** — Produces a single-file presentation with speaker attribution

### /readweave:podcast — Podcast series analysis
1. **Discover** — Parses RSS feed or audio files, identifies episodes and metadata
2. **Transcribe & Extract** — Parallel Whisper transcription across all episodes
3. **Map the Arc** — Traces recurring themes, the host's evolving perspective, and guest disagreements across episodes
4. **Curate & Select** — Selects excerpts showing thematic development across episodes
5. **Margin Annotations** — Writes cross-episode annotations showing how the show's ideas evolve
6. **Generate HTML Slides** — Produces a unified presentation organized by thematic threads

### /readweave:docs — Technical documentation learning path
1. **Discover & Map** — Crawls docs site structure, classifies pages into conceptual layers (Foundations, Core Concepts, Patterns, Deep Mechanics, Edge Cases, Ecosystem)
2. **Prioritize & Scope** — Selects 15-30 highest-value pages based on conceptual weight
3. **Fetch & Extract** — Parallel extraction focused on mental models, design decisions, patterns, and gotchas
4. **Build Learning Path** — Maps concept dependencies and designs progressive learning stages
5. **Margin Annotations** — Writes mental model, design space, gotcha, and cross-concept notes
6. **Generate HTML Slides** — Produces a presentation that transforms reference docs into a learning experience
7. **Refine** — Audits learning progression, mental model clarity, and practical value

### /readweave:thread — Discussion thread distillation
1. **Fetch & Extract** — Retrieves thread from any platform (Twitter/X, Reddit, HN, forums), maps conversation tree, identifies key participants
2. **Analyze & Select** — Maps the dialectic structure (claims, rebuttals, synthesis), selects 8-20 excerpts that capture the conversation's intellectual arc
3. **Margin Annotations** — Writes dialectic-aware, position-neutral annotations that illuminate argument structure and epistemic quality
4. **Generate HTML Slides** — Produces a single-file presentation preserving the multi-voice conversation format

## Supported formats

- `.epub` (extracted and read chapter-by-chapter)
- `.pdf` — text-based (read directly) and scanned/image-based (auto-OCR'd, then read)
- Plain text / `.txt`
- Web URLs (articles, blog posts, writings pages, documentation sites)
- YouTube URLs (individual videos and playlists)
- Audio files (`.mp3`, `.wav`, `.m4a`, `.ogg`, `.flac`, `.aac`) — transcribed via Whisper
- Podcast RSS feeds (episodes discovered and transcribed automatically)
- Discussion threads (Twitter/X, Reddit, Hacker News, Discourse, and other forums)

## Design

All HTML presentations use a parchment book-page aesthetic with Cormorant Garamond and EB Garamond fonts, margin notes in Caveat cursive, drop caps, and a responsive layout that hides margins below 900px. Every output is a single self-contained `.html` file that works offline.
