# 0002. IPC protocol between Neovim and the external preview process

## Status
accepted

## Context

Following [0001](0001-external-preview-window-technology.md), peep.nvim's
Neovim (Lua) side spawns a Go companion process that hosts a native
webview window. Per `docs/specs/core/preview-window.md` and
`docs/specs/core/window-keybinds.md` ([#5](https://github.com/Dijereo/peep.nvim/issues/5)),
the two sides need:

- Neovim → companion: push target-file content (including in-memory
  unsaved buffer content, not just disk content) to render/re-render a
  tab; trigger termination.
- Companion → Neovim: notify of interactive events inside the rendered
  content (e.g. a clicked file link should open a buffer in the spawning
  Neovim instance, per `docs/specs/rendered-markup/interactivity.md` —
  a separate ticket, but the transport built here must support this
  class of call).
- Lifecycle: Neovim must reliably detect the companion's exit and be
  able to force-terminate it; the companion must self-terminate
  (including its own child processes, e.g. a future LaTeX/Office
  conversion subprocess) when its window closes or it detects Neovim has
  exited.
- Async operation: slow companion-side work (LaTeX compilation, Office
  conversion, in later tickets) must never block the Neovim UI thread.

## Decision Drivers

- Must be genuinely bidirectional (Neovim-initiated and
  companion-initiated calls)
- Should not require inventing/maintaining a custom wire protocol if an
  adequate built-in mechanism exists
- Must integrate with Neovim's own job-control primitives for reliable
  lifecycle (spawn, detect exit, force-kill)
- Must support async/non-blocking calls so slow companion-side
  operations don't stall the Neovim UI
- Low implementation risk/effort given a maintained Go-side library
  exists

## Options Considered

- **Neovim's built-in RPC job channel** (`jobstart(cmd, {rpc = true})` +
  `github.com/neovim/go-client`) — chosen
- **Custom protocol over plain stdio pipes** (peek.nvim's approach: plain
  `jobstart`, no `rpc` flag, plus a hand-rolled length-prefixed binary
  framing) — considered as a lighter-weight alternative, particularly
  for a future high-frequency stream (e.g. sync-scroll's scroll-position
  updates), but rejected as this ticket's primary transport since it
  means designing and maintaining a bespoke protocol where Neovim
  already provides one
- **A separately-listening socket** (`vim.fn.sockconnect(..., {rpc =
  true})`) — confirmed equivalent to `jobstart`'s RPC channel (same
  msgpack-rpc underneath); offers no advantage for a process Neovim
  itself spawns and owns the lifecycle of, so not chosen
- **HTTP/WebSocket server hosted by the companion process** — rejected:
  adds a network-facing surface with no benefit over a direct job
  channel for a purely local, single-user, spawned-and-owned child
  process; complicates port/lifecycle management for no gain

## Decision

Chosen option: "Neovim's built-in RPC job channel," because it requires
no protocol design of our own, is confirmed genuinely bidirectional (a
job channel can be used for remote calls in both directions, with no
`:h remote-plugin` manifest/registration involved), and
`github.com/neovim/go-client` provides a working skeleton for exactly
this pattern: a Go child process serving msgpack-rpc over stdio while
running its own event loop concurrently. Its maintenance signal is
modest, not strong — its last commit (a one-line typo fix) landed in
March 2025 with none since, though the repository remains open (631
stars, 13 open issues) and not archived. Given the surface area this
project actually needs from it is small and stable (RPC serve/dispatch
over stdio), this is judged an acceptable risk to monitor rather than a
disqualifying one, but it should be named accurately rather than
described as more actively maintained than it is.

The companion process runs its RPC serve loop on a goroutine and the
webview's native event loop (`webview.Run()`) on the main goroutine,
using the webview library's own thread-safe `Dispatch()` to apply
RPC-triggered updates to the UI — a standard, confirmed-safe Go
concurrency pattern (documented directly in `webview_go`), not a novel
risk. `jobstart`'s `on_exit` callback and the RPC channel's own
connection-close event give Neovim reliable, built-in lifecycle signals
in both directions.

Within this channel, calls are split by whether blocking is safe:
Neovim-initiated calls that trigger any rendering work (even
"fast-path" non-conversion renders) use `vim.rpcnotify` (fire-and-forget),
with the companion reporting completion/failure back via its own
notification to Neovim, rather than `vim.rpcrequest`, which blocks
Neovim's main thread until the companion responds. `rpcrequest` is
reserved for calls guaranteed to return immediately (e.g. a pure state
query with no rendering/IO). This is required by this ADR's own
"must not block the Neovim UI" driver — a naive implementation using
`rpcrequest` for render triggers would violate it even for
non-conversion content, and outright stall the editor for the
inherently slower LaTeX/Office-conversion cases. This split is a
convention, not something the transport enforces on its own — ticket #5
must fund a single Lua-side wrapper (e.g. a `peep.rpc.notify_render(...)`
helper) that every render-triggering call site goes through, so the
sync/async discipline lives in one place rather than depending on every
call site getting it right independently.

The channel is treated as a trusted-but-narrow surface: Neovim's own
documentation notes that an RPC job channel is implicitly trusted and
can invoke any Neovim API function. Since the companion's webview
renders content from files that may be untrusted (a downloaded markdown
file, a shared notebook with crafted HTML output), the JS-to-native
bridge exposed *inside the webview's JS context* is a narrow, explicit
allow-list of application-level actions (open this file as a buffer,
edit this line, open this URL in the system browser) implemented by the
companion process — never a generic passthrough that lets in-page JS
issue arbitrary Neovim API calls over the RPC channel. The RPC
channel's own full trust is scoped to the companion process's own Go
code, not extended to whatever content it happens to be rendering.

## Consequences

- Good, because no custom wire protocol needs to be designed, versioned,
  or documented — Neovim's own RPC mechanism is already stable and
  documented.
- Good, because `go-client` gives a ready-made, low-risk skeleton for
  this exact "ad-hoc job-channel peer" pattern, with no
  `:UpdateRemotePlugins`/manifest step required.
- Good, because lifecycle detection (Neovim-side `on_exit`,
  companion-side channel-close) comes from the mechanism itself rather
  than a bespoke heartbeat/handshake.
- Bad, because every RPC call/response pair carries msgpack-rpc's
  framing overhead compared to a hand-rolled minimal protocol — judged
  acceptable since this ticket's traffic (open-file, edit-line,
  terminate) is not high-frequency.
- Scope note, not yet addressed: this design gives the companion process
  implicit full RPC trust, and the installation NFR makes a prebuilt
  binary the default install path — meaning most users will run a
  binary they didn't build, with full access to Neovim's API, sight
  unseen. Neither this ADR nor 0001 addresses binary provenance
  (checksums, signing) for that default path. This is explicitly
  deferred to whichever ticket sets up the release/build pipeline
  (the `deploy` skill's domain), not silently out of scope.
- Reversibility: moderate cost to change later, *assuming* the future
  sync-scroll ticket can extend this channel rather than needing a
  wholly separate parallel transport for its high-frequency stream (see
  the watch item below) — if it can't, treat this estimate as
  optimistic. Swapping transport would mean rewriting the Lua-side
  dispatch calls and the Go-side RPC handler registration, but both are
  already isolated behind the tab/content contract in
  `docs/architecture.md` rather than spread across format renderers, so
  a future change would not touch every renderer ticket individually —
  unlike 0001, which every renderer's UI code depends on directly.
- Watch item, not yet decided: real-world precedent (peek.nvim) needed
  explicit throttling for frequent updates and avoided stock RPC
  entirely for its own scroll/content-push stream in favor of a
  hand-rolled framed channel. This ticket's structured request/response
  calls are comfortably within RPC's sweet spot, but the future
  sync-scroll ticket — which needs a bidirectional, potentially frequent
  scroll-position stream — should re-evaluate whether stock RPC is
  adequate or whether a lighter, throttled channel is warranted for that
  specific traffic. This ADR does not decide that in advance.
