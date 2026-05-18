# Context Template

Copy this folder to create a new source-of-truth context.

Recommended destination:

```text
source-of-truth/contexts/<context-slug>/
```

## Required structure

```text
<context>/
├── README.md
├── manifest.json
├── events/
├── references/
└── facts/
```

## Event template

```text
events/YYYY/MM/DD/<event-slug>/
├── event.md
├── raw/
└── extracted/
```

## Reference template

```text
references/<reference-slug>/
├── reference.md
├── raw/
└── extracted/
```
