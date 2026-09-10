## Requirement: Convert and render Office documents in preview

As a Neovim user, I want docx, xlsx, and pptx files converted to PDF
and displayed in the preview window, so that I can view Office
documents without leaving Neovim or opening a separate application.

docx/xlsx/pptx are binary formats with no meaningful Neovim text
buffer, so target file content is always read from disk — the
generic no-buffer rule in `docs/specs/core/preview-window.md` applies
without a format-specific override. Sync-scroll does not apply (see
`docs/specs/core/sync-scroll.md`) since there's no source buffer to
sync from. Once converted, the resulting PDF renders per
`docs/specs/office-docs/pdf.md` (page navigation/zoom/search
controls specified generically in
`docs/specs/core/window-keybinds.md`).

### Scenario: Convert and preview a supported Office document
Given a supported docx, xlsx, or pptx target file (current buffer or file explorer selection)
When the user presses the preview keybind
Then peep.nvim converts the file to PDF using the configured converter and displays the resulting PDF in the preview tab

### Scenario: Show a converting indicator
Given the user has pressed the preview keybind on a docx, xlsx, or pptx target file
When conversion has not yet finished
Then the preview tab shows a visible "converting" indicator instead of appearing frozen or blank

### Scenario: Conversion tool not installed
Given the configured converter is not installed or not found
When the user presses the preview keybind
Then Neovim shows an error message stating the converter could not be found

### Scenario: Conversion fails
Given a docx, xlsx, or pptx file that is malformed, corrupted, or fails to convert
When the user previews it
Then no PDF is displayed and an error/notice describing the failure is shown within the preview window

### Acceptance Criteria
- [ ] Pressing the preview keybind on a supported docx/xlsx/pptx target file converts it with a configurable converter/command and displays the resulting PDF in the preview tab
- [ ] While conversion is in progress, the preview tab shows a visible converting indicator
- [ ] Re-invoking the keybind reconverts the file's current on-disk content and updates the same tab (per core preview-window behavior)
- [ ] A missing/not-found converter shows a Neovim error message (per core's launch-failure handling)
- [ ] Any conversion failure shows an in-window error/notice rather than crashing the preview
- [ ] Conversion output (the intermediate PDF and any converter artifacts) is written to an isolated temp/build directory, not alongside the source file

### Non-Functional Requirements
- [ ] The converter/command shall be configurable, not hardcoded to a single tool (e.g. LibreOffice headless is a reasonable default, but not the only option).
- [ ] Conversion shall run asynchronously so it does not block the Neovim UI while the document converts.
- [ ] The core preview-window's 500ms open/update latency target does not apply to Office document conversion, given conversion inherently takes longer.
