# Slack MCP — Reachability Probe (raw)

**Probe date:** 2026-06-06
**Probing context:** main session (Claude Code, fancylab-ops data-hygiene discovery pass)
**Verdict:** Slack read tools are **NOT reachable** from this (main) context.

---

## What was tried

The Slack MCP is exposed in Claude Code under the prefix `mcp__claude_ai_Slack__*`
(per `client-agent.md` step 6). To call any MCP tool, its JSON schema must first load
via `ToolSearch`. Five queries were run:

| # | ToolSearch query | Result |
|---|---|---|
| 1 | `slack` | No matching deferred tools found |
| 2 | `mcp__claude_ai_Slack` | No matching deferred tools found |
| 3 | `slack_search_channels read_channel message` | Returned ClickUp chat / SendMessage / Monitor — **no Slack** |
| 4 | `+slack channel` | No matching deferred tools found |
| 5 | `search public private channels workspace` | Returned ClickUp chat channels / Fireflies / WebSearch — **no Slack** |

## Cross-check against the deferred-tools inventory

The session's deferred-tools `<system-reminder>` was scanned directly. Every `mcp__*`
entry belongs to: `clickup`, `fireflies`, `plugin_figma_figma`, or
`plugin_vercel_vercel`. **Zero `mcp__claude_ai_Slack__*` entries exist.** Slack is not
deferred-but-available here — it is absent from this context entirely.

## "Try one lightweight read" sub-step

N/A — vacuously impossible. A tool with no loaded schema cannot be invoked
(calling it fails with InputValidationError before reaching Slack). Since no Slack
schema ever loaded, no live read (e.g. `slack_search_channels`) could be attempted.
No Slack call of any kind was made.

## Conclusion

- Slack read tools: **NOT reachable from main session.**
- The MCP **is** referenced by `client-agent.md` (a subagent definition with `tools: "*"`),
  which is consistent with Slack being subagent-scoped — but live reachability *from*
  that subagent was **not independently verified** in this pass.
- Live Slack capture (channel inventory, messages, member counts) is therefore deferred:
  see `[CAPTURE WHEN CONNECTED]` markers throughout the docs.

**Slack writes performed: ZERO** (no Slack tool was callable).
