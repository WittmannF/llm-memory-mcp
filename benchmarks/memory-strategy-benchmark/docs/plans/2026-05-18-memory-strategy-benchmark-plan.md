# Memory Strategy Benchmark / Hackathon Plan

> **For Hermes:** Use subagent-driven-development skill to implement this plan task-by-task.
>
> **Status:** Docs-only planning artifact. No implementation code has been created yet.

**Goal:** Create a benchmark/hackathon framework for comparing multiple LLM memory strategies against the same source-of-truth contexts.

**Architecture:** Shared fixtures live in the source-of-truth layout. Each strategy produces its own memory artifacts and MCP tools. The benchmark runs the same tasks against each strategy and scores quality, cost, latency, provenance, and update behavior.

**Tech Stack:** Markdown fixtures, JSON task specs, pytest/eval scripts later, optional evaluator LLM, cost/latency logging.

---

## 1. Motivation

The repo will contain multiple memory strategies. We need a fair way to compare them.

The benchmark should answer:

- Is a complex strategy actually better than a single context file?
- Does a strategy retrieve the right evidence?
- Does it cite sources correctly?
- Does it update after new evidence?
- How much does it cost?
- How much token budget does it use?
- How easy is the output for a human to inspect?

## 2. Competitors

Initial competitors:

1. `baseline-single-file`
2. `context-compiler`

Future competitors:

- vector RAG;
- graph RAG;
- episodic memory + reflection;
- Basic Memory-like markdown graph;
- hybrid search/context packer;
- long-context direct reader.

## 3. Fixture contexts

Use synthetic safe contexts first.

### Medical demo

Tests:

- timeline;
- extracted audio transcript;
- extracted exam PDF;
- stable facts;
- contradiction between sources;
- latest source update.

### Work demo

Tests:

- meetings;
- decisions;
- superseded decisions;
- stakeholders;
- next actions.

### Real-estate demo

Tests:

- property references;
- listings;
- comparables;
- temporal price changes;
- facts vs events.

## 4. Task types

### Question answering

Example:

```text
What is the current state of the medical situation and what changed recently?
```

### Evidence retrieval

Example:

```text
Find the source that mentions creatinine and antibiotic risk.
```

### Timeline reconstruction

Example:

```text
List events in chronological order and identify the latest exam.
```

### Contradiction handling

Example:

```text
Which sources disagree and what is uncertain?
```

### Update test

1. Run strategy on context v1.
2. Add new event/source.
3. Refresh strategy.
4. Ask what changed.

### Cost/latency test

Measure:

- input tokens;
- output tokens;
- model cost;
- execution time;
- files read;
- API/tool calls.

## 5. Scoring rubric

Score 0-3 per dimension.

### Faithfulness

- 0: unsupported or wrong.
- 1: partially supported.
- 2: mostly supported with minor issues.
- 3: fully supported and cites evidence.

### Completeness

- 0: misses central facts.
- 1: covers some central facts.
- 2: covers most important facts.
- 3: comprehensive for the task.

### Provenance

- 0: no sources.
- 1: vague sources.
- 2: file-level sources.
- 3: file + section/timestamp-level sources.

### Freshness

- 0: misses latest update.
- 1: mentions update but not impact.
- 2: incorporates latest update.
- 3: incorporates latest update and explains what changed.

### Contradiction handling

- 0: erases contradiction.
- 1: notices uncertainty vaguely.
- 2: identifies conflicting sources.
- 3: identifies conflict, dates, and evidence.

### Efficiency

Score based on normalized cost/latency relative to other strategies.

## 6. Benchmark artifacts

Planned structure:

```text
benchmarks/memory-strategy-benchmark/
├── fixtures/
│   ├── medical-demo/
│   ├── work-demo/
│   └── real-estate-demo/
├── tasks/
│   ├── medical-demo.json
│   ├── work-demo.json
│   └── real-estate-demo.json
├── results/
│   └── .gitkeep
└── reports/
    └── .gitkeep
```

## 7. Task spec draft

```json
{
  "id": "medical-current-state-001",
  "context": "medical-demo",
  "prompt": "What is the current state and what changed recently?",
  "expected_evidence": [
    "events/2026/05/18/doctor-call/extracted/audio.md",
    "events/2026/05/18/lab-result/extracted/exam.md"
  ],
  "must_mention": [
    "latest event",
    "uncertainty",
    "source paths"
  ],
  "task_type": "question_answering"
}
```

## 8. Hackathon protocol

Each competing model/agent gets:

- monorepo README;
- research docs;
- source-of-truth contract;
- strategy-specific folder;
- benchmark rules.

Each competitor must:

1. Implement or document a strategy under `strategies/<name>/`.
2. Consume the shared source-of-truth layout.
3. Expose strategy outputs in a documented way.
4. Run benchmark tasks or produce comparable outputs.
5. Record cost/latency assumptions.

## 9. Initial baseline comparison

Start with two strategies:

- `baseline-single-file`
- `context-compiler`

If the context compiler does not beat the baseline on the first fixtures, simplify it or revisit assumptions.

## 10. Acceptance criteria

- Benchmark has at least one fixture context.
- Benchmark has at least five tasks.
- Each task lists expected evidence.
- Baseline strategy can produce output for all tasks.
- Scoring rubric is documented.
- Results can be compared side by side.
