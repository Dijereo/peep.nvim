## Requirement: Render Word documents in preview

As a Neovim user, I want a Word (.docx) file's content rendered in
the preview window, so that I can view the document without leaving
Neovim or opening a separate word processor.

docx is converted and displayed per
`docs/specs/office-docs/office-conversion.md` (conversion, converting
indicator, missing-converter and conversion-failure handling, PDF
display). No docx-specific rendering behavior beyond that generic
requirement.

### Acceptance Criteria
- [ ] docx files preview per `docs/specs/office-docs/office-conversion.md`
