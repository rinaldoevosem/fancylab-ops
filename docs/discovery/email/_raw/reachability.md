# Gmail — Reachability Probe (raw)

**Probe date:** 2026-06-06
**Probing session:** main Claude Code session in `/Users/melo/fancylab/hub`
**Result:** NOT REACHABLE this session (live capture templated).

## What was probed

1. `ToolSearch query:"gmail mcp__claude_ai_Gmail search_threads list_labels"` → **No matching deferred tools found.**
2. `ToolSearch query:"select:mcp__claude_ai_Gmail__list_labels,mcp__claude_ai_Gmail__search_threads,mcp__claude_ai_Gmail__get_thread"` → **No matching deferred tools found.**
3. `ToolSearch query:"email inbox thread label draft"` → returned only ClickUp / Fireflies / NotebookLM / team-coordination tools. **Zero `mcp__claude_ai_Gmail__*` entries.**

## Interpretation (two distinct facts — do not collapse)

- **The Gmail tools are declared.** `email-agent.md` and `email-scout.md` frontmatter both list `mcp__claude_ai_Gmail__*` tools. The capability exists in the agent definitions.
- **The claude.ai Gmail MCP is not connected in this session.** The deferred-tool list in the session reminder contains no Gmail tools. This is the same claude.ai MCP suite (Gmail / Slack / Calendar / Drive) — Slack was independently confirmed unreachable this session, and the whole suite appears absent. The MCPs that *are* live this session are a different set: `mcp__clickup__*`, `mcp__fireflies__*`, `mcp__notebooklm-mcp__*`, `mcp__plugin_figma_*`, `mcp__plugin_vercel_*`.

So: **"not loaded in this session's tool surface" ≠ "the tools don't exist."** Live Gmail capture (label inventory, volume snapshot) is templated `[CAPTURE WHEN CONNECTED]`. Everything else in this discovery pass is consolidated from the existing agent assets, which do not require a live Gmail connection.

## To capture when connected

Run from a session where `mcp__claude_ai_Gmail__*` tools are loaded (read-only only):

- `list_labels` → full label inventory → replace the template in `_raw/labels.md`.
- `search_threads query:"in:inbox is:unread" pageSize:50` → unread volume + bucket distribution snapshot → fold into `02-intake.md`.
- `search_threads query:"in:inbox newer_than:7d" pageSize:50` → 7-day volume → `02-intake.md`.

**Read-only only.** No `create_draft`, `label_thread`, `unlabel_thread`, `create_label`, send, or any mutation during a discovery capture.
