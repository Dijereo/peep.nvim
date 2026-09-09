## Requirement: Render Quarto document content in preview

As a Neovim user, I want a Quarto (.qmd) buffer's content rendered as
formatted output in the preview window, so that I can read it in its
intended visual form.

Quarto is a markdown superset: standard markdown elements render per
`docs/specs/rendered-markup/markdown.md`'s rendering requirement
(headers, lists, code blocks, images, tables, front matter, horizontal
rules, mermaid diagrams, strikethrough, task lists). This requirement
covers only what's specific to Quarto syntax on top of that.

peep.nvim does not execute code chunks itself; rendering of
externally-produced execution outputs (plots, tables, printed results
from another tool/plugin) is deferred to
[#2](https://github.com/Dijereo/peep.nvim/issues/2).

### Scenario: Reuse markdown structural rendering
Given a Quarto buffer containing standard markdown elements
When the buffer is previewed
Then those elements render as specified for markdown

### Scenario: Render Quarto code chunks
Given a Quarto buffer containing a fenced code chunk tagged with a language in curly braces (e.g. ` ```{python} `)
When the buffer is previewed
Then the chunk renders as syntax-highlighted code for that language, including any chunk option comment lines (`#| key: value`) as part of the code

### Scenario: Render callout blocks
Given a Quarto buffer containing a callout div (`::: {.callout-note}`, `.callout-tip`, `.callout-warning`, `.callout-important`, or `.callout-caution`)
When the buffer is previewed
Then the block renders as a styled highlight box matching its callout type

### Scenario: Render an unrecognized callout type
Given a Quarto buffer containing a callout div with a type that doesn't match a known callout type
When the buffer is previewed
Then the block renders as a generic styled box rather than erroring

### Acceptance Criteria
- [ ] Quarto documents render standard markdown elements per markdown.md's rendering requirement
- [ ] Code chunks tagged with a language in curly braces (e.g. `{python}`, `{r}`) render as syntax-highlighted code for that language
- [ ] Chunk option comment lines (`#| ...`) render as part of the code block, unmodified
- [ ] Callout divs (`.callout-note`/`tip`/`warning`/`important`/`caution`) render as styled highlight boxes matching their type
- [ ] An unrecognized callout type renders as a generic styled box rather than erroring
- [ ] Execution outputs produced by other tools/plugins are not rendered by this requirement (see [#2](https://github.com/Dijereo/peep.nvim/issues/2))

Interactive links and task list checkboxes for Quarto previews are
specified generically in `docs/specs/rendered-markup/interactivity.md`
(shared with markdown).
