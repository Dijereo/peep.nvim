## Requirement: Render PDF documents in preview

As a Neovim user, I want to preview a PDF file's rendered content in
the preview window, so that I can view PDF documents without leaving
Neovim or opening a separate PDF viewer.

PDF is a standalone format with no source buffer — sync-scroll does
not apply (see `docs/specs/core/sync-scroll.md`). Content resolution
(buffer in-memory content vs. reading from disk) follows the generic
rule in `docs/specs/core/preview-window.md`. Viewing controls (page
navigation, zoom, text search) are specified generically in
`docs/specs/core/window-keybinds.md`.

### Scenario: Preview a PDF file
Given a supported PDF target file (current buffer or file explorer selection)
When the user presses the preview keybind
Then a preview tab opens showing the PDF's rendered pages

### Scenario: Malformed or corrupt PDF file
Given a PDF file that is not a valid PDF or is corrupted
When the user previews it
Then an error/notice is shown within the preview window rather than crashing the preview

### Acceptance Criteria
- [ ] Pressing the preview keybind on a supported PDF target file opens/updates a tab rendering the PDF's pages
- [ ] A malformed/corrupt PDF file shows an in-window error/notice rather than crashing the preview
- [ ] Page navigation, zoom, and text search controls are specified generically in `docs/specs/core/window-keybinds.md`
