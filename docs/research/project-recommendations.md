# Project Recommendations Based on Research

This document translates the research notes into concrete recommendations for `llm-memory-mcp`.

## Recommendation 1: Build a context compiler, not a RAG chatbot

Research basis:

- RAG improves factuality but struggles with global corpus questions.
- GraphRAG/RAPTOR show that intermediate summaries help answer broad questions.
- LLM Wiki patterns show value in compiling knowledge into durable Markdown.

Decision:

- The core product is a compiler from sources to durable context artifacts.
- Retrieval is a support function, not the product.

Architecture:

```text
sources/ -> extracted/ -> wiki + reports/topics/ -> summary-context.md -> build_context_pack
```

## Recommendation 2: Treat `extracted/` as first-class memory

Research basis:

- Episodic memory matters for long-term agents.
- Provenance and source inspection are necessary for trust.
- Hard sources are expensive for LLMs to inspect repeatedly.

Decision:

- Every PDF/audio/image/URL should have a durable Markdown derivative in `extracted/`.
- Extracted files are not temporary cache; they are canonical LLM-readable evidence.

Implementation implication:

- Manifest must track source -> extracted mapping.
- Search must index `extracted/` by default.
- `summary-context.md` should cite extracted files.
- MCP should expose `read_extracted_source`.

## Recommendation 3: Make `summary-context.md` the canonical root file

Research basis:

- MemGPT-style systems need a compact loaded memory tier.
- Hierarchical summarization needs a top-level abstraction.
- Other LLM clients need a single obvious entry point.

Decision:

- Each context has exactly one root `summary-context.md`.
- It is the first file for LLMs to read.
- It includes references to sources, extracted docs, reports, and open questions.

Implementation implication:

- Context creation should always initialize this file.
- UI should render `summary-context.html` from this file.
- MCP should expose `read_summary_context`.
- Summary staleness should be shown in status and UI.

## Recommendation 4: Add `build_context_pack` as the key MCP abstraction

Research basis:

- MemGPT frames context as memory pages loaded into a limited window.
- ReAct and MCP-style tool use need composable evidence-gathering actions.
- Self-RAG argues retrieval should be adaptive and critique-aware.

Decision:

Implement:

```text
build_context_pack(context, query, max_tokens, include_summary, include_sources, depth)
```

It should return:

- relevant parts of `summary-context.md`;
- relevant extracted sections;
- relevant topic reports;
- citations and paths;
- omitted/overflow note if budget was exceeded.

This tool should be preferred over raw `search_context` for complex tasks.

## Recommendation 5: Start with FTS5, then add hybrid retrieval later

Research basis:

- Basic search is often enough for file inspection and provenance.
- Vector search is useful but can hide exact evidence and increase complexity.
- GraphRAG/LightRAG show graph/hybrid retrieval can help later.

Decision:

MVP:

- SQLite FTS5 over all Markdown.
- Heading-aware chunks.
- Path/kind filters.
- Accent-insensitive tokenization where possible.

Later:

- embeddings;
- sqlite-vec;
- hybrid FTS + vector ranking;
- graph/wikilink boost;
- recency/importance boost.

## Recommendation 6: Use memory ranking signals beyond similarity

Research basis:

- Generative Agents uses recency, importance, and relevance.
- MemoryBank uses forgetting/reinforcement ideas.
- Real contexts often have time-sensitive information.

Decision:

Ranking should eventually combine:

- keyword relevance;
- semantic relevance;
- recency;
- source importance;
- document kind;
- link distance;
- manual pinning;
- whether the source is cited in `summary-context.md`.

MVP can store these fields even if ranking initially uses only FTS.

## Recommendation 7: Make reflection durable

Research basis:

- Reflexion stores verbal feedback/reflections.
- Generative Agents periodically reflect into higher-level memories.

Decision:

Reflections should become Markdown artifacts:

- topic reports;
- wiki pages;
- summary updates;
- explicit open questions;
- contradiction notes.

Avoid hidden background memory that cannot be inspected.

## Recommendation 8: Separate cheap extraction from strong synthesis

Research basis:

- Extraction/classification tasks are lower risk.
- Official summaries and contradiction resolution are higher risk.

Decision:

Use cheap models for:

- source-level summaries;
- tags;
- transcript cleanup;
- title/slug generation;
- topic suggestions.

Use stronger models for:

- `summary-context.md` refresh;
- medical/financial/legal synthesis;
- topic reports;
- contradiction resolution.

Implementation implication:

- LLM provider interface should expose roles like `cheap`, `strong`, `transcribe`, `vision`.
- Generated Markdown frontmatter should record model names.

## Recommendation 9: Add staleness and change tracking from day one

Research basis:

- Memory is dangerous when stale information looks current.
- Durable summaries are only useful when users know whether they reflect latest sources.

Decision:

Manifest should track:

- last source ingest;
- last extraction;
- last summary refresh;
- number of new sources since summary;
- failed jobs;
- stale/fresh status.

UI should prominently show:

```text
summary-context.md: stale — 3 new sources since last refresh
```

## Recommendation 10: Keep knowledge graph lightweight initially

Research basis:

- GraphRAG/HippoRAG show graph structures help multi-hop/global QA.
- Heavy graph infrastructure is not needed for MVP.

Decision:

MVP graph:

- parse `[[wikilinks]]`;
- parse source/report references;
- store edges in SQLite;
- expose backlinks/neighbors.

Avoid:

- Neo4j;
- complex entity resolution;
- automatic ontology building.

Later:

- source-topic graph;
- entity extraction;
- community summaries;
- graph-assisted retrieval.

## Recommendation 11: Build an inspection UI before advanced retrieval

Research basis:

- Memory systems fail silently if users cannot inspect evidence and generated summaries.
- Human trust depends on provenance and visibility.

Decision:

The UI should prioritize:

- summary viewer;
- sources/extracted viewer;
- search;
- topic reports;
- stale/fresh status;
- generated HTML;
- backlinks and source references.

Do not overbuild editing UX initially. Read-mostly inspection is enough for MVP.

## Recommendation 12: Define update policies per artifact type

Recommended policies:

### `sources/`

- immutable;
- never edited by LLM;
- replace only through explicit tool.

### `extracted/`

- regenerated when source hash or extraction model changes;
- may include manual correction note;
- should preserve full extracted content when possible.

### `summary-context.md`

- updated by explicit refresh/write tool;
- model and source set recorded;
- stale status tracked.

### `reports/topics/`

- append/create by query;
- stable snapshot of what was known at generation time;
- can be superseded but not silently overwritten unless explicit.

### `.llm-memory/`

- technical cache;
- rebuildable;
- not trusted as only source.

## Recommendation 13: Include evaluation fixtures early

Research basis:

- RAG/memory quality cannot be judged by unit tests alone.
- Faithfulness, retrieval, and staleness need scenario tests.

Decision:

Create synthetic fixtures:

```text
examples/contexts/medical-demo/
examples/contexts/work-demo/
```

Each fixture should include:

- a few sources;
- extracted Markdown;
- expected summary facts;
- queries with expected supporting sources;
- one contradiction;
- one stale-summary scenario.

Use these for regression tests.

## Recommendation 14: Make cross-client instructions explicit

Research basis:

- MCP clients differ in how much they infer.
- A global memory MCP must guide arbitrary LLM clients.

Decision:

Provide:

- MCP tool descriptions with clear usage guidance;
- prompt resources like `how_to_use_context`;
- README snippets for Hermes, Claude Desktop, Cursor, Codex;
- examples: "read summary first, then search, then read extracted source".

## Recommendation 15: Keep legal boundaries clear

Research basis:

- Basic Memory is a useful prior art but AGPL.

Decision:

- Do not copy implementation code.
- Do not vendor Basic Memory.
- If comparing behavior, write clean-room tests/specs.
- Use permissive dependencies where possible.
- Keep `basic-memory-review.md` as an explicit boundary note.

## Recommended MVP priorities after the plan

Order:

1. Context folder creation and manifest.
2. Markdown parser/render and HTML generation.
3. SQLite FTS over Markdown.
4. MCP tools for list/read/search/build context pack.
5. FastAPI/UI inspection.
6. Ingestion to `sources/` + `extracted/`.
7. LLM provider + summary/report generation.
8. Evaluation fixtures.
9. Semantic/hybrid retrieval.

Why this order:

- It validates the storage model before expensive LLM integrations.
- It gives usable inspection early.
- It avoids MCP tools becoming thin wrappers around unstable internals.
- It enables tests before adding nondeterministic model calls.
