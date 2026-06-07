# Email — Structure

How the Gmail surface is organized: the account, the label taxonomy, and the sender domain map the agents route against.

## Account

- **Single mailbox:** `rinaldo@evosem.com` (Rinaldo Melo, Web Director, Fancy Lab).
- **Single user**, no shared inbox. The agents operate on this one account.
- **Auth:** claude.ai Gmail MCP (`mcp__claude_ai_Gmail__*`). Read + label + draft scopes in the agent definitions; **no send scope** anywhere today (see `04-capabilities-and-gaps.md`).

## Label taxonomy

> **Live `list_labels` not captured this session.** The inventory below is the **"Current taxonomy" snapshot transcribed from `email-agent.md`** — flagged there as a snapshot ("call `list_labels` for the live state"). Treat as approximate. Full raw dump + template in `_raw/labels.md`. Replace with verbatim `list_labels` output `[CAPTURE WHEN CONNECTED]`.

### Current user labels (snapshot, not live)

| Group | Labels |
|---|---|
| **Status** | `HIGH/`, `HIGH/RI Jewelers`, `HIGH/Mervis Diamond` |
| **Support** (per-client) | `SUPPORT`, `SUPPORT/Narine`, `SUPPORT/Car Buying Pros`, `SUPPORT/Rockher`, `SUPPORT/ian.club`, `SUPPORT/Global Rings`, `SUPPORT/Luis`, `SUPPORT/kirkkara.com`, `SUPPORT/LJG`, `SUPPORT/Jason`, `SUPPORT/Ishkhan`, `SUPPORT/Barclays` |
| **Production** (per-client) | `PRODUCTION`, `PRODUCTION/Divinity Metals`, `PRODUCTION/erikarecords.com`, `PRODUCTION/Libra`, `PRODUCTION/Beverly Hills Watch`, `PRODUCTION/Mastercraft` |
| **Topical** | `MEETINGS`, `REPORTS`, `DATA`, `LEARN`, `JCK`, `clickup`, `IAN-DEALS`, `DESIGN` |
| **Marks** | `approved`, `✔`, `✔✔` |

**Known drift:** a single client can appear under both `SUPPORT/` and `PRODUCTION/` prefixes — the agent file calls this out as the main reason a redesign is proposed.

### Proposed redesign (a proposal, NOT live state)

`email-agent.md` also defines a *proposed* hierarchy for its "propose label hierarchy" workflow. **This is a redesign awaiting Rinaldo's approval, not the current taxonomy.** Headline of the proposal:

- `Clients/<DomainKey>` — one per active client, keyed on the master-sheet domain (replaces the `SUPPORT/` + `PRODUCTION/` split).
- `Stage/{Sales,Onboarding,Production,Support,Wind-Down}` — lifecycle, decoupled from client identity.
- `Priority/{HIGH,NOW}` — urgency override.
- `Team/{Internal,Harry,Carlos,Designers,Devs}` — internal traffic.
- `Topic/{Meetings,Reports,Design,Learn,Billing}` — context tags.
- `Automated/{GitHub,Shopify,Klaviyo,Calendar,ClickUp,Figma,LinkedIn}` + `Newsletter` — auto-archive candidates.
- Preserve as legacy: `JCK`, `IAN-DEALS`, `DATA`, `approved`, `✔`, `✔✔`.

Any `create_label` / `update_label` / `delete_label` requires explicit `AskUserQuestion` confirmation first — the redesign is never applied silently. Full detail in `_raw/labels.md` and `05-existing-assets.md`.

## Sender domain map

The agents route every thread by sender domain into one of three classes. (Client names from `email-agent.md`; canonical list resolves via `~/.config/gdocs-cli/master_sheet.py --list-domains` — the same source `client-agent` uses.)

### Internal — colleagues, NOT clients
`@evosem.com` and `@fancylab.com`.

| Person | Address | Role |
|---|---|---|
| Harry Kabadaian | `harry@evosem.com` | Founder & Director of Digital Strategy; escalations / "needs to pay" forwards |
| Carlos | `carlos@fancylab.com` | PM; production sync meetings |
| Asya | `asya@evosem.com` | Design / PM |
| Aida | `aida@evosem.com` | Design / PM |
| Luis | `luis@fancylab.com` | Dev / QA |
| Cynthia | `cynthia@fancylab.com` | Dev / QA (content) |
| Manuk | `manuk@fancylab.com` | Dev / QA |
| Robert | `robert@evosem.com` | Ops / marketing-adjacent (billing) |
| Ishkhan | `ishkhan@fancylab.com` | Production |

### Client — external (anything not `@evosem` / `@fancylab`)
Mostly Shopify jewelry/watch brands, plus a few non-jewelry verticals.

- **Jewelry / watch:** Kirk Kara, Bey-Berk, Watch Collectors, RI Jewelers, Mervis Diamond, Beverly Hills Watch, Mastercraft, Divinity Metals, Libra, Barclays, Rockher, Global Rings, LJG, Diamantaire.
- **Other verticals:** Select Dental, Erika Records, Christopher Salon, Riojoes, IAN.club, Leaselane, Narine, Car Buying Pros, Jason.

Canonical/current list lives in the master sheet — resolve live; the list above is illustrative.

### Vendor / tooling — automation + platform noise
`billing@shopify.com`, `notifications@github.com` (incl. `devactivity-app[bot]`), `klaviyo.com`, `figma.com`, `clickup.com`, `linkedin.com`, `stuller.com`, `calendar-notification@google.com`, Stripe, Slack notifications, `MAILER-DAEMON`, marketing/newsletter senders (Cursor, Railway, DeepLearning.ai, OfferUp, Slydpouch). These mostly land in `BILLING` or `AUTOMATED-NOISE` (see `02-intake.md`).
