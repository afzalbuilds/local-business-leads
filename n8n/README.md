# n8n workflow templates: Local Business Leads

Two importable n8n workflows that run the same Apify Actors as this plugin. Built with the official n8n skills plugin and n8n's instance-level MCP, on n8n Cloud with the Apify node (`@apify/n8n-nodes-apify` 0.9.0).

| File | What it does | Trigger | Needs |
|---|---|---|---|
| `google-maps-leads-to-sheets.json` | A form takes a business type, location, count and country, searches Google Maps and saves each business to a Google Sheet | n8n Form | Apify, Google Sheets |
| `weekly-contractor-leads.json` | Every Monday finds contractors for one trade and city, checks the emails it found and saves the leads to a Google Sheet | Schedule, Monday 07:00 | Apify, Google Sheets |

Actors used: `muhammadafzal/google-maps-scraper`, `muhammadafzal/contractor-lead-scraper` and `muhammadafzal/email-address-validator`.

## Import

1. In n8n, install the verified community node **Apify** (Settings → Community nodes, or search "Apify" in the node panel).
2. Create a workflow, open the **⋯** menu and choose **Import from file**, then pick a JSON file from this folder.
3. Open each node marked with a warning and select your Apify and Google Sheets credentials, and pick your spreadsheet in the **Save lead to sheet** node.
4. Publish the workflow.

## Sheet layout

Both workflows write to a tab named `Leads` and match rows on `Place ID`, so a business found again is updated, not duplicated.

Google Maps workflow header row:

`Place ID` | `Business` | `Category` | `Phone` | `Website` | `Address` | `City` | `Rating` | `Reviews` | `Google Maps URL` | `Search` | `Last seen`

Contractor workflow header row:

`Place ID` | `Business` | `Trade` | `Phone` | `Email` | `Email check` | `Website` | `Address` | `Rating` | `Reviews` | `Google Maps URL` | `Last seen`

`Email check` is `valid`, `risky` or `invalid`. Keep `valid` and `risky` rows and drop `invalid`. Google Maps rows carry no emails; use the contractor workflow when you need them.

## Changing the contractor search

Open the **Find contractor leads** node and edit `businessType`, `location` and `maxResults` in the JSON input. `businessType` accepts `HVAC Company`, `Plumber`, `Roofer`, `Electrician`, `General Contractor`, `Landscaper`, `Painter`, `Pest Control` or `Custom` (with a `searchQuery`).

## Cost

Billed by Apify to your own account. Google Maps: about USD 0.004 per place plus USD 0.006 per run; free Apify plans are capped at 20 places per run. Contractor leads: about USD 0.05 per lead that has a phone, website or email. Email validation: about USD 0.0015 per address. The workflows cap one run at USD 1 (Google Maps), USD 2 (contractors) and USD 1 (validation).

## What was tested

Each workflow passed n8n's validator and a test run with sample data in place of the Apify and Google Sheets calls. The join of email checks onto leads was checked on three sample leads, and the Google Maps form's end page was opened and read. A live run with real accounts was not done, so check the first real run yourself.
