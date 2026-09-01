# Nightwatch

An incident console where the AI agent's tool surface follows incident severity.
Tools mount as things get worse, unmount when they're over, and only the human
on call can change how bad things are.

One HTML file plus a vendored animation library. No build step, no backend.

**Live:** https://vocal-cascaron-3227b4.netlify.app
**Requires a WebMCP browser** — ChatGPT's desktop app browser (GPT-5.6 Sol or
Terra), or Chrome 149+ with `chrome://flags/#enable-webmcp-testing`.

## The idea

Most WebMCP pages register a static list of tools at load and leave it there.
Nightwatch treats the tool surface as state: what the agent is *able* to do is
a function of how serious the incident is, and severity belongs to the human.

| Severity   | Tools registered | What the agent can do |
| ---------- | ---------------- | --------------------- |
| **green**  | `get_console_state` `list_services` `check_health` `request_escalation` | Look, and ask. Nothing that touches production exists yet. |
| **yellow** | + `get_alert` `query_logs` `diff_last_deploy` `check_dependencies` | Read-only investigation — enough to find a root cause. |
| **red**    | + `rollback_deploy` `drain_region` `page_owner` `set_maintenance_mode` | Change production. Every one of these blocks on an in-page human approval before executing. |
| **blue**   | destructive four unregistered, + `generate_postmortem` `export_timeline` | Write the story down. The rollback tool no longer exists to fat-finger. |

Three rules live in code, not in a prompt:

1. **Severity is human-only.** The agent's `request_escalation` never changes
   it — it arms the button and states the evidence. You press.
2. **Destructive calls block on a click.** `rollback_deploy` and friends return
   nothing until the person approves or denies on the page itself.
3. **Out-of-severity tools are not registered.** Not hidden, not refused at
   call time — absent from the surface. The sidebar shows the browser's own
   `getTools()` count next to the console's, so the agreement is proved on
   screen, not asserted.

## How unmounting actually works

The WebMCP draft removed `unregisterTool()` on 23 April 2026. The only
removal path is an `AbortSignal` passed at registration:

```js
const ac = new AbortController();
await document.modelContext.registerTool(tool, { signal: ac.signal });
ac.abort();   // tool is gone
```

`syncTools()` keeps one `AbortController` per tool. On every severity change it
registers what became permitted and aborts what didn't, then re-reads
`getTools()` to display what the browser really holds. Verified in ChatGPT's
desktop browser — `probe.html` in this repo is the test that proved it, along
with every removal path that *doesn't* exist there (`unregisterTool`,
`provideContext`, handle methods — all absent; the signal is the way).

Registration feature-detects `document.modelContext` with a
`navigator.modelContext` fallback, since the API moved to `document` in the
21 July draft and Chrome 150 deprecates the old location.

## Why the data is synthetic

The five services, the alert, the logs and the deploy diff are all fixtures in
`index.html`. That is deliberate, not a shortcut.

An incident has to be reproducible. Anyone opening this — at any hour, from any
timezone — gets the same failure, the same root cause in the 03:04 deploy, and
the same fix. Wired to live telemetry there would be nothing wrong to
investigate most of the time, and a demo that only works during an outage is
not a demo.

What is *not* simulated is the part being demonstrated: tool registration
through `document.modelContext`, unregistration via `AbortSignal` verified
against the browser's own `getTools()`, destructive calls blocking on a real
DOM click for as long as it takes, and the agent reasoning its way to an
escalation request from tool output alone. All of it confirmed in ChatGPT's
desktop browser — `probe.html` is the test that established what that browser
actually implements.

Pointing this at a real estate means replacing fourteen function bodies with
calls to a real logging and deploy system. The tool definitions, the severity
lifecycle, and the approval gate would not change.

## Run it locally

WebMCP needs a secure context, so `file://` won't register tools.

- Drag this folder onto [app.netlify.com/drop](https://app.netlify.com/drop),
  or `vercel deploy`, or any static host. `motion.min.js` (vendored, from the
  `motion` npm package — see `package.json`) must ship next to `index.html`.
- Open the URL in ChatGPT's desktop browser on **GPT-5.6 Sol or Terra** (Luna
  has WebMCP disabled; Enterprise/Edu workspaces don't get site tools), or in
  Chrome 149+ with the flag above.
- In a browser without WebMCP the console still runs and explains itself; the
  tool surface just has nothing to attach to.
- Append `?selftest` and open the console for assertions on the severity
  rules: nothing writable at green, rollback absent from green and blue,
  postmortem only at blue, and the agent's escalation request never moving
  severity.

## Try this

Open at green and say: **"Something's wrong with checkout. Look into it."**

At green the agent has no logs and no deploy history — a good run calls
`request_escalation` and tells you why. Press **yellow**: four investigation
tools slide into the tray. It finds the 03:04 deploy that set
`max_connections` to 4 instead of 400 and asks for more. Press **red**, tell
it to roll back, and the approval card stops everything until you click. Press
**blue** and watch the four destructive tools leave the tray — and the
browser's registered count drop with them. Then ask it to roll back again,
and it will tell you the capability no longer exists.

## Files

- `index.html` — the whole app: UI, state, tools, registration lifecycle
- `motion.min.js` — vendored Motion (animation), loaded by a script tag
- `probe.html` — the API-surface probe that established what ChatGPT's
  browser supports, including the AbortSignal verdict
- `NOTES.md` — working notes and constraints for AI-assisted edits

## License

MIT.
