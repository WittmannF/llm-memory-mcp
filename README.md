# llm-memory-mcp

Global plug-and-play MCP for reusable LLM memory contexts.

## Status

Planning phase. No implementation code yet.

## Vision

`llm-memory-mcp` will manage portable Markdown-first memory contexts for LLMs:

- raw files in `sources/`;
- LLM-friendly source derivatives in `extracted/`;
- one canonical `summary-context.md` at each context root;
- HTML generated for human inspection;
- durable topic reports in `reports/topics/`;
- SQLite FTS indexes as reconstructible cache;
- MCP tools so Claude, Cursor, Codex, Hermes, and other clients can ingest, search, inspect, and update contexts.

## Current plan

See:

- [`docs/plans/2026-05-18-llm-memory-mcp-implementation-plan.md`](docs/plans/2026-05-18-llm-memory-mcp-implementation-plan.md)

## Research base

Start with:

- [`docs/research/README.md`](docs/research/README.md) — research index and high-level conclusions.
- [`docs/research/best-practices-llm-memory.md`](docs/research/best-practices-llm-memory.md) — best practices for LLM memory management.
- [`docs/research/literature-map.md`](docs/research/literature-map.md) — papers and systems that inform the design.
- [`docs/research/project-recommendations.md`](docs/research/project-recommendations.md) — recommendations translated into project decisions.
- [`docs/research/evaluation-and-quality.md`](docs/research/evaluation-and-quality.md) — evaluation plan for memory quality.
- [`docs/research/basic-memory-review.md`](docs/research/basic-memory-review.md) — Basic Memory review and license boundary.
- [`docs/research/claude-brainstorm.md`](docs/research/claude-brainstorm.md) — Claude/Cursor Agent brainstorm.

## Reference inspiration

The architecture is inspired by Karpathy's LLM Wiki pattern and by the product ideas in Basic Memory. Basic Memory is AGPL, so this project should reuse concepts only, not implementation code.
