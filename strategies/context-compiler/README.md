# Strategy: Context Compiler

This is the richer memory strategy originally planned for `llm-memory-mcp`.

It should now be treated as one strategy inside the monorepo, not the whole project.

## Goal

Consume the shared source-of-truth store and compile each context into durable memory artifacts:

- `summary-context.md`;
- `summary-context.html`;
- topic reports;
- optional wiki/topic pages;
- source-aware context packs;
- MCP tools for retrieval, reports, and summary refresh.

## Relationship to shared source-of-truth

This strategy should not own raw ingestion. It should consume evidence from:

```text
shared/source-of-truth-store
source-of-truth/contexts/<context>/events
source-of-truth/contexts/<context>/references
source-of-truth/contexts/<context>/facts
```

It owns the compiled artifacts.

## Plan

See:

```text
strategies/context-compiler/docs/plans/2026-05-18-context-compiler-strategy-plan.md
```
