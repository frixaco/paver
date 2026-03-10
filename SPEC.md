# Paver Spec

## 1. Product Thesis

- PDF reader for research papers + books
- Core job: read, question, summarize, extract, stay in flow
- Core surfaces:
  - reader
  - AI sidebar
  - selection toolbar
- Core promise:
  - PDF opens fast
  - AI stays grounded in document context
  - actions stay near cursor

## 2. Assumptions

- v1 = single active document
- v1 = desktop-first, mobile-safe
- v1 = text PDFs first; scanned PDFs may render with limited AI quality
- viewer usable before ingestion completes
- AI provider hidden behind server functions
- streaming is required in v1
- no persistence in v1
- BYOK = OpenRouter API key only

## 3. V1 Outcome

- Open PDF from file or URL
- Read with stable scrolling, zoom, page tracking
- Ask AI about book, page, selection
- Summarize same 3 scopes
- Every AI answer includes page citations
- Selecting text opens a floating toolbar near selection

## 4. Scope

### Must

- File input for PDF
- URL input for PDF
- EmbedPDF-based reader
- Page tracking
- Zoom + fit controls
- AI sidebar with scope-aware composer
- Streaming markdown message rendering
- Book / page / selection summaries
- Selection toolbar
- Citation jump-back into reader
- Cheap public model policy + BYOK mode
- Loading, empty, error states for import + AI

### Should

- Outline/bookmark panel
- Resizable AI sidebar
- Server-side URL proxy/fetch path for CORS failures
- Prompt presets for summarize / explain / extract claims

### Later

- Full-text document search UI
- Thumbnail rail
- Saved highlights / annotations
- Multi-document workspace
- OCR pipeline
- Export to notes / citation manager

## 5. Non-Goals

- Collaborative editing
- Google Docs-style comments
- Rich note graph
- Multi-user sync
- Offline-first architecture in v1

## 6. UX Principles

- Reader-first: PDF gets dominant width
- Cursor-proximate: selection actions appear at selection
- Quiet chrome: low-noise default UI
- Scope always visible before send
- Read before ready: ingestion lag must not block reading
- Credible AI: answers grounded with citations, never unsupported

## 7. Layout Model

### Desktop

- Top bar: app identity, source controls, current doc title, page jump, zoom, fit, outline toggle, sidebar toggle
- Main shell:
  - optional left rail for outline/thumbnails
  - center reader pane
  - right AI sidebar
- Width guidance:
  - left rail: 280-320px
  - AI sidebar: 360-420px
  - reader: remaining width, priority area

### Narrow Width

- Reader full width
- AI sidebar becomes sheet
- Outline becomes sheet
- Selection toolbar reduces to primary actions + overflow

## 8. Core Flows

### Open PDF

1. User chooses `File` or `URL`
2. App validates input immediately
3. Reader route loads
4. PDF renders as soon as source resolves
5. Ingestion starts in background
6. Sidebar status shows `Loading document`, `Indexing for AI`, `Ready`, or `Limited`

### Ask AI

1. User chooses scope
2. User enters prompt
3. App sends prompt plus normalized scope payload
4. Response returns answer, citations, scope label, completeness note
5. User can jump from citation back to page

### Select Text

1. User selects text in PDF text layer
2. Floating toolbar appears near selection anchor
3. User chooses `Ask AI`, `Summarize`, `Explain`, `Quote`, or `Copy`
4. AI action seeds composer with exact selection context

## 9. Functional Requirements

### A. Source Input

- Support local file upload and remote URL input
- Accept only PDF after validation
- Handle invalid URL, fetch failure, non-PDF response, inaccessible remote doc, duplicate import
- URL import path:
  - browser direct fetch if allowed
  - server fetch/proxy fallback if CORS blocks

### B. Reader

- Continuous vertical scroll default
- Fit width default on first load
- Zoom in/out
- Fit width / fit page
- Current page indicator
- Page jump input
- Outline/bookmarks panel when available
- Stable scroll when side panels open/close
- Preserve viewport during AI activity

### C. AI Sidebar

- Persistent + resizable on desktop
- Sheet on narrow screens
- Contains scope switcher, quick summary actions, chat thread, composer, citations
- Assistant messages render progressively while streaming
- Scope choices:
  - `Book`
  - `Page`
  - `Selection`
- Invalid scope behavior:
  - disable `Selection` if no active selection

### D. Selection Toolbar

- Trigger only on non-empty text selection in text layer
- Position above selection by default, below on top collision, clamped to viewport
- Dismiss on clear, `Esc`, click-away, or meaningful scroll
- No toolbar for accidental tiny selections or image-only regions

### E. AI Answers

- Must include:
  - answer text
  - page citations
  - cited snippets or excerpt anchors
  - scope metadata
  - completeness note if context partial
- Must not:
  - claim certainty without citations
  - silently answer outside active scope
  - hide incomplete ingestion state
- Markdown support:
  - headings, lists, code, tables, blockquotes
  - citations rendered as clickable page refs
  - incomplete markdown must render safely during stream

## 10. Acceptance Criteria

- Local PDF renders without server upload round-trip
- Remote PDF renders from URL when reachable
- Current page stays correct while scrolling
- Zoom changes do not break selection layer alignment
- Book answers cite real pages
- Page answers stay page-bounded unless app explicitly expands context
- Selection answers include exact selection text in retrieval context
- Selection toolbar appears quickly, stays unclipped, and seeds composer correctly

## 11. Technical Architecture

### Route Model

- `/`
  - landing + source input
- `/d/$documentId`
  - reader shell
  - reader state
  - AI sidebar

### TanStack Start

- Use route loaders for document shell data:
  - metadata
  - page count
  - outline
  - ingestion status
  - runtime summary readiness
- Use server functions for server boundaries:
  - `importDocument`
  - `resolveDocumentSource`
  - `getDocumentShell`
  - `askDocument`
  - `summarizeScope`
- Use validators on every server function input

### TanStack AI

- Use:
  - `@tanstack/ai`
  - `@tanstack/ai-react`
  - `@tanstack/ai-openrouter`
- Server path:
  - server function or route endpoint creates chat stream
  - return SSE stream to client
- Client path:
  - `useChat` manages thread state
  - `fetchServerSentEvents` connection default
- Reason:
  - typed messages
  - built-in streaming
  - provider adapter layer
  - easier future tool calls

### Client State Ownership

- Local UI state:
  - sidebar open/width
  - current page
  - zoom
  - current selection
  - current scope
  - composer draft
- Loader/server state:
  - document metadata
  - ingestion status
  - outline
  - chat responses

## 12. EmbedPDF Integration

### Foundation

- Use EmbedPDF React headless stack
- Verified base packages from current docs:
  - `@embedpdf/react`
  - `@embedpdf/pdfium`
  - `@embedpdf/core`
  - `@embedpdf/plugin-render`
  - `@embedpdf/plugin-viewport`
  - `@embedpdf/plugin-scroll`
  - `@embedpdf/plugin-tiling`
  - `@embedpdf/plugin-zoom`
  - `@embedpdf/plugin-document-manager`

### V1 Plugin Set

- `render`
- `viewport`
- `scroll`
- `tiling`
- `zoom`
- `document-manager`
- `selection`
- `bookmark`

### Later Plugin Set

- `search`
- `thumbnail`
- `annotation`
- `history`

### Integration Notes

- Depend on EmbedPDF plugin interfaces, not custom canvas logic
- Selection toolbar should use text-layer selection events where possible
- Citation jump-back should use viewer navigation primitives, not manual scroll guessing

## 13. AI / Retrieval Model

### Chat Rendering

- Use `comark` for assistant markdown rendering
- Reason:
  - React support
  - streaming-ready parser
  - auto-closes incomplete markdown while streaming
- Constraint:
  - wrap `comark` behind `MarkdownRenderer` component
  - do not leak library-specific API across app
  - reason: package is very new (`0.0.1`, modified 2026-02-13), so swap cost must stay low

### Chat Transport

- Default transport: SSE
- Streaming unit: append assistant content progressively into current message
- Preserve structured metadata outside markdown body:
  - citations
  - scope
  - warnings
  - token/model info

### Tool Model

- Expose exactly 2 document-reading tools in v1:
  - `read_page`
  - `read_book`
- Reason:
  - simpler than generic retrieval pipeline
  - easier token budgeting
  - easier debugging of grounded answers

### Tool Contracts

- `read_page`
  - input:
    - `pageNumber`
    - optional `selectionText`
  - output:
    - page text / markdown
    - page number
    - nearby metadata if available
- `read_book`
  - input:
    - optional `query`
    - optional `pageRange`
  - output:
    - compact book-level text / markdown context
    - matched page refs
    - truncation flag when content window is limited

### Document Representation

- Normalize PDF into page-addressable text in v1
- Preferred internal representation:
  - raw page text
  - page-level markdown when feasible
- Future direction:
  - stronger PDF -> Markdown pipeline
  - markdown becomes primary LLM context when extraction quality is good

### Scope Resolution

- `book`
  - model may call `read_book`
- `page`
  - model may call `read_page`
- `selection`
  - selected text passed directly plus optional `read_page` call for nearby context

### Response Contract

- `answer`
- `citations[]`
- `scope`
- `completeness`
- `warnings[]`

### Model Policy

- Public default model:
  - `deepseek/deepseek-v3.2`
  - rationale: strong cost/quality tradeoff from current OpenRouter pricing + usage signals
- BYOK:
  - OpenRouter API key only
  - user key takes precedence when configured
  - if user key has no credits or is invalid, request fails
- Public access:
  - backend uses app-owned OpenRouter API key
- Cost controls:
  - cap max output tokens per scope
  - tighter limits for summaries than free-form Q&A

## 14. Runtime Data Shape

### `activeDocument`

- `id`
- `title`
- `sourceType`
- `sourceUrl`
- `fileName`
- `mimeType`
- `pageCount`
- `outlineJson`
- `ingestionStatus`
- `ingestionError`

### `pageContent`

- `pageNumber`
- `text`
- `markdown`

### `chatMessage`

- `id`
- `role`
- `scopeType`
- `content`
- `citationsJson`
- `model`
- `tokenUsage`

## 15. UI Components

### Install via `shadcn`

```bash
bunx --bun shadcn@latest add input textarea separator sheet tooltip scroll-area resizable sidebar tabs card skeleton badge dialog dropdown-menu command
```

### Planned Components

- `AppShell`
- `TopBar`
- `DocumentSourceBar`
- `ReaderWorkspace`
- `PdfReaderPane`
- `PdfOutlinePanel`
- `AiSidebar`
- `ScopeSwitcher`
- `SummaryActions`
- `AiThread`
- `AiComposer`
- `SelectionToolbar`
- `CitationLink`
- `IngestionStatusBadge`
- `MarkdownRenderer`

## 16. Failure Modes

- URL blocked by CORS and proxy path fails
- Remote URL resolves to HTML, not PDF
- PDF renders but text extraction fails
- PDF has no outline
- Selection spans page boundaries
- Answer returns with stale page context after page changed
- Duplicate source imported twice

## 17. Milestones

### M1. Reader Shell

- source bar
- route model
- document open
- EmbedPDF base viewer
- page tracking
- zoom + fit

### M2. AI Foundation

- server functions for import + ask + summarize
- page-text extraction
- `read_page` / `read_book` tools
- book/page summaries
- cited answers

### M3. Selection Workflow

- text selection tracking
- floating toolbar
- seeded composer
- selection summaries + answers

### M4. Structure + Polish

- future PDF -> Markdown pipeline improvements
- sidebar resizing
- error-state polish
- search / thumbnails if base stable

## 18. Verification Strategy

- Reader shell:
  - manual open from file
  - manual open from URL
  - regression tests for zoom + page tracking
- AI server functions:
  - black-box tests for validated inputs
  - snapshot/golden tests for normalized citation payloads
- Selection workflow:
  - component tests for visibility + dismissal
  - regression tests for collision handling
- End-to-end:
  - open sample paper
  - ask book question
  - ask page question
  - ask selection question
  - jump from citation to page

## 19. Open Decisions

- exact PDF -> Markdown approach
- explicit support policy for scanned PDFs

## 20. Deferred Backend Findings

### Separate API / Service

- Likely yes
- Reason:
  - strongest PDF -> Markdown tools are Python-first
  - conversion can be CPU/GPU heavy and slow
  - isolates document-processing failures from web app request cycle
  - easier to swap converters later without rewriting app server
- Default shape if adopted later:
  - web app owns auth, reader UI, chat UI
  - document service owns PDF extraction, page text, page markdown, `read_page`, `read_book`

### `read_book` Implementation

- Do not send the whole book to the model
- Recommended v1 behavior:
  - extract page-level text once per active document
  - build a lightweight lexical index over pages
  - on `read_book(query)`:
    - rank relevant pages
    - include small neighbor windows when needed
    - compress into bounded context
    - return excerpts + page refs + `truncated`
- Simpler than embeddings for v1
- Easier to debug grounding and token cost
- Popular lightweight search options:
  - `MiniSearch`
  - `FlexSearch`

### PDF -> Markdown Tooling

- Most promising default:
  - `Docling`
  - strengths:
    - official CLI + Python APIs
    - Markdown and JSON output
    - document hierarchy support
    - good fit for future structured pipeline
- Fast/simple fallback:
  - `PyMuPDF4LLM`
  - strengths:
    - simple page-level markdown extraction
    - good fit for `read_page`
- Quality-first paper parsing options:
  - `Marker`
  - `MinerU`
- Tradeoff:
  - `Marker` and `MinerU` look stronger for hard scientific PDFs
  - but are heavier operationally than `Docling` / `PyMuPDF4LLM`

### Current Recommendation

- If a separate service is added later:
  - start with `Docling`
  - keep `PyMuPDF4LLM` as fallback / comparison baseline
- Keep conversion behind an internal adapter interface
- Future direction:
  - stronger PDF -> Markdown pipeline
  - possibly page markdown + structured JSON side-by-side

### Product Reference: alphaXiv

- Useful reference for UX, not implementation scope
- Relevant overlap seen publicly:
  - AI chat on paper
  - highlight-to-ask AI
- Difference:
  - alphaXiv is broader community + discussion platform
  - Paver v1 remains focused reader + AI assistant

## 21. Research Notes

- TanStack Start current docs support server functions with validators and middleware; fit for import, ingest, ask, summarize boundaries
- TanStack Router data-loading docs support route-bound loaders and revalidation; fit for document shell data
- TanStack AI current docs provide framework-agnostic core, React `useChat`, SSE/HTTP stream adapters, and an official OpenRouter adapter
- current `shadcn` CLI add command remains `bunx --bun shadcn@latest add <component>`
- EmbedPDF React docs show a headless plugin architecture with PDFium-backed rendering and document/render/viewport/scroll plugins
- Comark positions itself as a fast, streaming-ready markdown parser with React support and automatic closing of incomplete markdown during streaming
- OpenRouter BYOK docs support provider keys; v1 uses OpenRouter keys only
- OpenRouter rankings on 2026-03-10 show Deepseek V3.2 among top weekly usage models, with pricing still low enough for public fallback use
- EmbedPDF ecosystem currently includes selection, annotation, search, bookmark, thumbnail, history, render, scroll, viewport, tiling, zoom plugins

## 22. Primary Sources

- https://tanstack.com/start/latest/docs/framework/react/guide/server-functions
- https://tanstack.com/router/latest/docs/framework/react/guide/data-loading
- https://tanstack.com/ai/latest/docs/getting-started/overview
- https://tanstack.com/ai/latest/docs/api/ai-react
- https://tanstack.com/ai/latest/docs/adapters/openrouter
- https://ui.shadcn.com/docs/cli
- https://comark.dev/
- https://www.embedpdf.com/docs/react/headless/getting-started
- https://www.embedpdf.com/docs/react/headless/understanding-plugins
- https://www.embedpdf.com/docs/react/headless/full-example-mui-viewer
- https://docling-project.github.io/docling/v2/
- https://docling.site/cli/
- https://pymupdf.qubitpi.org/en/latest/pymupdf4llm/api.html
- https://github.com/datalab-to/marker
- https://github.com/opendatalab/MinerU
- https://github.com/lucaong/minisearch
- https://github.com/nextapps-de/flexsearch
- https://www.alphaxiv.org/changelog
- https://www.alphaxiv.org/signin?flow=%2Fprivate%2Fresources%2F4490ef1e-3060-42b2-8f98-46d829e5b359v1
- https://openrouter.ai/rankings
- https://openrouter.ai/docs/guides/overview/auth/byok
- https://openrouter.ai/api/v1/models
