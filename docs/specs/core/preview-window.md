## Requirement: Shared preview window management

As a Neovim user, I want a single keybind — usable on the current
buffer or on a file selected in a file explorer (neo-tree.nvim,
nvim-tree.nvim, netrw, or oil.nvim) — to open or update a shared
external preview window for that file's supported type, so that I can
preview files consistently, including ones with no open buffer,
without accumulating extra windows.

### Scenario: Open preview window for the first time
Given no preview window is currently open
When the user presses the preview keybind on a supported target file (current buffer or file explorer selection)
Then a new external preview window opens with one tab showing the rendered file

### Scenario: Trigger preview from a file explorer
Given the user has a supported file explorer plugin open with a file selected or under the cursor
When the user presses the preview keybind within the explorer
Then peep.nvim resolves that file as the target and opens/updates its preview tab the same as if triggered from a buffer

### Scenario: Reopen preview updates the same tab
Given a preview window is already open with a tab for target file A
When the user presses the preview keybind again for target file A after its content has changed
Then that tab re-renders and updates in place (no new window or tab opens)

### Scenario: Switch preview to a different target file
Given a preview window is already open with a tab for target file A
When the user presses the preview keybind on a different, supported target file B
Then a tab for target file B is created/switched to in the same window (no new window opens)

### Scenario: Preview reflects unsaved buffer changes
Given the target file has an open Neovim buffer with unsaved changes
When the user previews it
Then the rendered tab reflects the buffer's current in-memory content, not just the file on disk

### Scenario: Preview reads from disk when no buffer exists
Given the target file has no open Neovim buffer (e.g. selected via a file explorer)
When the user previews it
Then the rendered tab reflects the file's current content on disk

### Scenario: Renderer fails to launch
Given the external renderer/window process fails to launch (e.g. missing dependency, no display available)
When the user presses the preview keybind
Then Neovim shows an error message describing the failure

### Scenario: Unsupported file type
Given the target file's type has no supported viewer
When the user presses the preview keybind
Then Neovim shows an error message stating the file format is unsupported

### Acceptance Criteria

- [ ] Pressing the preview keybind with no window open launches a new external window with a rendered tab
- [ ] The preview keybind works both on the current buffer and on a file selected in a supported file explorer (neo-tree.nvim, nvim-tree.nvim, netrw, oil.nvim)
- [ ] Only one external preview window ever exists at a time
- [ ] The window shows one tab per currently-previewed target file
- [ ] Re-invoking on a target file with an existing tab re-renders that tab in place
- [ ] Re-invoking on a target file without an existing tab creates/switches to a new tab
- [ ] If a target file has an open Neovim buffer, rendered content reflects that buffer's in-memory content, including unsaved changes
- [ ] If a target file has no open Neovim buffer, rendered content is read directly from disk
- [ ] Renderer/window launch failure shows a descriptive Neovim error message
- [ ] Unsupported file type shows a Neovim error message naming the issue
- [ ] Any error, warning, alert, or info message displayed within the preview window is non-blocking and dismisses itself automatically, never requiring a click to dismiss
- [ ] The preview window can exist with zero tabs open (e.g. after all tabs have been individually closed); tab-closing behavior itself is specified separately

### Non-Functional Requirements

- [ ] When the Neovim instance that spawned the preview window exits, the system shall terminate the external preview window/process.
- [ ] When the user presses the preview keybind, the system shall open or update the preview window within 500ms.
- [ ] While no display is available (e.g. a headless/SSH session), when the user presses the preview keybind, the system shall show a descriptive Neovim error instead of hanging or crashing.
