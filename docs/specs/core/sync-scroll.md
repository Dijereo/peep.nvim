## Requirement: Toggle sync scroll for source-backed preview

As a Neovim user, I want to press a keybind to toggle scroll syncing
between a source buffer and its open rendered preview tab, so that both
views can stay aligned while I read or navigate either one.

Applies to file types where the preview is rendered from a distinct
source buffer with a meaningful text-position-to-rendered-position
mapping: markdown, Quarto (qmd), LaTeX (tex), and Jupyter notebooks
(ipynb). Does not apply to formats with no separate source buffer
(e.g. standalone PDF, images), nor to standalone Mermaid diagrams
(.mmd) — the whole buffer compiles to one diagram with no
line-to-region mapping to sync against (see
`docs/specs/rendered-markup/mermaid.md`).

### Scenario: Turn on sync scroll

Given a supported source buffer's preview tab is open with scroll sync off
When the user presses the sync-scroll keybind
Then scroll sync turns on, and the preview tab immediately scrolls to match the buffer's current position

### Scenario: Turn off sync scroll

Given scroll sync is on for a buffer's preview tab
When the user presses the sync-scroll keybind again
Then scroll sync turns off, and both views keep their current scroll positions independently

### Scenario: Bidirectional tracking while sync is on

Given scroll sync is on for a buffer's preview tab
When the user scrolls either the source buffer or the preview tab
Then the other view scrolls to match

### Scenario: Toggle sync scroll with no preview tab open

Given a supported buffer has no preview tab currently open
When the user presses the sync-scroll keybind
Then Neovim shows an info/warning message and no sync state changes

### Scenario: Sync scroll persists across sessions

Given scroll sync was previously turned on for a file
When Neovim and the preview window are closed, then the file is opened and previewed again in a new session
Then scroll sync is on for that file's preview tab without the user needing to toggle it again

### Acceptance Criteria

- [ ] Applies to source-backed formats: markdown, Quarto (qmd), LaTeX (tex), Jupyter notebooks (ipynb) — not standalone formats with no source buffer (e.g. PDF, images), nor standalone Mermaid diagrams (.mmd)
- [ ] Pressing the sync-scroll keybind on a buffer with an open preview tab toggles scroll sync on/off for that tab
- [ ] Turning sync on immediately aligns the preview tab's scroll position to the buffer's current position
- [ ] Turning sync off leaves both views at their current scroll positions, no longer tracking each other
- [ ] While sync is on, scrolling either the buffer or the preview tab scrolls the other to match
- [ ] Pressing the sync-scroll keybind on a buffer with no open preview tab shows an info/warning message in Neovim and has no other effect
- [ ] Scroll sync state is stored per file and persists across Neovim/preview window restarts

### Non-Functional Requirements

- [ ] The sync-scroll keybind shall be configurable/remappable, not hardcoded.
