# Evaluation and Quality Plan for LLM Memory

A memory MCP should be evaluated as a memory system, not only as an API server.

This document defines quality checks that should guide implementation and regression tests.

## 1. Core quality questions

A good memory context should answer yes to these questions:

1. Can another LLM understand the context by reading `summary-context.md` first?
2. Can the system retrieve the right evidence for a query?
3. Are generated claims traceable to extracted/source files?
4. Does the system detect when summaries are stale?
5. Does it preserve contradictions and uncertainty instead of smoothing them away?
6. Can hard sources be inspected through Markdown without reprocessing the original file?
7. Can the context be moved/copied and still work after reindexing?
8. Are generated artifacts reproducible enough for tests?

## 2. Suggested fixture contexts

Create synthetic fixtures that are safe to commit.

### `examples/contexts/medical-demo`

Purpose: test high-stakes summary behavior without real patient data.

Include:

- fake audio transcript in `extracted/audios/doctor-call.md`;
- fake PDF lab report in `extracted/pdfs/lab-report.md`;
- fake WhatsApp note in `extracted/whatsapp/family-update.md`;
- one contradiction between family note and lab report;
- `summary-context.md` initially stale;
- topic report about medication risk.

Queries:

- "What do we know about kidney function?"
- "What changed in the last 48 hours?"
- "Which claims are uncertain or contradictory?"

Expected evidence:

- lab report file for kidney function;
- doctor-call transcript for medication discussion;
- contradiction surfaced explicitly.

### `examples/contexts/work-demo`

Purpose: test project/work memory.

Include:

- meeting transcript;
- decision note;
- roadmap note;
- follow-up email;
- one outdated decision superseded later.

Queries:

- "What decisions were made about launch timing?"
- "What are the open risks?"
- "Which decision was superseded?"

## 3. Retrieval tests

For each fixture, maintain a file like:

```text
examples/contexts/<name>/.llm-memory/eval/queries.json
```

Example:

```json
[
  {
    "query": "kidney function and antibiotics",
    "expected_paths": [
      "extracted/pdfs/lab-report.md",
      "extracted/audios/doctor-call.md"
    ],
    "must_not_include": [
      "extracted/whatsapp/unrelated-logistics.md"
    ]
  }
]
```

Metrics:

- recall@k for expected paths;
- whether top result is useful;
- whether snippets include useful headings;
- whether `summary-context.md` is included when appropriate.

## 4. Context pack tests

`build_context_pack` should be tested separately from raw search.

Test cases:

- includes `summary-context.md` when `include_summary=true`;
- respects `max_tokens` or character budget;
- cites paths and headings;
- includes extracted source sections for detailed queries;
- includes topic report when query overlaps an existing report;
- emits an overflow/omission note when budget is exceeded.

Example assertion:

```text
Query: "What do we know about antibiotics and kidney function?"
Expected context pack contains:
- summary section mentioning current medication status;
- lab-report creatinine section;
- doctor-call transcript segment;
- explicit path references.
```

## 5. Summary quality tests

Generated summaries are partly subjective, but they can still be tested.

Use deterministic fake LLM provider for automated tests.

For real model evals, score manually or with an evaluator model on:

- completeness;
- faithfulness;
- freshness;
- uncertainty preservation;
- source citation quality;
- actionability;
- concision.

Suggested rubric:

```text
0 = absent/wrong
1 = partially present but flawed
2 = present and acceptable
3 = excellent
```

Dimensions:

1. Captures latest important sources.
2. Does not invent unsupported facts.
3. Cites extracted/source files.
4. Flags contradictions.
5. Separates confirmed facts from hypotheses.
6. Provides useful next inspection paths.

## 6. Provenance tests

For generated files:

- Every topic report should list input source paths in frontmatter or a "Sources" section.
- Every claim cluster should have at least one supporting source reference.
- Extracted files should point back to originals.
- Source hash in manifest should match current file.

Automated checks:

- parse all generated Markdown;
- verify referenced paths exist;
- verify no path traversal;
- verify source hashes;
- verify all reports have `query` and `sources` metadata.

## 7. Staleness tests

Scenarios:

1. New source ingested after summary generation.
2. Extracted file regenerated after source update.
3. Topic report generated before latest source.
4. Summary refresh fails.

Expected behavior:

- summary status becomes `stale` after new source;
- UI and MCP status report the reason;
- stale report is still readable but marked with generated timestamp;
- failed refresh is logged and does not corrupt previous summary.

## 8. Contradiction tests

Fixtures should include at least one contradiction.

Expected behavior:

- search retrieves both sides;
- context pack includes contradiction note if relevant;
- summary does not silently choose one side unless evidence supports it;
- generated topic report has an "Incertezas e contradições" section.

## 9. Portability tests

A context should be movable.

Test:

1. Create fixture context.
2. Copy folder to temp dir.
3. Delete `.llm-memory/index.sqlite`.
4. Run reindex.
5. Search returns expected results.
6. HTML can be regenerated.

This validates that Markdown/source files are the real source of truth.

## 10. Human-inspection tests

UI tests should verify:

- dashboard shows stale/fresh status;
- summary page renders;
- source/extracted mapping is visible;
- report list renders;
- search results link to files;
- copy/download links exist if implemented.

Do not require heavy browser automation for MVP. FastAPI + HTML response checks are enough initially.

## 11. Regression suite structure

Recommended test categories:

```text
tests/unit/
  test_paths.py
  test_frontmatter.py
  test_wikilinks.py
  test_manifest.py

tests/integration/
  test_context_create.py
  test_reindex_fts.py
  test_ingest_text.py
  test_render_html.py

tests/contract/
  test_mcp_tools.py
  test_http_api.py

tests/eval/
  test_retrieval_fixtures.py
  test_context_pack_fixtures.py
  test_staleness.py
  test_provenance.py
```

## 12. Manual acceptance checklist

Before a release:

- [ ] Create a context from scratch.
- [ ] Ingest a source.
- [ ] Confirm original appears in `sources/`.
- [ ] Confirm LLM-friendly Markdown appears in `extracted/`.
- [ ] Search finds the extracted content.
- [ ] MCP can read `summary-context.md`.
- [ ] MCP can build a context pack.
- [ ] Generate a topic report.
- [ ] HTML UI displays summary, sources, extracted files, and reports.
- [ ] Add a new source and confirm summary becomes stale.
- [ ] Refresh summary and confirm it becomes fresh.
- [ ] Move/copy context and reindex successfully.
