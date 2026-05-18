# Claude/Cursor Agent Brainstorm

Date: 2026-05-18
Tool: Cursor Agent CLI (`agent`) with Claude Opus thinking model

## Prompt summary

Asked Claude to propose architecture for `llm-memory-mcp`: a global plug-and-play MCP for multiple reusable contexts, Markdown as source of truth, raw `sources/`, LLM-friendly `extracted/`, root `summary-context.md`, HTML inspection, topic reports, SQLite FTS initially, cheap/strong model tiers, MCP stdio + HTTP/FastAPI/UI, and Basic Memory inspiration without copying AGPL code.

## Key recommendations from Claude

- Keep Markdown as source of truth; treat DB and HTML as reconstructible artifacts.
- Implement contexts as self-contained folders.
- Use MCP stdio as primary integration; HTTP/FastAPI and UI as secondary interfaces.
- Use cheap models for extraction/transcription/partial summaries and stronger models for official synthesis.
- Use SQLite FTS5 first; add vector/semantic search later.
- Avoid copying Basic Memory code due AGPL; reuse ideas only.
- Prefer FastAPI + Jinja + HTMX for lightweight UI.
- Add a `build_context`-like tool that packages relevant context and follows wikilinks.
- Add jobs/status observability for ingestion and synthesis.
- Use Pydantic schemas shared between MCP and HTTP.
- Add tests at unit, integration, MCP contract, HTTP contract, and E2E levels.

## Decisions incorporated into the plan

- The repo will be Apache-2.0 by default unless changed later.
- MVP avoids vector DBs and knowledge graph complexity beyond wikilink parsing.
- MVP includes one root `summary-context.md` per context; not multiple competing summaries.
- The global server can manage many contexts, but each context remains simple and portable.
- `extracted/` is first-class, not just cache, because it is the LLM-readable version of hard sources.
