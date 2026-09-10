## Requirement: Render raster images in preview

As a Neovim user, I want a PNG or JPG image rendered in the preview
window, so that I can view the image without leaving Neovim or
opening a separate image viewer.

PNG and JPG are binary formats with no meaningful Neovim text
buffer — target file content is always read from disk, per the
generic no-buffer rule in `docs/specs/core/preview-window.md` (no
format-specific override). Sync-scroll does not apply (see
`docs/specs/core/sync-scroll.md`) since there's no source buffer.
Zoom is specified generically, alongside PDF's viewing controls, in
`docs/specs/core/window-keybinds.md`.

### Scenario: Preview a raster image
Given a supported PNG or JPG target file (current buffer or file explorer selection)
When the user presses the preview keybind
Then a preview tab opens showing the image, scaled to fit the tab's width

### Scenario: Scale a large image to fit
Given an image whose native dimensions exceed the preview tab's width
When the image is previewed
Then it is scaled down to fit the tab's width while preserving aspect ratio

### Scenario: Malformed or corrupt image file
Given a file with a .png or .jpg extension that is not a valid image or is corrupted
When the user previews it
Then an error/notice is shown within the preview window rather than crashing the preview

### Acceptance Criteria
- [ ] Pressing the preview keybind on a supported PNG/JPG target file opens/updates a tab rendering the image
- [ ] Images wider than the preview tab are scaled down to fit the tab's width, preserving aspect ratio
- [ ] A malformed/corrupt image file shows an in-window error/notice rather than crashing the preview
- [ ] Zoom controls are specified generically in `docs/specs/core/window-keybinds.md`
