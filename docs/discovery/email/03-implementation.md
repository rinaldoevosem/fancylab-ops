# Email — Implementation (message out)

The outbound half of the loop: turning a needs-reply thread into a reply. This is the **most mature implementation side of the three tools** — but with one sharp caveat: **the mature capability is DRAFTING, not sending. There is no send capability today.**

## The capability today: draft-only

`email-agent`'s write ceiling is `create_draft`. The agent:
- writes a reply and attaches it to the existing thread (`threadId` set so it threads correctly, not a new message);
- can label and archive;
- **cannot send.** No send tool exists in `email-agent`, `email-scout`, or anywhere in the current toolset.

> **Hard rule #1 (`email-agent`):** *"Never send mail. You only `create_draft`. If the user asks you to send, decline and remind them drafts are the contract."*

A draft is a **proposal**. Rinaldo reads it and sends manually. The draft existing in the thread is the signal that Claude acted; the thread staying **unread** is the signal that Rinaldo still owes it a review (see `02-intake.md` read-state rule).

## Voice & format rules

Replies are written in Rinaldo's voice. The brief (from `email-agent.md`):

- **Short.** 2–5 sentences, usually.
- **Direct.** No filler — no "I hope this email finds you well," "just wanted to circle back," "per my last email."
- **Lowercase-comfortable but professional.** Punctuation matters.
- **Default structure for client replies:** **acknowledge → concrete next action → ETA or owner.**
- **Sign off `— Rinaldo`.**
- **When the answer isn't known:** draft an **internal forward** to the right teammate instead of guessing —
  - Carlos → PM / timeline
  - Luis / Manuk → dev
  - Asya / Aida → design
  - Cynthia → content
  - Robert → billing / marketing
  - Harry → escalation
- **CC discipline:** only CC stakeholders already on the thread; flag any added CC in the report.
- **Honest gaps over confident invention:** leave inline `[TODO: confirm X]` markers rather than fabricating quotes, dates, or commitments.
- **No fabrication, ever:** never invent quotes, commitments, dates, or names. If unknown, draft a holding reply that asks the right teammate.

### Worked examples (from the agent)

- Client DKIM question, no owner known → *"got it Milena — looping in our infra side to verify the DKIM record on Faire's end. will follow up by `<date>`. — Rinaldo"* + `[TODO: confirm who owns DKIM verification — Robert?]`.
- Harry forwards "get this guy to pay" → draft to Harry: *"on it — i'll reach out to `<customer>` today and follow up by EOD tomorrow. — Rinaldo"*, and optionally a separate gentle-nudge draft to the customer on its own thread.
- Design intake referred by Harry → label `DESIGN`, draft CC'ing Asya + Carlos.

### Draft hygiene
- `list_drafts` once at the start of the draft phase; **skip any thread that already has a draft** (never double-draft).
- Don't auto-draft a fifth acknowledgement to a thread-bombing sender (5+ emails, no reply from Rinaldo) — surface it as a "needs decision" instead.

## How automatable

Drafting is **highly automatable and already automated** — `email-agent` produces send-ready drafts unattended in a triage run. The human step is reduced to **read + tap send**. What is *not* automated (by deliberate design) is the send itself: that stays a human action.

## fancylab-ops loop intent: draft-on-merge

fancylab-ops closes the loop while keeping the human gate, recasting "draft → tap send" as **"draft → merge → send."** (`docs/specs/2026-06-06-fancylab-ops-design.md` §4.)

```
inbound client email
  → Scout opens a GitHub Issue (reply-needed)
  → Forager enriches (thread + client profile + SOP) — platform-neutral plan
  → Architect sets action-shape = reply, defines the wax
  → Builder scaffolds branch ops/NNNN-slug + outbox/NNNN-slug/reply.md
  → Worker fills the reply prose (reuses this voice brief) + QA
  → Ship opens the PR (= the proposed action)
  → Guard runs automated checks → PASS/BLOCK verdict on the PR
  → RINALDO reviews the reply diff and MERGES to ratify  ← the human gate
  → Executor (on merge) sends via Gmail → closes issue → 'sent' receipt  ← Honey
```

**The merge is the new "tap send."** A Bee drafts; Guard makes the proposal trustworthy; **Rinaldo's merge is what fires the send.** v1 default: Rinaldo confirms every send (trust can loosen per-category later).

### Critical: the send executor is DESIGNED, not BUILT

The send half is architected in fancylab-ops but **does not work today**:
- The hub's Google token is **read-only**; the executor needs a separate `gmail.send`-scoped refresh token (spec §7.1). *"The single live action does not work until this is done."*
- The executor (`executor/`, a local poller) is a v1 build target, not yet operational.
- **Exactly-once send** is a hard requirement: durable `sent` marker check → `sending` flag → send → write marker → close/receipt; a PR stuck in `sending` on restart **halts and alerts**, never auto-resends.

So today: **drafting = live and mature. Sending = on the roadmap, gated on the send-scope token and the executor build, zero capability right now.**
