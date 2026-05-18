# LLM Memory MCP Implementation Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task.
>
> **Status:** Docs-only planning artifact. No implementation code has been created yet.

**Goal:** Build `llm-memory-mcp`, a global plug-and-play MCP server for reusable LLM memory contexts, where each context is a portable Markdown-first knowledge base with raw sources, LLM-friendly extracted sources, a root `summary-context.md`, HTML inspection, search, and durable topic reports.

**Architecture:** A local-first Python service with one domain/service layer exposed through both MCP tools and FastAPI/HTTP. Markdown files are the source of truth; SQLite FTS, manifests, and HTML are generated/reconstructible artifacts. The system manages multiple context folders globally, but each context has a single canonical `summary-context.md` at its root.

**Tech Stack:** Python 3.11+ or 3.12+, FastAPI, Python MCP SDK/FastMCP, Pydantic v2, SQLite FTS5, Jinja2, markdown-it-py, Typer, pytest, optional LiteLLM, optional MarkItDown/faster-whisper/vision OCR adapters.

---

## 1. Product definition

`llm-memory-mcp` is a global memory/context compiler for LLMs.

It should let Fernando and other LLM clients:

1. Create or register reusable contexts, e.g.:
   - `saude-pai`
   - `trabalho`
   - `imoveis`
   - `buddy`
2. Add sources in hard-to-read formats:
   - audio;
   - PDFs;
   - images/screenshots;
   - WhatsApp exports;
   - pasted text;
   - URLs;
   - arbitrary files.
3. Automatically create LLM-friendly Markdown derivatives in `extracted/`.
4. Maintain a canonical context summary at root:
   - `summary-context.md`
   - `summary-context.html`
5. Search and inspect all context material.
6. Generate durable topic reports under `reports/topics/`.
7. Expose all of this as MCP tools so any AI connected to the server can reuse/update the contexts.
8. Provide a human UI for inspection and navigation.

The project is not just a RAG system. It is a **context compiler**:

```text
raw sources -> extracted markdown -> searchable/indexed knowledge -> summary-context.md -> topic reports -> HTML inspection
```

---

## 2. Non-goals for the MVP

The MVP should avoid unnecessary complexity:

- No mandatory vector database.
- No Neo4j/knowledge-graph database.
- No required Obsidian dependency.
- No required cloud service.
- No heavy React/Next.js frontend initially.
- No autonomous unattended rewriting of sensitive summaries without explicit tool call.
- No copying Basic Memory code due AGPL license.
- No multiple competing root summaries per context.

Allowed later:

- semantic search;
- embeddings;
- local/remote LLM providers;
- richer graph view;
- Obsidian compatibility;
- background watcher;
- scheduled refresh.

---

## 3. Research base

The expanded research base lives in `docs/research/` and should be treated as part of the planning source of truth:

- `docs/research/README.md` — research index and high-level conclusions.
- `docs/research/best-practices-llm-memory.md` — practical best practices for memory tiers, provenance, staleness, extraction, and context packs.
- `docs/research/literature-map.md` — papers/systems including RAG, ReAct, Reflexion, Generative Agents, MemoryBank, MemGPT, Self-RAG, RAPTOR, GraphRAG, HippoRAG, LightRAG, Basic Memory, and LLM Wiki references.
- `docs/research/project-recommendations.md` — concrete design recommendations for this repo.
- `docs/research/evaluation-and-quality.md` — retrieval, provenance, staleness, contradiction, portability, and UI inspection checks.

### Basic Memory

Reference: `basicmachines-co/basic-memory`

Useful ideas:

- local-first Markdown;
- MCP-native memory tools;
- project/context routing;
- search/read/write/build-context primitives;
- wikilinks as relations;
- optional semantic search;
- UI/render experiments;
- strong test coverage around MCP behavior.

Important constraint:

- Basic Memory is AGPL-3.0-or-later.
- This repo should **not copy or link to Basic Memory implementation code** unless we intentionally adopt AGPL.
- Use it as architecture inspiration only.

See: `docs/research/basic-memory-review.md`

### Claude/Cursor Agent input

Claude agreed with:

- Markdown as source of truth;
- generated HTML as inspection artifact;
- `extracted/` as first-class LLM-readable source layer;
- cheap model for low-risk extraction and strong model for final synthesis;
- SQLite FTS first;
- MCP stdio + FastAPI HTTP/UI;
- Pydantic shared contracts;
- tests for MCP and HTTP contract parity.

See: `docs/research/claude-brainstorm.md`

---

## 4. Core design principles

### 4.1 Markdown is the source of truth

All durable knowledge should be in Markdown files that can be inspected, edited, committed, copied, or zipped.

Generated indexes and HTML are caches/build outputs.

### 4.2 Every difficult source gets an LLM-friendly Markdown derivative

For every file in `sources/`, the ingestion pipeline should create a corresponding Markdown file in `extracted/` whenever the source format is not LLM-friendly.

Examples:

```text
sources/audios/audio-medico-2026-05-18.ogg
extracted/audios/audio-medico-2026-05-18.md

sources/pdfs/exame-sangue-2026-05-18.pdf
extracted/pdfs/exame-sangue-2026-05-18.md

sources/images/print-whatsapp-2026-05-18.jpg
extracted/images/print-whatsapp-2026-05-18.md
```

The extracted Markdown should include:

- link/path to original source;
- extraction metadata;
- summary;
- detailed summary if appropriate;
- full transcript/text/OCR when available;
- low-confidence sections;
- timestamps for audio/video;
- source hash.

### 4.3 One canonical root summary per context

Each context has exactly one primary summary:

```text
<context>/summary-context.md
<context>/summary-context.html
```

This file is the first thing another LLM should read.

It may contain short and long sections, but there should not be competing files like:

```text
summary-context-short.md
summary-context-complete.md
summary-context-llm.md
```

Topic-specific deep dives belong in `reports/topics/`.

### 4.4 Global server, portable contexts

The server is global and plug-and-play. It can list and update many contexts.

Each context remains portable:

- it is just a folder;
- it contains its own sources, extracted files, summary, reports, and technical metadata;
- it can be versioned in Git independently if desired.

### 4.5 LLMs are used deliberately

Use cheaper models for:

- first-pass source summaries;
- transcription post-processing;
- tags;
- title/slug suggestions;
- topic suggestions;
- extraction normalization.

Use stronger models for:

- official `summary-context.md` refresh;
- medical/legal/strategic synthesis;
- contradiction resolution;
- durable topic reports.

Privacy is not the blocker for Fernando's current use case, but model choice should remain configurable per context.

---

## 5. Context folder layout

Recommended layout for each context:

```text
<context-slug>/
├── summary-context.md
├── summary-context.html
├── index.md
├── log.md
│
├── sources/
│   ├── audios/
│   ├── pdfs/
│   ├── images/
│   ├── whatsapp/
│   ├── urls/
│   └── other/
│
├── extracted/
│   ├── audios/
│   ├── pdfs/
│   ├── images/
│   ├── whatsapp/
│   ├── urls/
│   └── other/
│
├── wiki/
│   ├── linha-do-tempo.md
│   ├── perguntas.md
│   ├── decisoes.md
│   ├── incertezas.md
│   └── fontes-importantes.md
│
├── reports/
│   └── topics/
│       └── <topic-slug>.md
│
├── html/
│   ├── index.html
│   ├── sources.html
│   ├── timeline.html
│   └── reports/
│
└── .llm-memory/
    ├── manifest.json
    ├── index.sqlite
    ├── jobs.sqlite
    └── cache/
```

### Why `.llm-memory/` instead of `indexes/`

Use `.llm-memory/` for technical artifacts to make it clear that these files are implementation details:

- reconstructible;
- not usually read by humans;
- safe to rebuild;
- optionally ignored from Git.

However, `manifest.json` should be human-readable and may be committed if useful.

---

## 6. Global layout

Default global root:

```text
~/llm-memory/
```

On Fernando's Rock host, we may use:

```text
/home/rock/workspace/repos/llm-memory-data/
```

or a configurable path.

Global config:

```text
~/.config/llm-memory-mcp/config.toml
```

Example:

```toml
memory_root = "/home/rock/workspace/repos/llm-memory-data"
default_context = "saude-pai"
default_cheap_model = "gpt-4o-mini"
default_strong_model = "gpt-5.5"

[http]
host = "127.0.0.1"
port = 8765
auth = "none"

[mcp]
stdio = true
http = true
```

MVP can start without requiring config: if no config exists, use `~/llm-memory`.

---

## 7. Markdown conventions

### 7.1 Extracted source Markdown

Template:

```markdown
---
id: <ulid>
kind: extracted
source_path: sources/audios/audio-medico-2026-05-18.ogg
source_type: audio
source_sha256: <hash>
created_at: 2026-05-18T00:00:00Z
updated_at: 2026-05-18T00:00:00Z
extraction_method: whisper-api
summary_model: gpt-4o-mini
tags: [medico, audio]
confidence: medium
---

# Transcrição — audio-medico-2026-05-18

## Referência

- Original: `sources/audios/audio-medico-2026-05-18.ogg`
- Duração: 38min12s
- Qualidade: média

## Resumo curto

...

## Resumo detalhado

...

## Pontos importantes

- ...

## Linha do tempo do áudio

### 00:00–03:15

...

### 03:16–08:42

...

## Transcrição completa

...

## Trechos incertos

- 12:44 — possível menção a creatinina, áudio ruim.
```

### 7.2 Summary context

Template generic:

```markdown
---
id: <ulid>
kind: summary
context: saude-pai
created_at: 2026-05-18T00:00:00Z
updated_at: 2026-05-18T00:00:00Z
status: fresh
summary_model: gpt-5.5
source_count: 37
tags: [summary-context]
---

# Summary Context — Saúde do Pai

## Como usar este arquivo

Este é o contexto consolidado. Leia primeiro. Para detalhes, siga as referências em `extracted/`, `wiki/` e `reports/topics/`.

## TL;DR

...

## Estado atual

...

## Linha do tempo consolidada

...

## Pontos confirmados

...

## Incertezas e contradições

...

## Fontes principais

- `extracted/audios/audio-medico-2026-05-18.md`
  - Original: `sources/audios/audio-medico-2026-05-18.ogg`
  - Relevância: ...

## Relatórios relacionados

- `reports/topics/infeccao.md`
```

### 7.3 Topic reports

Template:

```markdown
---
id: <ulid>
kind: topic_report
topic: infeccao-antibioticos
query: "Tudo que sabemos sobre infecção e antibióticos"
created_at: 2026-05-18T00:00:00Z
updated_at: 2026-05-18T00:00:00Z
model: gpt-5.5
sources:
  - extracted/audios/audio-medico-2026-05-18.md
  - extracted/pdfs/exame-sangue-2026-05-18.md
tags: [infeccao, antibioticos]
---

# Relatório — Infecção e antibióticos

## Resposta curta

...

## Tudo que sabemos

...

## Evidências por fonte

...

## Linha do tempo relacionada

...

## Incertezas

...

## Perguntas abertas

...
```

---

## 8. Manifest design

File:

```text
<context>/.llm-memory/manifest.json
```

Purpose:

- track source hashes;
- link source -> extracted Markdown;
- track generated summaries/reports;
- mark summary as stale/fresh;
- record models used;
- support idempotent ingestion.

Example:

```json
{
  "schema_version": 1,
  "context": "saude-pai",
  "created_at": "2026-05-18T00:00:00Z",
  "updated_at": "2026-05-18T00:00:00Z",
  "summary_context": {
    "path": "summary-context.md",
    "html_path": "summary-context.html",
    "status": "stale",
    "last_generated_at": "2026-05-18T00:00:00Z",
    "new_sources_since_last_generation": 2
  },
  "sources": [
    {
      "id": "audio-medico-2026-05-18",
      "type": "audio",
      "original_path": "sources/audios/audio-medico-2026-05-18.ogg",
      "extracted_path": "extracted/audios/audio-medico-2026-05-18.md",
      "sha256": "...",
      "ingested_at": "2026-05-18T00:00:00Z",
      "extraction_status": "done",
      "extraction_method": "whisper-api",
      "summary_model": "gpt-4o-mini"
    }
  ],
  "reports": [
    {
      "id": "infeccao-antibioticos",
      "path": "reports/topics/infeccao-antibioticos.md",
      "query": "Tudo que sabemos sobre infecção e antibióticos",
      "created_at": "2026-05-18T00:00:00Z",
      "model": "gpt-5.5"
    }
  ]
}
```

---

## 9. SQLite index design

File:

```text
<context>/.llm-memory/index.sqlite
```

### MVP tables

```sql
CREATE TABLE documents (
  id TEXT PRIMARY KEY,
  path TEXT NOT NULL UNIQUE,
  kind TEXT NOT NULL,
  title TEXT,
  source_path TEXT,
  source_sha256 TEXT,
  frontmatter_json TEXT NOT NULL DEFAULT '{}',
  content_sha256 TEXT NOT NULL,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL
);

CREATE TABLE chunks (
  id TEXT PRIMARY KEY,
  document_id TEXT NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  ordinal INTEGER NOT NULL,
  heading_path TEXT,
  text TEXT NOT NULL,
  char_start INTEGER,
  char_end INTEGER,
  token_estimate INTEGER
);

CREATE VIRTUAL TABLE chunks_fts USING fts5(
  text,
  heading_path,
  document_path UNINDEXED,
  document_id UNINDEXED,
  chunk_id UNINDEXED,
  tokenize='unicode61 remove_diacritics 2'
);

CREATE TABLE links (
  id TEXT PRIMARY KEY,
  src_document_id TEXT NOT NULL,
  dst_ref TEXT NOT NULL,
  dst_document_id TEXT,
  link_type TEXT NOT NULL DEFAULT 'wikilink'
);

CREATE TABLE tags (
  document_id TEXT NOT NULL,
  tag TEXT NOT NULL,
  PRIMARY KEY (document_id, tag)
);
```

### Later semantic tables

Add later:

- `embeddings` table;
- `sqlite-vec` virtual table;
- embedding model version in metadata;
- hybrid search result fusion.

---

## 10. MCP tool contract

MCP tools should be atomic and composable.

### 10.1 Context tools

#### `list_contexts`

Purpose: list all configured/discovered contexts.

Returns:

```json
{
  "contexts": [
    {
      "name": "saude-pai",
      "path": "/home/rock/llm-memory/saude-pai",
      "summary_status": "fresh",
      "source_count": 12,
      "report_count": 3,
      "updated_at": "2026-05-18T00:00:00Z"
    }
  ]
}
```

#### `create_context`

Args:

```json
{
  "name": "saude-pai",
  "template": "medical_simple",
  "description": "Contexto médico e familiar"
}
```

Behavior:

- create folder layout;
- create initial `summary-context.md`;
- create initial `index.md` and `log.md`;
- initialize manifest and SQLite index.

#### `get_context_status`

Return stale/fresh status, recent sources, jobs, index stats.

---

### 10.2 Ingestion tools

#### `ingest_file`

Args:

```json
{
  "context": "saude-pai",
  "path": "/tmp/audio.ogg",
  "source_type": "audio",
  "tags": ["medico"]
}
```

Behavior:

1. hash file;
2. detect duplicates;
3. copy into `sources/<type>/`;
4. create or queue `extracted/<type>/<slug>.md`;
5. index extracted Markdown;
6. update manifest;
7. mark `summary-context.md` as stale.

#### `ingest_text`

For pasted text/WhatsApp snippets.

#### `ingest_url`

For web sources. MVP may save URL metadata and extracted Markdown using simple HTTP/HTML extraction.

#### `extract_now`

Force synchronous extraction for a source.

---

### 10.3 Read/write tools

#### `read_summary_context`

Args:

```json
{"context": "saude-pai"}
```

Returns Markdown content plus status metadata.

#### `write_summary_context`

Allows an LLM to update the canonical summary after synthesis.

Must:

- write atomically;
- update frontmatter timestamp;
- render `summary-context.html`;
- update manifest status.

#### `read_markdown`

Read arbitrary Markdown by relative path/ref.

#### `write_markdown`

Write arbitrary Markdown under allowed folders:

- `wiki/`
- `notes/`
- `reports/topics/`
- root `summary-context.md` only through explicit path or specialized tool.

Should reject writes to:

- `sources/`;
- `.llm-memory/index.sqlite`;
- `.llm-memory/jobs.sqlite`.

#### `read_extracted_source`

Convenience tool for LLMs to inspect source-derived Markdown without knowing exact paths.

---

### 10.4 Search/context tools

#### `search_context`

Args:

```json
{
  "context": "saude-pai",
  "query": "creatinina antibiótico função renal",
  "kind": ["summary", "extracted", "report", "wiki"],
  "limit": 10
}
```

Returns:

- document path;
- heading path;
- snippet;
- score;
- source original if applicable.

#### `build_context_pack`

This is the most important Basic-Memory-inspired tool.

Input:

```json
{
  "context": "saude-pai",
  "query": "tudo sobre infecção e antibióticos",
  "max_tokens": 12000,
  "include_summary": true,
  "include_sources": true,
  "depth": 1
}
```

Output:

- `summary-context.md` relevant sections;
- top search results;
- linked extracted sources;
- related topic reports;
- citations as relative paths and headings.

This tool packages evidence. It does not have to produce the final answer.

---

### 10.5 Report/synthesis tools

#### `refresh_summary_context`

Generates or prepares an update to `summary-context.md`.

Two operating modes:

1. `mode="draft"`: returns proposed Markdown, caller decides to write.
2. `mode="write"`: writes directly and updates HTML/manifest.

MVP can implement draft-only first.

#### `generate_topic_report`

Args:

```json
{
  "context": "saude-pai",
  "query": "tudo que sabemos sobre função renal e antibióticos",
  "depth": "detailed",
  "write": true
}
```

Writes:

```text
reports/topics/funcao-renal-antibioticos.md
```

Returns path, Markdown, HTML path, and sources used.

#### `list_topic_reports`

List durable reports.

#### `read_topic_report`

Read report by id/path.

---

### 10.6 HTML/render tools

#### `render_html`

Render:

- summary context;
- a specific Markdown file;
- all UI HTML pages.

#### `get_context_dashboard_data`

JSON for UI:

- counts;
- recent files;
- stale/fresh;
- broken links;
- latest reports;
- recent logs.

---

## 11. HTTP and UI design

### 11.1 FastAPI endpoints

Use the same service layer as MCP tools.

Core endpoints:

```text
GET  /health
GET  /api/v1/contexts
POST /api/v1/contexts
GET  /api/v1/contexts/{context}/status
GET  /api/v1/contexts/{context}/summary.md
GET  /api/v1/contexts/{context}/summary.html
POST /api/v1/contexts/{context}/sources
POST /api/v1/contexts/{context}/search
POST /api/v1/contexts/{context}/reports/topics
GET  /api/v1/contexts/{context}/reports/topics
GET  /api/v1/contexts/{context}/files/{path:path}
GET  /api/v1/contexts/{context}/render/{path:path}
```

MCP HTTP:

```text
/mcp
```

### 11.2 UI pages

Use Jinja2 + HTMX/static CSS for MVP.

Pages:

```text
/                         Context list
/c/{context}              Dashboard
/c/{context}/summary      Rendered summary-context.html
/c/{context}/sources      Sources + extracted files
/c/{context}/search       Search UI
/c/{context}/reports      Topic reports
/c/{context}/view/{path}  Render arbitrary Markdown
/c/{context}/log          log.md rendered
```

Dashboard should show:

- summary fresh/stale;
- last update;
- source count;
- latest sources;
- latest reports;
- search box;
- copy summary Markdown button;
- regenerate summary button;
- generate topic report form.

---

## 12. Repository implementation layout

```text
llm-memory-mcp/
├── pyproject.toml
├── README.md
├── LICENSE
├── docs/
│   ├── plans/
│   │   └── 2026-05-18-llm-memory-mcp-implementation-plan.md
│   ├── research/
│   │   ├── basic-memory-review.md
│   │   └── claude-brainstorm.md
│   ├── adr/
│   └── architecture.md
│
├── src/llm_memory_mcp/
│   ├── __init__.py
│   ├── config.py
│   ├── cli.py
│   ├── paths.py
│   ├── schemas.py
│   │
│   ├── contexts/
│   │   ├── service.py
│   │   ├── templates.py
│   │   └── registry.py
│   │
│   ├── storage/
│   │   ├── fs.py
│   │   ├── manifest.py
│   │   ├── sqlite.py
│   │   ├── migrations.py
│   │   └── fts.py
│   │
│   ├── markdown/
│   │   ├── frontmatter.py
│   │   ├── parser.py
│   │   ├── wikilinks.py
│   │   └── render.py
│   │
│   ├── ingest/
│   │   ├── service.py
│   │   ├── extractors.py
│   │   ├── text.py
│   │   ├── pdf.py
│   │   ├── audio.py
│   │   ├── image.py
│   │   └── url.py
│   │
│   ├── search/
│   │   ├── service.py
│   │   ├── fts.py
│   │   └── context_pack.py
│   │
│   ├── reports/
│   │   ├── summary.py
│   │   ├── topics.py
│   │   └── prompts.py
│   │
│   ├── llm/
│   │   ├── provider.py
│   │   ├── fake.py
│   │   └── litellm_adapter.py
│   │
│   ├── mcp/
│   │   ├── server.py
│   │   └── tools.py
│   │
│   └── http/
│       ├── app.py
│       ├── routes.py
│       ├── templates/
│       └── static/
│
└── tests/
    ├── unit/
    ├── integration/
    ├── contract/
    └── fixtures/
```

---

## 13. Implementation phases

## Phase 0 — Repository and docs foundation

**Objective:** Establish a clean, docs-first repo with decisions captured.

Deliverables:

- `README.md`
- `LICENSE`
- `docs/plans/...`
- `docs/research/...`
- `docs/adr/0001-license.md`
- `docs/architecture.md` high-level overview

Acceptance:

- repo exists locally and on GitHub under `WittmannF/llm-memory-mcp`;
- plan committed and pushed;
- no implementation scaffolding beyond docs unless explicitly approved.

## Phase 1 — Python project scaffold

**Objective:** Add installable package and test/lint infrastructure.

Files:

- `pyproject.toml`
- `src/llm_memory_mcp/__init__.py`
- `src/llm_memory_mcp/cli.py`
- `tests/test_import.py`
- `.gitignore`
- `.github/workflows/test.yml`

Dependencies initially:

- `pydantic`
- `pydantic-settings`
- `typer`
- `pytest`
- `ruff`

Defer heavier deps until needed.

Verification:

```bash
uv sync
uv run pytest -q
uv run ruff check .
```

## Phase 2 — Context filesystem core

**Objective:** Create/list contexts and enforce folder structure.

Implement:

- context slug validation;
- default root resolution;
- folder creation;
- initial root Markdown files;
- manifest creation;
- `log.md` append helper.

CLI:

```bash
llm-memory context create saude-pai --template medical_simple
llm-memory context list
llm-memory context status saude-pai
```

Tests:

- creates expected directories;
- initializes `summary-context.md` at root;
- initializes `index.md` and `log.md`;
- rejects path traversal;
- idempotent create does not overwrite existing content unless `--force`.

## Phase 3 — Markdown parser/render core

**Objective:** Parse frontmatter, headings, chunks, wikilinks, and render HTML.

Implement:

- frontmatter extraction/update;
- heading path extraction;
- chunking by headings;
- wikilink extraction;
- Markdown -> HTML render;
- root summary HTML generation.

Tests:

- round-trip frontmatter;
- wikilinks from prose/list items;
- chunk heading paths;
- HTML contains safe rendered content;
- script tags escaped/sanitized or documented as trusted-local only.

## Phase 4 — SQLite FTS index

**Objective:** Build searchable index from Markdown files.

Implement:

- SQLite migration setup;
- document/chunk tables;
- FTS5 table;
- reindex command;
- search service.

CLI:

```bash
llm-memory reindex saude-pai
llm-memory search saude-pai "antibiótico creatinina"
```

Tests:

- indexes summary/wiki/extracted/reports;
- search returns path + heading + snippet;
- deleted files removed on reindex;
- handles Portuguese accents via FTS tokenizer.

## Phase 5 — MCP stdio MVP

**Objective:** Expose the core as MCP tools.

Tools:

- `list_contexts`
- `create_context`
- `get_context_status`
- `read_summary_context`
- `read_markdown`
- `write_markdown`
- `search_context`
- `build_context_pack`

Verification:

- MCP server starts over stdio;
- tools list successfully;
- at least one contract test calls `search_context` on fixture context;
- Hermes/Cursor/Claude config snippets documented.

## Phase 6 — FastAPI + UI MVP

**Objective:** Provide human inspection UI and HTTP API.

Implement:

- `/health`;
- `/api/v1/contexts`;
- `/api/v1/contexts/{context}/summary.md`;
- `/api/v1/contexts/{context}/search`;
- dashboard UI;
- summary viewer;
- source/extracted file viewer;
- rendered Markdown view.

Tests:

- HTTP status;
- JSON response schemas;
- rendered summary page includes expected text;
- UI does not require JS for basic navigation.

## Phase 7 — Ingestion MVP

**Objective:** Copy sources and create extracted Markdown.

Start with:

- text;
- Markdown;
- PDF via MarkItDown or pypdf;
- audio placeholder/fake provider in tests;
- image placeholder/fake provider in tests.

Implement provider abstraction before real providers:

```python
class ExtractionProvider:
    def extract_audio(...): ...
    def extract_pdf(...): ...
    def extract_image(...): ...
```

Tools:

- `ingest_file`
- `ingest_text`
- `read_extracted_source`

Tests:

- source copied to right folder;
- duplicate hash skipped;
- extracted Markdown created;
- summary marked stale;
- extracted file indexed.

## Phase 8 — LLM provider and synthesis

**Objective:** Generate summary drafts and topic reports.

Implement:

- fake deterministic provider for tests;
- optional LiteLLM adapter;
- prompt templates;
- `refresh_summary_context(mode="draft"|"write")`;
- `generate_topic_report(write=true|false)`.

Tests:

- fake provider produces deterministic summary;
- generated report saved under `reports/topics/`;
- paths and sources listed in frontmatter;
- `summary-context.html` generated after write.

## Phase 9 — HTML inspection polish

**Objective:** Make generated HTML useful for human inspection.

Features:

- table of contents;
- source cards;
- backlinks;
- stale/fresh badge;
- copy Markdown button;
- report list;
- search form;
- collapsible extracted sections.

## Phase 10 — Optional semantic search

**Objective:** Add embeddings only after FTS is useful.

Options:

- local `fastembed`/BGE;
- OpenAI embeddings;
- `sqlite-vec`;
- hybrid search via reciprocal rank fusion.

Acceptance:

- feature flag; not required for MVP;
- tests skip gracefully if dependencies missing;
- result output labels search mode.

---

## 14. Security and safety

Even if privacy is not a blocker, protect against accidental footguns:

- Reject path traversal in all paths.
- Never allow writes to files outside context root.
- Treat `sources/` as immutable after ingest unless explicit replace/delete tool.
- Avoid serving arbitrary local files through HTTP.
- Default HTTP binding to `127.0.0.1`.
- If binding to LAN, require optional token auth later.
- Log model/provider used for generated content.
- Mark low-confidence extraction sections clearly.

---

## 15. Git/versioning recommendations

Each context can optionally be its own Git repo, but the MCP should not require it.

For this project repo:

- implementation code versioned normally;
- sample contexts in `examples/contexts/` should use tiny synthetic fixtures only;
- never commit real private context data to this repo.

For user contexts:

- allow `llm-memory context git-init <context>` later;
- add optional auto-commit on writes later;
- avoid auto-push by default.

---

## 16. Open architecture decisions

### ADR-0001 — License

Recommendation: Apache-2.0.

Reason:

- permissive;
- enterprise-friendly;
- avoids AGPL contamination concern;
- compatible with most dependencies.

### ADR-0002 — MCP SDK

Need verify current Python MCP SDK/FastMCP APIs before implementation.

Decision likely:

- use official MCP SDK/FastMCP if stable enough;
- expose stdio first;
- add HTTP MCP through FastAPI only after tested.

### ADR-0003 — MarkItDown vs custom extractors

Recommendation:

- start with simple text/Markdown/PDF extraction;
- add MarkItDown as optional dependency if it works well on Rock host;
- keep extractor interface so we can swap.

### ADR-0004 — Config format

Recommendation:

- global TOML under XDG config;
- no required `context.yaml` in MVP;
- use `.llm-memory/manifest.json` for operational metadata.

If per-context model overrides are needed, add `.llm-memory/config.toml` later.

---

## 17. Detailed first implementation tasks

The tasks below assume we proceed beyond this docs-only plan.

### Task 1: Add license and ADR skeleton

**Objective:** Establish legal and architecture decision baseline.

**Files:**

- Create: `LICENSE`
- Create: `docs/adr/0001-license.md`
- Modify: `README.md`

**Steps:**

1. Add Apache-2.0 license.
2. Write ADR explaining why Basic Memory is reference-only due AGPL.
3. Update README with status: planning/MVP not implemented.
4. Commit: `docs: add license decision`

### Task 2: Add Python scaffold

**Objective:** Make repo installable/testable.

**Files:**

- Create: `pyproject.toml`
- Create: `src/llm_memory_mcp/__init__.py`
- Create: `tests/test_import.py`
- Create: `.gitignore`

**Steps:**

1. Add minimal dependencies.
2. Add import test.
3. Run `uv sync`.
4. Run `uv run pytest -q`.
5. Run `uv run ruff check .`.
6. Commit: `chore: scaffold python package`

### Task 3: Implement context create/list

**Objective:** Create the filesystem core.

**Files:**

- Create: `src/llm_memory_mcp/config.py`
- Create: `src/llm_memory_mcp/paths.py`
- Create: `src/llm_memory_mcp/contexts/service.py`
- Create: `tests/unit/test_contexts.py`

**TDD cases:**

- `create_context` creates folder layout.
- It writes `summary-context.md` at root.
- It writes `.llm-memory/manifest.json`.
- It rejects `../evil` slug.
- It is idempotent.

### Task 4: Add Markdown parser and HTML renderer

**Objective:** Render summaries and extract chunks.

**Files:**

- Create: `src/llm_memory_mcp/markdown/frontmatter.py`
- Create: `src/llm_memory_mcp/markdown/parser.py`
- Create: `src/llm_memory_mcp/markdown/render.py`
- Create: `tests/unit/test_markdown.py`

**TDD cases:**

- frontmatter parsed;
- headings extracted;
- chunks preserve heading path;
- HTML includes rendered headings;
- wikilinks detected.

### Task 5: Add SQLite FTS

**Objective:** Search Markdown documents.

**Files:**

- Create: `src/llm_memory_mcp/storage/sqlite.py`
- Create: `src/llm_memory_mcp/storage/fts.py`
- Create: `src/llm_memory_mcp/search/service.py`
- Create: `tests/integration/test_fts.py`

**TDD cases:**

- indexes fixture context;
- search returns expected file;
- reindex removes deleted content;
- accented Portuguese search works acceptably.

### Task 6: Add MCP tools

**Objective:** Connect LLM clients.

**Files:**

- Create: `src/llm_memory_mcp/mcp/server.py`
- Create: `src/llm_memory_mcp/mcp/tools.py`
- Create: `tests/contract/test_mcp_tools.py`

**TDD cases:**

- MCP server lists tools;
- `list_contexts` works;
- `read_summary_context` works;
- `search_context` works on fixture.

### Task 7: Add FastAPI/UI

**Objective:** Human inspection.

**Files:**

- Create: `src/llm_memory_mcp/http/app.py`
- Create: `src/llm_memory_mcp/http/routes.py`
- Create: `src/llm_memory_mcp/http/templates/*.html`
- Create: `tests/contract/test_http.py`

**TDD cases:**

- `/health` returns 200;
- context list endpoint works;
- summary page renders;
- search API returns schema.

### Task 8: Add ingestion/extracted layer

**Objective:** Store source and extracted Markdown.

**Files:**

- Create: `src/llm_memory_mcp/ingest/service.py`
- Create: `src/llm_memory_mcp/ingest/extractors.py`
- Create: `tests/integration/test_ingest.py`

**TDD cases:**

- ingested text writes source and extracted Markdown;
- duplicate hash skipped;
- extracted file indexed;
- summary marked stale.

### Task 9: Add LLM provider abstraction and report generation

**Objective:** Generate durable summaries/reports.

**Files:**

- Create: `src/llm_memory_mcp/llm/provider.py`
- Create: `src/llm_memory_mcp/llm/fake.py`
- Create: `src/llm_memory_mcp/reports/summary.py`
- Create: `src/llm_memory_mcp/reports/topics.py`
- Create: `tests/integration/test_reports.py`

**TDD cases:**

- fake provider generates deterministic summary;
- topic report saved under `reports/topics/`;
- HTML rendered for generated report;
- manifest updated.

---

## 18. Verification checklist for MVP

Before calling the MVP usable:

- [ ] Can create a context.
- [ ] Can ingest text file.
- [ ] Can ingest at least one PDF into `extracted/pdfs/*.md`.
- [ ] Can read `summary-context.md` via MCP.
- [ ] Can search context via MCP.
- [ ] Can generate or draft a topic report.
- [ ] Can render summary HTML.
- [ ] UI can browse summary, sources, extracted files, and reports.
- [ ] `pytest` passes.
- [ ] MCP contract smoke test passes.
- [ ] README explains configuration for Hermes/Cursor/Claude/Codex.

---

## 19. Recommended immediate next step

After this plan is reviewed:

1. Add license/ADR.
2. Scaffold Python package.
3. Implement context filesystem core.
4. Validate with a tiny synthetic `examples/contexts/saude-pai-demo`.
5. Only then add MCP tools.

This keeps the project grounded in files and avoids building an MCP facade before the context model is stable.
