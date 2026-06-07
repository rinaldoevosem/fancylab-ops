# Slack — Overview

> **Reachability status (2026-06-06): NOT reachable from main session.**
> The Slack MCP (`mcp__claude_ai_Slack__*`) did not load via `ToolSearch` and is
> absent from this context's deferred-tool inventory. It **is** referenced by the
> `client-agent` subagent definition. Live capture (channels, messages, member
> counts) is deferred — see `[CAPTURE WHEN CONNECTED]` markers. Probe detail in
> `_raw/reachability.md`. **Zero Slack writes were performed.**

## What Slack is used for in operations

Slack is one of the **intake + collaboration surfaces** in fancylab-ops, alongside
Gmail and ClickUp. In the current system it plays two roles:

1. **A signal source for client context.** The `client-agent` reads Slack to pull
   recent operational context for a client — "latest decisions, asks, escalations"
   (`client-agent.md` step 6) — when building Client Profiles. This is a *read* of
   Slack as a record of what's been discussed.
2. **A live human channel.** Day-to-day, Slack is where client and team conversation
   happens. Responses today are **manual** — Rinaldo types replies himself. There is
   no Slack scout and no automated write-back loop (unlike the email/ClickUp stacks).

## Who's on it

`[CAPTURE WHEN CONNECTED]` — member roster, per-channel membership, and client vs
internal participants are only knowable from a live `slack_search_channels` /
`slack_read_channel` pass. From spec: client work has **dedicated per-client channels**
(pattern `#{domain-slug}` or `#client-{name}`, `client-agent.md` step 6), implying a
mix of client-facing channels and internal team channels.

## The two-way loop (in one paragraph)

**Intake (signal in):** client questions, approval requests, dev blockers, escalations,
and FYIs land in Slack channels — today they ping Rinaldo directly, with no Scout
triaging or routing them. **Implementation (action out):** responses currently go back
the *manual* way — Rinaldo reads the thread and types a reply. The Slack MCP exposes a
write surface (`send_message`, reactions, etc.) that *could* close this loop
automatically, but it is **not exercised** today and there is no documented Slack
write-back automation. So the loop is currently **half-automated on the read side**
(client-agent consumes Slack context) and **fully manual on the write side.**

## Doc map

- `01-structure.md` — workspace anatomy, channel inventory (templated), client vs internal tagging
- `02-intake.md` — inbound signal types (the Scout view) — `[CAPTURE WHEN CONNECTED]`
- `03-implementation.md` — outbound: how replies/actions go back (manual today) + the write surface
- `04-capabilities-and-gaps.md` — read + write capability matrix and the gaps
- `05-existing-assets.md` — what already encodes Slack knowledge (`client-agent.md`)
- `_raw/` — `reachability.md` (probe result), `channels.md` (channel dump placeholder)
