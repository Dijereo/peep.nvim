# 0001. External preview window technology & runtime

## Status
accepted

## Context

peep.nvim needs a single external, non-Neovim-owned preview window (per
`docs/specs/core/preview-window.md` and `docs/specs/core/window-keybinds.md`,
tracked as [#5](https://github.com/Dijereo/peep.nvim/issues/5)) capable of:

- The ability to manage tabs — create, close, reopen, switch between
  them — for one tab per previewed target file, regardless of whether
  tabs are implemented as native OS windows or an in-page UI construct
- A full custom keybind set (tab close/reopen/switch, scroll/pan, page
  jump, zoom, search, manual reload, help) that must be fully capturable,
  including `C-w` closing the entire window — nothing may be reserved by
  a browser's own chrome
- Non-blocking, self-dismissing in-window messages
- Rendering both HTML-ish content (markdown/Quarto tables, syntax
  highlighting, inline images, Mermaid diagrams) and PDF-quality paged
  content (PDF.js, for PDF files, LaTeX-compiled PDFs, and
  Office-converted PDFs)
- Reliable process lifecycle: hard-terminate the entire backing process
  (window + any child/server process) when the window closes or when the
  spawning Neovim instance exits
- Sub-500ms open/update latency for non-conversion content
- A graceful, descriptive error when no display is available, rather
  than hanging or crashing
- Low installation friction for ordinary Neovim users

## Decision Drivers

- Needs a real, browser-grade rendering engine (WebKit/Chromium-class) —
  PDF.js alone rules out lightweight/limited engines
- Custom keybinds, including `C-w`, must be capturable with nothing
  reserved by browser chrome
- Single-window, multi-tab enforcement and reliable close/exit detection
  are required
- Sub-500ms latency target rules out heavyweight runtimes with slow cold
  starts
- Installation should default to a prebuilt binary, low-friction path
- The cost of maintaining a second-language codebase should be
  proportionate to what it buys

## Options Considered

- **Native webview embedded in a Go companion process (`webview/webview_go`)**
  — chosen
- **Native webview embedded in a Go companion process (`abemedia/go-webview`,
  CGO-free)** — same end-user runtime dependency as `webview_go` (both
  ultimately load the same OS-provided engine); differs only in
  build/CI/local-fallback convenience (no C toolchain needed to
  cross-compile or to build locally)
- **Browser tab + local web server** — rejected: can't reliably enforce
  single-window-ness; the browser reserves core shortcuts (`Ctrl+W/T/N/Q`)
  that page JS cannot override; "did the user close it" is a fragile
  signal to recover
- **Driving an already-installed Chromium-based browser via CDP
  (`chromedp`)** — rejected: confirmed the same browser-reserved-shortcut
  problem persists even in `--app=` mode (Chrome/Chromium deliberately
  reserve `Ctrl+W/T/N/Q` at the browser level); doesn't reliably solve
  the install-dependency problem either, since most Linux desktops
  default to Firefox, not a Chromium-based browser; window-close
  detection has no first-class event and is an open upstream issue
- **Electron-based companion app** — rejected: ~100MB+ footprint and
  slower cold start work against the 500ms latency target, with no
  capability advantage over a native webview for this project's needs
- **CEF (bundled Chromium, no OS dependency)** — rejected: no actively
  maintained Go binding exists (all found are archived/stale); would
  mean authoring and maintaining our own from-scratch binding, and its
  redistributable is as large as Electron's
- **Self-contained lightweight engine (Ultralight, via a custom Go
  binding)** — rejected: Ultralight's own documented feature gaps (WebGL
  and canvas `putImageData`/`getImageData` unsupported) directly conflict
  with PDF.js, which depends on `putImageData` for compositing; no
  evidence anyone has run PDF.js under Ultralight. The binding itself
  would have been a tractable, bounded effort against a documented C
  API, but that's moot given the rendering incompatibility with a
  requirement covering over half this plugin's format matrix (PDF,
  LaTeX, Office conversion all render through this path)
- **Terminal graphics protocols (kitty graphics/sixel)** — rejected:
  cannot support clickable links/checkboxes, true PDF pagination
  fidelity, or the window/tab model the specs require

## Decision

Chosen option: "Native webview embedded in a Go companion process via
`webview/webview_go`," because it's the only evaluated option that
satisfies every hard requirement at once — full keybind capture (a bare
webview widget has no omnibox/tab-strip/reserved accelerators to fight),
a real WebKit/Chromium-class engine capable of PDF.js, a lightweight
native window with fast cold start, and a mature, widely-used binding
closely tracking the established upstream `webview/webview` C library
rather than a from-scratch binding effort (as Ultralight or CEF would
require).

The spec requires tab management, not a specific implementation
mechanism — running multiple `webview/webview` instances in one process
has been reported to crash (an observed, practical constraint:
[webview/webview#647](https://github.com/webview/webview/issues/647),
not a documented library guarantee — its own README says nothing about
a single-window limit). Given that constraint, this project's own tab
management is implemented as a single native window whose content is an
in-page JS/HTML shell that swaps rendered panes — our design choice to
satisfy the tab-management requirement within a real limitation, not a
workaround we're forced into or a pattern the library documents and
endorses. If a future library version or binding removes this
constraint, native multi-window tabs would equally satisfy the
requirement.

`webview_go`'s CGO dependency (vs. the CGO-free `abemedia/go-webview`
alternative) was accepted as a qualitative judgment call, not a
quantified comparison: the harder problem — the end-user OS-level
runtime engine dependency (WebKitGTK on Linux, WebView2 on
pre-Windows-11) — is identical between the two bindings, so this choice
affects only build/CI/local-build-fallback convenience. `webview_go`'s
longer track record and direct correspondence to the reference
`webview/webview` C library were preferred over `abemedia/go-webview`'s
lighter build story for a project just getting started, without hard
data (star counts, CI cost estimates) backing that preference either
way.

## Consequences

- Good, because full custom keybind capture (including `C-w`) is
  achievable with no browser-chrome shortcut conflicts.
- Good, because PDF.js, Mermaid.js, highlight.js, and markdown-it all run
  natively in a real WebKit/Chromium-class engine with no rendering
  compatibility risk.
- Good, because tabs-as-in-page-content sidesteps `webview/webview`'s
  observed single-window-per-process constraint entirely, without
  depending on that constraint ever being lifted.
- Bad, because Linux end users must have WebKitGTK installed separately
  — it is not reliably preinstalled, including on fresh desktop installs
  and minimal/server/WSL/Docker environments. This is a real case for
  this project's own likely dev/CI environment, not just a hypothetical
  edge case: WSL2 needs WSLg (bundled by default on current Windows 11,
  available via `wsl --update` on Windows 10 21H2+) for any GUI window
  to appear at all, on top of the same `apt install
  libwebkit2gtk-4.1-0`-style step a native Linux desktop would need. The
  remediation is low-effort once diagnosed (one package-manager command,
  plus enabling WSLg where it isn't already on), but only if the
  companion process actually detects and reports the failure — it must
  be actively detected (engine-load failure) and surfaced as a specific,
  descriptive error per `docs/specs/core/preview-window.md`'s
  launch-failure requirement, and documented prominently in install
  instructions. peek.nvim, the closest prior art, has a real history of
  silent failures and unhelpful raw exceptions from exactly this gap.
- Bad, because Windows users on older/unpatched Windows 10 or Windows 10
  LTSC/Server editions may lack the WebView2 runtime; same
  detect-and-report requirement applies, with the official ~2MB
  bootstrapper installer as the remediation path.
- Bad, because CGO forces native per-OS CI runners for prebuilt binaries
  (no cheap cross-compilation), and a local-build fallback additionally
  requires a C compiler and platform webview development headers, not
  just the Go toolchain — heavier than the CGO-free alternative would
  have been. The installation NFR's local-build fallback tier must
  document this accurately rather than imply a bare `go install` suffices.
- Scope note, not yet addressed: a browser-grade engine will execute
  script from rendered file content (markdown/notebooks with embedded
  HTML/JS output are already in this plugin's planned scope), and
  neither this ADR nor 0002 defines a content-sandboxing policy (CSP,
  disabling script execution for rendered file content, devtools
  reachability in release builds). This is explicitly deferred, not
  resolved: it should become a decision driver in whichever format
  ticket first renders content capable of carrying arbitrary
  script/HTML (the notebook or markdown rendering ticket), rather than
  staying an implicit non-issue.
- Bad, because switching webview bindings later, or moving off an
  OS-embedded webview entirely, would require re-plumbing the
  window/tab-shell/JS-bridge layer this ticket builds — a costly-to-reverse
  decision, which is why it warrants this ADR.
