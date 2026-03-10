# Phase 6: Rendering, Eval, And Hardening

Goal

- turn structured IR into stable user-facing output
- harden runtime for real browser use

Scope

- HTML renderer
- Markdown renderer
- metadata/debug outputs
- perf + memory + cache hardening

Deliverables

- HTML renderer from IR
- Markdown renderer from HTML or direct IR
- debug bundle export:
  - page images
  - IR JSON
  - overlays
  - markdown output
- benchmark suite:
  - first-page latency
  - full-doc latency
  - memory
  - model startup cost

Renderer rules

- canonical source = IR
- markdown = render target, not source of truth
- preserve page anchors where useful
- prefer HTML fallback for unreliable tables/complex blocks

Hardening work

- worker cancellation
- streaming page-by-page processing
- weight caching
- partial-page retry
- graceful fallback when WebGPU unavailable
- memory guardrails for large PDFs

Quality gates

- first page visible quickly
- page-by-page progressive extraction
- deterministic output for same PDF + same model set
- no catastrophic memory growth on long docs

Failure modes

- markdown renderer loses structure not lost in IR
- browser tab OOM on large PDFs
- workers deadlock on large model downloads
- model cache corruption / version drift

Verification

- snapshot tests:
  - IR JSON
  - HTML
  - Markdown
- manual review set:
  - digital paper
  - scanned page
  - table-heavy report
  - formula-heavy paper

Exit criteria

- browser-native pipeline stable on reference corpus
- markdown quality competitive on born-digital PDFs
- known weak cases explicitly gated behind fallback behavior
