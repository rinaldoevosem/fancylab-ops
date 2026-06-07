# fancylab-ops — Discovery & Data-Hygiene

Read-only documentation of the **real operating context** of each tool, captured before resuming the fancylab-ops build. Every tool is documented as a **two-way loop**: *intake* (signal in) and *implementation* (message/action out). Gather, invent nothing.

> **Status:** discovery phase. The build (the unmerged `plan1/foundation-executor` branch) stays parked until this is reviewed and Rinaldo says resume. **No tool was written to** — no email sent/drafted, no ClickUp task/comment/status changed, no Slack post, no label edited.

## Map

| Folder | Tool | Loop maturity | Live capture this session |
|---|---|---|---|
| [`slack/`](slack/00-overview.md) | Slack | intake *partial*, implementation *manual* | ❌ MCP not reachable → consolidated + templated |
| [`clickup/`](clickup/00-overview.md) | ClickUp | intake ✓ (Scout), implementation ✗ (read-only) | ✅ live read-only capture |
| [`email/`](email/00-overview.md) | Email (Gmail) | intake ✓, implementation ✓ (draft-only), + memory loop | ❌ MCP not reachable → consolidated + templated |
| [`cross-tool.md`](cross-tool.md) | All | shared assets, linkage gaps, capability matrix | synthesis |

Each tool folder follows the same template: `00-overview` · `01-structure` · `02-intake` · `03-implementation` · `04-capabilities-and-gaps` · `05-existing-assets` · `_raw/`.

## Three cross-cutting findings (detail in `cross-tool.md`)

1. **Reachability split (this session).** Only **ClickUp, Fireflies, NotebookLM, Figma** were live. The **claude.ai MCP suite — Gmail, Slack, Calendar, Drive — was not connected.** So live capture (and any future automated action) for email and Slack depends on that suite being reachable in the operating context. Email/Slack here were documented from their existing agents, with the live layer marked `[CAPTURE WHEN CONNECTED]`.
2. **Maturity asymmetry.** Email is the reference model (Scout + draft implementation + a self-tuning memory loop). ClickUp has intake but **no write-back loop**. Slack has only partial intake and no scout/implementation/memory.
3. **No cross-tool linkage.** Nothing ties the same client's Slack thread ↔ Gmail thread ↔ ClickUp task together. The one cross-tool synthesizer, `client-agent`, reads all sources per-client on demand but persists nothing as a shared graph.

## What this is for

Repurposable source material — for SOPs, a knowledge base, Forager context, and the facts that should drive the **resume-the-implementation** conversation. It is *documentation*, not decisions; `cross-tool.md` lists implications, not commitments.

## PRs

Slack → PR #2 · ClickUp → PR #3 · Email → PR #4 · Cross-tool + index → this PR.
