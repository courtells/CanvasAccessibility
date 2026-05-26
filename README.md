# Canvas Accessibility Checker

A self-contained, browser-based accessibility auditing tool for exported Canvas LMS course shells. Faculty or instructional designers upload a `.imscc` or `.zip` export from Canvas and receive a detailed WCAG 2.1 AA audit report — with no server, no login, and no data leaving the browser.

---

## Features

- **Zero setup** — a single HTML file. Open it in any modern browser and it works.
- **Drag-and-drop upload** of `.imscc` or `.zip` Canvas course exports
- **Optional Canvas URL** — paste your course URL to generate direct "Open in Canvas" links alongside each finding
- **13 WCAG 2.1 AA checks** across text, images, media, and structure (see [Checks](#checks) below)
- **Two result views**
  - *By Module* — findings grouped by Canvas module and page, matching your course structure
  - *By Check Type* — findings grouped by accessibility category across all pages
- **Inline fix checklist** — check off issues as you fix them; progress persists while the page is open
- **Expand all with issues** button to open every accordion containing unfixed items at once
- **Filter bar** — show all results, issues only, manual checks only, or passed checks
- **PDF export** — clean, printable audit report with checkboxes for each issue
- **Spreadsheet export** — `.xlsx` file with all findings, locations, and fix suggestions; ready for sharing or tracking in Excel or Google Sheets
- **Suggestions written for faculty** — every fix recommendation uses plain language and Canvas-specific instructions, not developer jargon

---

## Getting Started

1. **Export your Canvas course**
   In Canvas, go to *Settings → Export Course Content → Course* and download the `.imscc` file.

2. **Open the tool**
   Open `index.html` in any modern browser (Chrome, Firefox, Edge, or Safari).

3. **Upload your export**
   Drag the `.imscc` file onto the upload zone, or click to browse for it.

4. **Optionally enter your Canvas URL**
   Paste the URL from your browser while viewing any page in your Canvas course (e.g. `https://canvas.instructure.com/courses/123456`). This enables "Open in Canvas" links on each finding.

5. **Run the audit**
   Click *Run accessibility audit*. Results appear in seconds.

6. **Review and fix**
   - Switch between *By Module* and *By Check Type* views using the toggle at the top of the results
   - Click any issue's checkbox to mark it as fixed — the progress bar at the top tracks your overall progress
   - Use *Expand all with issues* to open all relevant accordions at once
   - Download a PDF or spreadsheet to share with colleagues or track fixes over time

---

## Checks

The tool audits for the following, organised by WCAG 2.1 success criterion:

| Check | WCAG Criterion | Type |
|---|---|---|
| Page titles | 2.4.2 Level A | Automated |
| Heading structure & labels | 1.3.1 Level A / 2.4.6 Level AA | Automated |
| Logical reading order | 1.3.2 Level A / 2.4.3 Level A | Manual |
| Descriptive link text | 2.4.4 Level A | Automated |
| Color contrast (inline styles) | 1.4.3 Level AA | Automated + Manual |
| Image alt text | 1.1.1 Level A / 1.4.5 Level AA | Automated |
| Long descriptions for complex images | 1.1.1 Level A | Manual |
| Accessible tables | 1.3.1 Level A | Automated |
| Video captions | 1.2.2 Level A | Manual |
| Transcripts for audio/video | 1.2.1 Level A | Manual |
| Audio descriptions | 1.2.3 Level A / 1.2.5 Level AA | Manual |
| Seizure triggers — flashing content | 2.3.1 Level A | Manual |
| Animated/auto-updating content | 2.2.2 Level A | Manual |

**Automated** checks scan the exported HTML directly. **Manual** checks flag items that require a human to verify in the live Canvas environment, with plain-language guidance on what to look for.

> **Note on color contrast:** The tool checks inline `style` attributes only. Colors applied via Canvas themes or editor CSS classes are flagged for manual review, since they are not present in the exported HTML.

---

## Exports

### PDF Report
A formatted, printable document organised by the active view (By Module or By Check Type). Each finding includes its status, WCAG criterion, description, page location, and a fix suggestion. Includes empty checkboxes so the report can be used as a physical checklist.

### Spreadsheet (.xlsx)
An Excel workbook with three sheets:
- **Summary** — overall pass/fail/manual counts and a per-module breakdown
- **By Module** — one row per finding, with module, page, check type, WCAG reference, issue description, location, and fix suggestion
- **By Check Type** — the same findings reorganised by accessibility category

---

## Dependencies

All dependencies are loaded from CDN at runtime. No `npm install` or build step is required.

| Library | Version | Purpose |
|---|---|---|
| [JSZip](https://stuk.github.io/jszip/) | 3.10.1 | Reads the `.imscc` / `.zip` archive in the browser |
| [jsPDF](https://github.com/parallax/jsPDF) | 2.5.1 | Generates the downloadable PDF report |
| [SheetJS (xlsx)](https://sheetjs.com/) | 0.18.5 | Generates the downloadable `.xlsx` spreadsheet |
| [DM Sans / DM Mono](https://fonts.google.com/specimen/DM+Sans) | — | UI typography via Google Fonts |

---

## Architecture

The entire tool is a **single HTML file** with no build process, no framework, and no backend.

```
index.html
├── <style>          — All CSS; design tokens as CSS custom properties
├── HTML             — Static shell: upload zone, results container, modals
└── <script>
    ├── State & file handling
    ├── Audit runner (async, with progress reporting)
    ├── Manifest parser (reads imsmanifest.xml for module/page structure)
    ├── Link text helpers
    ├── Color contrast engine (WCAG luminance calculations)
    ├── Main audit (performAudit — per-page HTML analysis)
    ├── Rendering — module view, check-type view
    ├── PDF export (pdfModuleView, pdfChecksView)
    ├── Spreadsheet export
    └── Inline fix checklist (clChecked state, progress bar)
```

### Key data flow

1. User uploads a `.imscc` / `.zip` file
2. JSZip unpacks the archive in memory
3. `parseManifest()` reads `imsmanifest.xml` to build the module/page map and ordering
4. `performAudit()` iterates over every HTML page, runs all checks, and returns a `auditData` object containing both a `byModule` map and a flat `sections` array
5. Each issue object is stamped with a unique `_clId` at this point, used to track checklist state across both views
6. `renderResults()` dispatches to `renderModuleView()` or `renderSections()` based on the active view toggle

---

## Limitations

- **Canvas export format only** — the tool reads `.imscc` files exported from Canvas. It will not audit live Canvas pages directly, nor exports from other LMS platforms.
- **Static analysis** — the audit parses the exported HTML. It cannot run JavaScript, render pages, or detect issues that only appear in a live browser context (e.g. dynamic content, keyboard focus traps).
- **Color contrast** — only inline `style` attributes are checked automatically. Canvas theme colors and rich-text editor CSS classes require manual verification on the live course.
- **Media checks are manual** — captions, transcripts, and audio descriptions cannot be verified from static file analysis. The tool identifies media files and prompts the user to check them.
- **No persistent storage** — checklist progress is held in memory for the current browser session only. Closing or refreshing the page resets all progress.

---

## Browser Compatibility

Tested in current versions of Chrome, Firefox, Edge, and Safari. Requires a browser that supports:
- `FileReader` / `File` API
- `DOMParser`
- ES2018+ (async/await, optional chaining)

Internet Explorer is not supported.

---

## Contributing

Contributions are welcome. Some areas where the tool could be improved:

- Additional automated checks (e.g. `lang` attribute on `<html>`, form label associations, `<iframe>` titles)
- Support for other LMS export formats
- Persistent checklist storage using `localStorage` (for use outside Claude.ai)
- Improved detection of images of text
- Unit tests for the contrast engine and audit logic

To contribute, fork the repository, make your changes to `index.html`, and open a pull request with a description of what was changed and why.

---

## License

MIT License. See `LICENSE` for details.

---

## AI Use Declaration

This project was built collaboratively with an AI assistant. See [AI_DECLARATION.md](./AI_DECLARATION.md) for a full account of how AI was used, what the human contributor directed, and why this is being disclosed.

---

## Acknowledgements

Developed to support accessible course design for college faculty. Accessibility guidance references the [Web Content Accessibility Guidelines (WCAG) 2.1](https://www.w3.org/TR/WCAG21/) and resources from [WebAIM](https://webaim.org), [Arizona State University Accessibility](https://accessibility.asu.edu), and the [DCMP Captioning Key](https://dcmp.org/learn/captioningkey).
