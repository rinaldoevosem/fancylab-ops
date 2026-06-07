# Slack — Structure (workspace anatomy)

> Reachability: **NOT reachable from main** on 2026-06-06. The live channel inventory
> below is `[CAPTURE WHEN CONNECTED]`. Naming conventions and the client-channel
> pattern are **confirmed from `client-agent.md`**; everything requiring live counts /
> flags / topics is templated. Raw dump placeholder: `_raw/channels.md`.

## Workspace shape

`[CAPTURE WHEN CONNECTED]` — workspace name, total channel count, total member count,
and the public/private split require a live `slack_search_channels` enumeration.

## Channel inventory

The full inventory lives in `_raw/channels.md` (placeholder until captured). Summary
table here once captured:

| Channel | Public/Private | Members | client-facing / internal | Topic |
|---|---|---|---|---|
| `[CAPTURE WHEN CONNECTED]` | | | | |

## Client-facing vs internal — tagging heuristic

Tag each channel from its name/topic. The client-channel pattern is **confirmed**;
the internal patterns are **inferred conventions** (typical workspace layout), to be
validated against the live list.

**Client-facing** (confirmed pattern — `client-agent.md` step 6, lines 95–97):
- `#{domain-slug}` — e.g. `#truwild`, `#morpho`
- `#client-{name}` — e.g. `#client-truwild`
- Heuristic: channel named after a client brand / domain → client-facing.

**Internal / team** (inferred — validate on capture):
- `#general`, `#random`, `#dev`, `#design`, `#eng`, `#ops`, `#announcements`, etc.
- Heuristic: generic function/team name, not a client brand → internal.

When a channel is ambiguous, read its topic/purpose (and who's in it — external email
domains imply client-facing) before tagging.

## Public vs private

`[CAPTURE WHEN CONNECTED]` — `slack_search_channels` returns each channel's privacy
setting (per its result shape used by `client-agent`). Private client channels and
public team channels are both expected; capture the real split.

## Activity / dormancy

`[CAPTURE WHEN CONNECTED]` — read recent messages per channel (`slack_read_channel`,
last ~20) to tag **active vs dormant**. Expect active channels for in-flight client
projects and dormant channels for launched/parked clients.
