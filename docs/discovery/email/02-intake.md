# Email — Intake (signal in)

The inbound half of the loop: how a raw inbox becomes triaged signal. Two complementary classifiers run here — the **6-bucket taxonomy** (`email-agent`, action-oriented) and the **4-tier urgency rubric** (`email-scout`, read-only digest) — plus sender routing and Calendar/ClickUp cross-references.

## The 6-bucket taxonomy (`email-agent`)

Every unread thread is bucketed at **snippet level first** (no deep read), then the 5–8 highest-signal threads get a full `get_thread` read.

| Bucket | What goes here | Disposition |
|---|---|---|
| `URGENT-CLIENT` | External client (not `@evosem`/`@fancylab`) needing reply or decision — escalations, "needs to pay" forwards, DKIM/domain issues, test-link requests, kickoff replies, contract/scope changes | Label, **draft a reply**, stays unread + in inbox |
| `TEAM-INTERNAL` | Threads from Harry / Carlos / Asya / Aida / Luis / Cynthia / Robert / Manuk / Ishkhan where Rinaldo is on To: or directly asked | Label, draft if Rinaldo owes a reply, stays unread |
| `MEETING-PREP` | Calendar invites, agenda forwards, or threads whose company/participants match an event in the next 7 days (Calendar cross-ref) | Label `MEETINGS`, surface in Calendar-prep section, stays unread |
| `CLICKUP-LINKED` | Subject or sender domain matches a known ClickUp client list with open tasks (resolved only for high-signal `URGENT-CLIENT` threads) | Label, attach ClickUp context, stays unread |
| `BILLING` | `billing@shopify.com`, ClickUp trial reminders, Slack/Stripe payment notices — non-urgent but Rinaldo should see them | Label, stays in inbox (auto-read only with prior approval) |
| `AUTOMATED-NOISE` | GitHub PR bot summaries, Shopify settings/order notifications, "no events today" digests, Figma seat requests, marketing/newsletters, `MAILER-DAEMON` delays, daily automation self-reports | **Archive** (remove `INBOX` + `UNREAD` together) |

**Read-state rule (load-bearing):** analyzing a thread is **not** acting on it. Only `AUTOMATED-NOISE` (archived) loses `UNREAD`. Every other bucket **stays unread even after labeling + drafting** — the draft signals "Claude acted," the unread state signals "Rinaldo still owes it a review." (`BILLING` may auto-read only with prior explicit approval.)

## The 4-tier urgency rubric (`email-scout`)

`email-scout` is the **read-only** triage agent. It scores each thread into exactly one tier and emits a digest — it never labels, drafts, or archives.

| Tier | Definition |
|---|---|
| **Urgent** | Explicit time-sensitive ask (today / EOD / ASAP / by `<hour>`), payment/billing problems, security alerts, client escalations, deliverability failures, calendar conflicts in-window |
| **Needs reply** | Direct question to Rinaldo, decision requested, his input blocks someone else, signed contracts/quotes awaiting his action |
| **FYI** | Status updates, meeting recaps (Fireflies/Fathom/Otter), CC'd informational threads, project notifications, automated reports worth noting |
| **Low priority** | Newsletters, marketing, automated digests, receipts, already-accepted invites, generic platform notifications |

Rule: **when in doubt between two adjacent tiers, place it in the higher-urgency one** and let Rinaldo decide.

### needs-reply vs FYI vs noise — the cut lines

- **Needs-reply** = a direct ask, a decision, or your reply is someone's blocker. → in fancylab-ops this is the only class that earns a full flight to a draft-reply PR.
- **FYI** = you're informed, not asked. Status/recaps/CCs. → in fancylab-ops: issue labeled `fyi`, auto-closed, **no PR**.
- **Noise** = newsletters, platform notifications, bot summaries, receipts. → archive (agent) / `fyi` auto-close (ops).
- **Uncertain** → stays in inbox / unread; ops opens an issue but **no PR** — left for Rinaldo.

## Sender routing

The first cut is always **sender domain** (full map in `01-structure.md`):

- **Internal** (`@evosem` / `@fancylab`) → `TEAM-INTERNAL` if Rinaldo is addressed/asked. Harry's forwards are often client escalations in disguise ("get this guy to pay") → deep-read, may spawn a client-facing draft.
- **Client** (external, not internal) → `URGENT-CLIENT` if it needs a reply/decision.
- **Vendor/tooling** → `BILLING` (money-relevant) or `AUTOMATED-NOISE` (everything else).

## Cross-references

Intake is enriched by two read-only cross-references so meeting- and ticket-linked threads rank first.

### Email → Calendar
For `URGENT-CLIENT` / `MEETING-PREP` threads, extract the company / external participants, then **one batched** `list_events(now → now+7d, fullText:"<company>")` per unique company. Matches surface in the report's Calendar-prep section. A pre-report `list_events(now → now+48h)` flags RSVPs needed (`responseStatus: needsAction`) and client meetings with no prep thread. **Calendar is read-only — surface, never `respond_to_event`.**

### Email → ClickUp
Only for `URGENT-CLIENT`. Resolve the client domain → match against `clickup_get_workspace_hierarchy` (cached once per run) → if the client has a list, `clickup_filter_tasks list_ids:[<id>] include_closed:false` → surface 1–3 most-recently-updated open tasks as operational context. **ClickUp is read-only — never create/update/comment.**

## Volume

`[CAPTURE WHEN CONNECTED]` — live `search_threads` was not reachable this session (see `_raw/reachability.md`). When connected, capture read-only:

- `search_threads query:"in:inbox is:unread" pageSize:50` → unread count + bucket distribution.
- `search_threads query:"in:inbox newer_than:7d" pageSize:50` → 7-day inbound volume.
- Note the per-bucket split (how much is `AUTOMATED-NOISE` vs `URGENT-CLIENT` vs `TEAM-INTERNAL`) — this drives how aggressive auto-archive should be.

**Scan caps (known from the assets, regardless of live volume):**
- `email-scout`: ~50 threads per pass; reports how many matched-but-skipped.
- `email-agent` triage sweep: `pageSize:50`, paginate until exhausted, but **stop and ask** if the unread queue is 200+ (that's a "long-form cleanup session" decision, not daily triage).
- Label-proposal sampling: up to 200 threads max (`newer_than:90d` + starred + important).
