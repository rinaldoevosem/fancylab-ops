# Gmail — Label Inventory (raw)

**Status:** `[CAPTURE WHEN CONNECTED]` — live `list_labels` was not reachable this session (see `reachability.md`).

The inventory below is the **"Current taxonomy" snapshot transcribed from `email-agent.md`** (lines ~146–154), NOT a live `list_labels` dump. The agent file itself flags this as a snapshot: *"call `list_labels` for the live state."* Treat as approximate / possibly stale.

## Current user labels (from agent file, not live)

### Status prefixes
- `HIGH/`
- `HIGH/RI Jewelers`
- `HIGH/Mervis Diamond`

### Support
- `SUPPORT`
- `SUPPORT/Narine`
- `SUPPORT/Car Buying Pros`
- `SUPPORT/Rockher`
- `SUPPORT/ian.club`
- `SUPPORT/Global Rings`
- `SUPPORT/Luis`
- `SUPPORT/kirkkara.com`
- `SUPPORT/LJG`
- `SUPPORT/Jason`
- `SUPPORT/Ishkhan`
- `SUPPORT/Barclays`

### Production
- `PRODUCTION`
- `PRODUCTION/Divinity Metals`
- `PRODUCTION/erikarecords.com`
- `PRODUCTION/Libra`
- `PRODUCTION/Beverly Hills Watch`
- `PRODUCTION/Mastercraft`

### Topical
- `MEETINGS`
- `REPORTS`
- `DATA`
- `LEARN`
- `JCK`
- `clickup`
- `IAN-DEALS`
- `DESIGN`

### Marks
- `approved`
- `✔`
- `✔✔`

## Proposed redesign (NOT live — a proposal in `email-agent.md`)

`email-agent.md` also carries a *proposed* label hierarchy for the "propose label hierarchy" workflow. This is a redesign awaiting Rinaldo's approval, **not the current state**:

- `Clients/<DomainKey>` — one per active client, keyed on master-sheet domain (replaces the `SUPPORT/{name}` + `PRODUCTION/{name}` split, which causes drift).
- `Stage/Sales`, `Stage/Onboarding`, `Stage/Production`, `Stage/Support`, `Stage/Wind-Down` — lifecycle, decoupled from identity.
- `Priority/HIGH`, `Priority/NOW` — urgency override (collapses `HIGH/RI Jewelers`-style labels).
- `Team/Internal`, `Team/Harry`, `Team/Carlos`, `Team/Designers`, `Team/Devs`.
- `Topic/Meetings`, `Topic/Reports`, `Topic/Design`, `Topic/Learn`, `Topic/Billing`.
- `Automated/GitHub`, `Automated/Shopify`, `Automated/Klaviyo`, `Automated/Calendar`, `Automated/ClickUp`, `Automated/Figma`, `Automated/LinkedIn` — auto-archive candidates.
- `Newsletter` — marketing / promo / digest (auto-archive).
- **Preserve as legacy** (don't touch unless asked): `JCK`, `IAN-DEALS`, `DATA`, `approved`, `✔`, `✔✔`.

## When capturing live

Replace the "Current user labels" section above with the verbatim `list_labels` output (label `id` ↔ `name` pairs, including system labels: `INBOX`, `UNREAD`, `STARRED`, `IMPORTANT`, `SENT`, `DRAFT`, `SPAM`, `TRASH`, `CATEGORY_*`). Note which proposed labels (if any) have since been created.
