# fancylab-ops — Design Spec (v1, first slice)

**Date:** 2026-06-06
**Status:** Draft for review
**Owner (Beekeeper):** Rinaldo (rinaldo@evosem.com)
**Repo:** https://github.com/rinaldoevosem/fancylab-ops

---

## 0. Purpose

Turn Rinaldo's director role into a reviewable, mostly-automated operation. Every signal
that would normally ping him across channels (email, ClickUp, Slack) is detected, triaged,
and — where it needs a response — turned into a **proposed action** he can approve from one
place. Nothing is actioned without his approval.

This spec covers **only the first slice**: a wide-watch attention router plus one fully
closed action loop (email reply). Everything else is named in §6 as out-of-scope-for-v1 so
the boundary is honest.

The system is an instance of the **Purple Bee** framework
(`/Users/melo/shopify-builder/PURPLE-BEE.md`) applied to the *operations* domain.

---

## 1. Three resolved decisions (load-bearing)

1. **System of record = literal GitHub.** Real issues and PRs in a private repo are the
   operational store. The hub reads the GitHub API. **Merge is what fires an action.**
2. **Wide watch, one live action.** Ingestion is wide (email + ClickUp now, Slack once its
   MCP is wired; client-facing *and* internal). Execute-on-merge is narrow: only an **email
   reply** actually executes in v1. ClickUp/Slack issues are opened and visible but
   watch-only.
3. **Review surface = GitHub-native + a read-only hub Inbox.** Approve/merge happens in the
   GitHub app/web (free, mobile, robust). The hub gains a read-only Inbox (reads the GitHub
   API) as the single-glance board. Full in-hub merge UI is a later upgrade.

---

## 2. Purple Bee mapping (the operations Hive)

**Hive:** Rinaldo's operations as one definition — *source of truth* = comms + knowledge
base (Gmail threads, ClickUp tasks, Slack msgs, CLIENT DATABASE, SOPs); *target* = action
systems (Gmail send, ClickUp update, Slack post); *repository* = `fancylab-ops`. On-disk
workspace = `/Users/melo/fancylab/`.

**Two languages, one seam** (the portability principle):
- **Scout + Forager speak the universal "what"** — never name GitHub or Gmail. Decompose the
  firehose into per-signal requests; gather nectar and refine into a platform-neutral plan.
- **Architect + Builder + Worker + Guard speak the platform "how"** — GitHub branches, the
  reply-file format, Gmail send mechanics.
- Adding Slack/ClickUp execution later reuses the discovery layer; only the build layer gets
  new briefs.

| Bee | In this domain | Speaks |
|---|---|---|
| **Beekeeper** | Claude main session + owner of the knowledge base / SOPs / memory Foragers read. Ignites flights, surfaces gates. | — |
| **Scout** | Per-source watcher (email, ClickUp, Slack-later). Decomposes firehose → one *request* (= one GitHub issue) per real signal. Proposes, never acts. | universal |
| **Forager** | Enriches one issue: thread + client facts + matching SOP → a platform-neutral *plan* comment. Flags gaps, invents nothing. | universal |
| **Architect** | Decides action-shape (`reply` / `none` / `escalate`) and the wax of the response. | platform |
| **Builder** | Scaffolds: branch + draft-reply file with markers + PR wiring. | platform |
| **Worker** | Fills the reply prose; runs QA. | platform |
| **Guard** | **HITL** gate. Runs automated checks and proposes a verdict (PASS/BLOCK); the verdict is final only when **Rinaldo ratifies it by merging**. Not autonomous in v1. | platform |

**Guard is human-in-the-loop.** Guard's automated checks make the proposal trustworthy; the
**merge is Guard's human approval step**, performed by Rinaldo. Per Purple Bee principle 5,
trust loosens per-gate over time — low-risk categories could let a Guard PASS auto-merge
later — but v1 default is: Rinaldo confirms every send.

---

## 3. The artifact chain, on GitHub

One private repo, **`fancylab-ops`** (production level). Hub code stays out of it (separate
concern; honors "meta vs production must never mix" and "no git on the hub yet"). A meta/code
repo comes later.

The framework's artifact chain maps 1:1 onto GitHub primitives, all carrying the shared
`NNNN-slug` ID so one flight is traceable end-to-end:

| Artifact | Bee | GitHub representation |
|---|---|---|
| request | Scout | **Issue** — `Reply needed: <subject>`, body = signal summary + source link |
| plan | Forager | structured **comment** on the issue (the enriched, platform-neutral plan) |
| architecture | Architect | action-shape decision recorded on the issue |
| scope + scaffold | Builder | **branch** `ops/NNNN-slug` + `outbox/NNNN-slug/reply.md` (front-matter: to, subject, thread-id; body = markers) |
| filled | Worker | the completed `reply.md` |
| shipped | Ship | the **Pull Request** = the proposed action |
| VERIFIED | Guard | automated checks → **PASS/BLOCK** status + verdict comment on the PR; gates the merge button |
| **Honey** | executor (on merge) | merge → executor sends the email → closes the issue → `sent` receipt comment |

### Label taxonomy (classification = labels, not separate pipelines)

- `source:email` · `source:clickup` · `source:slack`
- `facing:client` · `facing:internal`
- `client:<name>`
- `level:production` (vs `level:meta` later)
- `priority:p1` · `priority:p2` · `priority:p3`
- `action:email-reply` (only one with a live executor in v1) · `action:none` · `action:manual`
- `guard:pass` · `guard:block`
- `fyi` (no-reply, auto-closed)

### No PR spam

Most email is FYI/newsletter/cc. Scout + Forager classify:
- **reply-needed** → full flight to a PR
- **no-reply** → issue labeled `fyi`, auto-closed
- **uncertain** → issue stays open for Rinaldo, **no PR**

Only genuine reply-needed signals ever become PRs.

### What Forager reads (v1, minimal)

The existing CLIENT DATABASE (`/Users/melo/Library/CloudStorage/GoogleDrive-.../CLIENT
DATABASE/`) + memory files + a thin seed `sops/` folder in this repo. The full knowledge
base is a later subsystem.

### Data sensitivity (conscious v1 decision)

Storing operations in GitHub means full email-thread bodies and CLIENT DATABASE facts land
in the repo (issue bodies, Forager plan comments, `outbox/*/reply.md`). **v1 decision:** the
repo is **private** and single-user, so full content is stored **unredacted** — accepted as
a deliberate trade for simplicity and fidelity, not an oversight. Guard's checks still scan
the *outbound* reply for leaks (e.g., quoting another client). Redaction-at-rest of issue
bodies is a later option, noted here so the choice is explicit, not implicit.

---

## 4. The first Flight — one email, ping → sent

1. **Ignition (scheduled).** A cron wakes the **Email Scout** every N minutes.
2. **Scout** decomposes new Gmail threads → per-thread requests; classifies reply-needed /
   fyi / uncertain. Reply-needed client email → opens Issue #NNNN with labels.
3. **Forager** gathers nectar (thread + client profile + SOP) → posts the plan comment;
   flags gaps.
4. **Architect** sets action-shape `reply`; defines the wax.
5. **Builder** creates branch `ops/NNNN-slug`; scaffolds `outbox/NNNN-slug/reply.md`.
6. **Worker** fills the reply in Rinaldo's voice (reuses email-agent voice brief); runs QA.
7. **Ship** commits, pushes, opens the PR; dispatches Guard.
8. **Guard (HITL)** runs automated checks → posts PASS/BLOCK. BLOCK routes back to the owning
   Bee and loops.
9. **Rinaldo** sees the PR (GitHub app or hub Inbox), reads the reply diff, **merges to
   ratify** — or comments to send it back. (Guard's human step.)
10. **Executor** (local poller) sees the merged `action:email-reply` PR → reads `reply.md` →
    **sends via Gmail** → closes #NNNN → drops a `sent` receipt. **= Honey.**

Scout's next run sees the closed issue and won't re-open it (dedup by thread-id).

---

## 5. Components to build (v1)

1. **`fancylab-ops` repo structure** — label set, `outbox/`, `sops/` seed, `executor/`, Bee
   brief files (`bees/*.md`), this `docs/`.
2. **Email Scout** — extends existing `email-scout`; writes GitHub issues instead of a
   markdown report. Classifies reply-needed / fyi / uncertain.
3. **ClickUp Scout** — extends existing `ClickUp-Scout`; writes GitHub issues (watch-only,
   `action:manual`). Proves wide-watch ingestion.
4. **Email Flight** — Forager → Architect → Builder → Worker → Ship, orchestrated as
   subagents per reply-needed issue. Produces the draft-reply PR.
5. **Guard** — automated checks (answers the ask? tone? PII/leak? no unverified commitment?)
   → posts PASS/BLOCK verdict; on BLOCK routes findings to the owning Bee.
6. **Executor** — local poller in `executor/` (Node script, own `.env` with a Gmail
   `send`-scoped refresh token). Polls GitHub for merged `action:email-reply` PRs → sends via
   Gmail API → closes issue → `sent` receipt. Runs on a schedule; later can move into the hub
   / become always-on via Vercel.
   - **Exactly-once send (hard requirement).** Double-sending a client email is real harm, so
     the send must be idempotent against crashes and re-polls. Sequence per merged PR:
     (a) check for a durable `sent` marker (issue label `sent` **and/or** committed
     `outbox/NNNN-slug/.sent` receipt) — if present, **skip**; (b) flip issue to `sending`;
     (c) send via Gmail; (d) **immediately** write the `sent` marker; (e) then close the issue
     + post receipt (these are secondary — a crash here must NOT cause a resend because the
     marker already exists). A PR found stuck in `sending` on restart **halts and alerts** —
     it is never auto-resent; Rinaldo confirms whether it actually went out.
7. **Hub Inbox page** — new read-only route in the hub that reads the GitHub API and renders
   the open issue queue + PR/Guard state. The single-glance board; links out to GitHub to
   act. (Read-only; no mutations from the browser.)
8. **Scheduling** — cron to run the Scouts (via the schedule skill / CronCreate).

**Build order (skeleton-first).** Prove the novel, risky path before adding breadth: build
**email → PR → Guard → merge → real send (exactly-once)** end-to-end *first* (components
1, 2, 4, 5, 6). Only once a single real email has gone out the door do we add the ClickUp
Scout (3) and the Hub Inbox (7). Ingestion breadth and UI are additive once the loop holds.

---

## 6. Out of scope for v1 (named, not forgotten)

- ClickUp / Slack **execute-on-merge** (watch-only in v1)
- **Slack ingestion** (needs a Slack MCP connected first)
- Full **knowledge base / SOP system** (v1 = seed SOPs only)
- **In-hub merge UI** (v1 = review/merge in GitHub; hub is read-only)
- **Metrics / director dashboards**
- **Queen** autonomous orchestration

### Known v1 limitations (accept or escalate)

- **Merge-on-phone is not send-on-phone.** Review/merge is free and mobile (GitHub app), but
  the executor runs on Rinaldo's Mac — so merging from a phone sends the email only when the
  Mac is awake and next polls (could be deferred hours; sends in order on wake). If
  near-real-time send from mobile is required, the executor must move to an always-on home
  (hub on Vercel / a small always-on worker) sooner rather than later. **Needs Rinaldo's
  explicit OK in v1.**

---

## 7. Prerequisites (Rinaldo actions / setup)

1. **Gmail `send` scope** — the hub's Google token is read-only today. The executor needs a
   `gmail.send`-scoped refresh token. One-time: re-run the OAuth bootstrap with the added
   scope and re-consent. **The single live action does not work until this is done.**
2. **GitHub repo** — exists ✓; `gh` authed as `rinaldoevosem` with `repo` scope ✓; cloned to
   `/Users/melo/fancylab/fancylab-ops` ✓.
3. **Slack MCP** — connect whenever; not blocking v1.

---

## 8. Open questions for the implementation plan

1. **Executor home & schedule** — standalone script in `executor/` with its own token (keeps
   ops self-contained) vs a route in the hub (reuses `google-auth.ts`). Leaning standalone
   for v1 isolation. Cron cadence?
2. **Flight orchestration** — Beekeeper (main session) spawns the Bee subagents on demand,
   vs a scheduled routine that runs the whole flight per issue. v1 likely: scheduled Scout
   opens issues; flight runs per reply-needed issue.
3. **Guard checks** — exact automated checks and how BLOCK routes back (label flip + reopen
   PR comment vs new commit request).
4. **Dedup key** — thread-id for email; task-id for ClickUp. Where stored (issue body
   front-matter vs a search by label+external-id).
5. **PR mechanics** — branch protection? Require Guard `pass` status before merge is allowed?
6. **Voice brief reuse** — point Worker at the existing email-agent voice brief, or copy a
   trimmed version into `bees/worker.md`.

---

## 9. Success criteria for v1

- A real inbound client email produces a labeled issue, an enriched plan, and a draft-reply
  PR with a Guard PASS — without manual intervention up to the merge gate.
- Merging that PR causes the email to actually send, the issue to close, and a receipt to
  post.
- **Exactly-once:** no inbound email ever produces two outbound sends, even across executor
  crashes, restarts, or re-polls.
- ClickUp pings appear as issues in the same queue (watch-only).
- The hub Inbox shows the live queue from the GitHub API.
- No duplicate issues across Scout runs.
