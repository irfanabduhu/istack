---
description: "Reverse-engineer a library, framework, or codebase. Trace architecture, core abstractions, and design decisions."
argument-hint: "[library/framework name or repo URL]"
---

Reverse-engineer and explain the internals of: $ARGUMENTS

1. **Orientation** — identify the project, fetch the source (clone or browse). Map the directory structure, entry points, and build system. Determine the core problem it solves and its key constraints.
2. **Architecture** — trace the main abstractions: what are the 3-5 core types/interfaces? How does data flow through the system? What are the module boundaries and dependency directions? Draw the conceptual architecture.
3. **Mechanics** — pick the 3 most important code paths (e.g., a request lifecycle, a compilation pass, a render cycle). Walk through each step-by-step with actual source references. Show how the abstractions interact in practice.
4. **Design decisions** — identify the non-obvious engineering choices. What trade-offs did they make? What alternatives exist and why were they rejected? Where are the clever parts and where are the known warts?
5. **Patterns extracted** — distill reusable patterns, idioms, and techniques. What can the reader steal for their own projects?

Output as `[project-name]-dissection.md`. Include actual code excerpts (with file paths), not summaries. Prioritize *why* over *what* — the reader can read the code themselves.
