# Monorepo Architecture: LLM Memory Strategies

## Purpose

This repo is being restructured from a single MCP plan into a monorepo for researching and benchmarking multiple memory strategies for language models.

The design separates three concerns:

1. **Source-of-truth layer** — stores raw evidence and LLM-ready Markdown derivatives.
2. **Strategy layer** — transforms or indexes that source-of-truth into a memory product.
3. **Benchmark layer** — evaluates strategies against common fixtures and tasks.

## Why a monorepo

A monorepo lets different strategies share:

- research notes;
- context data layout;
- ingestion conventions;
- benchmark fixtures;
- evaluation rubrics;
- common MCP/API contracts;
- implementation utilities later.

It also makes it easier to run a "hackathon" where different models or agents implement competing approaches under the same rules.

## Boundary between source-of-truth and strategy

The source-of-truth layer should not decide how memory is summarized or retrieved. It only guarantees that context evidence is stored in a consistent, LLM-consumable way.

It owns:

- raw files;
- extracted Markdown derivatives;
- context/event/reference metadata;
- stable facts if explicitly curated;
- search/list/read APIs over evidence.

Strategies own:

- summaries;
- indexes beyond basic source access;
- topic reports;
- knowledge graphs;
- embeddings;
- retrieval policies;
- context packing;
- answer/report generation.

## Shared source-of-truth shape

Each context has temporal and atemporal material.

```text
<context>/
├── events/       # dated things that happened
├── references/   # atemporal or slow-changing resources/artifacts
└── facts/        # optional curated stable facts
```

### Events

Events are dated, ordered, and useful for timelines.

Examples:

- medical consultation;
- exam result;
- hospital update;
- work meeting;
- WhatsApp message batch;
- product decision call.

### References

References are stable or slow-changing artifacts.

Examples:

- identity document;
- project brief;
- organization chart;
- contract;
- medical baseline document;
- property listing snapshot that is used as a reference rather than a timeline event.

### Facts

Facts are curated, already-processed stable claims.

Examples:

- person's full name;
- relationship;
- birth date;
- project constants;
- account identifiers;
- canonical names.

Facts are not raw evidence. They are processed memory. They can live in the shared source layer only if they remain minimal, explicitly curated, and provenance-linked.

## Strategies currently planned

### Context Compiler

Richer strategy that compiles source evidence into:

- root `summary-context.md`;
- topic reports;
- optional wiki/topic pages;
- HTML inspection;
- context packs.

### Baseline Single File

Simplest possible strategy:

- one canonical generated Markdown file per context;
- no graph, no topic pages, no embeddings initially;
- serves as benchmark baseline.

## Benchmark/hackathon

The benchmark should evaluate strategies as memory systems:

- Can they answer correctly?
- Can they cite the right evidence?
- Do they update after new sources?
- Do they preserve uncertainty and contradictions?
- What is the token/API cost?
- How easy is the result for a human to inspect?

## Decision summary

Recommended next implementation order:

1. Specify the source-of-truth store and context layout.
2. Implement/plan the baseline single-file strategy.
3. Update the context-compiler plan to consume the shared source layer.
4. Define the benchmark fixture/rubric before building advanced retrieval.
5. Only then implement competing strategies.
