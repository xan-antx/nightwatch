# Nightwatch

An incident console where the agent's tool surface is mounted and unmounted by incident severity.

One HTML file. No build, no dependencies, no backend.

## The idea

Most WebMCP apps register a static array of tools at page load and leave it there. Nightwatch
treats the tool surface as state. What the agent is allowed to do is a function of how bad the
incident is, and only the on-call human can change that.

| Severity | Mounted | Effect |
|---|---|---|
| **green** | `get_console_state`, `list_services`, `check_health`, `request_escalation` | The agent can look and ask. It cannot touch production. |
| **yellow** | + `get_alert`, `query_logs`, `diff_last_deploy`, `check_dependencies` | Read-only investigation. Enough to find a root cause. |
| **red** | + `rollback_deploy`, `drain_region`, `page_owner`, `set_maintenance_mode` | Write access, granted because a human declared an incident. Each call stops for approval. |
| **blue** | destructive tools unmount, + `generate_postmortem`, `export_timeline` | No fat-fingering a rollback after the incident is over. |

Three rules live in code, not in a prompt:

1. **Severity is human-only.** `request_escalation` never changes it — it arms the button you press.
2. **Destructive tools block on a human click.** `rollback_deploy`, `drain_region`, `page_owner` and
   `set_maintenance_mode` return nothing until you approve or deny in the page.
3. **Out-of-severity tools are not registered.** Not hidden, not guarded at call time — absent from
   the surface the agent can see.

## Run it

WebMCP needs a secure context, so `file://` won't register tools. Deploy first.

- Drag this folder onto [app.netlify.com/drop](https://app.netlify.com/drop), or `vercel deploy`.
- **ChatGPT:** desktop app → built-in browser → your URL. Use **GPT-5.6 Sol or Terra** (Luna has
  WebMCP disabled). Not available in Enterprise or Edu workspaces.
- **Chrome:** 149+, enable `chrome://flags/#enable-webmcp-testing`, restart.
- **Self-check:** append `?selftest`, open the console. Asserts that green mounts nothing that
  writes, that rollback is absent at green and gone again at blue, and that the agent's escalation
  request does not move severity.

## Spec notes

The API moved from `navigator.modelContext` to `document.modelContext` in the 21 July 2026 draft and
Chrome 150 deprecates the old location, so registration feature-detects both.

For unmounting, `syncTools()` prefers `provideContext({tools})` — replacing the whole surface on
every severity change — and falls back to `registerTool()` with `unregister()` handles. If a browser
supports neither, the sidebar says so, and out-of-severity tools still refuse at call time. The
sidebar always shows which path is live.

## Demo script

Open in ChatGPT's browser at green and say:

> Something's wrong with checkout. Look into it.

At green it has no logs and no deploy history. A good run calls `request_escalation` and tells you
why. Press **yellow**: four investigation tools animate into the tray. Ask it to continue and it
finds the deploy at 03:04 that set `max_connections` to 4 instead of 400, then asks for red.

Press **red**: the destructive tools mount. Tell it to roll back. It calls `rollback_deploy`, the
approval card appears, you approve, error rate drops to 0.02%. Press **blue** and watch the
rollback and drain tools unmount while the postmortem tools appear.

## License

MIT.
