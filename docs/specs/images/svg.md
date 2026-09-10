## Requirement: Render SVG images in preview

As a Neovim user, I want an SVG image rendered in the preview
window, so that I can view the image without leaving Neovim or
opening a separate image viewer.

Unlike PNG/JPG, SVG is a text/XML format and can have an open
Neovim buffer. Content resolution (buffer in-memory content vs.
reading from disk) follows the generic rule in
`docs/specs/core/preview-window.md` — no SVG-specific override.
Sync-scroll does not apply (see `docs/specs/core/sync-scroll.md`)
since SVG has no notion of scroll position to sync. Zoom is
deferred, alongside PDF's viewing controls, to
[#3](https://github.com/Dijereo/peep.nvim/issues/3).

### Scenario: Preview an SVG image
Given a supported SVG target file (current buffer or file explorer selection)
When the user presses the preview keybind
Then a preview tab opens showing the image rendered as vector graphics, scaled to fit the tab's width

### Scenario: Preview reflects unsaved SVG buffer changes
Given the SVG file has an open Neovim buffer with unsaved changes
When the user previews it
Then the rendered tab reflects the buffer's current in-memory content, not just the file on disk

### Scenario: Malformed or invalid SVG file
Given a file with a .svg extension that is not valid XML or is not a valid SVG document
When the user previews it
Then an error/notice is shown within the preview window rather than crashing the preview

### Acceptance Criteria
- [ ] Pressing the preview keybind on a supported SVG target file opens/updates a tab rendering the image as vector graphics (not rasterized), scaled to fit the tab's width
- [ ] If the SVG file has an open Neovim buffer, rendered content reflects that buffer's in-memory content, including unsaved changes (per core preview-window behavior)
- [ ] A malformed/invalid SVG file shows an in-window error/notice rather than crashing the preview
- [ ] Zoom controls are out of scope for this requirement (see [#3](https://github.com/Dijereo/peep.nvim/issues/3))
