---
description: "Deep language mastery guide. Five phases from primitives to production codebase analysis, rooted in SICP's principles."
argument-hint: "[language name]"
---

Generate a comprehensive learning guide for: $ARGUMENTS

Languages consist of **primitives**, **means of combination**, and **means of abstraction**.

## Phases

### 1. Primitives
Basic types, variables, constants, control structures, core data structures, basic functions.
Must include: installation/tooling, REPL, hello world through small programs, type system basics, stdlib overview.
Goal: write simple scripts fluently.

### 2. Combinations
Advanced data structure ops, function composition, pattern matching, higher-order functions, concurrency primitives.
Must include: map/filter/reduce, pattern matching, composition techniques, error handling basics, simple concurrency.
Goal: compose complex operations idiomatically.

### 3. Abstractions
Module/package systems, interfaces/protocols/typeclasses, metaprogramming (macros, generics), error handling patterns, process/thread models.
Must include: module system deep dive, type classes/interfaces/protocols, metaprogramming, testing frameworks, build/package management.
Goal: architect systems and create reusable components.

### 4. Patterns
Architectural patterns, domain modeling, testing strategies, performance, complete working project.
Must include: complete project (100+ lines), testing strategy, performance considerations, deployment/distribution.
Goal: ship production code.

### 5. Codebase Analysis
Study exemplary open source: stdlib implementation, framework internals, production architecture, idioms.
Must include: 3-5 projects analyzed, conceptual architecture diagrams, key idioms extracted, contribution guidelines.
Goal: read and contribute to professional codebases.

## Language-Specific Emphasis

| Type | Emphasis |
|------|----------|
| Functional | Immutability, composition, type systems |
| Object-Oriented | Inheritance vs composition, design patterns |
| Concurrent | Goroutines/actors/processes, message passing |
| Systems | Memory management, zero-cost abstractions |
| Dependently Typed | Curry-Howard, proof techniques, verification |

## Per Phase

- 5 exercises: trivial to challenging, edge cases, one open-ended design problem
- Quick reference: syntax cheatsheet, common idioms, stdlib essentials, tool commands, debugging tips

## Output

```
[language]/
├── README.md              # Overview, learning path, resources
├── 00_quick_reference.md  # One-page cheatsheet
├── 01_primitives.md       # Phase 1
├── 02_combinations.md     # Phase 2
├── 03_abstractions.md     # Phase 3
├── 04_patterns.md         # Phase 4 + complete project
└── 05_codebases.md        # Phase 5
```

## Success Metrics

A seasoned programmer can: write idiomatic code after Phase 2, design systems after Phase 3, ship production code after Phase 4, contribute to major projects after Phase 5.

Tone: direct, dense, practical. Every example runnable. Type signatures and expected outputs included. Assume intelligence, explain nuances.
