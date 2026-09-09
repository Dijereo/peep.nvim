## Requirement: Shared preview window management

As a Neovim user, I want a single keybind to open or update a shared
external preview window for the current buffer's supported file type,
so that I can preview files consistently without accumulating extra
windows.

### Scenario: Open preview window for the first time

Given no preview window is currently open
When the user presses the preview keybind on a buffer of a supported file type
Then a new external preview window opens with one tab showing the rendered buffer

### Scenario: Reopen preview updates the same tab

Given a preview window is already open with a tab for buffer A
When the user presses the preview keybind again after buffer A's content has changed
Then that tab re-renders and updates in place (no new window or tab opens)

### Scenario: Switch preview to a different buffer

Given a preview window is already open with a tab for buffer A
When the user presses the preview keybind on a different, supported buffer B
Then a tab for buffer B is created/switched to in the same window (no new window opens)

### Scenario: Preview reflects unsaved changes

Given a buffer has unsaved changes
When the user presses the preview keybind
Then the rendered tab reflects the buffer's current in-memory content, not just the file on disk

### Scenario: Renderer fails to launch

Given the external renderer/window process fails to launch (e.g. missing dependency, no display available)
When the user presses the preview keybind
Then Neovim shows an error message describing the failure

### Scenario: Unsupported file type

Given the current buffer's file type has no supported viewer
When the user presses the preview keybind
Then Neovim shows an error message stating the file format is unsupported

### Acceptance Criteria

- [ ] Pressing the preview keybind with no window open launches a new external window with a rendered tab
- [ ] Only one external preview window ever exists at a time
- [ ] The window shows one tab per currently-previewed buffer
- [ ] Re-invoking on a buffer with an existing tab re-renders that tab in place
- [ ] Re-invoking on a buffer without an existing tab creates/switches to a new tab
- [ ] Rendered content reflects in-memory buffer content, including unsaved changes
- [ ] Renderer/window launch failure shows a descriptive Neovim error message
- [ ] Unsupported file type shows a Neovim error message naming the issue
- [ ] Any error, warning, alert, or info message displayed within the preview window is non-blocking and dismisses itself automatically, never requiring a click to dismiss
- [ ] The preview window can exist with zero tabs open (e.g. after all tabs have been individually closed); tab-closing behavior itself is specified separately

### Non-Functional Requirements

- [ ] When the Neovim instance that spawned the preview window exits, the system shall terminate the external preview window/process.
- [ ] When the user presses the preview keybind, the system shall open or update the preview window within 500ms.
- [ ] While no display is available (e.g. a headless/SSH session), when the user presses the preview keybind, the system shall show a descriptive Neovim error instead of hanging or crashing.
