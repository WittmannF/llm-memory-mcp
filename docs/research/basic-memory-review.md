# Basic Memory Review for `llm-memory-mcp`

Reviewed: 2026-05-18
Reference: https://github.com/basicmachines-co/basic-memory

## Why it matters

Basic Memory is the closest mature reference for a local-first Markdown memory exposed through MCP. It validates several product assumptions for `llm-memory-mcp`:

- Markdown files can remain the durable source of truth.
- MCP is a good interoperability layer across Claude, Codex, Cursor, ChatGPT-compatible clients, and IDEs.
- Context/project routing is necessary once multiple memory roots exist.
- Search/read/write/build-context tools are a useful mental model for LLM clients.
- A UI/view layer can coexist with a filesystem-first architecture.

## License caution

Basic Memory is AGPL-3.0-or-later. We should **not copy code** or link/import implementation modules unless we intentionally adopt AGPL obligations. Treat it as an architectural reference only.

Recommended license for this repo: Apache-2.0 or MIT. Apache-2.0 is preferable if this becomes a reusable infra project.

## Useful ideas to reuse conceptually

### 1. Local-first Markdown

Basic Memory's README emphasizes that notes are plain Markdown on disk and remain readable/editable outside the app. We should keep the same principle:

- `summary-context.md` is the canonical LLM entry point per context.
- `sources/` preserves raw originals.
- `extracted/` contains LLM-friendly Markdown versions of hard-to-read sources.
- `wiki/`, `notes/`, or `topics/` hold human/agent-authored structured knowledge.
- `reports/topics/` holds durable query-specific reports.
- DB and HTML are reconstructible cache/build artifacts.

### 2. MCP-native tools

Basic Memory exposes tools like:

- `search_notes`
- `read_note`
- `write_note`
- `edit_note`
- `build_context`
- `recent_activity`
- project-management tools

For `llm-memory-mcp`, comparable tools should exist, but tuned to context-compilation:

- `list_contexts`
- `create_context`
- `read_summary_context`
- `refresh_summary_context`
- `search_context`
- `read_markdown`
- `write_markdown`
- `ingest_source`
- `read_extracted_source`
- `generate_topic_report`
- `build_context_pack`

### 3. Project/context routing

Basic Memory has project/workspace routing. We need a simpler global-context registry:

- Global root: configurable, e.g. `~/llm-memory`.
- Contexts: subdirectories under root.
- Registry: `contexts.json` or auto-discovery by folders containing `summary-context.md`/`.llm-memory/manifest.json`.
- MCP tools accept `context` argument; if omitted, use configured default.

### 4. Build-context pattern

Basic Memory's `build_context` can return compact Markdown or JSON built from graph relations. We should implement a simpler version:

- Input can be a query, file path, report id, extracted source id, or wikilink.
- It searches/indexes relevant Markdown, follows local links, and returns a bounded context pack.
- It should cite relative paths and headings.
- It should not silently invent facts; it packages evidence for the caller LLM.

### 5. Semantic search as optional enhancement

Basic Memory includes semantic-search components (`fastembed`, `sqlite-vec`, OpenAI options). For MVP we should start with SQLite FTS5, then optionally add:

- local embeddings;
- sqlite-vec;
- hybrid search with reciprocal rank fusion.

### 6. UI render layer

Basic Memory has MCP UI experiments/templates. For our project, UI should be deliberately simpler:

- FastAPI + Jinja2 + HTMX/static CSS.
- Read-mostly inspection dashboard.
- Render Markdown to HTML.
- Show backlinks, sources, extracted files, stale/fresh status, and topic reports.

## What is different in `llm-memory-mcp`

Basic Memory is closer to general AI memory / note graph. `llm-memory-mcp` should focus on **context compilation**:

- Every difficult source creates a Markdown derivative in `extracted/`.
- `summary-context.md` is always at context root and is the primary artifact for LLMs.
- Topic reports are durable generated artifacts, not ephemeral answers.
- HTML is generated for human inspection, not the source of truth.
- The pipeline explicitly uses cheap models for extraction/initial summaries and stronger models for official synthesis.
- The MCP should be global and plug-and-play, allowing any connected AI to reuse/update selected contexts.

## Avoid copying

Do not copy:

- Basic Memory's implementation of MCP tools.
- Basic Memory's SQLAlchemy models or repository/service code.
- Basic Memory's parser implementation.
- Basic Memory's UI template code.

Safe to copy conceptually:

- tool categories;
- naming inspiration;
- local-first principles;
- project routing idea;
- Markdown + wikilinks + graph concept.
