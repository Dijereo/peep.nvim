## Requirement: In-window keybinds for tab management, navigation, zoom, and search

As a Neovim user, I want a consistent set of keybinds within the
external preview window itself — independent of Neovim's own
keybinds — for managing tabs, scrolling/paging through content,
zooming, and searching text, so that I can navigate previewed
content without needing a mouse or switching back to Neovim.

These keybinds operate on the external preview window described in
`docs/specs/core/preview-window.md`. They resolve that requirement's
deferred "tab-closing behavior itself is specified separately" note,
and resolve [#3](https://github.com/Dijereo/peep.nvim/issues/3)
(PDF/image viewing controls).

### Scenario: Close a tab while others remain
Given the preview window has more than one open tab (regular and/or the help tab)
When the user presses `q` while a tab is active
Then that tab closes, and the tab to its right becomes active (or the new rightmost tab, if the closed tab was rightmost)

### Scenario: Close the last regular tab
Given exactly one regular (non-help) tab is open, and the help tab is not currently open
When the user presses `q` on that tab
Then it closes and the keybind help tab automatically opens and becomes active, rather than leaving the window with zero tabs

### Scenario: Close the last regular tab when the help tab is already open
Given exactly one regular tab is open alongside an already-open help tab
When the user presses `q` on the regular tab
Then it closes and the existing help tab becomes active (not duplicated)

### Scenario: Close the window via the help tab
Given the help tab is the only open tab
When the user presses `q`
Then the entire preview window closes

### Scenario: Close the window directly
Given the preview window is open, with any number of tabs
When the user presses `C-w`
Then the entire preview window closes immediately, regardless of how many tabs are open

### Scenario: Reopen a closed tab
Given the user has closed one or more tabs during the current window's lifetime
When the user presses `u`
Then the most recently closed tab reopens and becomes active, re-rendering the target file fresh

### Scenario: Reopen further back in closed-tab history
Given the user has already reopened the most recently closed tab with `u`
When the user presses `u` again without closing any new tab since
Then the next-most-recently closed tab (further back in history) reopens, like a multi-level undo stack

### Scenario: Switch to the next/previous tab
Given the preview window has more than one open tab
When the user presses `L` or `H`
Then the next tab (`L`) or previous tab (`H`) becomes active

### Scenario: Scroll content
Given the active tab shows scrollable content (any rendered format)
When the user presses `j`/`k` (scroll down/up slightly) or `C-d`/`C-u` (scroll down/up a larger amount)
Then the content scrolls accordingly within the tab

### Scenario: Pan while zoomed in
Given the active tab's content is zoomed in beyond fit-to-width (PDF, image, or mmd diagram content)
When the user presses `h` (pan left) or `l` (pan right)
Then the visible viewport pans accordingly within the zoomed content

### Scenario: Pan with no effect at default zoom
Given the active tab's content is at the default fit-to-width zoom level (nothing to pan)
When the user presses `h` or `l`
Then nothing happens (no panning, no error)

### Scenario: Jump to the next/previous page
Given the active tab shows PDF-backed content (pdf.md, tex.md, or office-conversion.md)
When the user presses `J` (next page) or `K` (previous page)
Then the view jumps to the next or previous discrete page

### Scenario: Next/previous page not applicable
Given the active tab shows continuous-scroll content (markdown, Quarto, or notebook rendering, with no discrete pages)
When the user presses `J` or `K`
Then an info/warning message is shown within the preview window and no page jump occurs

### Scenario: Jump to start/end of content
Given the active tab shows scrollable content
When the user presses `C-g` (jump to start) or `G` (jump to end)
Then the view jumps to the very beginning or very end of the content

### Scenario: Zoom in/out on PDF, image, or diagram content
Given the active tab shows PDF, image (raster/SVG), or standalone Mermaid diagram (.mmd) content
When the user presses `+` (zoom in), `-` (zoom out), or `0` (reset zoom)
Then the content's zoom level changes accordingly, or resets to the default fit-to-width level

### Scenario: Text input for page entry or search
Given the active tab is showing any content
When the user presses `:` or `/`
Then a visible text input field opens showing the characters typed as the user types, until the user confirms (Enter) or cancels (Esc, or Backspace while the input is empty)

### Scenario: Cancel the text input
Given the text input field is open
When the user presses Esc, or presses Backspace while the input is empty
Then the input field closes and no page jump or search is performed

### Scenario: Jump to a specific page
Given the active tab shows PDF-backed content (pdf.md, tex.md, or office-conversion.md)
When the user presses `:`, types a page number into the text input, and confirms
Then the view jumps to that page

### Scenario: Page entry not applicable
Given the active tab shows continuous-scroll content (markdown, Quarto, or notebook rendering, with no discrete pages)
When the user presses `:`
Then an info/warning message is shown within the preview window and no page jump occurs

### Scenario: Search text within a tab
Given the active tab shows content containing text (any rendered format)
When the user presses `/`, types a search term into the text input, and confirms
Then matches are highlighted and the view jumps to the first match

### Scenario: Jump between search matches
Given an active search with one or more matches in the current tab
When the user presses `n` (next match) or `N` (previous match)
Then the view jumps to the corresponding match, wrapping around at the start/end

### Scenario: Manually reload the active tab
Given the active tab is showing previously rendered content for a format with no format-specific reload indicator (e.g. markdown, Quarto, notebooks, PDF, images, mmd)
When the user presses `e`
Then peep.nvim re-renders the target file into that tab, keeping the existing content visible with a generic "reloading" info bar shown until the new render is ready, then swapping it in

### Scenario: Manually reload a tab with its own indicator
Given the active tab is a LaTeX (tex.md) or Office-conversion-backed (office-conversion.md) tab
When the user presses `e`
Then peep.nvim re-renders the target file using that format's own existing indicator ("compiling" for tex, "converting" for Office documents) instead of the generic "reloading" bar, keeping the existing content visible until the new render is ready

### Scenario: Open the keybind help tab
Given the preview window is open
When the user presses `?`
Then a help tab opens listing all in-window keybinds and their descriptions, or becomes active if it's already open (reused, not duplicated)

### Scenario: Search with no matches
Given the user searches for a term with no matches in the current tab's content
When the search is confirmed
Then an info/warning message is shown within the preview window stating no matches were found

### Acceptance Criteria
- [ ] `q` closes the active tab; focus moves to the tab that was to its right, or the new rightmost tab if the closed tab was rightmost
- [ ] Closing the last regular tab automatically opens (or switches to, if already open) the keybind help tab, so the window is never left with zero tabs
- [ ] Closing the help tab while it's the only open tab closes the entire preview window
- [ ] `C-w` closes the entire preview window immediately, regardless of how many tabs are open
- [ ] `u` reopens the most recently closed tab, re-rendering it fresh and making it active
- [ ] Repeated `u` presses (with no new closures in between) walk back through a multi-level closed-tab history
- [ ] `H`/`L` switch to the previous/next open tab
- [ ] `j`/`k` scroll the active tab's content down/up slightly; `C-d`/`C-u` scroll down/up a larger amount
- [ ] `h`/`l` pan left/right within zoomed-in PDF, image, or mmd content; no effect when at the default fit-to-width zoom level
- [ ] `J`/`K` jump to the next/previous discrete page, for PDF-backed tabs (pdf.md, tex.md, office-conversion.md)
- [ ] `J`/`K` on a continuous-scroll tab (markdown/Quarto/notebook) shows an in-window info/warning instead of jumping
- [ ] `C-g`/`G` jump to the very start/end of the active tab's content
- [ ] `+`/`-`/`0` zoom in/out/reset on PDF and image (raster/SVG) tabs
- [ ] `e` manually reloads/re-renders the active tab's target file, keeping the existing content visible until the new render replaces it
- [ ] The reload shows a generic "reloading" info bar, except for tex and Office-conversion-backed tabs, which reuse their own existing "compiling"/"converting" indicators instead
- [ ] `:` or `/` opens a visible text input field showing typed characters live, until confirmed (Enter) or cancelled (Esc, or Backspace while empty), with no action taken on cancel
- [ ] `:` followed by a page number jumps to that page, for PDF-backed tabs (pdf.md, tex.md, office-conversion.md)
- [ ] `:` on a continuous-scroll tab (markdown/Quarto/notebook) shows an in-window info/warning instead of jumping
- [ ] `/` followed by a search term highlights matches and jumps to the first one, for any tab with text content
- [ ] `n`/`N` jump to the next/previous search match, wrapping around at the start/end
- [ ] A search with no matches shows an in-window info/warning stating no matches were found
- [ ] `?` opens a help tab listing all in-window keybinds and their descriptions, reusing the same tab if it's already open rather than duplicating it

### Non-Functional Requirements
- [ ] All keybinds in this requirement are configurable/remappable, not hardcoded, consistent with other configurable behaviors in this project (e.g. the LaTeX engine, the Office converter).
