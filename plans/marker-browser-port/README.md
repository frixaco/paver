# Marker Browser Port Plan

Assumptions

- target = browser-native reimplementation of Marker architecture
- target runtime = JS/TS + WASM + workers + ONNX/WebGPU where needed
- not using Python, Pyodide, server OCR, or remote inference
- primary input = born-digital PDFs first; scanned PDFs via fallback path

Phase files

- [PHASE-01-foundation.md](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-01-foundation.md)
- [PHASE-02-provider-and-ir.md](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-02-provider-and-ir.md)
- [PHASE-03-layout-and-ocr-routing.md](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-03-layout-and-ocr-routing.md)
- [PHASE-04-structure-and-processors.md](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-04-structure-and-processors.md)
- [PHASE-05-tables-and-advanced-regions.md](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-05-tables-and-advanced-regions.md)
- [PHASE-06-rendering-eval-and-hardening.md](/home/frixa/Documents/paver/plans/marker-browser-port/PHASE-06-rendering-eval-and-hardening.md)

Marker mapping

- `PdfProvider` -> browser PDF engine wrapper
- `DocumentBuilder` -> worker pipeline bootstrap
- `LayoutBuilder` -> browser layout model stage
- `LineBuilder` -> native-text-vs-OCR routing
- `OcrBuilder` -> block OCR fallback stage
- `StructureBuilder` + processors -> deterministic postprocess passes
- renderers -> browser HTML/Markdown emitters

Guiding rules

- keep Marker architecture; do not port Python code literally
- keep structured IR as canonical source of truth
- use native PDF text first; OCR only where needed
- ship deterministic passes before large ML passes
- benchmark every phase on same golden PDFs
