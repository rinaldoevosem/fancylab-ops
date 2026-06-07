# Email (Gmail) — Discovery Overview

**Tool:** Gmail (`rinaldo@evosem.com`), via the claude.ai Gmail MCP (`mcp__claude_ai_Gmail__*`).
**Owner:** Rinaldo Melo — Web Director at Fancy Lab (Shopify-focused digital agency).
**Discovery date:** 2026-06-06
**Scope of this pass:** read-only documentation. **Zero Gmail writes, drafts, labels, sends, or any mutation were performed.**

## Reachability — decisive, up top

**Gmail was NOT reachable in this discovery session.** The claude.ai Gmail MCP is declared in `email-agent` / `email-scout` frontmatter, but the live tools were **not loaded into this session's tool surface** — `ToolSearch` for `mcp__claude_ai_Gmail__*` returned nothing on every probe. (Same story as Slack, confirmed unreachable this session; the whole claude.ai MCP suite — Gmail / Slack / Calendar / Drive — appears absent. The MCPs live this session are ClickUp, Fireflies, Figma, Vercel.)

Consequence: **live label inventory and inbox volume are templated `[CAPTURE WHEN CONNECTED]`.** Everything else here is consolidated from the mature existing email assets, which do not need a live connection. See `_raw/reachability.md` for the full probe log.

> **"Not loaded this session" ≠ "doesn't exist."** The Gmail tools are real and declared; they're simply not connected in *this* session.

## Email's role in fancylab-ops

Email is the **most mature stack** of the three watched tools (email / ClickUp / Slack). It is the only domain that has both:
- a working **Scout** (`email-scout` — read-only triage), and
- a **tuning loop** (`email-queen` reads session reports and proposes prompt edits).

It is also the **one domain with a live action loop designed in fancylab-ops v1**: an inbound client email is the first-class signal that runs the full Purple Bee flight (Scout → Forager → Architect → Builder → Worker → Guard) and, on merge, sends a reply. ClickUp and Slack are watch-only in v1. (See `docs/specs/2026-06-06-fancylab-ops-design.md` §4.)

## The two-way loop in one paragraph

Email is a **two-way loop: a signal comes in, and a message goes out.** **Intake** (signal in): the Scout sweeps the inbox, buckets every thread (6-bucket taxonomy / 4-tier urgency rubric), separates needs-reply from FYI from noise, routes by sender domain (internal / client / vendor), and cross-references Calendar and ClickUp so meeting- and ticket-linked threads surface first. **Implementation** (message out): for the threads that need a reply, a draft is written in Rinaldo's voice — *acknowledge → concrete next action → ETA/owner, 2–5 sentences, signed `— Rinaldo`* — and that draft is **a proposal, never a send**. Today the write ceiling is `create_draft`; in fancylab-ops the loop closes as **draft-on-merge**: a Bee drafts, Guard checks, and Rinaldo taps merge/send. The whole thing is governed by hard read-only-on-everything-but-draft constraints (never send, never write Calendar/ClickUp, never delete, confirm bulk ops).

## Document map

| File | Contents |
|---|---|
| `00-overview.md` | this file — role, the loop, reachability |
| `01-structure.md` | account, label taxonomy (templated + current snapshot), sender domain map |
| `02-intake.md` | inbound signals: 6-bucket taxonomy, 4-tier rubric, routing, cross-refs, volume |
| `03-implementation.md` | outbound: draft-only capability, voice/format rules, draft-on-merge intent |
| `04-capabilities-and-gaps.md` | read+write capability matrix, scan caps, gaps |
| `05-existing-assets.md` | email-agent / email-scout / email-queen / session-report loop, digested |
| `_raw/reachability.md` | probe log |
| `_raw/labels.md` | label dump (templated; current snapshot from agent file) |
