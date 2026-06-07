# Slack — Capabilities & Gaps

> Reachability: **NOT reachable from main** on 2026-06-06. The matrix mixes
> **confirmed** entries (the 3 read tools + the deep-read cap, from `client-agent.md`)
> with **task-asserted** write entries (not introspected) and **inferred** scoping.
> Provenance is tagged per row. **Zero Slack writes performed.**

## Capability matrix

| Tool / capability | Read/Write | Reachable from main? | Reachable from subagent? | Auth | Limits | Provenance |
|---|---|---|---|---|---|---|
| `slack_search_channels` | Read | **No** (probe 2026-06-06) | Defined for `client-agent`; live reachability **not verified** this pass | Claude Code Slack MCP (`mcp__claude_ai_Slack__*`) | — | **Confirmed** — `client-agent.md` step 6 (L95) |
| `slack_read_channel` | Read | **No** | Defined for `client-agent`; **not verified** live | same | **max 1 channel deep-read per run** (client-agent); ~last 20 msgs read in practice | **Confirmed** — `client-agent.md` step 6 (L96) + cap L413 |
| `slack_search_public_and_private` | Read | **No** | Defined for `client-agent`; **not verified** live | same | top ~5 results used | **Confirmed** — `client-agent.md` step 6 (L97) |
| `send_message` (class) | **Write** | **No** | Unknown — not introspected | same (assumed) | — | **Task-asserted**; tool ID unverified |
| reactions | **Write** | **No** | Unknown | same | — | **Task-asserted** |
| pins | **Write** | **No** | Unknown | same | — | **Task-asserted** |
| channel creation | **Write** | **No** | Unknown | same | — | **Task-asserted** |
| edits | **Write** | **No** | Unknown | same | — | **Task-asserted** |

### Reading the matrix

- **Read tools (rows 1–3):** the only Slack capabilities **confirmed by a primary
  source**. They appear in `client-agent.md` step 6 and the hard-caps section (L413).
  They are **not reachable from the main session** (probe-confirmed — see
  `_raw/reachability.md`).
- **"Reachable from subagent?" column:** honest status is **defined-but-unverified**.
  `client-agent` has `tools: "*"` and explicitly calls these three tools, so they are
  *scoped to* that subagent by design. But this pass did **not** independently run them
  from the subagent, so "subagent-scoped and working" is an **inference, not a
  confirmed fact.** Verify by running a read inside `client-agent` (or any context where
  the schema loads).
- **Write rows (4–8):** enumerated from the discovery task's forbidden-writes list, **not
  introspected** from the MCP. Tool names/signatures are placeholders pending a live
  introspection.

## Gaps

1. **No dedicated Slack Scout.** Email and ClickUp have Scouts that detect/route signals;
   Slack does not. Inbound Slack signals ping Rinaldo directly with no triage/routing
   layer. (The Purple Bee `Email Scout` / `ClickUp Scout` have no Slack peer.)
2. **No write-back loop.** No agent drafts or posts Slack replies; outbound is fully
   manual (`03-implementation.md`). No PR-style propose→review→send surface for Slack.
3. **No cross-run memory.** `client-agent`'s Slack read is per-invocation and ephemeral
   (max 1 channel, last ~20 msgs, then discarded). Nothing persists Slack state, threads,
   or decisions across runs — each run re-reads cold.
4. **No Slack ↔ email ↔ ClickUp linkage.** Slack is read in isolation per client. There's
   no entity graph tying a Slack thread to the same client's Gmail threads or ClickUp
   tasks — so a decision made in Slack isn't automatically reflected in ClickUp, and vice
   versa. Correlation today is manual / human-in-the-loop.
5. **Reachability fragility (this pass).** Slack isn't reachable from the main session at
   all — only (by design) from the `client-agent` subagent. Any hub-level or main-session
   Slack feature would need the MCP wired into that context first.

## To close the gaps (pointers, not built)

- A **Slack Scout** (Intent & Router Bee Type) reading channels, classifying the signal
  types in `02-intake.md`, routing to the right Forager.
- A **write-back path** ending in a Guard/hub review before any `send_message` fires
  (production-level → propose→merge, never auto-post).
- A shared client entity key linking Slack ↔ Gmail ↔ ClickUp so signals correlate.
