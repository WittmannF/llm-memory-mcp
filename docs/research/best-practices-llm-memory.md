# Best Practices for LLM Memory Management

This note summarizes practical design principles for long-lived LLM memory systems, especially systems that need to be reused by many AI clients through MCP.

## 1. Separate memory into explicit tiers

Do not treat "memory" as one blob. Long-lived LLM systems work better when memory has tiers with different update rules.

Recommended tiers for `llm-memory-mcp`:

```text
Working memory
  - transient context pack for the current query/task
  - built from summary + search + selected sources

Episodic memory
  - chronological records of events/interactions/sources
  - raw sources and extracted Markdown

Semantic memory
  - stable facts, concepts, summaries, topic pages
  - summary-context.md, wiki/, reports/topics/

Procedural memory
  - instructions, templates, domain rules, update policies
  - README, schema docs, context templates, prompts

Indexes/cache
  - FTS, vector embeddings, graph edges, HTML
  - always reconstructible
```

Design implication: `summary-context.md` should not try to be every source. It should be the semantic top layer that points to episodic evidence in `extracted/` and raw evidence in `sources/`.

## 2. Make source-of-truth and derived artifacts obvious

A memory system decays when it is unclear which files are canonical.

Recommended rule:

- Canonical human/LLM-readable knowledge: Markdown.
- Canonical raw evidence: `sources/` originals.
- Canonical LLM-readable source derivatives: `extracted/*.md`.
- Generated views: HTML.
- Generated indexes: SQLite FTS / embeddings.

Never make SQLite or HTML the only place where knowledge exists.

## 3. Preserve provenance at every layer

Every generated summary, topic report, or extracted file should cite its inputs.

Minimum provenance:

- relative path to original source;
- relative path to extracted Markdown;
- source hash;
- extraction method/model;
- summary/synthesis model;
- generated timestamp;
- cited sections/headings where possible.

Example:

```markdown
- Claim: Creatinine was mentioned as a monitoring concern.
  - Evidence: `extracted/audios/audio-medico-2026-05-18.md#12-44-15-10`
  - Original: `sources/audios/audio-medico-2026-05-18.ogg`
```

Best practice: generated claims in `summary-context.md` and `reports/topics/*.md` should reference extracted Markdown, not only raw files.

## 4. Store LLM-friendly derivatives for hard formats

LLMs should not need to replay an audio file, OCR an image, or parse a PDF every time they need a detail.

For each hard source, create an extracted Markdown derivative:

```text
sources/audios/foo.ogg    -> extracted/audios/foo.md
sources/pdfs/bar.pdf      -> extracted/pdfs/bar.md
sources/images/baz.jpg    -> extracted/images/baz.md
sources/urls/page.url     -> extracted/urls/page.md
```

Extracted Markdown should include:

- short summary;
- detailed summary;
- full transcript/text/OCR when available;
- timestamped sections for audio/video;
- low-confidence or unclear segments;
- metadata and source path.

This makes later retrieval, inspection, and synthesis much cheaper and more reliable.

## 5. Use hierarchical summarization instead of one-shot summarization

Large contexts should be summarized in layers:

```text
source chunks
  -> source-level extracted summaries
  -> topic-level notes/reports
  -> summary-context.md
  -> query-specific context pack
```

This follows the same intuition as RAPTOR/GraphRAG-style systems: summarize local units first, then synthesize global answers from the intermediate summaries and evidence.

Avoid regenerating `summary-context.md` directly from hundreds of raw chunks unless the context is tiny.

## 6. Keep `summary-context.md` canonical, but not magic

`summary-context.md` should be the first file an LLM reads, but not the only file available.

It should include:

- how to use the context;
- TL;DR;
- current state;
- timeline / state changes;
- durable facts;
- open questions;
- contradictions;
- most important sources;
- related topic reports;
- instructions for when to inspect `extracted/`.

It should not hide uncertainty. If the underlying evidence is weak, the summary should say so.

## 7. Retrieval should be adaptive

Do not always retrieve a fixed number of chunks.

Good retrieval should consider:

- query intent;
- whether the root summary already answers the question;
- recency;
- source importance;
- semantic similarity;
- keyword match;
- links/backlinks;
- user-specified scope;
- maximum token budget.

The MCP tool `build_context_pack` should be the adaptive retrieval/synthesis boundary:

```text
query + context + budget -> compact evidence pack for the caller LLM
```

## 8. Use reflection, but make it durable and reviewable

Agent-memory papers repeatedly show the value of reflection: periodically turning raw observations into higher-level lessons.

For `llm-memory-mcp`, reflections should become files, not hidden model state:

- `wiki/*.md`
- `reports/topics/*.md`
- sections inside `summary-context.md`

Useful reflection triggers:

- N new sources ingested;
- summary is stale;
- a query produced a useful synthesis;
- contradictions detected;
- a source changed;
- user explicitly asks for consolidation.

## 9. Use forgetting/decay carefully

Some memory systems use importance, recency, and decay. This is useful for retrieval ranking, but dangerous for deletion.

Recommendation:

- Use recency/importance/decay to rank retrieval.
- Do **not** delete or overwrite evidence automatically.
- Archive rather than delete.
- Keep stale or superseded information with timestamps and status.

For factual domains, old evidence may still matter.

## 10. Track staleness explicitly

A memory context needs to know when the summary is behind the sources.

Manifest should track:

- source count;
- sources added since last summary;
- extracted files added since last summary;
- last summary generation time;
- model used for summary;
- summary status: `fresh | stale | missing | failed`.

UI should show stale/fresh prominently.

## 11. Prefer durable topic reports over ephemeral answers

If a query is complex and likely useful again, save it.

Example:

```text
reports/topics/funcao-renal-antibioticos.md
reports/topics/decisoes-de-produto-q2.md
reports/topics/mercado-imoveis-batel.md
```

A topic report should include:

- original query;
- answer;
- evidence by source;
- uncertainty;
- contradiction notes;
- open questions;
- generation metadata.

This avoids re-deriving expensive context again and again.

## 12. Separate extraction models from synthesis models

Cheap models are good for:

- source summaries;
- tags;
- chunk titles;
- OCR cleanup;
- transcript formatting;
- topic suggestions.

Stronger models are better for:

- official `summary-context.md` updates;
- high-stakes medical/legal/financial synthesis;
- contradiction resolution;
- final topic reports.

This gives a good cost/quality trade-off.

## 13. Design for MCP clients with limited context

MCP tools should not return huge files by default.

Recommended pattern:

- `read_summary_context`: returns canonical summary.
- `search_context`: returns ranked snippets.
- `read_extracted_source`: returns one source derivative by id/path.
- `build_context_pack`: returns a bounded package with the most relevant sections.
- `generate_topic_report`: writes a durable report and returns path + summary.

Expose raw reading, but make bounded/context-aware tools the defaults.

## 14. Build observability into memory updates

Every mutation should be visible.

Recommended logs:

- `log.md` for human-readable chronological changes;
- manifest metadata for machine status;
- optional job logs for ingestion/synthesis failures.

Every generated file should answer:

- what generated me;
- when;
- from which sources;
- using which model;
- with what confidence.

## 15. Evaluate memory quality, not just feature completion

A memory system can have search and still be bad.

Evaluate:

- Can it retrieve the right source for a known question?
- Does `summary-context.md` reflect all important recent changes?
- Are answers faithful to cited sources?
- Are contradictions preserved rather than erased?
- Is stale information detected?
- Can another LLM use the context without special instructions?

See `evaluation-and-quality.md` for a suggested test suite.

## 16. Keep humans in the loop at the right layer

Humans should not need to manage chunk indexes, but they should be able to inspect:

- original sources;
- extracted Markdown;
- generated summaries;
- topic reports;
- provenance;
- stale/fresh status;
- contradictions.

That is why the project should generate HTML for inspection but keep Markdown as the source.
