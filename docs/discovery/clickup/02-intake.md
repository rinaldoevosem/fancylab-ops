# ClickUp — Intake (signal in)

The inbound half of the loop: how work and decisions arrive at Rinaldo. This is the
**ClickUp-Scout's domain** — it reads these signals and ranks them. (Method digest in 05.)

## The signals the Scout surfaces

The Scout produces a daily Markdown "what's on your plate" report, grouped by client
project (= ClickUp folder/list). It detects three signal types:

1. **Mention pings** — comments that @-mention Rinaldo (matched by his `user_id` in the
   comment's `mentions` array, or the standalone word "rinaldo" case-insensitive). These
   are direct requests for a decision: "Is this page approved?", "we need the figma file
   here", "who is going to do this demo?". The Scout records the most recent matching
   comment per task (author, date, ≤200-char snippet) plus a ping count.
2. **Overdue** — `due_date < now` and not null. Heavily present: in the 2026-05-24 sample,
   most projects had multiple overdue tasks, some lapsed by weeks/months.
3. **Status gates** — tasks parked at decision statuses, especially **`client approval`**
   and **`review`**. These are work that is *done* but blocked on Rinaldo's sign-off
   (e.g. anglodiamond's "Add Natural Diamonds Page" sitting at `client approval`,
   ian.club's "Benefits - Aero" at `client approval` since January).

## Structural intake funnels (signal in, made concrete)

Beyond per-task signals, the workspace has **dedicated intake lists/channels** where new
work enters before it's routed to a client list:

- **`TICKETS`** (list 901111987101, WEB DEVELOPMENT) — folderless ticket funnel; holds
  inbound items not yet tied to a client (e.g. "Please Review: E-commerce 1308334 Grand
  American", spam-form fixes). Empty `content` field — used purely as a queue.
- **`Campaign Intake`** (901112439092) and **`Requests`** (901109643593) — MARKETING space
  intake queues.
- **`Incoming Requests`** lists inside the MARKETING folders (Campaign & Promotion
  Management, Content Management, Marketing Sprint) — per-workflow intake.
- **`marketing-tickets`** (901112326298) — marketing-side ticket funnel.
- **Gate chat channels:** `CLIENT APPROVAL` (private) and `REVIEW-FEED` mirror the
  `client approval` / `review` task statuses in chat.

## Sort / priority logic (how the Scout ranks)

From `ClickUp-Scout.md`:
- **Projects** sorted: those with ≥1 mention ping first (by ping count desc), then
  remaining projects by overdue-count desc.
- **Tasks within a project:** mention-pinged tasks first (latest ping date desc), then
  the rest by due date asc (no due date sinks to bottom).
- Scope is **open tasks only** (status type ≠ closed/done), assigned to Rinaldo, in space
  `90110691628`. Read-only throughout.

## Volume snapshot (representative, NOT exhaustive)

> ⚠ This snapshot is **Rinaldo-assigned open tasks in the WEB DEVELOPMENT space only** —
> it is not workspace-wide volume.

- **~130 open tasks** assigned to Rinaldo in WEB DEVELOPMENT (sampled 2026-06-06:
  `clickup_filter_tasks` returned a full page of 100 + a second page of 30). The
  2026-05-24 Scout run independently reported **135 assigned** — consistent.
- These span **~30+ client projects** (the Scout's 2026-05-24 run counted 30 projects,
  21 with mention pings).
- **Status mix observed** in the sample: heavy `Open` / `pending`, plus clusters at
  `client approval` (esp. luxefinejewelers.com — 7+ design tasks awaiting sign-off),
  `review`, `in progress`, and `backlog` (mostly SEO tasks).
- **Priority:** many tasks flagged `urgent` / `high` (e-commerce breakages on
  divinitymetals.com, bey-berk.com checkout/login).
- **Structure scale (workspace, from hierarchy):** 8 spaces; WEB DEVELOPMENT alone holds
  ~90 client lists; 175 chat channels total.

The picture: a high-volume, single-reviewer bottleneck. Many done-but-unapproved tasks and
many overdue tasks converge on one person, which is exactly what the Scout is built to
triage — and exactly why an *implementation* (write-back) loop is the missing piece (03).
