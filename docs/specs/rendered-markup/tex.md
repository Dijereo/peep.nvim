## Requirement: Compile and render LaTeX documents in preview

As a Neovim user, I want peep.nvim to compile my LaTeX (.tex) buffer and
display the resulting PDF in the preview window, so that I can see the
rendered document without leaving my editor or running the compiler
myself.

### Scenario: Compile and preview a LaTeX document
Given a .tex buffer with valid LaTeX content
When the user presses the preview keybind
Then peep.nvim compiles the document using the configured LaTeX engine and displays the resulting PDF in the preview tab

### Scenario: Show a compiling indicator
Given the user has pressed the preview keybind on a .tex buffer
When compilation has not yet finished
Then the preview tab shows a visible "compiling" indicator instead of appearing frozen or blank

### Scenario: Recompile on re-invoke
Given a .tex buffer's preview tab is already open
When the user presses the preview keybind again after the buffer's content has changed
Then peep.nvim recompiles the document and updates the same tab with the new PDF, per core preview-window behavior

### Scenario: Compile a sub-file with a declared root document
Given a .tex buffer contains a root-file marker comment (e.g. `% !TEX root = main.tex`) referencing another file in the same project
When the user presses the preview keybind
Then peep.nvim compiles the declared root file instead of the current buffer, and displays its resulting PDF

### Scenario: Compile a sub-file with no declared root
Given a .tex buffer has no root-file marker and is not itself a complete compilable document
When the user presses the preview keybind
Then compilation fails and an error/notice describing the failure is shown within the preview window

### Scenario: Compilation fails
Given a .tex buffer that fails to compile (syntax error, missing package, etc.)
When the user presses the preview keybind
Then no PDF is displayed and an error/notice with the compiler's error output is shown within the preview window

### Scenario: LaTeX engine not installed
Given the configured LaTeX engine is not installed or not found
When the user presses the preview keybind
Then Neovim shows an error message stating the engine could not be found

### Acceptance Criteria
- [ ] Pressing the preview keybind on a .tex buffer compiles it with a configurable LaTeX engine/command and displays the resulting PDF in the preview tab
- [ ] While compilation is in progress, the preview tab shows a visible compiling indicator
- [ ] Re-invoking the keybind after changes recompiles and updates the same tab (per core preview-window behavior)
- [ ] A `% !TEX root = <file>` marker comment causes peep.nvim to compile the declared root file instead of the current buffer
- [ ] A sub-file with no root marker that fails to compile standalone shows an in-window error/notice
- [ ] Any compilation failure shows an in-window error/notice including the compiler's error output, without crashing the preview
- [ ] A missing/not-found LaTeX engine shows a Neovim error message (per core's launch-failure handling)
- [ ] Compilation artifacts (.aux, .log, .synctex.gz, etc.) are written to an isolated temp/build directory, not alongside the source file
- [ ] Compilation enables SyncTeX output so that source-to-PDF sync scroll (see core/sync-scroll.md) is possible

### Non-Functional Requirements
- [ ] The LaTeX engine/compilation command shall be configurable, not hardcoded to a single engine.
- [ ] Compilation shall run asynchronously so it does not block the Neovim UI while the document compiles.
- [ ] The core preview-window's 500ms open/update latency target does not apply to LaTeX compilation, given compilation inherently takes longer.
