# Slack — Existing Assets (what already encodes Slack knowledge)

> Reachability: **NOT reachable from main** on 2026-06-06. This doc inventories the
> assets that already reference Slack. The authoritative one is `client-agent.md`.

## 1. `client-agent.md` — step 6 (the only operational Slack spec)

**Path:** `/Users/melo/.claude/agents/client-agent.md`
**Section:** Step 6, "Slack (max 1 channel deep-read)" — lines 93–98.

This is the **only place in the system that defines how Slack is used operationally.**
It is read-only: the agent pulls recent client context from Slack to enrich Client
Profiles. Exact tools + approach, quoted verbatim (lines 93–98):

```
### 6. Slack (max 1 channel deep-read)

- `mcp__claude_ai_Slack__slack_search_channels` for `{company_name}` and `{domain}` — identify a dedicated channel if one exists (common pattern: `#{domain-slug}` or `#client-{name}`).
- If a channel matches, `slack_read_channel` last 20 messages for recent operational context (latest decisions, asks, escalations).
- If no dedicated channel, run `slack_search_public_and_private` for the company name, top 5 results.
```

**Hard cap (line 413):** `Max 1 Slack channel deep-read.`
**Failure handling (lines 417):** if the Slack MCP fails (auth/429/permission), the agent
writes `_Source unavailable_` for Slack-derived fields and proceeds — no retry > once.

What this asset establishes (confirmed):
- The three read tools in use: `slack_search_channels`, `slack_read_channel`,
  `slack_search_public_and_private`.
- The MCP prefix: `mcp__claude_ai_Slack__*`.
- The client-channel naming convention: `#{domain-slug}` / `#client-{name}`.
- The signal types it reads for: "latest decisions, asks, escalations."
- The depth budget: 1 channel, last 20 messages, top-5 search fallback.

Slack output feeds the Client Profile **Key Links → Slack Channel** field
(`client-agent.md` step 8 template, line 192) and the Special Notes context.

## 2. No standalone Slack agent exists

There is **no dedicated Slack agent, Slack Scout, or Slack skill** in `~/.claude/agents/`
or the skills list. Slack appears only as **step 6 of `client-agent`** — a sub-step of a
read-only profile-building agent, never as its own pipeline stage. (Contrast: email and
ClickUp each have their own Scout in the Purple Bee model.)

## 3. Indirect references

- **`client-agent` description** (`client-agent.md` line 3) lists Slack among the sources
  it synthesizes ("Gmail … Google Drive, ClickUp, **Slack**, the Master Client Database").
- **`client-agent` Figma/GitHub steps** (7b §3, 7c §2) name Slack as one of the content
  surfaces to regex-scan for pasted Figma / GitHub URLs — i.e. Slack messages are also
  mined opportunistically for links, not just operational context.
- The **hub CLAUDE.md** (`/Users/melo/fancylab/hub/CLAUDE.md`) does **not** list Slack in
  its nav/integrations (Today = ClickUp + Gmail + Calendar). Slack is **not** surfaced in
  the hub today — another sign there's no Slack pipeline yet.

## Summary

| Asset | Path | What it encodes |
|---|---|---|
| `client-agent.md` step 6 | `/Users/melo/.claude/agents/client-agent.md` (L93–98, cap L413) | The only operational Slack spec — 3 read tools, naming pattern, 1-channel cap, signal types |
| (none) | — | No standalone Slack agent / Scout / skill exists |
| hub CLAUDE.md | `/Users/melo/fancylab/hub/CLAUDE.md` | Confirms Slack is NOT in the hub integrations today |
