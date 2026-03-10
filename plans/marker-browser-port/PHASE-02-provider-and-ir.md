# Phase 2: Provider And IR

Goal

- rebuild Marker `PdfProvider` and `Document` concepts in browser-native form

Scope

- no ML yet
- no OCR yet
- focus on extraction, geometry, and canonical data model

Deliverables

- `PdfProvider` browser wrapper over chosen PDF engine
- `DocumentIR`, `PageIR`, `BlockIR`, `LineIR`, `SpanIR`, `CharIR`
- extraction output for:
  - page bbox
  - page image lowres/highres
  - native text spans
  - char offsets if available
  - links / refs
  - outline if available
- JSON snapshot serializer for debug/regression tests

Required behavior

- open PDF from file bytes
- enumerate pages
- extract native text layer with geometry
- render lowres + highres page images
- preserve page-local coordinate system

Suggested IR

- `DocumentIR`
  - `source`
  - `pages`
  - `metadata`
- `PageIR`
  - `pageNumber`
  - `width`
  - `height`
  - `nativeTextSpans`
  - `links`
  - `outlineRefs`
  - `images`
  - `blocks`
- `SpanIR`
  - `text`
  - `bbox`
  - `fontName?`
  - `fontSize?`
  - `fontWeight?`
  - `charStart?`
  - `charEnd?`

Marker parity target

- match what Marker gets from `pdftext` + `pypdfium2`
- not exact API parity
- semantic parity only

Failure modes

- engine returns text items with unusable reading order
- font metadata unavailable
- link extraction missing
- rendered coordinates do not align with extracted spans

Verification

- golden JSON snapshots per sample PDF
- visual debug overlay:
  - page image
  - native span bboxes
  - link boxes

Exit criteria

- one stable IR shape
- provider can dump deterministic JSON for sample corpus
- extracted geometry aligns with page render
