# Source-of-Truth Store MCP Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task.
>
> **Status:** Docs-only planning artifact. No implementation code has been created yet.

**Goal:** Build a shared source-of-truth ingestion/access layer that all memory strategies can consume.

**Architecture:** A local-first MCP/FastAPI service over a folder of context data. It preserves raw files and generates LLM-ready Markdown derivatives. Strategy packages consume this service rather than implementing their own ingestion and evidence storage.

**Tech Stack:** Python, FastAPI, MCP stdio/HTTP, Pydantic, Markdown, SQLite FTS optional for evidence search, pluggable extractors for audio/PDF/image/URL/text.

---

## 1. Problem

If each memory strategy implements its own ingestion/storage, strategies become impossible to compare fairly.

The shared layer should define one stable contract:

```text
context -> event/reference -> raw source -> extracted Markdown derivative
```

Then strategies can compete on summarization, retrieval, indexing, and context packing.

## 2. Data model

### Context

A named project/domain:

- `saude-pai`
- `trabalho`
- `imoveis`
- `buddy`

Each context is isolated.

### Event

A dated thing that happened.

Examples:

- consultation;
- exam result;
- meeting;
- phone call;
- message batch;
- decision event.

Path:

```text
contexts/<context>/events/YYYY/MM/DD/<event-slug>/
```

### Reference

A stable/atemporal artifact.

Examples:

- ID document;
- baseline document;
- contract;
- project brief;
- person profile;
- stable property reference.

Path:

```text
contexts/<context>/references/<reference-slug>/
```

### Fact

Optional curated stable information.

Facts should be small, provenance-linked, and not used as a dumping ground for summaries.

## 3. File layout

### Event folder

```text
events/YYYY/MM/DD/<event-slug>/
├── event.md
├── raw/
│   ├── audio.ogg
│   └── exam.pdf
└── extracted/
    ├── audio.md
    └── exam.md
```

### Reference folder

```text
references/<reference-slug>/
├── reference.md
├── raw/
└── extracted/
```

### Metadata file

`event.md` or `reference.md` should contain frontmatter:

```yaml
---
id: <ulid>
kind: event
context: saude-pai
date: 2026-05-18
title: Medical consultation
tags: [medical, consultation]
source_count: 2
created_at: 2026-05-18T00:00:00Z
updated_at: 2026-05-18T00:00:00Z
---
```

## 4. Extracted Markdown requirements

Each extracted file should include:

- pointer to original raw file;
- source hash;
- extraction method;
- model used, if any;
- confidence;
- short summary;
- detailed summary;
- full transcript/OCR/text where possible;
- timestamps for audio/video;
- unclear segments.

## 5. Image policy

Images are ambiguous because multimodal LLMs can inspect them directly, but image tokens are more expensive.

Support per-context or per-source policy:

- `markdown_only`: OCR/description only.
- `image_plus_markdown`: expose both raw image and extracted Markdown.
- `image_on_demand`: use Markdown by default; allow strategies to request raw image.

Default: `image_on_demand`.

## 6. MCP tools

### Context tools

- `list_contexts`
- `create_context`
- `get_context_manifest`

### Ingestion tools

- `create_event`
- `create_reference`
- `ingest_file`
- `ingest_text`
- `ingest_url`
- `extract_source`

### Read/search tools

- `list_events`
- `list_references`
- `read_event`
- `read_reference`
- `read_extracted_source`
- `search_evidence`
- `get_timeline`

### Maintenance tools

- `rebuild_evidence_index`
- `check_source_integrity`
- `list_unextracted_sources`

## 7. HTTP/UI

FastAPI should expose the same domain service:

- `GET /contexts`
- `POST /contexts`
- `GET /contexts/{context}/events`
- `POST /contexts/{context}/events`
- `POST /contexts/{context}/ingest`
- `GET /contexts/{context}/timeline`
- `POST /contexts/{context}/search`
- `GET /contexts/{context}/files/{path:path}`

UI should be read-mostly:

- context list;
- event timeline;
- reference list;
- source/extracted pair viewer;
- extraction status;
- search.

## 8. Implementation phases

### Phase 1 — Docs and schemas

- Define JSON/Markdown schemas.
- Create fixtures.
- Add validation rules.

### Phase 2 — Filesystem service

- Create contexts/events/references.
- Copy raw files.
- Write metadata.
- Compute hashes.

### Phase 3 — Extractors

- Text and Markdown first.
- PDF second.
- Audio and image via pluggable provider.

### Phase 4 — MCP tools

- Expose create/list/read/search/ingest.
- Contract tests.

### Phase 5 — HTTP/UI

- Add FastAPI and inspection UI.

### Phase 6 — Strategy integration

- Baseline single-file consumes source-of-truth API.
- Context compiler consumes source-of-truth API.

## 9. Acceptance criteria

- Can create a context.
- Can create an event with date.
- Can ingest a raw file into the event.
- Can generate or attach extracted Markdown.
- Can list timeline events.
- Can read extracted source through MCP.
- Can search evidence.
- Can copy the context folder and rebuild the index.
