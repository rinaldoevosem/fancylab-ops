# Slack — Intake (inbound signals · the Scout view)

> Reachability: **NOT reachable from main** on 2026-06-06. This whole doc is the
> **inbound / Scout** side of the loop. Live message capture (real examples, volumes,
> urgency timing) is `[CAPTURE WHEN CONNECTED]`. The signal *types* below are
> **inferred from spec** — `client-agent.md` step 6 names "latest decisions, asks,
> escalations" as what it reads Slack for — plus standard agency-Slack patterns.
> No real message has been read; do not treat the examples as captured.

## What pings Rinaldo via Slack

Slack is a direct, low-latency channel: signals land in client/team channels and
ping Rinaldo with **no Scout in between** triaging or routing them. The expected
intake signal types:

| Signal type | What it is | Representative example (illustrative — `[CAPTURE WHEN CONNECTED]`) | Typical urgency |
|---|---|---|---|
| **Client question** | Client asks about scope, status, a feature, or a how-to | "Hey, can we get the new PDP layout on staging before Friday?" | Medium — same-day reply expected |
| **Approval request** | Client/team asks Rinaldo to sign off on a deliverable or decision | "Design's ready for review — good to push to prod?" | Medium–High — blocks work until answered |
| **Dev blocker** | Team flags something stuck needing a decision or input | "Theme build failing on the checkout extension — need the API key" | High — actively blocking delivery |
| **Escalation** | A problem surfaced that needs Rinaldo's attention now | "Client's live store is showing the wrong price on bundles" | Critical — drop-everything |
| **FYI / status** | Non-actionable updates, launch notices, context | "Launched truwild.com 🎉 monitoring for issues" | Low — read, no reply needed |

> Note: `client-agent.md` step 6 explicitly characterizes its Slack read as pulling
> "latest decisions, asks, escalations" — i.e. the **decision / ask / escalation**
> rows above are the spec-named signal types. The others (question, blocker, FYI) are
> standard agency-Slack categories, to be confirmed against live message capture.

## Volume / activity

`[CAPTURE WHEN CONNECTED]` — rough message volume per channel and which channels are
hot vs dormant come from `slack_read_channel` (last ~20 per channel). Expect:
- High volume in channels for in-flight client projects.
- Low/dormant volume in launched or parked client channels.
- Steady-but-low volume in internal team channels (`#general`, etc.).

## Urgency patterns

`[CAPTURE WHEN CONNECTED]` — confirm timing once captured. Inferred pattern: escalations
and dev blockers carry implicit "now" urgency (often with @-mentions); client questions
expect same-day; FYIs are read-only. A future Slack Scout would route by exactly this
axis — see `04-capabilities-and-gaps.md`.

## Capture checklist (when connected)

- [ ] Pick 3–5 representative channels (mix of client + internal).
- [ ] `slack_read_channel` last ~20 messages each (respect client-agent's **max 1
      channel deep-read per run** cap if running inside that agent; a dedicated capture
      run can read several).
- [ ] Bucket messages into the signal types above; replace illustrative examples with
      1–2 **real, short** examples per type (do NOT dump full logs; scrub `$` figures).
- [ ] Record approximate volume + which channels are active vs dormant.
