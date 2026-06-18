# Wiki Schema

> Multi-domain knowledge base. Raw sources are flat (classified via frontmatter). Knowledge flows by maturity.

## Architecture

```
vault/
├── raw/                        ← Layer 1: source material (immutable, flat files)
│   └── *.md                    ← classification via frontmatter, no subfolders
│
├── wiki/                       ← Layer 2: processed knowledge
│   ├── literature/             ← literature notes (single-source extraction, your own words)
│   ├── concepts/               ← permanent notes (synthesized concepts)
│   ├── entities/               ← permanent notes (people, companies, models, assets)
│   ├── comparisons/            ← permanent notes (A vs B)
│   ├── index.md                ← top-level directory
│   └── log.md                  ← changelog
│
├── projects/                   ← project notes (temporary, archive or delete when done)
│   └── {project-name}/
│
├── SCHEMA.md                   ← this file
│
└── personal notes              ← legacy content, not part of the wiki system
```

## Knowledge Flow

```
inspiration strikes
    ↓
wiki/literature/xx.md   status: fleeting    ← capture fast, don't polish
    ↓ add sources and context
wiki/literature/xx.md   status: draft       ← has citations
    ↓ synthesize 2+ sources, form original view
wiki/concepts/xx.md     status: done        ← permanent note
    ↓ serve a specific task
projects/some-project/                      ← project note, dies with the project
```

Promotion rules:

| Condition | Action |
|-----------|--------|
| First reading of raw source → write key points in your own words | Create `wiki/literature/`, status=fleeting |
| Added citations and context | status → draft |
| Synthesized 2+ sources, formed independent view | Move to `wiki/concepts/`, `entities/`, or `comparisons/`, status=done |
| fleeting untouched for 30+ days | Lint flags it: develop or delete |

## File Conventions

- Filenames: lowercase, hyphens, no spaces (`dragon-tactics.md`, `cortex-m3-nvic.md`)
- Every wiki page links to at least 2 other pages
- Files in `raw/` are never modified — corrections go in the wiki layer
- Creating or deleting a wiki page requires updating the domain index
- Every action appends to `wiki/log.md`
- A note growing from fleeting to permanent moves directories and changes status — don't duplicate

## Frontmatter

### raw/ files

```yaml
---
type: article | paper | transcript
domain: [trading, embedded]       ← can be multiple
source_url: https://...
ingested: YYYY-MM-DD
---
```

### wiki/ files

```yaml
---
title: Page Title
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: concept | entity | comparison | literature
domain: [trading, embedded]
status: fleeting | draft | done   ← required for literature layer
tags: [keyword, ...]
sources: [raw/xxx.md]
confidence: high | medium | low
---
```

- `domain`: array of domains this page belongs to (can be multiple)
- `status`: maturity tracker for literature layer; permanent notes default to done
- `confidence`: single-source or subjective content → medium or low
- `tags`: freeform keywords describing the topic (not the domain — that's already in `domain`)

## Domain Labels

Allowed values for the `domain` field (multi-select):

| Value | Meaning |
|-------|---------|
| `trading` | Financial trading, A-shares, quantitative |
| `ai` | LLMs, agents, workflows |
| `embedded` | STM32, PCB, microcontrollers |
| `optoelectronics` | Semiconductor physics, spectroscopy, solar cells |
| `media` | Content creation, ASMR, platform operations |

Register new domains here before using them.

## Page Creation Thresholds

- **Create a page** when an entity/concept appears in 2+ sources, or is central to one source
- **Append to existing page** when the source overlaps with what's already covered
- **Don't create** pages for passing mentions
- **Split** pages exceeding ~200 lines
- **Archive** when content is fully superseded — move to `_archive/`, remove from index

## Update Policy

When new information contradicts existing content:
1. Check dates — newer sources generally supersede older ones
2. If genuinely contradictory: keep both claims, annotate with dates and sources
3. Add `contested: true` and `contradictions: [page-name]` to frontmatter
4. Flag for human review

## Lint Checklist

- Broken wikilinks (pointing to nonexistent pages)
- Orphan pages (zero inbound links)
- Fleeting notes untouched for 30+ days
- Missing frontmatter
- Pages over 200 lines not split
- Unresolved contradiction markers
- `log.md` exceeding 500 entries — rotate it

## Personal Notes Rule

Markdown files at the vault root (not under `raw/`, `wiki/`, or `projects/`) are personal notes. The agent reads them but does not modify them unless explicitly asked.
