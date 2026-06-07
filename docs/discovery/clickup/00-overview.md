# ClickUp — Discovery Overview

**Reachability: ✓ reachable** (all read tools responded; ZERO writes performed).
Captured 2026-06-06 via the ClickUp MCP server. Workspace (team) `8519953`.

## ClickUp's role in operations

ClickUp is the **system of record for production work** at Evosem/FancyLab — the place
where every client project, ticket, and deliverable lives as a task. Each client is a
**list**; lists are grouped into **folders** by lifecycle stage (MARKETING/active,
SUPPORT, BACKLOG, DISCOVERY) inside the **WEB DEVELOPMENT** space. A parallel **MARKETING**
space runs campaign/content work with its own richer schema. Rinaldo (Web Director) sits
at the center as reviewer/approver — the human **Guard** in Purple Bee terms.

## The two-way loop, in one paragraph

ClickUp is a closed loop with two halves. **Intake (signal in):** work arrives as tasks
and signals — new tickets land in funnel lists (`TICKETS`, `Campaign Intake`, the
`Incoming Requests` / `Requests` lists), teammates @-mention Rinaldo in task comments to
ask for a decision, due dates lapse into overdue, and tasks pile up at gate statuses
(`client approval`, `review`) waiting on his sign-off. **Implementation (action out):**
once Rinaldo decides, the response goes *back into ClickUp* — a comment reply, a status
change (e.g. `client approval → Open`), a reassignment, or a new task. Today the ClickUp
**Scout reads the intake side only**; the implementation side is still done **by hand** in
the ClickUp UI. There is no documented automated write-back loop — that gap is the headline
finding (see 03 and 04).

## Document map

| File | Contents |
|---|---|
| `01-structure.md` | Spaces → folders → lists tree, statuses, members/roles, custom fields, chat channels |
| `02-intake.md` | Inbound signals: mention pings, overdue, status gates, intake funnels, sort logic + volume snapshot |
| `03-implementation.md` | Outbound: how work is actioned back into ClickUp (manual today) + the write-tool surface |
| `04-capabilities-and-gaps.md` | Read + write tool capability matrix, limits, and gaps |
| `05-existing-assets.md` | ClickUp-Scout.md (method) + scout-2026-05-24.md (sample) digested |
| `_raw/` | Raw hierarchy, members, custom-fields, chat-channels snapshots |
