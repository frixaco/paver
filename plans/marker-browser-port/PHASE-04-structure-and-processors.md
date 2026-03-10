# Phase 4: Structure And Processors

Goal

- rebuild the deterministic passes that make Marker output readable

Scope

- grouping
- ordering
- continuation
- heading inference
- TOC
- basic cleanup

Deliverables

- `StructureBuilder` equivalent:
  - caption grouping around figures/tables
  - list grouping
  - unmark isolated fake list items
- processor passes:
  - order repair
  - line merge
  - text continuation across columns/pages
  - section header level inference
  - document TOC generation
  - footnote/header/footer suppression

Priority order

1. order repair
2. text continuation
3. heading inference
4. list grouping
5. caption grouping
6. TOC

Marker-inspired notes

- `OrderProcessor`
  - only needed when layout slicing or extraction reordering causes drift
- `TextProcessor`
  - key for multi-column/page continuation quality
- `SectionHeaderProcessor`
  - use deterministic clustering over line heights/font metadata first
- `DocumentTOCProcessor`
  - derive from section headers; no separate model

Simplifications vs Marker

- no LLM processors
- no sklearn dependency; plain TS clustering or threshold bucketing
- no Python block inheritance tree

Failure modes

- continuation merges unrelated columns
- heading inference overfits font size noise
- grouped captions swallow nearby body text
- TOC polluted by false headers

Verification

- golden markdown/JSON snapshots on:
  - 2-column papers
  - books with chapter-like headings
  - figure/table caption-heavy docs
- visual diff of:
  - block ordering
  - continuation flags
  - heading levels

Exit criteria

- output reads coherently on clean digital papers without OCR
- section headers and TOC look sane on sample corpus
