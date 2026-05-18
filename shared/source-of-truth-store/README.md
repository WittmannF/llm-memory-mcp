# Source-of-Truth Store

Common ingestion and access layer for all memory strategies in this monorepo.

## Goal

Provide a shared MCP/API service that stores and serves source-of-truth context data in an LLM-ready structure.

This layer should not decide the final memory strategy. It should only make evidence easy to ingest, inspect, search, and consume.

## Responsibilities

- Create/list contexts.
- Ingest raw sources into event or reference folders.
- Generate extracted Markdown derivatives.
- Maintain source/extracted mappings and hashes.
- Serve raw and extracted files to strategies.
- Provide timeline/list/search APIs over source evidence.
- Expose MCP tools for ingestion and consumption.

## Non-responsibilities

- It should not own `summary-context.md` for advanced strategies.
- It should not create graph/wiki/topic reports by default.
- It should not decide strategy-specific retrieval policy.
- It should not delete evidence automatically.

## Plan

See `docs/plans/2026-05-18-source-of-truth-store-plan.md`.
