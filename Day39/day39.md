# Day 39 – PDF Splitter & Merger

## Overview

A single-file HTML application that splits and merges PDF documents entirely in the browser, with no file ever leaving the user's device. It provides two primary tools — a PDF Splitter with nine splitting methods and visual page thumbnails, and a PDF Merger with drag-and-drop reordering and live file previews. All processing happens client-side using JavaScript, making it comparable to commercial tools like ILovePDF or Smallpdf but with the privacy advantage of never uploading files to a server.

The problem it addresses is that most online PDF tools require uploading sensitive documents to third-party servers. This application performs all splitting, merging, and extraction locally in the browser, which makes it suitable for confidential documents. The educational objective is understanding client-side file processing, drag-and-drop interactions, and commercial-grade UI/UX design.

---

## Prompt Template

The following prompt was used to generate the PDF Splitter & Merger application:

```text
You are an expert UI/UX designer, frontend developer, document processing specialist, and JavaScript engineer.

Before generating anything, ask the user the following question.

1. Would you like Claude to automatically design the application, or would you like to customize its features?

If the user chooses customization, ask which additional PDF features they would like included.

After collecting the response, generate a premium single-page interactive HTML application called 'PDF Splitter & Merger'.

The application should provide two primary tools:

PDF Splitter:
Allow users to upload a PDF and automatically detect the total number of pages. Display visual page thumbnails for every page and allow users to preview the document before splitting. Users should be able to split the PDF by entering page numbers, selecting custom page ranges, splitting after specific pages, splitting every N pages, or extracting selected pages into one or more new PDF files. Allow users to create multiple split ranges in a single operation, validate all page ranges, preview the resulting document structure before processing, and clearly highlight invalid inputs.

PDF Merger:
Allow users to upload multiple PDF files using drag-and-drop or file selection. Display all uploaded files in a sortable list with page counts and visual previews. Users should be able to reorder the PDFs using drag-and-drop before merging. Display the total number of files, total page count, and estimated output before generating the merged document. Generate the merged PDF and provide an easy download option.

Perform all PDF processing entirely within the browser using client-side JavaScript. Do not upload files to external servers or rely on backend services. Use reliable browser-compatible libraries where necessary and ensure the application continues to work offline after the initial page load.

Include drag-and-drop uploads, processing indicators, loading animations, responsive layouts, dark mode, accessibility features, intuitive error handling, keyboard shortcuts where appropriate, and smooth micro-interactions throughout the application.

Generate everything as a single self-contained HTML file using HTML, CSS, and JavaScript only.

Design the interface as a polished commercial application comparable to professional PDF utilities, with exceptional UI/UX, beautiful typography, modern layouts, smooth animations, intuitive navigation, and an experience users would genuinely choose over existing online PDF tools.
```

---

## Features

- **Two primary tools in a tabbed sidebar layout**: PDF Splitter and PDF Merger, each with its own workspace and dedicated upload zone.
- **PDF Splitter with 9 methods**: Split Every Page (one PDF per page), Split Every N Pages (fixed chunks), Split After Page (break at a specific page), Custom Range (e.g., pages 2–5), Multiple Outputs (many ranges at once), Extract Selected (click thumbnails to pick), Odd Pages, Even Pages, and Reverse Extraction (flip page order).
- **Visual page thumbnails** rendered for every page in the uploaded PDF, with zoom controls, selection checkboxes, individual page actions (delete, rotate, extract), and **drag-to-reorder** — drag any thumbnail to a new position and the other pages shift smoothly to fill the gap.
- **PDF Merger with drag-and-drop reordering**: upload multiple PDFs, see file thumbnails, page counts, and total estimated output, then reorder by dragging before merging.
- **Real-time validation** of page ranges with clear error highlighting for invalid inputs (out-of-range, reversed, overlapping).
- **Processing indicators** with animated progress overlays showing the current operation stage and estimated time remaining.
- **Result screens** with success animations, file details (pages processed, processing time, file size), and download buttons.
- **Settings panel** with thumbnail size slider, animation speed control, and reduced-motion option. The theme section has been removed from settings entirely — instead, a theme icon in the top bar cycles through 7 themes: Dark (🌙), Light (☀️), Blue (🟦), Red (🟥), Green (🟩), Yellow (🟨), and White (⬜). Each click advances to the next theme with a toast notification.
- **Session statistics** tracked locally: files processed, pages processed, downloads completed — displayed in the sidebar and footer.
- **Accessibility**: keyboard navigation, ARIA labels, focus-visible outlines, reduced-motion support, and responsive layout for mobile/tablet/desktop.
- **Seven cycling themes** with contrasting button colors: Dark (dark navy with purple accents), Light (lavender white with purple accents), Blue (light blue with blue accents), Red (light pink with red accents), Green (light green with green accents), Yellow (light yellow with amber accents), and White (pure white with black accents). Button text uses `white-space:nowrap` and `flex-shrink:0` to prevent overflow, ensuring labels stay properly contained within their buttons.

---

## Screenshots

### Dashboard / Home

The landing dashboard shows two feature cards (PDF Splitter and PDF Merger) with descriptions, badges, and session statistics. The light theme with blue accents is visible throughout.

![Dashboard](Screenshots/01-dashboard.png)

### Splitter — Upload Screen

The splitter's empty state with a drag-and-drop upload zone. The dashed border and floating icon invite the user to drop a PDF or browse for one.

![Splitter upload](Screenshots/02-splitter-upload.png)

### Splitter — Thumbnails Loaded

After uploading the 8-page "Security and Privacy of Public WiFi" PDF, visual thumbnails render for every page. The summary card shows filename, total pages (8), file size (172.8 KB), orientation, and active page count. The method selector grid appears below with 9 splitting options.

![Splitter thumbnails](Screenshots/03-splitter-thumbnails.png)

### Splitter — Custom Range Method

The Custom Range method selected, with start (2) and end (5) page inputs. This allows extracting a specific page range into a single new PDF.

![Custom range](Screenshots/04-splitter-custom-range.png)

### Splitter — Extract Selected

The Extract Selected method with pages 1, 3, and 5 selected by clicking thumbnails. Selected pages show a teal border and glow effect.

![Extract selected](Screenshots/05-splitter-extract-selected.png)

### Splitter — Drag-to-Reorder

Pages can be reordered by dragging any thumbnail to a new position. Here, page 1 was dragged to position 5 — the other pages shift smoothly to fill the gap. The page numbers update automatically to reflect the new order.

![Reordered](Screenshots/06-splitter-reordered.png)

### Merger — Upload Screen

The merger's empty state with a drag-and-drop zone that accepts multiple PDF files.

![Merger upload](Screenshots/08-merger-upload.png)

### Merger — Files Loaded

Two PDFs loaded (document-A.pdf with 8 pages and document-B.pdf with 4 pages). The summary shows 2 files, 12 total pages, 11.1 KB estimated output, and 0.2s estimated processing time. Files can be reordered via drag-and-drop.

![Merger files](Screenshots/09-merger-files.png)

### Merger — Result

After merging, the result screen shows a success animation with the merged file details: 12 pages processed, 355ms processing time, 7.1 KB total size, and a download button for merged-document.pdf.

![Merger result](Screenshots/10-merger-result.png)

### Splitter — Result

After splitting pages 3–6 from the 8-page Security PDF, the result screen shows 4 pages processed, processing time, output size, and a download button for the extracted PDF.

![Splitter result](Screenshots/07-splitter-result.png)

### Settings Panel

The settings modal with thumbnail size slider, animation speed control, and reduced-motion option. The theme section has been removed entirely — themes are controlled via the theme icon in the top bar which cycles through 7 options.

![Settings](Screenshots/11-settings.png)

### Seven Themes

The theme icon in the top bar cycles through 7 themes. Each click advances to the next theme with a toast notification. All themes use contrasting button colors for readability.

**Dark (🌙)** — Dark navy background with purple/cyan accents:

![Dark theme](Screenshots/theme-01-dark.png)

**Light (☀️)** — Light lavender background with purple accents:

![Light theme](Screenshots/theme-02-light.png)

**Blue (🟦)** — Light blue background with blue accents:

![Blue theme](Screenshots/theme-03-blue.png)

**Red (🟥)** — Light pink background with red accents:

![Red theme](Screenshots/theme-04-red.png)

**Green (🟩)** — Light green background with green accents:

![Green theme](Screenshots/theme-05-green.png)

**Yellow (🟨)** — Light yellow background with amber accents:

![Yellow theme](Screenshots/theme-06-yellow.png)

**White (⬜)** — Pure white background with black accents:

![White theme](Screenshots/theme-07-white.png)

### Dashboard — Final View

The dashboard after completing both split and merge operations, showing updated session statistics in the sidebar.

![Dashboard final](Screenshots/12-dashboard-final.png)

---

## Technologies Used

- HTML5
- CSS3 (glassmorphism, backdrop-filter, CSS custom properties, grid/flexbox layouts, @keyframes animations, responsive breakpoints)
- Vanilla JavaScript (ES6+)
- PDF-lib (embedded in the HTML file for client-side PDF processing)
- Canvas API (for rendering page thumbnails)
- File API and DataTransfer (for drag-and-drop file handling)
- localStorage (for session preferences and statistics)

No external servers or APIs — all PDF processing happens entirely in the browser.

---

## Key Learnings

### Technical Learnings

- **Client-side PDF processing requires the File API and DataTransfer for headless testing.** Standard file input uploads don't trigger change events in headless browsers because `element.files` is read-only. The workaround is fetching the file as a blob, constructing a `File` object, using `DataTransfer.items.add()` to populate the file list, and dispatching a synthetic `change` event. This is the only reliable way to simulate file uploads in automated browser testing.

- **PDF-lib handles page extraction and merging with array operations.** Splitting a PDF to pages 2–5 means copying pages at indices 1–4 (0-indexed) from the source document into a new `PDFDocument`, then saving as bytes. Merging multiple PDFs means iterating through each source document and copying all its pages into a single output document. The library handles all the internal PDF structure — cross-reference tables, stream compression, metadata — so the developer only deals with page-level operations.

- **Thumbnail rendering requires the canvas API and async page rendering.** Each page thumbnail is rendered by loading the PDF with PDF-lib, extracting the page, and drawing it to a canvas element. For large PDFs, this is CPU-intensive and must be done asynchronously with progress indicators to avoid blocking the UI thread. The app uses skeleton loaders (shimmer animation) while thumbnails generate, which is a better UX than a blank grid.

- **Drag-and-drop reordering needs both mouse and touch event handling.** The HTML5 drag-and-drop API (`dragstart`, `dragover`, `drop`) works for desktop but not mobile. A production implementation needs touch event listeners (`touchstart`, `touchmove`, `touchend`) as a fallback. The app's merger file list uses the native API for desktop and would need a polyfill for mobile — a common gap in PDF utility tools.

### Conceptual Learnings

- **Client-side processing is a privacy feature, not just a technical choice.** The app's value proposition isn't speed or convenience — it's that no file ever leaves the user's device. This matters for confidential documents (contracts, medical records, financial statements) where uploading to a third-party server is a non-starter. The "🔒 Offline processing" badge in the UI makes this explicit, and it's the kind of framing that changes how users evaluate a tool.

- **Nine split methods cover distinct use cases, not variations of the same thing.** "Split Every Page" serves archival use cases (one file per page). "Extract Selected" serves research use cases (pull specific pages for reference). "Reverse Extraction" serves printing use cases (print last page first). The variety isn't feature bloat — each method maps to a real workflow that users would otherwise do manually.

- **Visual thumbnails change how users interact with PDFs.** Without thumbnails, splitting is a guessing game — you enter page numbers and hope the output matches what you intended. With thumbnails, users can see exactly which pages they're selecting, which reduces errors and builds confidence in the operation. The visual feedback loop (select thumbnail → see selection state → preview output structure) is what makes the app feel professional rather than utilitarian.

### Personal Reflection

The most impressive thing about this app is how it makes a complex operation feel simple. Splitting a PDF by custom range, extracting selected pages, and merging multiple documents are all operations that traditionally require desktop software like Adobe Acrobat or a trip to an online tool that uploads your files. Having all of this happen in a single HTML file, offline, with visual previews, felt like using a commercial product rather than a practice exercise. The drag-to-reorder feature on the splitter thumbnails was the feature that impressed me most — being able to physically rearrange pages by dragging them, watching the other pages shift smoothly to fill the gap, and seeing the page numbers update automatically is the kind of interaction that makes the tool feel designed rather than functional. The theme toggle icon in the top bar that instantly switches between light and dark modes without opening settings is a small but meaningful UX detail. The light theme with blue accents gave it a clean, professional appearance that I'd genuinely choose over some of the ad-supported online PDF tools I've used.

---

## Project Structure

```
day39/
├── pdf-splitter-merger.html
├── day39.md
├── test-document.pdf (Security and Privacy of Public WiFi — 8 pages)
├── Screenshots/
  ├── 01-dashboard.png
  ├── 02-splitter-upload.png
  ├── 03-splitter-thumbnails.png
  ├── 04-splitter-custom-range.png
  ├── 05-splitter-extract-selected.png
  ├── 06-splitter-reordered.png
  ├── 07-splitter-result.png
  ├── 08-merger-upload.png
  ├── 09-merger-files.png
  ├── 10-merger-result.png
  ├── 11-settings.png
  ├── 12-dashboard-final.png
  ├── theme-01-dark.png
  ├── theme-02-light.png
  ├── theme-03-blue.png
  ├── theme-04-red.png
  ├── theme-05-green.png
  ├── theme-06-yellow.png
  └── theme-07-white.png
```

---

## Final Thoughts

PDF Splitter & Merger does what it set out to do: it provides a complete client-side PDF utility that feels like a commercial product rather than a practice project. The nine splitting methods cover the vast majority of real-world use cases, the merger's drag-and-drop reordering is genuinely useful, and the visual thumbnails make every operation confidence-inspiring rather than blind. The single-file constraint with embedded PDF-lib is impressive — everything runs offline with no server dependency. The one thing I'd improve is mobile drag-and-drop support: the merger works perfectly on desktop but would need touch event handling to match the desktop experience on tablets and phones. That said, for a single-file vanilla JavaScript application, the depth of functionality and the polish of the UI are comparable to many modern commercial PDF platforms while remaining completely offline. Building this project reinforced how much a well-designed prompt can influence architecture, usability, and overall user experience — not just the final appearance of an application.
