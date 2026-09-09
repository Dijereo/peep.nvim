Shared interactive-preview requirements for all markdown-syntax formats
(currently: markdown, Quarto). Referenced from each format's own spec
file rather than duplicated.

## Requirement: Interactive links in preview

As a Neovim user, I want to click links in the rendered preview, so
that I can navigate to web pages, sections, or referenced files without
manually typing paths or URLs.

### Scenario: Click a web link

Given the rendered preview contains a link to a web URL (http/https)
When the user clicks the link
Then the link opens in the user's default system browser

### Scenario: Click a section link

Given the rendered preview contains a link to a heading anchor within the same document
When the user clicks the link
Then the preview scrolls to that section

### Scenario: Click a file link

Given the rendered preview contains a link to another file, referenced by a relative or absolute path
When the user clicks the link
Then the target file opens as a buffer in the Neovim instance that spawned the preview window

### Scenario: Click a section link with no matching heading

Given the rendered preview contains a link to a heading anchor that does not match any heading in the document
When the user clicks the link
Then no navigation occurs and an error/notice is shown within the preview window

### Scenario: Click a file link to a nonexistent file

Given the rendered preview contains a link to a file that does not exist
When the user clicks the link
Then an error/notice is shown within the preview window instead of opening a buffer

### Acceptance Criteria

- [ ] Web links (http/https) open in the user's default system browser
- [ ] Section/anchor links scroll the preview to the matching heading within the same document
- [ ] File links open the target file as a buffer in the Neovim instance that spawned the preview window
- [ ] File paths in links are resolved relative to the document's own location
- [ ] Clicking a section link with no matching heading shows an error/notice within the preview window instead of silently doing nothing
- [ ] Clicking a file link to a nonexistent file shows an error/notice within the preview window instead of silently doing nothing

## Requirement: Interactive task list checkboxes in preview

As a Neovim user, I want to click a task list checkbox in the rendered
preview, so that I can toggle it without switching back to the editor.

### Scenario: Check an unchecked task
Given the preview shows an unchecked task list item
When the user clicks its checkbox
Then the checkbox displays as checked and the corresponding line in the source buffer updates to a checked task item

### Scenario: Uncheck a checked task
Given the preview shows a checked task list item
When the user clicks its checkbox
Then the checkbox displays as unchecked and the corresponding line in the source buffer updates to an unchecked task item

### Scenario: Click a checkbox on a read-only buffer
Given the source buffer is read-only or otherwise non-modifiable
When the user clicks a task checkbox in the preview
Then an error/notice is shown within the preview window and the checkbox state does not change

### Acceptance Criteria
- [ ] Clicking an unchecked task's checkbox in the preview checks it and updates the corresponding line in the source buffer
- [ ] Clicking a checked task's checkbox in the preview unchecks it and updates the corresponding line in the source buffer
- [ ] The buffer update is in-memory (reflected like any other unsaved change), not an automatic save to disk
- [ ] Clicking a checkbox on a read-only/non-modifiable buffer shows an error/notice within the preview window and leaves the checkbox unchanged
