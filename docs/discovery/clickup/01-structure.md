# ClickUp — Workspace Structure

Read-only snapshot, 2026-06-06. Full tree in `_raw/hierarchy.md`.

## Anatomy: spaces → folders → lists

Workspace `8519953` contains **8 spaces**:

| Space | ID | Role | Shape |
|---|---|---|---|
| **WEB DEVELOPMENT** | 90110691628 | Primary production ops (Scout target) | ~90 client lists across 5 folders + many folderless lists |
| **MARKETING** | 90110694465 | Marketing/content/campaign ops | ~60 client lists + sprint/content folders |
| Marketing Client with Location | 26363363 | Discipline-based lists | Design, Technical, Paid, SEO, Link Building, Local SEO |
| Sales & Marketing | 90110748762 | Pipeline | Leads, fancylab.com, idealevolved.com, 📋 Form |
| FANCYLAB | 90110717386 | Internal agency ops | Customer Support, Website, Design, Billing |
| PERFAME | 90110723117 | Brand ops | Shipping, Simple CRM |
| TEMPLATES | 90113933614 | Reusable templates | Content Marketing, Wine Onboarding (Phase 0–4), Website |
| SOPs | 90113935066 | Process docs | List |

**Organizing principle in WEB DEVELOPMENT:** one **list per client domain**; folders
group clients by lifecycle/stage:
- `MARKETING` folder (90117805155) — 26 active client lists
- `SUPPORT` folder (90112350219) — 15 lists (live clients in support)
- `BACKLOG` folder (90116124766) — 19 lists (parked)
- `DISCOVERY` folder (90117999772) — 13 lists (pre-sales / scoping)
- `MAGENTO` folder (90117805168) — empty
- plus ~25 folderless lists incl. the intake funnels `TICKETS`, `RING BUILDER`,
  `DIAMOND SEARCH`, `PROJECT MANAGEMENT`.

## Statuses (observed in WEB DEVELOPMENT)

ClickUp's list API did not return status *definitions*; the following are the statuses
**observed on Rinaldo's open tasks** in the WEB DEVELOPMENT space (other spaces may differ):

`backlog` · `pending` · `Open` · `in progress` · `review` · `client approval`

Of these, **`review`** and **`client approval`** are the operational **gate** statuses —
tasks parked there are waiting on a human decision (see 02). Native task `priority`
(urgent / high / normal / low) is used heavily and is separate from status.

## Members / roles

**23 members** (full list + IDs in `_raw/members.md`). Key constraint: **ClickUp's member
API does not expose role or permission level** — it returns only id/name/username/email.
Functions below are **inferred from Scout activity**, not the API:

- **Reviewer / Guard:** Rinaldo (12760847, rinaldo@evosem.com) — Web Director
- **PM / account:** Asya (75437457), Carlos Castaneda (81312375)
- **Design:** Aida (75437459), Manuk Navasardyan (87337492), Ericka Simanca (87433848)
- **Dev / integrations:** Luis Grosso (81590812), Juan FancyLab (87418081),
  Ishkhan Ghukasyan (81562587), Jason Van Hil (12837311)
- **Support / SEO:** Miguel Rosales (87434010), Azniv Alvrtsyan (81558701)
- **Leadership / sales:** Harry Kabadaian (10728498)
- **External guests:** Olga Garibian & Daniel Quintana (@theclassictshirt.com, client),
  John Dorsey (@idealbrandmarketing.com, partner)

Internal staff split by domain: `@evosem.com` and `@fancylab.com`.

## Custom fields (what tasks carry)

Schema is **domain-specific** (sampled two scopes; full detail in `_raw/custom-fields.md`):

- **WEB DEVELOPMENT (sparse):** only **Phase** (12-stage lifecycle dropdown:
  Discovery → P0 Sales & Billing → … → P10 Post Launch) and **Progress Updates** (an
  AI-autofilled text field summarizing incomplete subtasks). No client-name/budget/hours
  fields — client = the list name; priority = native field.
- **MARKETING (rich, ~25 fields):** Budget, Spend, CPC, CPM, CTR, Clicks, Impr., Conv.,
  Conversion Rate, Primary Marketing Goal, Marketing Task Type, Channel (multi-label),
  Audience Funnel (Cold/Warm/Hot), Requester, Draft/Review/Launch dates, Published Link.

## Chat channels

**175 channels total** (fully enumerated over 2 pages; detail in `_raw/chat-channels.md`).
Mostly one public channel per client domain, plus space-level channels
(`WEB DEVELOPMENT`, `MARKETING`, `SUPPORT`, `All Clients`), ~20 DMs, and dev/role channels
(`Dev-Luis`, `seo`, `Ring Builder`). Notable **gate channels**: `CLIENT APPROVAL`
(private), `REVIEW-FEED`, `PRODUCTION`, `WEB DESIGN` — the chat-side mirror of the
`client approval` / `review` task statuses.
