---
name: readweave
description: Distill books (EPUB, PDF, TXT), academic papers (PDF), web writings (URLs), video (YouTube), and blogs into elegant single-file HTML slide presentations and markdown reading guides. Use this skill for summarizing long-form content, creating study materials, surveying research threads, or "weaving" multiple sources into a unified intellectual map.
---

# ReadWeave Skill

You are an expert at distilling complex long-form content into elegant, readable summaries. This skill automates the process of extracting key insights and presenting them in a high-quality "parchment" aesthetic.

## Available Workflows

Refer to the following command definitions in the `commands/` directory of this skill's installation path:

- **Single Book Distillation**: Use `commands/read.md`. Supports .epub, .pdf (auto-OCR for scanned), and .txt.
- **Technical Book Mastery**: Use `commands/master.md`. CS/technical book with code slides, active recall, and challenges.
- **Multi-Book Synthesis**: Use `commands/study.md`. Unified maps for multiple book files.
- **Narrative Retelling**: Use `commands/narrate.md`. Friendly, parchment-style narration with extensive quotations from any source type.
- **Discord (Narration + Debate)**: Use `commands/discord.md`. Narrates a work then stages a vigorous simulated forum debate (20-40 HN/Reddit comments) arguing for and against the ideas.
- **Single Paper Deep-Dive**: Use `commands/paper.md`. Deep reading of a technical/academic paper (PDF).
- **Multi-Paper Survey**: Use `commands/survey.md`. Trace a research thread across related papers.
- **Web Article Deep-Dive**: Use `commands/digest.md`. Best for individual URLs or essays.
- **Topic Research**: Use `commands/research.md`. Discovers high-quality sources across the web and builds a curated learning path.
- **Blog Thread Weaving**: Use `commands/crawl.md`. Groups blog posts into thematic threads.
- **YouTube Video Deep-Dive**: Use `commands/watch.md`. Single video transcript analysis.
- **YouTube Playlist**: Use `commands/playlist.md`. Curated sequence of videos.
- **Single Podcast/Audio Episode**: Use `commands/listen.md`. Transcribes and distills a podcast episode or audio file (mp3, wav, m4a).
- **Podcast Series**: Use `commands/podcast.md`. Traces themes across multiple podcast episodes from an RSS feed or audio files.
- **Technical Documentation**: Use `commands/docs.md`. Crawls a documentation site and builds a progressive learning-path presentation.
- **Discussion Thread**: Use `commands/thread.md`. Captures the dialectic structure of Twitter/X threads, Reddit posts, HN discussions, and forum threads.

## Core Mandates

1. **Parallel Extraction**: Always use parallel agents (tools) to read different chapters or URLs simultaneously to minimize latency.
2. **High-Fidelity Excerpts**: Never paraphrase key passages. Extract 60-200 word paragraphs that capture the author's original voice.
3. **Margin Annotations**: Write analytical, contextual notes that explain *why* a passage matters, its historical context, or its connection to other ideas.
4. **Single-File Output**: All HTML presentations must be self-contained (CSS/JS inlined) and use the templates in the `assets/` directory.

## Execution Strategy

When a user asks to "read", "study", or "weave" content:
1. Identify the input type (file path or URL).
2. Load the appropriate logic from the `commands/` markdown file.
3. Follow the multi-phase execution (Extraction -> Selection -> Annotation -> Generation).
4. Save progress to `.readweave/` checkpoints as defined in `references/checkpoints.md`.
