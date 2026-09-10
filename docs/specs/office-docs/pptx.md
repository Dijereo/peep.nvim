## Requirement: Render PowerPoint presentations in preview

As a Neovim user, I want a PowerPoint (.pptx) file's content
rendered in the preview window, so that I can view the presentation
without leaving Neovim or opening a separate presentation
application.

pptx is converted and displayed per
`docs/specs/office-docs/office-conversion.md` (conversion, converting
indicator, missing-converter and conversion-failure handling, PDF
display).

### Scenario: Render a multi-slide presentation
Given a pptx file with multiple slides
When the user previews it
Then each slide renders as one PDF page, in slide order

### Acceptance Criteria
- [ ] pptx files preview per `docs/specs/office-docs/office-conversion.md`
- [ ] A multi-slide presentation renders all slides, in slide order, one slide per page
