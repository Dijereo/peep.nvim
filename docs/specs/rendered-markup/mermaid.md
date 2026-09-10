## Requirement: Render standalone Mermaid diagram files in preview

As a Neovim user, I want a standalone Mermaid diagram (.mmd) file
rendered as a diagram in the preview window, so that I can view it
without leaving Neovim.

Unlike markdown/Quarto, where a mermaid block is one fenced code
block among other content, a .mmd file's entire content is a single
Mermaid diagram source. Rendering (and invalid-syntax handling)
reuses the same behavior specified for mermaid blocks in
`docs/specs/rendered-markup/markdown.md`.

Sync-scroll does not apply (see `docs/specs/core/sync-scroll.md`) —
the whole file compiles to one diagram with no line-to-region
mapping to sync against. Zoom is specified generically, alongside
PDF/image zoom, in `docs/specs/core/window-keybinds.md`.

### Scenario: Render a Mermaid diagram file
Given a .mmd file containing valid Mermaid diagram syntax
When the user previews it
Then the preview tab renders the file's entire content as a single diagram

### Scenario: Invalid Mermaid syntax
Given a .mmd file containing invalid Mermaid diagram syntax
When the user previews it
Then an error/placeholder renders in the preview tab rather than failing silently or crashing the preview

### Acceptance Criteria
- [ ] Previewing a .mmd file renders its entire content as a single Mermaid diagram
- [ ] Invalid Mermaid syntax renders an error/placeholder in the preview tab rather than crashing the preview
