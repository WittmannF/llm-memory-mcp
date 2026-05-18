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
- [`docs/research/basic-memory-review.md`](docs/research/basic-memory-review.md)
- [`docs/research/claude-brainstorm.md`](docs/research/claude-brainstorm.md)

## Reference inspiration

The architecture is inspired by Karpathy's LLM Wiki pattern and by the product ideas in Basic Memory. Basic Memory is AGPL, so this project should reuse concepts only, not implementation code.
