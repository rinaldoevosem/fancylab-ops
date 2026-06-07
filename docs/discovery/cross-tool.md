# Cross-tool — shared assets, linkage gaps, capability matrix

Synthesis across Slack / ClickUp / email. See each tool's folder for detail.

## Shared assets (the connective tissue today)

| Asset | What it is | Where | Who uses it |
|---|---|---|---|
| **CLIENT DATABASE** | Per-domain folders (`{domain}/raw.md` profile + synced Google Docs) — the canonical client record | `/Users/melo/Library/CloudStorage/GoogleDrive-rinaldo@evosem.com/My Drive/CLIENT DATABASE/` | `client-agent` reads/writes the CLIENT PROFILE doc |
| **Master Client spreadsheet** | Canonical row per client (name, contact, platform, status, Drive link, ClickUp list ref, Figma link) | Google Sheet `1qHphT19x9ik_f1BvfO0EiTx4N1XNnEMLT_JJABHcAdg`, tab **WEB DEVELOPMENT** | `client-agent`, `master_sheet.py` CLI |
| **Fireflies** | Meeting transcripts/summaries (decisions, action items, launch dates) | `mcp__fireflies__*` (reachable ✓) | `client-agent` step 4 only — **not** wired into email/ClickUp scouts |
| **Email memory loop** | Per-run session reports → prompt tuning | `/Users/melo/fancylab/email/.claude/feedback/SESSION-*.md` → `email-queen` | the only self-tuning loop in the system |
| **client-agent** | The only **cross-tool synthesizer** — reads Gmail + Drive + ClickUp + Slack + Fireflies + master sheet to build a Client Profile | `/Users/melo/.claude/agents/client-agent.md` | on-demand, per client; persists to the CLIENT PROFILE doc, not a shared graph |

## The cross-tool linkage gap (the big one)

There is **no shared client entity key** tying a client's Slack thread ↔ Gmail thread ↔ ClickUp task ↔ CLIENT DATABASE record together. Consequences:

- A decision made in **Slack** isn't reflected in **ClickUp**; a **ClickUp** "client approval" gate isn't visible from the **email** triage; an inbound **email** complaint about an overdue **ClickUp** task is correlated only by a human eyeballing a task-ID.
- `client-agent` *assembles* a cross-tool view on demand (per client), but it's ephemeral — nothing persists the correlation. Each scout reads its own tool in isolation.
- The unifying primitive a unified operations system needs is a **client/entity key** (domain-slug is the de-facto candidate: `#{domain-slug}` Slack channels, `{domain}/` DATABASE folders, client domains in email routing, client lists in ClickUp). Worth making explicit when the build resumes.

## Consolidated capability matrix

| Tool | Intake (read) | Implementation (write) | Memory / tuning | Live this session | Maturity |
|---|---|---|---|---|---|
| **Email** | ✅ Scout + 6-bucket taxonomy + 4-tier rubric + sender routing | ⚠️ **draft-only** (drafts in Rinaldo's voice; **no send**) | ✅ `email-queen` session-report loop | ❌ Gmail MCP absent (declared, not connected) | **High** — reference model |
| **ClickUp** | ✅ Scout taxonomy (mention pings, overdue, status gates) | ❌ **none** — Scout is read-only; write surface exists (`create_task_comment`, `update_task`, …) but unused | ❌ none | ✅ read tools reachable | **Medium** — intake only |
| **Slack** | ⚠️ partial (embedded in `client-agent` step 6; **no scout**) | ❌ manual (write surface exists, unused) | ❌ none | ❌ MCP not reachable | **Low** — context-only |

Legend: ✅ present · ⚠️ partial/constrained · ❌ absent.

## Reachability reality (and why it matters)

This session: **ClickUp, Fireflies, Figma = connected.** **Gmail, Slack, Calendar, Drive (the claude.ai suite) = not connected.** The session-start note warns claude.ai MCPs "may be absent in headless/cron runs." Implication for the build:

- Anything that must run **unattended/scheduled** (a Slack scout, an email scout, draft-on-merge) needs the claude.ai MCP **reliably reachable in that execution context** — which is not guaranteed today. This is a foundational dependency to resolve, separate from the application logic.
- ClickUp automation (the missing write-back loop) has **no such dependency** — its MCP is reachable now. That makes ClickUp write-back the lowest-friction first automation to actually run.

## Implications for the resume conversation (observations, not decisions)

1. **Email** already has the shape the others lack (Scout + implementation + memory). Treat it as the template to replicate for ClickUp and Slack.
2. **ClickUp's write-back loop** is the highest-value untapped automation *and* the most reachable — a strong candidate for the first real closed loop, starting with `create_task_comment` behind the Guard.
3. **Email & Slack automation are gated on MCP reachability**, not on missing design. Decide how the operating context guarantees the claude.ai suite is connected before relying on them.
4. **A shared client/entity key** is the missing primitive for a *unified* operations view — promote `domain-slug` from convention to explicit join key.
5. **Fireflies** is reachable but under-used (only `client-agent`). Meeting decisions/action-items are a rich, currently-untapped intake source for both email and ClickUp scouts.

*These are inputs for the strategy discussion — the build remains parked until Rinaldo decides how to proceed.*
