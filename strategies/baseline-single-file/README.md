# Strategy: Baseline Single File

This is the simplest possible memory strategy and should be used as a baseline benchmark.

## Goal

For each context, read the shared source-of-truth and generate exactly one canonical Markdown file:

```text
single-context.md
```

This file contains everything the LLM should know about that context, organized into predictable sections.

## Why this baseline matters

A richer memory strategy is only worth it if it beats this simple approach on quality, cost, latency, maintainability, or inspectability.

## Plan

See:

```text
strategies/baseline-single-file/docs/plans/2026-05-18-baseline-single-file-plan.md
```
