# NOTES — working context for AI-assisted edits

Read this before touching anything.

## What this is

Nightwatch: a single-file WebMCP demo (`index.html`, vanilla JS, no build) for
the WebMCP Challenge (deadline Sep 3, 2026, 1pm PT). An incident console where
the agent's tool surface mounts and unmounts with severity. **The build is
verified working in ChatGPT's desktop browser.** Remaining work is packaging
(video, Devpost text), not features.

## Current architecture — the truth as of the last verified run

- 14 tools in the `ALL` array, each with a `states` list naming the severities
  it exists at. green=4, yellow=8, red=12, blue=10.
- `syncTools()` keeps a `controllers` Map (name → AbortController). Registration
  passes `{ signal }`; unmounting is `controller.abort()`. This is the spec's
  only removal path (`unregisterTool` was removed 23 Apr 2026) and it is
  **verified working in ChatGPT's browser** via probe.html: after abort,
  `getTools()` no longer lists the tool.
- After every sync, `state.browserTools` is re-read from `ctx.getTools()` and
  the UI shows the browser's count beside the console's. They agree by
  construction; showing both is the point.
- Severity changes are human-only (`setSeverity` is called from buttons).
  `request_escalation` records a request and arms the button — never changes
  state.
- Destructive tools (`rollback_deploy`, `drain_region`, `page_owner`,
  `set_maintenance_mode`) await `humanApproval()` — a promise resolved only by
  an in-page click. ChatGPT's browser tolerates 100+ second tool calls; do not
  add timeouts.
- Registration targets `document.modelContext ?? navigator.modelContext`.
  ChatGPT exposes only: `registerTool`, `getTools`, `executeTool`,
  `codexGetTools`, `codexExecuteTool`. `registerTool` returns undefined and
  duplicate names throw — never re-register an existing name.
- Motion (`motion.min.js`, vendored) animates exactly three moments: cards
  mounting (stagger, slide), cards leaving (ghost fade-out), and the recon
  numbers ticking. `canAnimate()` gates on Motion presence and
  prefers-reduced-motion. Nothing else animates.

## Do not touch without explicit instruction

- `syncTools()`, the `controllers`/`registered` bookkeeping, `spec()`
- The `ALL` tools array structure and every tool **description string** — the
  agent reads those and they are tuned; the unprompted `request_escalation`
  behavior depends on them
- `humanApproval()`
- The severity state machine (`setSeverity`, `SEV`, human-only rule)

## Working rules

- One scoped change at a time; show it, wait for go-ahead.
- `node --check` on the extracted script after every edit.
- Anything touching registration must be re-verified in ChatGPT's real
  browser before it counts as done — stubs prove logic, not the browser.
- Commit after each verified change with a message describing what a judge
  would see.

## History worth knowing

The project briefly shipped a "sealed" state (tools registered but refusing out
of severity) built on the wrong assumption that this browser couldn't unmount.
probe.html then established the AbortSignal path works, and the seal was
removed in favour of real unmounting. The probe file stays in the repo as the
evidence trail.
