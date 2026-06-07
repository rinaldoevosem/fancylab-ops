# ClickUp — Chat Channels (raw snapshot)

Captured read-only via `clickup_get_chat_channels` (2 pages, fully enumerated).
Date: 2026-06-06. **Total = 175 channels** (page 1 = 100 `has_more:true`, page 2 = 75
`has_more:false`).

Composition (approximate, by inspection):
- **Per-client channels** (majority): one public channel per client domain, e.g.
  `anglodiamond.com`, `divinitymetals.com`, `ian.club`, `feyjewelers.com`,
  `barclaysjewelers.com`, `luxefinejewelers.com`, `Steindiamonds.com`, etc. Many clients
  have both a `domain.com` channel and a display-name channel ("Diamond Wish", "Kirk Kara").
- **Space-level channels:** `WEB DEVELOPMENT`, `MARKETING`, `ADVERTISING`, `SUPPORT`,
  `Sales & Marketing`, `PERFAME`, `DATA & INVENTORY`, `All Clients`, `FANCY LAB`.
- **DMs:** ~20 private 1:1 direct-message channels (type `DM`).
- **Dev/role channels:** `Dev-Luis`, `Dev-Vahagn`, `Dev-Narine`, `Dev-Emil`,
  `Dev: @Jason + Harry`, `Data-Lilit`, `seo`, `Shopify Theme`, `Ring Builder`,
  `RING BUILDER`, `DIAMOND SEARCH`, `Inventory Management`.

## Intake / gate channels (the "signal in" and "review" surfaces)

These non-client channels function as operational gates rather than client comms:

| Channel | Visibility | Creator | Role |
|---|---|---|---|
| `CLIENT APPROVAL` (8408h-40711) | PRIVATE | Asya (75437457) | Client sign-off gate |
| `REVIEW-FEED` (8408h-31071) | PUBLIC | Rinaldo (12760847) | Review queue / feed |
| `PRODUCTION` (8408h-28951) | PUBLIC | Rinaldo (12760847) | Production hand-off |
| `WEB DESIGN` (8408h-20211) | PRIVATE | Asya | Design coordination |
| `Welcome` / `Template Guide` channels | PUBLIC | — | Onboarding |

> `CLIENT APPROVAL` and `REVIEW-FEED` are the chat-side mirror of the `client approval`
> and `review` task statuses (see 02-intake.md) — the same gate appears as both a task
> status and a chat channel.

Note: channel message *content* was NOT read (would require `clickup_get_chat_channel_messages`);
this is a structural inventory only, consistent with the read-only / general-depth scope.
