# Phase 1: Foundation

Goal

- lock runtime architecture
- lock core engine choices
- de-risk browser constraints before feature work

Why first

- wrong engine choice poisons every later phase
- Marker depends on strong PDF-native extraction; browser port needs same

Deliverables

- architecture decision record for:
  - PDF engine
  - worker graph
  - model runtime
  - artifact loading/cache strategy
- benchmark harness for:
  - PDF open time
  - page render time
  - text extraction time
  - memory growth
- reference corpus:
  - clean digital papers
  - multi-column books
  - scanned pages
  - table-heavy docs
  - formula-heavy docs

Choices to settle

- PDF engine:
  - preferred: `PDFium WASM`
  - fallback candidate: `MuPDF.js`
- model runtime:
  - preferred: `onnxruntime-web`
- threading:
  - parse worker
  - model worker
  - optional render worker
- storage:
  - memory first
  - IndexedDB for weights/cache later

Inputs

- PDF bytes
- page count
- page dimensions
- extracted text spans if available
- rendered page bitmaps

Failure modes

- engine cannot expose chars/spans/links cleanly
- memory spikes on large PDFs
- workers blocked by large structured clones
- model weight downloads too large for acceptable UX

Exit criteria

- one engine selected
- worker boundaries documented
- benchmark harness runs locally in browser
- golden corpus checked into plan/test process

Non-goals

- markdown quality
- OCR quality
- table support
