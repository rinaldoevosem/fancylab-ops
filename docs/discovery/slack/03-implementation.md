# Slack — Implementation (outbound · the action-out side)

> Reachability: **NOT reachable from main** on 2026-06-06. The write surface below is
> **asserted by the discovery task** (the forbidden-writes list) — it was **not
> introspected** (no Slack tool schema loaded, so the real MCP tool set could not be
> enumerated). **No write tool was exercised. Zero Slack writes performed.**

## How responses go back through Slack today: manually

The outbound half of the Slack loop is currently **100% manual**. When a signal lands
(see `02-intake.md`), Rinaldo reads the thread and **types the reply himself** in Slack.
There is:

- **No Slack Scout** — nothing triages/routes inbound Slack signals (contrast the
  email and ClickUp stacks, which have Scouts).
- **No write-back loop** — no agent drafts or posts Slack replies for review/approval.
- **No PR-style review surface for Slack** — the hub's "Claude proposes → you merge"
  model is not wired to Slack outbound.

The only automated Slack interaction that exists today is **read-only**: the
`client-agent` consumes Slack context to enrich Client Profiles (`client-agent.md`
step 6). It never writes.

## The write surface (NOT exercised — enumeration not introspected)

The Slack MCP exposes write capabilities. These are the **implementation surface** a
future write-back loop would use. Listed here for planning only — none were called, and
this list is **not authoritative** (the live MCP tool set could not be introspected
because Slack did not load):

| Write capability | Effect | Source of claim |
|---|---|---|
| `send_message` | Post a message to a channel/thread | task-asserted (forbidden-writes list) |
| reactions | Add an emoji reaction to a message | task-asserted |
| pins | Pin a message in a channel | task-asserted |
| channel creation | Create a new channel | task-asserted |
| edits | Edit an existing message | task-asserted |

> **Caveat:** the exact tool names/signatures (e.g. whether `send_message` is
> `mcp__claude_ai_Slack__slack_send_message` or similar) are **unverified** — confirm by
> introspecting the MCP when it is reachable. Treat the column above as *categories of
> write*, not confirmed tool IDs.

## How automatable is a Slack reply?

Mechanically, **highly automatable** — a single `send_message`-class call posts a reply,
so the action itself is one tool call. The hard part is **not** the mechanics; it's the
**governance**:

- A Slack reply to a client is a **production-level** action (client-facing comms),
  which under the hub model must go **Claude proposes (PR) → Rinaldo reviews → merge =
  send** — never auto-posted.
- This needs (a) a Slack Scout to detect/route the inbound signal, (b) a drafting step
  that produces a proposed reply, and (c) a Guard/review surface in the hub before the
  `send_message` fires.

None of (a)/(b)/(c) exist for Slack today. So: **technically a 1-call action, but
operationally gated behind infrastructure that isn't built.** See the gaps in
`04-capabilities-and-gaps.md`.
