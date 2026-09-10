## Requirement: Render Jupyter notebook content in preview

As a Neovim user, I want a Jupyter notebook (.ipynb) file's cells and
stored outputs rendered as formatted output in the preview window, so
that I can read the notebook's content and results without executing
it myself.

Notebook outputs are read directly from the .ipynb file's own stored
data (nbformat embeds outputs alongside each executed cell) — no
execution or external tooling is required, unlike Quarto.

Markdown cells render per `docs/specs/rendered-markup/markdown.md` and
`docs/specs/rendered-markup/interactivity.md` (links, task checkboxes)
— reused, not duplicated.

### Scenario: Render an executed code cell
Given a notebook code cell that has been executed (has an execution count)
When the notebook is previewed
Then the cell renders as syntax-highlighted code for the notebook's kernel language, prefixed with its execution count (e.g. `In [3]:`)

### Scenario: Render an unexecuted code cell
Given a notebook code cell that has not been executed (no execution count)
When the notebook is previewed
Then the cell renders as syntax-highlighted code without an execution count prompt

### Scenario: Render markdown cells
Given a notebook markdown cell
When the notebook is previewed
Then the cell renders per the markdown rendering and interactivity requirements

### Scenario: Render a raw cell
Given a notebook raw cell
When the notebook is previewed
Then the cell renders as a plain, unstyled text block with no syntax highlighting or markdown processing

### Scenario: Render stream/text output
Given a code cell with stored stdout/stderr stream output
When the notebook is previewed
Then the output renders as plain text beneath the cell

### Scenario: Render rich display output
Given a code cell with stored rich display data (e.g. image/png, image/jpeg, or text/html)
When the notebook is previewed
Then images render inline and HTML output renders as formatted content (e.g. tables)

### Scenario: Render error output
Given a code cell with a stored error/exception output
When the notebook is previewed
Then the traceback renders as a highlighted error block beneath the cell

### Scenario: Cell with no output
Given a code cell with no stored output
When the notebook is previewed
Then no output section renders beneath the cell

### Scenario: Truncate large output
Given a code cell with stored output exceeding a size threshold (very long text or a very large image)
When the notebook is previewed
Then the output is truncated/scaled to fit, with a visible indicator that it was truncated

### Scenario: Unsupported kernel language
Given a notebook whose declared kernel language has no supported syntax highlighter
When the notebook is previewed
Then code cells render as plain, unhighlighted code

### Scenario: Malformed notebook file
Given a .ipynb file that is not valid JSON or does not conform to the nbformat schema
When the buffer is previewed
Then an error/notice is shown within the preview window rather than crashing the preview

### Acceptance Criteria
- [ ] Code cells render as syntax-highlighted code for the notebook's declared kernel language
- [ ] Executed code cells show their execution count prompt (e.g. `In [3]:`); unexecuted cells show no execution count
- [ ] Markdown cells render per rendered-markup/markdown.md and rendered-markup/interactivity.md
- [ ] Raw cells render as plain, unstyled text with no processing
- [ ] Stream/stdout/stderr output renders as plain text beneath its cell
- [ ] Rich display outputs (image/png, image/jpeg, text/html) render inline (images) or as formatted content (HTML)
- [ ] Error/exception output renders as a highlighted error block beneath its cell
- [ ] Cells with no stored output show no output section
- [ ] Output exceeding a size threshold is truncated/scaled with a visible indicator, rather than rendered in full or crashing the preview
- [ ] An unsupported kernel language falls back to plain, unhighlighted code
- [ ] A malformed/invalid .ipynb file shows an in-window error/notice rather than crashing the preview

### Non-Functional Requirements
- [ ] Text output beyond approximately 500 lines shall be truncated with a visible "truncated" indicator rather than rendered in full.
- [ ] Large images shall be scaled to fit the preview window's width rather than rendered at full/native size.
