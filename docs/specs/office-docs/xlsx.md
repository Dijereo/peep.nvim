## Requirement: Render Excel spreadsheets in preview

As a Neovim user, I want an Excel (.xlsx) file's content rendered in
the preview window, so that I can view the spreadsheet without
leaving Neovim or opening a separate spreadsheet application.

xlsx is converted and displayed per
`docs/specs/office-docs/office-conversion.md` (conversion, converting
indicator, missing-converter and conversion-failure handling, PDF
display).

### Scenario: Render a multi-sheet workbook
Given an xlsx file with multiple sheets
When the user previews it
Then each sheet renders as one or more PDF pages, in sheet order, using the converter's default print/page setup for that sheet (no forced fit-to-page or print-area correction)

### Acceptance Criteria
- [ ] xlsx files preview per `docs/specs/office-docs/office-conversion.md`
- [ ] A multi-sheet workbook renders all sheets, in sheet order, using the converter's default page setup per sheet
