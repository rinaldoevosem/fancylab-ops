# Slack — Channel Inventory (raw dump)

**Status:** `[CAPTURE WHEN CONNECTED]` — Slack MCP was not reachable from the main
session on 2026-06-06 (see `reachability.md`). No live channel list was retrieved.

This file is a **placeholder schema**, not a captured dump. When Slack read tools are
reachable (run from a context where `mcp__claude_ai_Slack__slack_search_channels` loads),
fill the table below. Do not fabricate rows.

---

## Capture procedure (when connected)

1. `slack_search_channels` with a broad/empty query (or per-letter) to enumerate channels.
2. For client-domain channels, also `slack_search_channels` on each known client
   `{company_name}` / `{domain}` (the pattern `client-agent.md` step 6 already uses).
3. Record name, topic/purpose, member count, public/private, last-activity date.
4. Tag each `client-facing` vs `internal/team` by name/topic heuristic (see `01-structure.md`).

## Channel table — `[CAPTURE WHEN CONNECTED]`

| Channel | Public/Private | Members | Topic / Purpose | client-facing / internal | Last activity |
|---|---|---|---|---|---|
| `[CAPTURE]` | `[CAPTURE]` | `[CAPTURE]` | `[CAPTURE]` | `[CAPTURE]` | `[CAPTURE]` |

## Observed naming patterns (confirmed from spec, not from live data)

Per `client-agent.md` step 6 (lines 95–97), dedicated **client** channels follow:
- `#{domain-slug}` — e.g. `#truwild`
- `#client-{name}` — e.g. `#client-truwild`

These are the search keys the client-agent already relies on, so they are the
expected client-channel convention. Internal channel names (`#general`, `#dev`,
`#design`, etc.) are **inferred conventions**, not confirmed from this workspace.
