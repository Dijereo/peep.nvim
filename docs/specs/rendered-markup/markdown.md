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

### Scenario: Render strikethrough and task lists

Given a markdown buffer containing strikethrough text (`~~text~~`) and task list items (`- [ ]` / `- [x]`)
When the buffer is previewed
Then strikethrough text renders with a line through it, and task list items render with checkbox indicators reflecting their checked state

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
- [ ] Strikethrough text renders with a line through it
- [ ] Task list items render with checkbox indicators matching their checked/unchecked state
- [ ] Target markdown flavor is GitHub-Flavored Markdown (GFM) plus mermaid diagram support

Interactive links and task list checkboxes for markdown previews are
specified generically in `docs/specs/rendered-markup/interactivity.md`
(shared with other markdown-syntax formats, e.g. Quarto).
