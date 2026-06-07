# Email — Existing Assets

Email is the **reference model** the other tools lack: a working read-only Scout, an action-capable agent, AND a tuning loop that feeds session feedback back into the Scout's prompt. This is the maturity bar ClickUp and Slack should aim for. The assets, digested and linked.

## `email-agent` — the action-capable executive assistant

**File:** `/Users/melo/.claude/agents/email-agent.md` · **Model:** sonnet

Triages the inbox, applies/maintains the label taxonomy, archives noise, and **drafts replies in Rinaldo's voice**. Cross-references Calendar + ClickUp (read-only) so meeting/ticket-linked threads surface first. **Never sends, never writes Calendar/ClickUp, never deletes.**

- **6-bucket taxonomy:** `URGENT-CLIENT`, `TEAM-INTERNAL`, `MEETING-PREP`, `CLICKUP-LINKED`, `BILLING`, `AUTOMATED-NOISE` (see `02-intake.md`).
- **Default workflow:** load taxonomy → sweep unread (`pageSize:50`) → snippet-bucket every thread → deep-read 5–8 highest-signal → label → archive noise → draft replies → emit a scannable Markdown report.
- **Alternate workflow:** "propose label hierarchy" — samples ≤200 threads, clusters by domain, cross-refs the master sheet + ClickUp, presents a **diff** (renames / new / deprecate), and requires `AskUserQuestion` before any label mutation.
- **Voice brief + hard rules:** see `03-implementation.md` and `04-capabilities-and-gaps.md`.
- **Read-state discipline:** read ≠ act; only archived noise loses `UNREAD`.

> In fancylab-ops, the Worker reuses this voice brief; the Scout's bucketing maps onto the issue labels (`source:email`, `facing:client`, `action:email-reply` / `fyi`).

## `email-scout` — read-only triage

**File:** `/Users/melo/.claude/agents/email-scout.md` · **Model:** sonnet · **Tools:** `search_threads`, `get_thread`, `list_labels`, `AskUserQuestion`, `advisor`, `Write`

Single job: *surface signal, hide noise, never touch inbox state.* The strictly read-only counterpart to `email-agent`.

- **4-tier rubric:** Urgent / Needs reply / FYI / Low priority (see `02-intake.md`). Ties break toward higher urgency.
- **Workflow:** ask for time range first (unless given) → translate to a Gmail query (`newer_than:Nd`, `after:`/`before:`, default scope `in:inbox -category:promotions -category:social`) → `search_threads` (~50 cap) → `get_thread` each → score → emit the digest (no emojis, newest-first, `(none)` for empty buckets) → one-line top-action summary.
- **Hard rules:** read-only (no labeling/drafting/archiving/marking — *"if you find yourself reaching for one, stop"*); never claim an action it can't take; parse-don't-guess on ranges; cap ~50/pass; always run Step 7.
- **In fancylab-ops:** the Email Scout *extends* this agent — same triage brain, but it **writes GitHub issues** instead of a markdown report (spec §5.2).

### Step 7 — the Session Report (the feedback hook)

Every scout run ends with a self-review that makes the tuning loop possible:
1. Call `advisor()` (no args; it gets the full transcript) → *"what went wrong / what to improve."* Quote **verbatim**.
2. Compose a **Session Report** block: time range + threads scanned, **User feedback** (quoted), **Changes / decisions**, **Advisor critique** (verbatim).
3. **Persist to disk** via `Write` at `/Users/melo/fancylab/email/.claude/feedback/SESSION-<UTC-ISO>.md` (`:` → `-`). Scoped to that dir only; never overwrites; fresh timestamp per run.

## `email-queen` — the tuning loop

**File:** `/Users/melo/fancylab/email/.claude/agents/email-queen.md` · **Model:** opus · **Tools:** `Read`, `Edit`, `Write`, `AskUserQuestion`, `advisor`

The prompt editor for `email-scout`. Closes the learning loop: scout's session reports become proposed edits to scout's own prompt.

- **Locked scope:** may only read/edit/write **one** file — `/Users/melo/.claude/agents/email-scout.md` (plus read-only access to `SESSION-*.md` feedback files). Refuses any other file.
- **Inputs:** (1) a direct instruction in the invocation, or (2) the most recent **3** session reports when invoked with no instruction. Direct instruction wins; reports are supplementary.
- **Workflow:** read the current scout file in full (never edit blind) → read relevant session reports → classify **trivial vs non-trivial** → for non-trivial, call `advisor()` once to pressure-test → produce a **literal diff** (old_string / new_string + one-line rationale) → `AskUserQuestion` with **Apply / Modify / Cancel** → only on **Apply** does any write happen.
- **Guardrails:** never write without explicit Apply; preserve scout's foundational rules (read-only, no emojis, ask-for-range, ~50 cap, always-Step-7) unless explicitly told to change them — and require a second confirmation if a change would weaken one; show the diff, don't summarize; no silent scope expansion.

> This is the piece ClickUp/Slack have no equivalent of: a governed, human-gated mechanism for the tool's triage logic to **improve from its own run history.**

## The session-report memory loop (worked example)

**File:** `/Users/melo/fancylab/email/.claude/feedback/SESSION-2026-05-27T00-00-00Z.md`

A real scout session report. Shows exactly what the loop records and why it's valuable:

- **Header:** time range `2026-05-26`, 39 threads scanned.
- **User feedback:** `(none)` this run.
- **Changes / decisions:** range supplied in invocation (Step 1 skipped); a prior run hit a socket error and this one resumed from pre-gathered data; two judgment-call reclassifications (Countdown Timer → FYI; Vendor Update → FYI on re-fetch); 5 stale threads with no same-day activity surfaced as Low-priority with a transparency note; a duplicate-send thread flagged.
- **Advisor critique (verbatim):** caught that the digest **dropped real threads** between analysis and final output (Thirty1 Jewelry, Business Cards, Ooma SMS, Ishkhan calendar), that **silently excluding stale matches is wrong** (list them or add a transparency line), a **count error** (39 not 38), and the **Step-7 order of operations** (advisor → compose with quote → write file → print block).

This is the loop in action: a concrete, specific critique that `email-queen` can turn into a prompt edit — e.g. a "never drop analyzed threads from the digest" rule, or a "always reconcile final count against pages fetched" check. **The feedback isn't abstract; it's a diff-ready improvement.**

## Why this stack is the reference model

```
email-scout (read-only triage)
   │  emits Session Report (advisor critique + decisions) → disk
   ▼
email-queen (reads reports) → proposes scout prompt edit → Rinaldo Apply-gates → scout improves
```

Plus `email-agent` for the action side (label + draft). No other watched tool (ClickUp, Slack) has this combination of a dedicated read-only Scout, an action agent, and a self-improving tuning loop. fancylab-ops builds on it directly: the Email Scout extends `email-scout`; the Worker reuses `email-agent`'s voice brief; the rest of the flight (Forager → Architect → Builder → Guard → executor) is new.
