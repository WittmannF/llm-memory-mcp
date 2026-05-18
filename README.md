# LLM Memory Strategies Monorepo

This repository is a research and implementation monorepo for testing different strategies for **memory management for language models**.

The goal is not to bet on a single architecture too early. The repo is organized so multiple LLM memory strategies can be designed, implemented, benchmarked, and compared against the same shared source-of-truth layer.

## Core idea

Different memory strategies should consume the same canonical data layer:

```text
source-of-truth contexts
  -> shared ingestion/access MCP
  -> strategy-specific memory builders
  -> strategy-specific MCP tools
  -> benchmark/hackathon evaluation
```

The shared layer stores raw evidence plus LLM-ready Markdown derivatives. Strategies can then compete on how well they summarize, retrieve, update, and package that memory for LLM clients.

## Monorepo layout

```text
.
├── README.md
├── docs/
│   ├── architecture/                 # monorepo-level architecture notes
│   └── research/                     # literature, best practices, recommendations
│
├── source-of-truth/                  # canonical context data layout, templates only
│   ├── README.md
│   ├── contexts/README.md            # real contexts live here locally/private, ignored by git
│   └── templates/context/            # template for one context
│
├── shared/
│   └── source-of-truth-store/        # common MCP/access layer plan
│       ├── README.md
│       └── docs/plans/
│
├── strategies/
│   ├── context-compiler/             # richer strategy: extracted, wiki, reports, summary-context
│   │   ├── README.md
│   │   └── docs/plans/
│   │
│   └── baseline-single-file/         # simplest baseline: one generated context file
│       ├── README.md
│       └── docs/plans/
│
└── benchmarks/
    └── memory-strategy-benchmark/    # benchmark/hackathon plan for comparing strategies
        ├── README.md
        └── docs/plans/
```

## Shared source-of-truth model

The shared source-of-truth is organized by **contexts** or **projects**:

```text
source-of-truth/contexts/<context>/
├── README.md
├── manifest.json
├── events/               # dated, temporal evidence
│   └── YYYY/MM/DD/<event-slug>/
│       ├── event.md
│       ├── raw/
│       └── extracted/
├── references/           # atemporal or slow-changing artifacts
│   └── <reference-slug>/
│       ├── reference.md
│       ├── raw/
│       └── extracted/
└── facts/                # optional curated immutable/slow-changing facts
    └── facts.md
```

Examples:

- `events/2026/05/18/medical-consultation/` — a consultation, exam, meeting, call, or message batch.
- `references/person-id-card/` — stable document or identity artifact.
- `facts/facts.md` — curated facts like names, relationships, stable identifiers, project constants.

The important rule: every hard-to-read raw source should have an LLM-ready Markdown derivative.

```text
raw/audio.ogg       -> extracted/audio.md
raw/exam.pdf        -> extracted/exam.md
raw/screenshot.jpg  -> extracted/screenshot.md, optionally raw image available too
```

For images, the system should support a policy flag:

- `markdown_only`: use OCR/description only;
- `image_plus_markdown`: include both image and Markdown derivative for multimodal models;
- `image_on_demand`: use Markdown by default and load image only if needed.

## Current strategies

### Strategy 1 — Context Compiler

Path: `strategies/context-compiler/`

A richer strategy inspired by LLM Wiki, Basic Memory, MemGPT, GraphRAG, RAPTOR, and long-term agent memory literature.

It builds:

- `summary-context.md`;
- extracted source indexes;
- optional wiki/topic pages;
- durable topic reports;
- HTML inspection;
- MCP tools such as `build_context_pack` and `generate_topic_report`.

### Strategy 2 — Baseline Single File

Path: `strategies/baseline-single-file/`

The simplest possible baseline:

- read the shared source-of-truth;
- generate exactly one canonical context file per context;
- keep sections for TL;DR, facts, timeline, sources, contradictions, and open questions;
- expose that single file through MCP.

This strategy is intentionally simple so richer strategies can prove they are worth their extra complexity.

## Shared component

### Source-of-Truth Store MCP

Path: `shared/source-of-truth-store/`

This is the common ingestion/access layer for all strategies. It should handle:

- creating contexts;
- ingesting sources;
- storing raw files;
- generating LLM-ready Markdown derivatives;
- listing/searching events and references;
- exposing context data through MCP and HTTP.

## Benchmark / Hackathon

Path: `benchmarks/memory-strategy-benchmark/`

This will define a benchmark/hackathon where different strategies compete on:

- answer faithfulness;
- retrieval quality;
- context completeness;
- update quality after new evidence;
- handling of contradictions;
- token cost;
- LLM/API cost;
- latency;
- implementation complexity;
- human inspectability.

## Research base

Start here:

- [`docs/research/README.md`](docs/research/README.md)
- [`docs/research/best-practices-llm-memory.md`](docs/research/best-practices-llm-memory.md)
- [`docs/research/literature-map.md`](docs/research/literature-map.md)
- [`docs/research/project-recommendations.md`](docs/research/project-recommendations.md)
- [`docs/research/evaluation-and-quality.md`](docs/research/evaluation-and-quality.md)

## Status

Docs/planning phase. No implementation code has been created yet.
