# peep.nvim architecture

A single living architecture doc for the whole plugin (flat, like the
ADRs under `docs/adrs/`). Updated in place as each design ticket lands;
cross-references the ADRs that motivate each decision rather than
restating their reasoning.

## Components

- **Neovim plugin (Lua)** — the `peep.nvim` runtime loaded into the
  user's Neovim. Resolves the preview target (current buffer or a
  supported file explorer selection), reads its content (buffer
  in-memory content if a buffer exists, disk otherwise), spawns/reuses
  the companion process, and drives it over the RPC channel
  ([0002](adrs/0002-neovim-companion-process-ipc.md)). Also receives
  companion-initiated calls (e.g. "open this file as a buffer" from a
  clicked link).
- **Companion process (Go binary, `peep-server`)** — spawned via
  `jobstart(cmd, {rpc = true})`, one instance per Neovim session that
  has ever triggered a preview. Hosts:
  - An **RPC serve loop** (goroutine), built on
    `github.com/neovim/go-client`, handling Neovim-initiated requests
    and issuing companion-initiated calls back over the same channel.
  - A **native webview window** (`webview/webview_go`,
    [0001](adrs/0001-external-preview-window-technology.md)), whose
    event loop (`webview.Run()`) owns the main goroutine; RPC handlers
    apply UI changes via the library's thread-safe `Dispatch()`.
- **In-webview tab shell (HTML/CSS/JS)** — the single webview's own page
  content. Implements tab management (create/close/reopen/switch, one
  tab per previewed target file) as an in-page construct: a tab bar,
  active-tab switching, and per-tab content panes. The spec requires
  tab management, not native OS tabs specifically; this project
  implements it in-page because running multiple `webview/webview`
  instances in one process is an observed practical constraint (see
  [0001](adrs/0001-external-preview-window-technology.md)), not because
  native tabs were ruled out on their own merits. Hosts the renderers
  each format ticket plugs into: an HTML/CSS pipeline
  (markdown-it/highlight.js/mermaid.js for markdown, Quarto, notebooks,
  images) and PDF.js for paged content (PDF, LaTeX-compiled,
  Office-converted).
  - Switching to an already-rendered, inactive tab does not force a
    re-render — its rendered state is preserved when memory permits.
    "Torn down" applies specifically to reclaiming heavy content (live
    PDF.js instances, notebook DOMs) from tabs that have been inactive
    for a while or under memory pressure; reactivating a torn-down tab
    requires a fresh render, the same as the explicit manual-reload
    (`e`) behavior. The exact threshold/policy for when teardown kicks
    in is not decided here — left to ticket #5's implementation.
  - The HTML-pipeline/PDF-pipeline split below is a provisional
    taxonomy for ticket #5, not a separately-ADR'd decision — revisit
    it (and whether it deserves its own ADR) once format tickets
    actually start building against it, rather than treating it as
    settled now.
  - `+`/`-`/`0` zoom is spec'd as one shared keybind across both
    pipelines (CSS-zoom for the HTML pipeline, PDF.js zoom for the PDF
    pipeline); whether the two feel consistent (step size, fit-to-width
    baseline) under that one keybind is not yet verified and should be
    checked once both pipelines exist.

## Process & window lifecycle

- One companion process per Neovim session; one native window per
  companion process; one tab per currently-previewed target file within
  that window.
- Neovim tracks the job's channel id; `on_exit` plus the RPC channel's
  own close event give lifecycle signals in both directions.
- The companion terminates itself (and any of its own child processes,
  e.g. a future LaTeX/Office conversion subprocess) when its window
  closes, or when it detects Neovim has exited.
- On companion start, before opening the window, it checks whether the
  OS's native webview engine is actually loadable (WebKitGTK on Linux,
  WebView2 on Windows, WKWebView on macOS) and whether a display is
  available; either failure is reported back over the RPC channel as a
  specific, descriptive error rather than a hang, crash, or silent
  no-op — learned directly from peek.nvim's own bug history of exactly
  this failure mode going unreported.

## Tab / content contract

Each tab is backed by one of two rendering families, chosen by target
file type:

- **HTML pipeline** — markdown, Quarto, notebooks, raster/SVG images,
  standalone Mermaid (`.mmd`). Rendered as an HTML fragment/page inside
  the tab shell's per-tab pane.
- **PDF pipeline** — PDF files, LaTeX-compiled PDFs, Office-converted
  PDFs. Rendered via PDF.js inside the tab shell's per-tab pane.

Format-specific tickets ([design](https://github.com/Dijereo/peep.nvim/issues/5)'s
downstream tickets) plug into whichever pipeline applies; this contract
(not yet finalized in code, since ticket #5 defers actual content
rendering) is what lets the generic keybind dispatch below stay
generic.

## Keybind dispatch

The in-window keybind set (`docs/specs/core/window-keybinds.md`) is
handled generically by the tab shell, dispatched per the active tab's
content family:

- Tab management (`q`/`u`/`H`/`L`/`?`) and window close (`C-w`) are
  handled uniformly regardless of content family.
- Scroll/pan/page-jump/zoom (`j`/`k`/`C-d`/`C-u`/`h`/`l`/`J`/`K`/`C-g`/`G`/
  `+`/`-`/`0`) dispatch to whichever the active tab's content family
  implements — the HTML pipeline maps these onto standard
  scroll/CSS-zoom; the PDF pipeline maps them onto PDF.js's own page
  navigation and zoom APIs.
- Text input (`:`/`/`) opens a generic overlay; its confirm action
  dispatches to page-jump (PDF pipeline only) or text search (either
  pipeline).
- Manual reload (`e`) shows a generic "reloading" indicator, except tex/
  Office-conversion-backed tabs, which reuse their own
  "compiling"/"converting" indicator instead (per
  `docs/specs/core/window-keybinds.md`).

## IPC

See [0002](adrs/0002-neovim-companion-process-ipc.md) for the full
decision. Summary: `jobstart(cmd, {rpc = true})` on the Lua side,
`github.com/neovim/go-client`'s RPC serve loop on the Go side, running
concurrently with the webview's own event loop via `Dispatch()`.
Structured request/response calls (open/render a target file, edit a
line in a buffer, terminate) run over this channel; a future
high-frequency stream (sync-scroll's scroll-position updates) may
warrant a separate, lighter mechanism — not yet decided (see 0002's
Consequences).

Any Neovim-initiated call that triggers rendering work uses
`vim.rpcnotify` (async), never blocking `vim.rpcrequest`, so the editor
is never stalled waiting on a render — including the inherently slower
LaTeX/Office-conversion cases. The webview's in-page JS reaches Neovim
only through a narrow, explicit allow-list of actions the companion
implements (open file as buffer, edit a line, open a URL in the system
browser) — never a generic passthrough to the companion's own
(implicitly-trusted) RPC access — since the webview may be rendering
untrusted file content.

## Installation

Prebuilt `peep-server` binaries are published per release for
Linux/macOS/Windows (native per-OS CI builds, required by
`webview_go`'s CGO dependency — see
[0001](adrs/0001-external-preview-window-technology.md)'s
consequences). A plugin-manager `build` hook fetches the matching
binary by default; building locally (Go toolchain + a C compiler +
platform webview development headers) remains available as a fallback,
not disallowed, per `docs/specs/core/preview-window.md`'s installation
NFR. End users are also responsible for the OS-level webview engine
itself being present (WebKitGTK on Linux is not reliably preinstalled;
WebView2 is bundled on Windows 11 and most, not all, Windows 10; WKWebView
is always present on macOS) — the companion process detects and reports
this explicitly rather than failing silently. WSL2 specifically also
needs WSLg (bundled by default on current Windows 11, available via
`wsl --update` on Windows 10 21H2+) for any GUI window to appear at all,
on top of the same Linux package step.

## Security (deferred items)

Two items this design deliberately does not resolve yet, tracked here
so they aren't silently dropped:

- **Content sandboxing** — the webview is a full browser-grade engine
  and will execute script from rendered file content (planned scope
  includes notebooks/markdown with embedded HTML/JS output). No CSP,
  script-execution policy, or devtools-reachability decision has been
  made. Revisit as a decision driver in whichever format ticket first
  renders content capable of carrying arbitrary script/HTML.
- **Prebuilt binary provenance** — the default install path is a
  prebuilt `peep-server` binary, given implicit full RPC trust by
  design ([0002](adrs/0002-neovim-companion-process-ipc.md)). No
  checksum/signing story exists yet. Revisit when the release/build
  pipeline is set up.

## Tech stack summary

| Layer | Choice |
| --- | --- |
| Neovim-side plugin | Lua |
| Companion process | Go |
| Native window/engine | `webview/webview_go` (OS-provided WebKitGTK/WebView2/WKWebView) |
| Neovim ↔ companion transport | Neovim RPC job channel (`github.com/neovim/go-client`) |
| In-window UI | HTML/CSS/JS (tab shell, HTML rendering pipeline) |
| Paged content | PDF.js |
