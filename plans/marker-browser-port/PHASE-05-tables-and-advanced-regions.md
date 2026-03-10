# Phase 5: Tables And Advanced Regions

Goal

- close the biggest quality gap between a basic converter and Marker-class output

Scope

- table structure
- cell text assignment
- code / formula / complex region strategy

Deliverables

- table detection + structure stage
- cell text assignment:
  - native PDF text first
  - OCR fallback per cell
- HTML table emitter
- block policies for:
  - code
  - inline math
  - display math
  - complex regions

Recommended order

1. table detection
2. cell structure
3. native text -> cells
4. OCR per cell fallback
5. table HTML/Markdown rendering
6. formula/code handling

Marker parity target

- emulate `TableProcessor` behavior, not exact implementation
- treat tables as separate subsystem
- keep them out of generic text merging path

Model options

- initial:
  - compact table detector / structure model via ONNX
- later:
  - stronger specialized model if browser budget allows

Rules

- do not OCR full page for one bad table
- table cells own their own text lines
- markdown tables only when structure is reliable
- otherwise fall back to embedded HTML table

Failure modes

- cell structure wrong but looks plausible
- OCR per cell too slow on dense tables
- formulas pollute surrounding text
- code blocks rendered as prose

Verification

- separate table-heavy golden corpus
- cell-grid overlay debug view
- compare:
  - row/col counts
  - cell text assignment
  - final HTML/Markdown table quality

Exit criteria

- tables no longer destroy surrounding reading order
- table output good enough for downstream AI ingestion
