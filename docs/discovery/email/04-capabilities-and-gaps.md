# Email — Capabilities & Gaps

A precise read of what the Gmail tooling can and cannot do today, across the two distinct agent tool surfaces, plus the gaps fancylab-ops still has to close.

## Two distinct tool surfaces

The Gmail capability differs by agent. They are NOT the same toolset:

- **`email-scout`** — strictly **read-only**: `search_threads`, `get_thread`, `list_labels`. No mutation tools are even in its toolset. Its job is a digest; it can never change inbox state.
- **`email-agent`** — **read + label mutation + draft**. It can label/unlabel, archive, manage drafts. Its write ceiling is `create_draft`. **It cannot send.**

## Capability matrix

| Capability | Tool | Class | `email-scout` | `email-agent` |
|---|---|---|---|---|
| Search threads | `search_threads` | read | ✅ | ✅ |
| Read full thread | `get_thread` (FULL_CONTENT) | read | ✅ | ✅ |
| List labels | `list_labels` | read | ✅ | ✅ |
| Label / unlabel thread | `label_thread` / `unlabel_thread` | **write** | ❌ | ✅ |
| Label / unlabel message | `label_message` / `unlabel_message` | **write** | ❌ | ✅ |
| Archive (remove `INBOX`+`UNREAD`) | via `unlabel_thread` | **write** | ❌ | ✅ (noise only) |
| Create / update / delete label | `create_label` etc. | **write** | ❌ | ✅ (confirm first) |
| Create draft | `create_draft` | **draft** | ❌ | ✅ |
| List drafts | `list_drafts` | read | ❌ | ✅ |
| **Send mail** | — | **send** | ❌ | ❌ **(no tool exists)** |
| Delete / trash | — | destructive | ❌ | ❌ **(forbidden)** |
| Calendar read | `list_events` / `get_event` / `list_calendars` / `suggest_time` | read | ❌ | ✅ (read-only) |
| Calendar write | — | write | ❌ | ❌ **(forbidden)** |
| ClickUp read | `clickup_filter_tasks` / `get_task` / `search` etc. | read | ❌ | ✅ (read-only) |
| ClickUp write | — | write | ❌ | ❌ **(forbidden)** |

**Write ceiling = draft.** The single most important line: there is **no send capability anywhere today.** Send is the action fancylab-ops is being built to add (gated on a `gmail.send` token + the executor — see `03-implementation.md`).

## Hard constraints (governance, from the agents)

- **Never send** (draft is the contract).
- **Never write Calendar** (no create/update/delete/respond — read-only cross-ref only).
- **Never write ClickUp** (no task/comment/tag — read-only cross-ref only).
- **Never delete / never touch `TRASH` or `SPAM`.**
- **Confirm before bulk:** any single op touching **>20 threads**, or any label create/update/delete, requires `AskUserQuestion` first. Per-thread labeling during normal triage is fine.
- **No fabrication** of quotes, dates, commitments, names.
- **Read ≠ act:** analyzing a thread never marks it read (except archived noise).

## Scan caps

- `email-scout`: **~50 threads per pass**; reports matched-but-skipped count.
- `email-agent` triage: `pageSize:50`, paginate to exhaustion, but **halt at 200+ unread** and ask.
- Label-proposal sampling: **200 threads max**.
- Deep `get_thread` reads: only the **5–8 highest-signal** threads per run (snippets handle the rest).

## Reachability scope

- **Live Gmail tools were NOT reachable in this main session** (see `_raw/reachability.md`). The claude.ai Gmail MCP suite is absent here.
- The Gmail tools are **subagent-scoped**: declared in `email-agent` / `email-scout` frontmatter. They run when those subagents run in a session where the claude.ai MCP suite is connected — not guaranteed from the main session.

## Gaps

1. **No send capability.** The biggest gap and the whole point of the fancylab-ops executor. Until the `gmail.send`-scoped token + executor exist, nothing actually goes out; drafts pile up for manual send.
2. **No read-back of outcome.** Nothing confirms whether a draft was ever sent, or whether a thread got a reply. The agents create drafts but have no signal that Rinaldo acted on them. (fancylab-ops fixes this for ops-routed replies via the `sent` receipt + issue close — but not for ad-hoc drafts.)
3. **Fireflies is reachable but NOT wired into `email-scout`.** The Fireflies MCP (`mcp__fireflies__*`) IS live this session, but `email-scout` does **not** query it — it only *recognizes Fireflies/Fathom/Otter recap emails as FYI text* in the inbox. Meeting-transcript context is not pulled into triage. Wiring Fireflies into the Forager (meeting → reply context) is unbuilt.
4. **No email ↔ Slack ↔ ClickUp linkage.** Cross-references are one-directional and read-only (email reads Calendar + ClickUp). There is no unified thread that ties an email, its ClickUp task, and a Slack discussion together. Slack ingestion isn't connected at all (no Slack MCP this session).
5. **Label taxonomy drift, unresolved.** Clients appear under both `SUPPORT/` and `PRODUCTION/`; the redesign (`Clients/` + `Stage/` + `Priority/`) is proposed but not applied — needs Rinaldo's approval before any `create_label`.
6. **Live inventory + volume uncaptured this session.** Label inventory and inbox volume are templated; capture when the Gmail MCP is connected.
