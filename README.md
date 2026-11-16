# Local Business Leads: Google Maps, Yellow Pages & BBB inside Claude

Build local business lead lists without leaving Claude. Ask for any business type in any city, region or ZIP code and the plugin returns a table with phone, website, address, opening hours, rating and review count, plus business emails for dentists and contractors. It draws on **Google Maps (any niche, 12 countries), Yellow Pages USA and Canada, BBB profiles**, dedicated dentist, contractor and restaurant lists, and an email validator for cleaning the list before outreach.

It works in Claude chat, in Cowork and in Claude Code, and the same repository also installs as a native plugin in OpenAI Codex, Gemini CLI, GitHub Copilot, Grok, Cursor, Hermes Agent and OpenCode, with the skills usable in many more agents (see [Other AI agents](#other-ai-agents-codex-gemini-cli-copilot-and-more)). There are no developer accounts to set up and nothing to configure. Lookups run as pay-per-record Apify Actors on your own Apify account, and you only pay for rows that come back.

## What you can ask

- "Find 30 plumbers in Chicago with phone numbers and websites."
- "Dentists in Denver and Boulder with emails, as a CSV."
- "Roofing companies in Tampa with verified emails and social links."
- "Top-rated sushi restaurants in Austin with their latest reviews."
- "Give me HVAC companies in Phoenix, then check their BBB ratings."
- "Here are 400 emails from my CRM export. Which ones bounce?"
- "Compare what Google Maps and Yellow Pages list for electricians in Boston."
- "What will it cost to get 500 contractor leads across five Texas cities?"

Claude picks the right source for the niche and country, runs it, waits for the result, removes duplicates and shows one table. When fewer rows come back than you asked for, it tells you why (Google listed fewer places, the free-plan cap, a spending limit).

## Sources covered

| Need | Source used | Rows per run | What you get |
|---|---|---|---|
| Any business type, cheapest list, 12 countries (US, UK, CA, AU, DE, FR, ES, IT, JP, BR, IN, MX) | Google Maps Scraper | up to 200 per search, several searches per run (20 per run on free Apify plans) | name, phone, website, address, city, postal code, rating, review count, category, hours, claimed flag, map link |
| Dentists, orthodontists, dental clinics, worldwide | Dentist Leads Scraper | up to 5,000 across several cities | the above plus specialties, practice email from the website, new-patient flag |
| Plumbers, roofers, electricians, HVAC, general contractors, landscapers, painters, pest control, any custom trade | Contractor Leads Scraper | up to 500 per trade and city | the above plus website emails with a mailbox check, Facebook, Instagram, LinkedIn |
| Restaurants, cafes, bars, any cuisine | Google Maps Restaurant Scraper | up to 120 per search | the above plus cuisine, price level, menu link, rating breakdown, latest reviews |
| Second source for the US | Yellow Pages Scraper USA | up to 10,000 | name, phone, email when listed, website, address, categories, services, payment methods, hours, years in business, BBB-accredited flag |
| Second source for Canada | Yellow Pages Canada Scraper | up to 10,000 | same, with hours, services and social links on request |
| Trust check, US and Canada | BBB Business Scraper | up to 1,000 | BBB rating, accreditation, years in business, complaints, reviews, entity type |
| Clean emails before outreach | Email Validator | up to 10,000 | valid, risky or invalid with reasons; syntax, DNS, mail-server, disposable, role and free-provider checks, without sending mail |

Not included: lists of private individuals, home addresses, personal phone numbers, social media followers, or review scraping beyond what Google Maps shows publicly.

## How to set it up

1. Install the plugin from the Claude directory (claude.ai > Customize > Plugins) or in Claude Code with `claude plugin install local-business-leads@imzlab-local-leads` after adding the marketplace `afzalbuilds/local-business-leads`.
2. Open the plugin's **Connectors** tab and connect the **business-leads** server. It is Apify's official MCP server (`mcp.apify.com`) limited to the eight listing Actors, so a browser window opens for you to sign in or create a free Apify account. Free accounts include monthly usage credit, so you can try it without adding a card. Claude may note that the server is not a separately registered directory connector; that is expected, because it is Apify's hosted server rather than one Imzlab runs.
3. Ask Claude for a list.

In Claude Code, `/mcp` shows the connection state, and `/local-business-leads:leads plumbers in Chicago 30` runs a lookup directly.

## Other AI agents: Codex, Gemini CLI, Copilot and more

The skills and the **business-leads** server are the same in every app; only the manifest each app reads differs. Every app signs you in to Apify in the browser, and lists are billed to your own Apify account exactly as in Claude.

| App | Install | What loads |
|---|---|---|
| Claude (chat, Cowork, Claude Code) | [How to set it up](#how-to-set-it-up) | skills, commands, lead-list-builder agent, business-leads server |
| OpenAI Codex (CLI and app) | [OpenAI Codex](#openai-codex) | skills, business-leads server |
| Gemini CLI (and Qwen Code) | [Gemini CLI](#gemini-cli) | skills, commands, lead-list-builder subagent, business-leads server |
| GitHub Copilot CLI and VS Code | [GitHub Copilot](#github-copilot-cli-and-vs-code) | skills, commands, lead-list-builder agent, business-leads server |
| Grok Build | [Grok Build](#grok-build) | skills, commands, lead-list-builder agent, business-leads server |
| Cursor and Grok Bot | [Grok Bot and Cursor](#grok-bot-and-cursor) | skills, commands, lead-list-builder agent, business-leads server |
| Hermes Agent | [Hermes Agent](#hermes-agent) | skills, business-leads server |
| OpenCode | [OpenCode](#opencode) | skills, commands, lead-list-builder subagent, business-leads server |
| Antigravity, Kiro, Devin, Factory Droid, Augment, Junie, Mistral Vibe | [More agents](#more-agents) | skills and business-leads server, plus commands and agent where the app supports them |
| Any other agent with skills support | [More agents](#more-agents) | skills, plus the server URL added by hand |

Checked on 2026-11-16: the plugin installs, and its skills and server load, in Codex 0.167, Gemini CLI 0.68.0, GitHub Copilot CLI 1.0.112 and OpenCode 1.18.27, and it passes the plugin validators of Claude Code, Gemini CLI, Grok Build and Hermes Agent. Apify sign-in and a live list were not repeated in those apps. The apps under [More agents](#more-agents) follow each app's published plugin format but were not installed.

### OpenAI Codex

Needs Codex CLI 0.142 or newer (`codex update`). The Codex app uses the same settings, so the plugin appears there too.

```bash
codex plugin marketplace add afzalbuilds/local-business-leads
```

```bash
codex plugin add local-business-leads@imzlab-local-leads
```

```bash
codex mcp login business-leads
```

The last command opens Apify's sign-in page in your browser. Ask Codex for a list. You can also call a skill by name, for example `$find-business-leads`. Codex loads the skills and the business-leads server, but not the plugin's commands or the lead-list-builder agent, so ask for a multi-city list in plain words there.

### Gemini CLI

```bash
gemini extensions install https://github.com/afzalbuilds/local-business-leads
```

When Gemini CLI first connects to the business-leads server it opens Apify's sign-in page in your browser; if it does not, run `/mcp auth business-leads` inside Gemini CLI. The extension adds the skills, the `/leads`, `/leads-report` and `/bbb-check` commands and the lead-list-builder subagent. Update it later with `gemini extensions update local-business-leads`.

Qwen Code installs the same extension: `qwen extensions install afzalbuilds/local-business-leads`.

### GitHub Copilot CLI and VS Code

```bash
copilot plugin marketplace add afzalbuilds/local-business-leads
```

```bash
copilot plugin install local-business-leads@imzlab-local-leads
```

In a Copilot CLI session, run `/mcp auth business-leads` to sign in to Apify. The plugin adds the skills, the `/leads`, `/leads-report` and `/bbb-check` commands, the lead-list-builder agent and the business-leads server.

In VS Code, turn on the `chat.plugins.enabled` setting, run **Chat: Install Plugin from Source** from the Command Palette and enter `afzalbuilds/local-business-leads`.

### Grok Build

```bash
grok plugin install afzalbuilds/local-business-leads
```

Grok asks you to confirm that you trust the plugin; its business-leads server stays off until you do. Then open `/mcps`, select **business-leads** and press `i` to sign in to Apify. Ask Grok for a list.

### Grok Bot and Cursor

The plugin is not listed in their marketplaces yet. Once it is, add it from the marketplace and sign in to Apify when the business-leads server asks.

On grok.com you can use the Actors today without the skills: open Connectors, add a custom connector and paste the server URL below.

### Hermes Agent

Needs Hermes 0.21.6 or newer, which loads Agent Plugins packages (a root `plugin.json`).

```bash
hermes plugins install afzalbuilds/local-business-leads --no-enable
```

```bash
hermes plugins enable local-business-leads
```

The package declares the business-leads server, but Hermes signs in to a remote server only when your own configuration asks for it. Register the server under the same name with browser sign-in, so Hermes uses your Apify account for it. Run this in a terminal, not from a script, because Hermes starts the sign-in only when it can talk to you:

```bash
hermes mcp add business-leads --auth oauth --url "https://mcp.apify.com/?tools=muhammadafzal/google-maps-scraper,muhammadafzal/contractor-lead-scraper,muhammadafzal/worldwide-dentist-scraper,muhammadafzal/google-maps-restaurant-scraper,muhammadafzal/yellow-pages-us-scraper,muhammadafzal/yellow-pages-ca-scraper,muhammadafzal/bbb-business-scraper,muhammadafzal/email-address-validator"
```

Hermes opens Apify's consent page in your browser. After you allow access it lists the server's tools (the eight listing Actors plus the run, dataset and record tools); press Enter to enable them all. `hermes mcp test business-leads` confirms the connection, and `hermes mcp login business-leads` signs in again later if the access is ever revoked.

Every list request is a paid Actor run on your Apify account, so the package marks the server `untrusted`: Hermes asks before each call and refuses them in unattended runs. Your own server entry replaces the package's entry, including that setting; add `trust: untrusted` under `mcp_servers.business-leads` in `~/.hermes/config.yaml` to keep the prompts.

To install only one skill instead of the whole package: `hermes skills install afzalbuilds/local-business-leads/skills/find-business-leads`. The business-leads server still has to be added as above.

### OpenCode

OpenCode has no plugin bundle, so the skills, the server and the extras are added separately:

```bash
npx skills add afzalbuilds/local-business-leads -a opencode -g
```

```bash
opencode mcp add business-leads --url "https://mcp.apify.com/?tools=muhammadafzal/google-maps-scraper,muhammadafzal/contractor-lead-scraper,muhammadafzal/worldwide-dentist-scraper,muhammadafzal/google-maps-restaurant-scraper,muhammadafzal/yellow-pages-us-scraper,muhammadafzal/yellow-pages-ca-scraper,muhammadafzal/bbb-business-scraper,muhammadafzal/email-address-validator"
```

```bash
opencode mcp auth business-leads
```

For the `/leads`, `/leads-report` and `/bbb-check` commands and the lead-list-builder subagent, copy the `opencode/commands` and `opencode/agents` folders of this repository into `~/.config/opencode/`.

### More agents

| App | How to install |
|---|---|
| Google Antigravity | `agy plugin install https://github.com/afzalbuilds/local-business-leads` |
| Kiro | Powers panel, **Add Custom Power**, **Import power from GitHub**, then `https://github.com/afzalbuilds/local-business-leads` |
| Devin (formerly Windsurf) | `devin plugins install afzalbuilds/local-business-leads` |
| Factory Droid | `droid plugin marketplace add afzalbuilds/local-business-leads`, then `droid plugin install local-business-leads@imzlab-local-leads --scope user` |
| Augment (Auggie CLI) | `auggie plugin marketplace add afzalbuilds/local-business-leads`, then enable the plugin in `/plugins` |
| JetBrains Junie CLI | `/extensions marketplace add afzalbuilds/local-business-leads`, then `/extensions install local-business-leads` |
| Mistral Vibe | clone this repository into `~/.vibe/plugins/local-business-leads`, run `/reload-plugins`, then `/mcp login business-leads` |
| Cline, Kilo Code, Amp, Goose and other agents that read Agent Skills | `npx skills add afzalbuilds/local-business-leads` installs the skills; then add the server URL below as a remote MCP server with OAuth in that app |

### Server URL

Any MCP client can use the Actors directly by adding this URL as a remote (streamable HTTP) server with OAuth:

```
https://mcp.apify.com/?tools=muhammadafzal/google-maps-scraper,muhammadafzal/contractor-lead-scraper,muhammadafzal/worldwide-dentist-scraper,muhammadafzal/google-maps-restaurant-scraper,muhammadafzal/yellow-pages-us-scraper,muhammadafzal/yellow-pages-ca-scraper,muhammadafzal/bbb-business-scraper,muhammadafzal/email-address-validator
```

## n8n workflow templates

Two ready-made n8n workflows use the same lead Actors: a form that saves Google Maps leads to Google Sheets, and a weekly contractor search with email validation. Import the JSON files from the [n8n folder](n8n/README.md); they need the Apify node for n8n and your own Apify account.

## Pricing

The plugin itself is free. Each row is billed by Apify on your account at the Actor's pay-per-event price. Prices below are the Apify Store prices on the FREE tier as of 2026-11-13. Paid Apify plans get a discount, empty or blocked searches charge no rows, and contractor leads with no phone, website or email are free.

| Source | Per row | Per run start | Example |
|---|---|---|---|
| Google Maps (any niche) | USD 0.004 | USD 0.006 | 100 places in 2 searches: about USD 0.41 |
| Restaurants | USD 0.004 | USD 0.006 | 40 restaurants with reviews: about USD 0.17 |
| Yellow Pages Canada | USD 0.005 | USD 0.013 | 60 listings: about USD 0.31 |
| Yellow Pages USA | USD 0.011 | USD 0.019 | 60 listings: about USD 0.69 |
| BBB profiles | USD 0.025 | USD 0.010 | 10 profiles: about USD 0.26 |
| Dentists (with emails) | USD 0.05 | USD 0.005 | 30 dentists: about USD 1.51 |
| Contractors (with emails and checks) | USD 0.05 | USD 0.0001 | 25 roofers: about USD 1.25 |
| Email validation | USD 0.0015 | USD 0.006 | 1,000 addresses: about USD 1.51 |

Before anything estimated above USD 0.50, Claude tells you the estimate and asks before running. Ask "what will this cost" to get an estimate without running anything.

## Data and privacy

- Your search (niche, place, options) is sent to Apify's hosted MCP server (`mcp.apify.com`), which runs the listing Actor on your Apify account. The Actor reads public business listings and, for emails, the public pages of each business's own website.
- Results are stored in your Apify account's datasets for your plan's retention period, where you can download them as CSV, Excel or JSON. The plugin stores nothing itself and sends nothing to any other service.
- Results are business listings. Use them for business-to-business outreach and research in line with the marketing laws that apply to you (for example CAN-SPAM, CASL, PECR and GDPR) and honour opt-out requests.
- You authorize Apify through OAuth in the plugin's Connectors tab. The plugin itself holds no Apify account details.
- Full policy: https://imzlab.com/privacy#local-leads. Terms: https://imzlab.com/terms#local-leads. Support: https://imzlab.com/support#local-leads

## Frequently asked questions

**Does Google Maps give me emails?** No. Google Maps listings have no emails. The dentist and contractor sources open each business website to find a published email, and Yellow Pages lists emails for some businesses. For other niches, ask for Yellow Pages as a second source, or validate emails you already have.

**How many rows can I get in one go?** Google Maps lists about 120 places per search, so Claude splits big areas into cities or neighbourhoods and merges the results. Free Apify plans are capped at 20 Google Maps places per run; the other sources have no free-plan cap.

**Which countries work?** Google Maps: US, UK, Canada, Australia, Germany, France, Spain, Italy, Japan, Brazil, India and Mexico. Dentists: any country. Contractors: US and Canada work best. Restaurants: any country Google Maps covers. Yellow Pages: US and Canada. BBB: US and Canada.

**What does "risky" mean on an email?** It passed the checks but is lower quality for business outreach: a consumer domain such as gmail.com, a role address such as info@, or a check that was switched off. "Invalid" means bad syntax, a dead domain, no mail server or a disposable domain.

**Is this legal?** The sources read public business listings that the businesses publish to be found by customers. You are responsible for how you contact them.

**Can I import into HubSpot or Salesforce?** Yes. Ask for a HubSpot or Salesforce CSV and Claude names the columns for the import, or asks the contractor source for its HubSpot or Salesforce format directly.

**Is this an official Google, Yellow Pages or BBB tool?** No. This plugin and the Actors are independent and not affiliated with Google, Yellow Pages, Thryv, BBB or Apify.

## Skills and commands in this plugin

- **find-business-leads**: any niche and place, routed to the right source, deduplicated, one table.
- **lead-list-cleanup**: merge lists, remove duplicates, validate emails, export CSV or CRM columns.
- **business-trust-check**: BBB rating, accreditation and complaints for a shortlist.
- **leads-cost-estimate**: estimate the Apify cost before running.
- **/local-business-leads:leads**, **/local-business-leads:leads-report** and **/local-business-leads:bbb-check** commands (Claude Code, Cowork, Gemini CLI, Copilot, Grok Build, OpenCode).
- **lead-list-builder** agent (Claude Code, Cowork, Gemini CLI, Copilot, Grok Build, OpenCode): unattended multi-city build with a spending cap, CSV and summary on disk.

Codex and Hermes Agent load the skills and the business-leads server only; there, ask for a multi-city list in plain words. In Codex the skills can also be called by name, for example `$find-business-leads`.

## Source and support

Source: https://github.com/afzalbuilds/local-business-leads. Issues and feature requests: https://github.com/afzalbuilds/local-business-leads/issues. The Actors are published on Apify Store by Imzlab Technologies under the `muhammadafzal` account.

Last verified: 2026-11-16. Checked with the plugin validators of Claude Code, Gemini CLI, Grok Build and Hermes Agent 0.23.1; install and loading checked in Codex 0.167, Gemini CLI 0.68.0, GitHub Copilot CLI 1.0.112 and OpenCode 1.18.27.
