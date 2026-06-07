# ClickUp — Implementation (action out)

The outbound half of the loop: how a decision becomes an action **back inside ClickUp**.

## Status today: manual / human

**There is no automated write-back loop.** The ClickUp-Scout is **read-only** — its tool
allowlist contains only read tools (`filter_tasks`, `get_task_comments`,
`get_workspace_hierarchy`, `get_workspace_members`, `find_member_by_name`), and its prompt
explicitly forbids any create/update/comment/assign/move/tag/delete. So every action-out
step is currently performed **by Rinaldo by hand in the ClickUp UI**:

| Decision | Manual action today |
|---|---|
| Approve work at a gate | Change status `client approval` / `review` → `Open`/done in the UI |
| Reply to a mention ping | Type a comment on the task in the UI |
| Reassign / delegate | Edit assignees in the UI |
| File new work | Create a task in the right client list in the UI |
| Re-prioritize / set due date | Edit the task in the UI |

This is the **implementation gap**: the intake side is instrumented (Scout reads it), but
the action side is entirely off-tool. Automating it is the obvious next build — and maps to
the Purple Bee model: Scout (read) → … → a **Worker** that writes back, behind the **Guard**
(Rinaldo approves before the write fires).

## The write surface (documented, NOT exercised)

The ClickUp MCP server exposes the write tools below. **None were called in this discovery**
(read-only constraint). Listed by tool name only — schemas were not loaded. Automatability
is an assessment of how cleanly each could be driven from a Scout decision.

| Write tool | What it does | Maps to decision | Automatability |
|---|---|---|---|
| `clickup_create_task_comment` | Post a comment on a task | Reply to a mention ping | High — text in, comment out; the natural first write-back |
| `clickup_update_task` | Change status / priority / due date / name / desc | Approve at a gate, re-prioritize | High — but status changes are high-stakes (must stay behind the Guard) |
| `clickup_create_task` | Create a new task | File new work from a ticket/email | High — needs target list resolution |
| `clickup_add_task_to_list` / `clickup_remove_task_from_list` | Multi-list membership | Route a ticket to a client list | Medium |
| `clickup_move_task` | Move a task between lists | Triage `TICKETS` → client list | Medium — destination must be resolved correctly |
| `clickup_add_tag_to_task` / `clickup_remove_tag_from_task` | Tag management | Label (seo/development/etc.) | High — low-risk, reversible |
| `clickup_resolve_assignees` (read) + `clickup_update_task` | Reassign | Delegate to a dev/designer | Medium — assignee resolution then update |
| `clickup_create_task_dependency` / `clickup_add_task_link` | Link/sequence tasks | Wire up blockers | Medium |
| `clickup_create_list` / `clickup_create_folder` / `clickup_create_list_in_folder` | New client list/folder | Onboard a new client | Low frequency, high impact |
| `clickup_create_document` / `clickup_create_document_page` / `clickup_update_document_page` | Docs | SOPs, briefs | Medium |
| `clickup_send_chat_message` | Post to a chat channel | Notify `REVIEW-FEED` / client channel | High — but crosses into comms |
| `clickup_create_reminder` / `clickup_update_reminder` | Reminders | Follow-ups | High |
| `clickup_add_time_entry` / `start`/`stop_time_tracking` | Time tracking | Log effort | High |
| `clickup_delete_task` | Delete | (rare) close stale tickets | Keep human-only |

### Recommended first write-back (when built)

`clickup_create_task_comment` behind the Guard — drafting a reply to a mention ping and
presenting it for one-click approval in the hub — is the lowest-risk, highest-value first
step, because comments are additive and reversible (unlike a status change that unblocks a
client-facing deliverable). Status changes (`clickup_update_task` to clear a
`client approval` gate) should come second and always require explicit human merge.
