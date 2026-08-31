# SUBMISSION KIT — not part of the deployed app
# Copy from here into Devpost / GitHub / your teleprompter. Do not deploy this file.

═══════════════════════════════════════════════════════════════════
1. GITHUB ABOUT (repo page → gear icon next to "About")
═══════════════════════════════════════════════════════════════════

Description:
Incident console where the AI agent's tool surface follows severity — tools
mount as things get worse, unmount when they're over, and a human holds the
keys. Built for the WebMCP Challenge.

Website: https://vocal-cascaron-3227b4.netlify.app   ← update if you rename

Topics: webmcp, ai-agents, incident-response, mcp, chatgpt, human-in-the-loop

═══════════════════════════════════════════════════════════════════
2. DEVPOST SUBMISSION TEXT
═══════════════════════════════════════════════════════════════════

## Nightwatch — the agent's permissions follow the incident

**Why this use case is a strong fit for WebMCP**

WebMCP's defining property is that the tool surface belongs to the page and
follows the page's state. Nightwatch is built entirely on that property: it is
an incident console where the set of tools the agent can call is a function of
incident severity. At green the agent gets four read-only tools and nothing
that touches production even exists. Declaring an incident registers
investigation tools; escalating to red registers destructive ones; resolving
to blue aborts their registration signals and they are genuinely gone — not
hidden, not refused, unregistered. Recent security research (arXiv:2606.06387)
argues the WebMCP tool surface must be treated as a security boundary rather
than a neutral interface. Nightwatch is that argument implemented as a
product: the tool surface *is* the permissions model, and a human holds it.

**How it creates a better user experience**

The person on call sees exactly what the agent can do at every moment — an
"Agent capabilities" tray that visibly gains and loses tools as severity
changes, with the browser's own getTools() count displayed beside the
console's so the claim is proved on screen. Escalation is a conversation with
teeth: the agent must present evidence ("checkout-api at 14.2% error rate,
1,840ms p99") to request access, and the request lights up a button only the
human can press. Every destructive call freezes mid-flight on an in-page
approval card. The human is never reviewing a transcript after the fact;
they're holding the keys during.

**What people and agents can do together that was difficult or impossible
before**

Incident response is exactly the task you want to hand an agent and exactly
the task you can't hand an agent: 3am, high stakes, and one wrong rollback
makes it worse. Before WebMCP the options were an agent with standing
API credentials (too much trust, all the time) or no agent (all the toil, all
the time). Nightwatch demonstrates a third option: trust that expands and
contracts with the situation, granted by a human, in the open. The agent does
the reading, correlating, and proposing at machine speed; the person does the
judging and the authorizing with one click; and when the incident ends, the
dangerous capabilities stop existing. In our test runs the agent, finding
itself without log access at green, argued for escalation unprompted — and
after de-escalation, correctly reported that the rollback capability no longer
existed and asked for it back. That interplay is the product.

**How we implemented WebMCP**

Fourteen tools registered imperatively via
document.modelContext.registerTool() (with a navigator.modelContext fallback,
tracking the API's July move to Document). Each tool declares the severities
it exists at; a sync layer keeps one AbortController per tool, registering
newly permitted tools and aborting no-longer-permitted ones on every severity
change — AbortSignal being the spec's only unregistration path since April.
After each sync the page re-reads getTools() and displays the browser's count
next to its own. Read tools carry readOnlyHint; destructive tools carry
destructiveHint and block on a promise resolved only by a human click (verified
to survive 100-second waits in ChatGPT's browser). The repo includes
probe.html, the API-surface probe we wrote to establish exactly what ChatGPT's
browser implements — including verifying abort-based unregistration against
getTools() — plus a ?selftest mode asserting the severity rules. Vanilla JS,
one HTML file, no backend; the tools call the same functions the buttons do.

**Built with:** javascript, html, css, webmcp, motion, netlify

═══════════════════════════════════════════════════════════════════
3. VIDEO SCRIPT — target 2:40, hard cap 3:00
═══════════════════════════════════════════════════════════════════

Setup before recording:
- Fresh reload of the live URL in ChatGPT's desktop browser, severity green
- ChatGPT on GPT-5.6 Sol or Terra, site already approved once so no permission
  popup interrupts the take
- Window sized so the capabilities tray and chat are both visible
- Practice the run once; agent turns take 30-60s, so record the agent's turns
  and trim the waiting in the edit — don't narrate over dead air

[0:00-0:15 — over the green console]
"This is Nightwatch, an incident console. The right column is every tool the
AI agent is allowed to call — right now, four, all read-only. The rollback
tool doesn't exist yet. Not disabled. Not registered."

[0:15-0:45 — type: Something's wrong with checkout. Look into it.]
"At green it has no logs and no deploy history. So watch what it does — it
can't escalate itself. It has to convince me."
(show the agent's request_escalation, the yellow button arming, its evidence
line on screen)

[0:45-1:15 — press YELLOW]
"I press yellow, and four investigation tools mount — watch the tray, and
watch the browser's own tool count go from four to eight."
(agent finds the deploy: max_connections 4, should be 400; asks for red)

[1:15-1:50 — press RED, type: Roll back checkout-api.]
"Red registers the dangerous tools. It calls rollback — and everything stops.
The tool call is frozen mid-flight until I click. The agent cannot click this
button."
(hold a beat on the approval card, click Approve, error rate drops to 0.02%)

[1:50-2:20 — press BLUE]
"Incident over. I press blue — and the four destructive tools unregister.
Watch the count: fourteen becomes ten, and that number is the browser's own
getTools(), not my UI."
(type: Roll back checkout-api. Show the agent reporting the capability no
longer exists and requesting red — read its actual reply aloud)

[2:20-2:50 — over the tray]
"That's the whole idea. Security research already calls the WebMCP tool
surface a security boundary. Nightwatch makes it one you can see: the agent's
permissions follow the incident, escalation must be argued for, destructive
actions block on a human, and when it's over the keys stop existing. One HTML
file, no backend — the lifecycle is registerTool with an AbortSignal, verified
live in this browser. Every rule you watched is enforced in code, not in a
prompt."

[2:50-3:00 — end card: live URL + repo URL on screen]

═══════════════════════════════════════════════════════════════════
4. FINAL CHECKLIST — in order
═══════════════════════════════════════════════════════════════════

[ ] Claim the Netlify site into your account (anonymous drops expire —
    judging runs to Sep 21) and optionally rename it; update the URL in
    README, About, and the Devpost form if you do
[ ] Replace README.md and NOTES.md with the rewritten versions; commit, push
[ ] Fill GitHub About (section 1 above); confirm MIT still shows
[ ] Redeploy the folder (README changes don't need it, but keeps parity)
[ ] One full clean run in ChatGPT's browser: green → blue, 10 = 10 at blue
[ ] Record the video (section 3), trim to <3:00, upload to YouTube as PUBLIC,
    confirm it plays logged out
[ ] Submit on Devpost: live URL, video URL, repo URL, text (section 2) —
    at least a day before Sep 3, 1pm PT (= 1:30am Sep 4 IST)
[ ] After submitting, change nothing on main until judging ends
