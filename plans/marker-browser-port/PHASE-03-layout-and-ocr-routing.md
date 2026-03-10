# Phase 3: Layout And OCR Routing

Goal

- replicate Marker’s most important decision:
  - use native PDF text when good
  - use OCR only where needed

Scope

- layout detection
- line detection fallback
- OCR quality gating
- page/block routing

Deliverables

- layout stage producing block candidates:
  - text
  - table
  - figure
  - picture
  - caption
  - footnote
  - section header
- OCR routing stage:
  - page-level native-text quality check
  - block-level OCR scheduling
- initial OCR stack for fallback pages/blocks

Suggested browser stack

- layout:
  - small `RT-DETR` or compact YOLO model via ONNX
- OCR fallback:
  - `PP-OCRv4` det + cls + rec via ONNX/WebGPU
- quality routing:
  - deterministic heuristics first
  - no learned OCR-error classifier in v1

Port of Marker logic

- `LayoutBuilder`
  - lowres page image -> labeled block boxes
- `LineBuilder`
  - inspect native text quality
  - compare native text coverage against layout blocks
  - mark page/block as `native` or `ocr`
- `OcrBuilder`
  - crop only scheduled blocks
  - merge OCR results back into IR

Routing heuristics to implement

- empty native text
- too many replacement chars / garbage chars
- low alnum ratio
- low coverage of layout text regions
- overlapping / out-of-bounds native spans

Failure modes

- routing too aggressive -> unnecessary OCR cost
- routing too weak -> broken native text leaks through
- OCR crops misaligned with page coordinates
- layout labels too noisy for downstream processors

Verification

- per-page debug report:
  - native-text score
  - OCR scheduled yes/no
  - layout overlay
  - OCR block overlay
- compare routed output against chosen sample PDFs

Exit criteria

- clean digital papers mostly stay on native-text path
- broken/scanned pages trigger OCR path
- OCR output reinserts into IR with correct geometry
