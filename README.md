# Browser-Native PDF Converter

- https://www.datalab.to/app/playground/documents/new
- https://pdf-to-markdown.com/#examples

This project is building a browser-native document conversion pipeline for PDFs.

Target:

- run fully on the user's machine
- no Python
- no document-processing API
- convert PDFs into structured data and Markdown
- handle research papers and books well, especially born-digital PDFs

Main idea

- use a native-grade PDF engine in the browser
- extract native PDF text first
- fall back to OCR only when native text is missing or broken
- reconstruct document structure locally:
  - reading order
  - headings
  - lists
  - captions
  - tables
- emit Markdown as a final render target

Project principles

- browser-first, local-first
- structured IR is canonical; Markdown is derived output
- Marker architecture is the main reference, not a literal code port
- deterministic passes before heavy ML passes
- per-page and per-block processing, not blind full-document OCR

Planned stack

- app: TanStack Start + React
- UI: shadcn/ui
- PDF engine: likely `PDFium WASM`
- model runtime: likely `onnxruntime-web`
- acceleration: workers + WebGPU where available

Current focus

- architecture and research
- browser-native replacement for Marker-style PDF conversion stages
- phased port plan for provider, IR, layout, OCR routing, structure passes, tables, and rendering

Planning docs

- [Marker browser port plan](/home/frixa/Documents/paver/plans/marker-browser-port/README.md)
- [Phase 1](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-01-foundation.md)
- [Phase 2](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-02-provider-and-ir.md)
- [Phase 3](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-03-layout-and-ocr-routing.md)
- [Phase 4](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-04-structure-and-processors.md)
- [Phase 5](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-05-tables-and-advanced-regions.md)
- [Phase 6](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-06-rendering-eval-and-hardening.md)

Non-goals right now

- remote parsing services
- Python-based conversion stack
- literal runtime parity with Docling/Marker/MinerU

Near-term success criteria

- strong extraction and Markdown output for digital research PDFs
- clear browser-native architecture
- measurable quality/perf benchmarks on a fixed PDF corpus
