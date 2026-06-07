# ClickUp — Capabilities & Gaps

## Read capability matrix (exercised this session ✓)

| Tool | R/W | Reachable | Auth | Notes / limits |
|---|---|---|---|---|
| `clickup_get_workspace_hierarchy` | R | ✓ | session-detected workspace | depth 0–2; paginates spaces (≤50/page); returned all 8 spaces in one call |
| `clickup_get_workspace_members` | R | ✓ | " | 23 members; **no role/permission field** in response |
| `clickup_get_custom_fields` | R | ✓ | " | per list/folder/space/workspace scope |
| `clickup_get_chat_channels` | R | ✓ | " | 100/page, cursor pagination; 175 total over 2 pages |
| `clickup_filter_tasks` | R | ✓ | " | page size 100; **~10-page practical cap** (Scout truncates at 10×100); assignees must be numeric IDs |
| `clickup_get_list` | R | ✓ | " | returns id/name/space/url — **not** status definitions |
| `clickup_get_task` | R | ✓ | " | `detail_level` summary/detailed; auto-summarizes >50k tokens |
| `clickup_search` | R | ✓ (not needed) | " | global search across tasks/docs/chat/etc. |
| `clickup_get_task_comments` | R | ✓ (per Scout) | " | the mention-ping source; 1 call/task → expensive at scale |

> **Limits worth remembering:** task reads are paginated at ~10 pages × 100 = ~1,000 tasks
> per filter; comment scanning is 1 call per task (the Scout's 2026-05-24 run made ~270 MCP
> calls for 135 tasks). Deep-reading individual tasks should be capped (~5) to stay fast.

## Write capability surface (NOT exercised — read-only discovery)

Available but never called: `clickup_create_task`, `clickup_update_task`,
`clickup_create_task_comment`, `clickup_add_tag_to_task`/`remove_tag_from_task`,
`clickup_move_task`, `clickup_add_task_to_list`/`remove_task_from_list`,
`clickup_create_list`/`create_folder`/`create_list_in_folder`, `clickup_update_list`/
`update_folder`, `clickup_create_task_dependency`/`remove`, `clickup_add_task_link`/`remove`,
`clickup_attach_task_file`, `clickup_create_document`/`create_document_page`/
`update_document_page`, `clickup_send_chat_message`, `clickup_create_reminder`/
`update_reminder`, `clickup_add_time_entry`/`start_time_tracking`/`stop_time_tracking`,
`clickup_delete_task`. (Full mapping + automatability in 03.)

## Gaps

1. **No write-back loop.** The Scout reads intake but cannot act. Every approval, reply,
   reassignment, and new task is manual in the UI. This is the central gap (see 03).
2. **No custom-field reading in the Scout.** The Scout's allowlist omits
   `clickup_get_custom_fields`, so it never sees `Phase`, marketing Budget/Channel/Goal,
   etc. Triage is blind to lifecycle stage and campaign metadata that exist on the tasks.
3. **No role/permission data.** ClickUp's member API exposes no role/seat/permission field;
   functional roles are inferred from activity only. Any "who can approve / who owns this"
   logic must be encoded elsewhere, not read from ClickUp.
4. **No task ↔ email ↔ Slack linkage.** A `client approval` task, the client email that
   triggered it, and any chat thread are not connected. Cross-system correlation (e.g.
   "this overdue task is the same thread as that unanswered Gmail") is unbuilt.
5. **Comment-scan cost.** Mention detection is 1 API call per task; at ~130 tasks this is
   the dominant cost and limits how often/broadly the Scout can run.
6. **No status-definition source.** `get_list` doesn't return a list's status set;
   statuses are only learned by observing tasks, so the Scout can't enumerate gate statuses
   per space authoritatively.
7. **Domain schema asymmetry.** WEB DEVELOPMENT tasks carry almost no structured data
   (client = list name); MARKETING tasks carry rich metrics. Unified reporting must handle
   both shapes.
