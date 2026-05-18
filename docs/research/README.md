# Research Index

This directory captures the research base for `llm-memory-mcp`: prior art, literature, best practices, and design recommendations.

The goal is to make sure this project is not just a convenient MCP wrapper, but a memory system grounded in what works for long-lived LLM agents and knowledge bases.

## Start here

1. [`best-practices-llm-memory.md`](best-practices-llm-memory.md) — practical best practices for LLM memory management.
2. [`literature-map.md`](literature-map.md) — papers and systems that inform the architecture.
3. [`project-recommendations.md`](project-recommendations.md) — concrete recommendations for `llm-memory-mcp` based on the research.
4. [`evaluation-and-quality.md`](evaluation-and-quality.md) — how to test whether memory is actually working.

## Existing reference notes

- [`basic-memory-review.md`](basic-memory-review.md) — what to reuse conceptually from Basic Memory and what to avoid due AGPL.
- [`claude-brainstorm.md`](claude-brainstorm.md) — brainstorming output from Claude via Cursor Agent CLI.

## High-level conclusion

The strongest architecture for this project is a **context compiler**, not a pure RAG app:

```text
raw sources
  -> LLM-friendly extracted Markdown
  -> indexed evidence and links
  -> durable summaries/reflections/topic reports
  -> one canonical summary-context.md
  -> MCP tools that package the right context for any LLM client
```

Key design implications:

- Keep Markdown as the durable source of truth.
- Treat SQLite FTS, embeddings, and HTML as rebuildable indexes/artifacts.
- Preserve provenance from every generated claim back to extracted/source files.
- Use a memory hierarchy: working context, episodic source history, semantic summaries, procedural/project rules.
- Build context packs on demand instead of blindly dumping all files into an LLM.
- Use cheap models for extraction and classification, but stronger models for canonical summary updates.
- Add evaluation early: retrieval quality, provenance accuracy, staleness detection, and answer faithfulness.
