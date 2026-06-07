# ClickUp — Custom Fields (raw snapshot)

Captured read-only via `clickup_get_custom_fields`. Date: 2026-06-06.
Two representative scopes sampled: WEB DEVELOPMENT space (sparse) vs a MARKETING
space list (rich). They differ sharply — the custom-data schema is domain-specific.

---

## WEB DEVELOPMENT space (90110691628) — and list anglodiamond.com (901101906101)

Both space-level and list-level returned the **same two fields** (list inherits space):

1. **Phase** — `drop_down`. 12 options modeling the delivery lifecycle:
   `Discovery → P0 Sales & Billing → P1 Client Onboarding → P2 Strategy & Planning →
   P3 Design → P4 Project Setup → P5 Inventory → P6 Development → P7 Testing & QA →
   P8 Client Review & Pre-Launch → P9 Launch → P10 Post Launch`
2. **Progress Updates** — `text` with an **AI auto-fill** config: prompt = "Provide quick
   and short overview of subtasks that are not COMPLETED/DONE … If all items in the
   checklist are completed return COMPLETED". Source = activity, window = 30d.

> WEB DEVELOPMENT carries NO client-name, budget, hours, or priority custom fields.
> Priority is the native ClickUp task `priority` (urgent/high/normal/low), not a custom
> field. Client identity = the **list name** (each client is its own list).

---

## MARKETING space — list "Campaigns" (901109856691) — rich schema (~25 fields)

| Field | Type | Notes |
|---|---|---|
| Primary Marketing Goal | drop_down | awareness / engagement / rank / traffic / leads / revenue / customer value / authority |
| Marketing Task Type | drop_down | Campaign / Promotion / Website / Blog / Social / Email / Creative / SEO / Event / Asset |
| Channel | labels (multi) | Website, Blog, Email, Google, Facebook, Instagram, Twitter, LinkedIn, YouTube, Print, TV, Radio, Podcast, Outdoor |
| Audience Funnel | drop_down | Cold / Warm / Hot |
| Target Audience | short_text | |
| Objective | short_text | |
| Budget | currency (USD) | |
| Spend | currency (USD) | |
| CPC / CPM | currency (USD) | |
| CTR | text | |
| Clicks / Impr. / Conv. | number | ad-performance metrics |
| Conversion Rate | manual_progress + a formula variant | round(100*conv/clicks) |
| Campaign Progress | automatic_progress | rolls up subtasks/checklists/assigned comments |
| Requester | users (single) | who asked for the work — intake attribution |
| Draft Date / Review Date / Launch Date | date | editorial/campaign workflow gates |
| Published Link | url | |
| Email | email | |
| Mockups / Inspiration | attachment | |

> Takeaway: marketing tasks carry real budget/performance/attribution data; web-dev
> tasks do not. Any cross-domain reporting must account for this asymmetry.
