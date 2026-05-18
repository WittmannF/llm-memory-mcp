# Baseline Single File Strategy Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task.
>
> **Status:** Docs-only planning artifact. No implementation code has been created yet.

**Goal:** Build the simplest memory strategy: one generated Markdown context file per context, derived from the shared source-of-truth.

**Architecture:** The strategy reads events, references, facts, raw/extracted metadata, and generated Markdown derivatives from the source-of-truth store. It produces a single canonical `single-context.md` file and exposes it through MCP.

**Tech Stack:** Markdown, source-of-truth MCP client, optional LLM provider, optional HTML render, simple file watcher or explicit refresh.

---

## 1. Purpose

This strategy is the baseline for the benchmark.

It intentionally avoids:

- embeddings;
- graph retrieval;
- many topic pages;
- complex indexing;
- multi-step memory hierarchy.

It does one thing:

```text
all relevant source-of-truth data -> one detailed context file
```

## 2. Output file

Per context:

```text
outputs/<context>/single-context.md
outputs/<context>/single-context.html
```

Or, if strategy artifacts live inside each context later:

```text
<context>/.memory-strategies/baseline-single-file/single-context.md
```

## 3. Suggested sections

```markdown
# Single Context — <Context Name>

## How to use this file

## TL;DR

## Stable facts

## Current state

## Timeline

## Events by date

## References

## Important source excerpts

## Contradictions and uncertainties

## Open questions

## Source inventory

## Generation metadata
```

### Stable facts

This section consumes curated facts from source-of-truth `facts/facts.md`, plus stable facts inferred from references/events only if provenance is cited.

### Timeline

Built from `events/YYYY/MM/DD` ordering.

### References

Built from atemporal `references/` folders.

### Important source excerpts

Short curated excerpts from `extracted/` files. Avoid dumping full transcripts unless needed.

## 4. Update behavior

On refresh:

1. Read manifest from source-of-truth.
2. Read facts.
3. List references.
4. List events chronologically.
5. Read extracted Markdown for each relevant source.
6. Generate a full draft of `single-context.md`.
7. Write atomically.
8. Render HTML.
9. Record generation metadata.

## 5. MCP tools

- `refresh_single_context(context, mode="draft|write")`
- `read_single_context(context)`
- `get_single_context_status(context)`
- `render_single_context_html(context)`

Optional:

- `ask_single_context(context, question)` — not needed for MVP; benchmark can ask external LLMs using the file.

## 6. Strengths

- Very easy for LLMs to consume.
- Low implementation complexity.
- Human inspectable.
- Good baseline for cost and quality.
- No retrieval failures if file fits in context.

## 7. Weaknesses

- Can become too large.
- Refresh may be expensive.
- Harder to cite fine-grained evidence if not carefully structured.
- May blur contradictions if the summarizer is not conservative.
- Less suitable for huge contexts.

## 8. Benchmark hypotheses

This strategy may win for:

- small/medium contexts;
- contexts where a complete summary fits comfortably into an LLM context window;
- high-inspection workflows;
- low engineering complexity.

It may lose for:

- very large contexts;
- queries needing obscure details;
- high-frequency incremental updates;
- cases requiring granular provenance.

## 9. Implementation phases

### Phase 1 — File format and fixture

- Create a synthetic fixture context.
- Write expected `single-context.md` manually.
- Define acceptance rubric.

### Phase 2 — Source reader

- Read facts, events, references from source-of-truth store.
- Build deterministic data bundle.

### Phase 3 — Draft generator

- Use fake LLM for tests.
- Use real LLM provider later.
- Generate the sections above.

### Phase 4 — MCP tools

- Expose refresh/read/status.

### Phase 5 — Benchmark integration

- Add this strategy as the first benchmark competitor.

## 10. Acceptance criteria

- Can generate `single-context.md` from a fixture.
- Includes facts, timeline, references, source inventory, uncertainties.
- Cites extracted/source paths.
- Marks stale/fresh based on source-of-truth manifest.
- Can be read through MCP.
- Can be benchmarked against question set.
