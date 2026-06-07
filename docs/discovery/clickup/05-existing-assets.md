# ClickUp — Existing Assets

Two existing assets define today's ClickUp automation. Both are **read-only**; **no
write-back agent exists**.

## 1. `ClickUp-Scout.md` (the method)

Source: `/Users/melo/fancylab/clickup/.claude/agents/ClickUp-Scout.md`
(Claude Code sub-agent, model: sonnet, **read-only** by tool allowlist.)

**Purpose:** a daily triage agent. Produces one Markdown report — "what's on Rinaldo's
plate" — grouped by client project, with an executive summary per project and mention
pings highlighted.

**Hard scope:** workspace `8519953`, space `90110691628` (WEB DEVELOPMENT); open tasks only
(status type ≠ closed/done); assigned to Rinaldo (user_id `12760847`). Never writes —
"create, update, comment, assign, move, tag, or delete" are all forbidden, enforced by the
allowlist.

**Tools (all read):** `find_member_by_name`, `filter_tasks`, `get_task_comments`,
`get_workspace_hierarchy`, `get_workspace_members`.

**Procedure (digest):**
1. Resolve Rinaldo's user_id (known: 12760847).
2. **Bucket A** — `filter_tasks` for his open tasks in the space, paginated (100/page,
   hard cap 10 pages); defensively drop any closed/done that leak through.
3. **Mention scan** — for *every* Bucket A task, `get_task_comments`; a comment matches if
   it @-mentions his user_id or contains the word "rinaldo" (word-boundary). Record latest
   matching comment (author/date/≤200-char snippet) + count.
4. **Group by project** = `task.folder.name` (fallback "No Folder"); track total / overdue /
   mention counts per project.
5. **Render** — header (counts), per-project blocks sorted *mention-projects first
   (by ping count), then by overdue count*; tasks sorted *pinged-first (latest date),
   then due-date asc*. Footer with totals + MCP call count + errors.

**Cost profile:** ~1 filter call + 1 comment call per task. The 2026-05-24 run = ~270 MCP
calls for 135 tasks, 0 errors.

## 2. `scout-2026-05-24.md` (sample output)

Source: `/Users/melo/fancylab/clickup/scout-2026-05-24.md` — an actual Scout run.

**Headline:** `135 assigned · 21 with mention pings · 30 projects · 270 MCP calls · 0 errors`.

What it surfaces, concretely:
- **Mention pings** as direct asks: Asya "Is this page approved?" (ian.club Benefits-Aero,
  overdue since Jan), Miguel "we need the figma file here" (rijewelers), Harry "who is
  going to do this demo?" (barclaysjewelers), Miguel "Not a Shopify Store" flags
  (moderncarats, lawkaufman, TICKETS) suggesting re-route/close.
- **Status gates** dominating blocked work: many tasks at **`client approval`** and
  **`review`** awaiting sign-off (anglodiamond pages, harryritchies Locations Page, etc.).
- **Overdue clusters** signalling at-risk clients: divinitymetals (6/7 overdue — B2B
  discount, missing photos, password-reset emails — flagged "needs a prioritization call"),
  feyjewelers (all 3 overdue), Steindiamonds (4/4 overdue).
- **Sort order** in action: pinged projects first, then by overdue density; a final
  "Other active projects (no mention pings)" table sweeps up the long tail.

This output is the **intake report** (02). It is presented for human reading — nothing in
the pipeline writes a response back into ClickUp.

## What's missing

- **No write-back / implementation agent.** Nothing converts a Scout finding into a comment,
  status change, reassignment, or new task. (The write tools exist — see 03/04 — but no
  agent is wired to them.)
- **No custom-field awareness.** The Scout never reads `Phase` or marketing metrics.
- **No cross-system linkage** to Gmail/Slack/Calendar.

These define the build backlog for closing the ClickUp loop.
