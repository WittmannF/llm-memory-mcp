# Source of Truth

This directory documents the canonical data layout that all memory strategies should consume.

Real context data may be private and is intentionally ignored by git under `source-of-truth/contexts/*`. Keep real contexts in a private clone, private branch, encrypted storage, or external path.

## Purpose

The source-of-truth layer stores evidence in a way that is immediately consumable by LLMs:

- raw originals are preserved;
- each hard-to-read source has an LLM-ready Markdown derivative;
- contexts are separated by project/domain;
- temporal events are separated from stable references;
- optional curated facts can be stored with provenance.

## Context layout

```text
source-of-truth/contexts/<context>/
├── README.md
├── manifest.json
├── events/
│   └── YYYY/MM/DD/<event-slug>/
│       ├── event.md
│       ├── raw/
│       └── extracted/
├── references/
│   └── <reference-slug>/
│       ├── reference.md
│       ├── raw/
│       └── extracted/
└── facts/
    └── facts.md
```

## Events

Use `events/` for dated observations or episodes:

- consultations;
- exams;
- meetings;
- calls;
- message batches;
- status updates;
- decisions made at a point in time.

The date path makes timeline reconstruction cheap and reliable.

## References

Use `references/` for atemporal or slow-changing resources:

- identity documents;
- baseline medical documents;
- project briefs;
- contracts;
- schemas;
- stable account/project metadata;
- documents that are facts/resources rather than events.

## Facts

Use `facts/` sparingly for curated stable facts.

Facts are processed memory, not raw evidence. They should include provenance links back to events or references whenever possible.

## LLM-ready derivatives

For every raw source that is not naturally easy for an LLM to inspect, create an extracted Markdown file.

```text
raw/audio.ogg       -> extracted/audio.md
raw/exam.pdf        -> extracted/exam.md
raw/screenshot.jpg  -> extracted/screenshot.md
raw/document.docx   -> extracted/document.md
```

Images should support a policy:

- `markdown_only`: use OCR/description only;
- `image_plus_markdown`: expose both image and Markdown;
- `image_on_demand`: default to Markdown, load image only if a multimodal model needs it.

## Template

See `source-of-truth/templates/context/` for the proposed context skeleton.
