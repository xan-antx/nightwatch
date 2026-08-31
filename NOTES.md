# Nightwatch — session handoff notes

_Last updated: 2026-08-31_

## What this project is

A single-file WebMCP demo: `index.html`, vanilla JS, no build step, no
dependencies. It's a mock incident console where an AI agent's tool surface
grows and shrinks with incident severity (green → yellow → red → blue). The
core demo point: ChatGPT's browser exposes `registerTool`/`getTools` but **no
way to unregister a tool**, so the page mounts tools add-only and "seals" them
(refuses calls in the tool's `execute` wrapper) when severity drops back out
of their range. Only a human can change severity; destructive tools stop at a
`humanApproval()` modal.

Tested and working in ChatGPT's built-in browser. `probe.html` is a scratch
probe file, not part of the demo.

## Do not touch (tuned / verified working)

- `syncTools()`, `spec()`, the `ALL` tools array, `registerTool`/`getTools`
  calls, the `registered` set, `humanApproval()`
- **Tool description strings** — the agent reads these and they are tuned
- Registration is add-only and re-registering a name throws; don't "fix" that

`node --check` the extracted `<script>` after any edit (pull the script body
out of index.html and check it; there's no build or test runner).

## What was done recently (all committed, tree clean except .claude/)

1. `de2d426` — Intro panel (`#intro`) at the top of the left column: three
   lines of product copy for a judge opening the page in plain Chrome with no
   agent. Quiet styling, reuses existing CSS vars, no colored callout.
2. `3c5bcbd` —
   - `#webmcpnote`: a dim sentence appended to the intro **only when WebMCP is
     unavailable** (unhidden in the boot else-branch), pointing at ChatGPT's
     browser or Chrome 149+ with `chrome://flags/#enable-webmcp-testing`. The
     old equivalent timeline log line was removed so the message isn't in two
     places.
   - `#recon`: the reconciliation stats ("N permitted at <sev> · N sealed ·
     N registered per browser") promoted from the bottom `#truth` box to
     directly under the "Agent capabilities" heading — three large
     Space Grotesk numbers with small uppercase labels. `#sealnote` below it
     appears only when sealed > 0 and explains the no-unregister limitation.
   - `#truth` now shows only the sync-mode line and is hidden until
     `syncTools()` has run.

**Untested path:** the sealed > 0 state (stats + `#sealnote`) needs a real
WebMCP browser — register tools at yellow/red, then drop severity. Verify once
in ChatGPT's browser; everything else was checked visually in a plain browser.

## New: .claude/skills/ (uncommitted, gitignored)

`npx ui-ux-pro-max-cli init --ai claude` installed 7 project skills
(~4.6 MB, 172 files) under `.claude/skills/`: ui-ux-pro-max, design,
design-system, brand, ui-styling, slides, banner-design. They load at session
start, so they were installed but **never active in the previous session** —
the restart this handoff precedes is what activates them. The directory is
in `.gitignore` (vendored content, regenerable with the npx command above).

Note: some of these skills assume stacks (shadcn/Tailwind, Gemini image
generation) that don't apply to this vanilla single-file project. Use
ui-ux-pro-max for styling/UX advice; ignore the rest unless asked.

## Next steps / open items

- Verify the sealed-state UI in ChatGPT's browser (see "Untested path").
- The user may want UI polish passes using the newly installed skills — that
  was the motivation for installing them. Same constraints will likely apply:
  single file, no deps, match existing CSS variables, don't touch tool
  descriptions or registration logic.
- Workflow so far: make one scoped change, show it in the browser pane,
  stop and wait for the user's go-ahead before the next task.
