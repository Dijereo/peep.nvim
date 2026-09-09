## Requirement: Render markdown structural content in preview

As a Neovim user, I want a markdown buffer's structural content rendered
as formatted output in the preview window, so that I can read it in its
intended visual form.

### Scenario: Render markdown structural elements

Given a markdown buffer containing headers, lists, code blocks, images, and tables
When the buffer is previewed
Then headers render as formatted headings, lists as formatted lists, code blocks with syntax highlighting for their language, images inline, and tables as formatted tables

### Scenario: Render YAML front matter

Given a markdown buffer starting with a YAML front matter block delimited by `---` lines as the very first content in the file
When the buffer is previewed
Then the front matter renders as a formatted metadata table of its key/value pairs, matching GitHub's front matter rendering, distinct from the document body

### Scenario: Render horizontal rules

Given a markdown buffer containing a `---` line that is not a leading front matter delimiter
When the buffer is previewed
Then the line renders as a horizontal rule

### Scenario: Render mermaid diagrams

Given a markdown buffer containing a fenced code block tagged mermaid
When the buffer is previewed
Then the block renders as a diagram instead of as plain code

### Scenario: Code block with unsupported language tag

Given a markdown buffer containing a fenced code block tagged with an unrecognized or unsupported language
When the buffer is previewed
Then the block renders as plain, unhighlighted code

### Scenario: Invalid mermaid syntax

Given a markdown buffer containing a fenced mermaid code block with invalid diagram syntax
When the buffer is previewed
Then an error/placeholder renders in place of the diagram, rather than failing silently or crashing the preview

### Acceptance Criteria

- [ ] Headers render as formatted headings matching their level
- [ ] Ordered and unordered lists render as formatted lists
- [ ] Code blocks render with syntax highlighting matching their declared language
- [ ] Code blocks with an unrecognized/unsupported language tag render as plain, unhighlighted code
- [ ] Images render inline
- [ ] Tables render as formatted tables (GFM table syntax)
- [ ] A YAML front matter block is recognized only when it is the very first content in the file, and renders as a formatted metadata table (GitHub-style) distinct from the document body
- [ ] A `---` line that is not a leading front-matter delimiter renders as a horizontal rule
- [ ] Fenced code blocks tagged mermaid render as diagrams
- [ ] A mermaid block with invalid syntax renders an error/placeholder in place of the diagram
- [ ] Target markdown flavor is GitHub-Flavored Markdown (GFM) plus mermaid diagram support

## Requirement: Interactive links in markdown preview

As a Neovim user, I want to click links in the rendered markdown preview,
so that I can navigate to web pages, sections, or referenced files
without manually typing paths or URLs.

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
- [ ] File paths in links are resolved relative to the markdown file's own location
- [ ] Clicking a section link with no matching heading shows an error/notice within the preview window instead of silently doing nothing
- [ ] Clicking a file link to a nonexistent file shows an error/notice within the preview window instead of silently doing nothing
