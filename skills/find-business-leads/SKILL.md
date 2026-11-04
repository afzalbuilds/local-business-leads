---
name: find-business-leads
description: Find local business leads for any niche and city and return a list with phone, website, address, rating, hours and business email. Use when the user says "find leads", "lead list", "list of plumbers in", "dentists in Denver with emails", "restaurants in Austin", "scrape Google Maps", "Yellow Pages", "local businesses", "prospect list", "companies in my area", "build a B2B list", "who are the HVAC companies in", or asks for businesses of any type in a city, region, ZIP or country, with or without contact details.
---

# Find business leads

You build lead lists by running Imzlab Technologies' listing Actors on Apify through the **business-leads** connector. Each Actor is a tool on that connector whose name ends with the Actor slug, for example `muhammadafzal--google-maps-scraper`. The connector also exposes `get-actor-run`, `get-dataset-items` and `get-key-value-store-record`, which you need to collect results.

Read `references/actor-contracts.md` for the exact input and output of every Actor before your first call in a conversation. `references/niche-routing.md` says which Actor to use for which niche and country, and `references/lead-schema.md` defines the normalized columns, deduplication and CSV export.

## Steps

1. **Pin down the request.** You need a niche (business type), a place, a count, and what the user wants per row. Infer what you can: "plumbers in Chicago" is niche plus city; default count is 20; default row is phone, website, address, rating and review count. Ask only when the niche or the place is missing. Note extras the user asked for: emails, reviews, hours, social links, a second source, a trust check, or "cheapest".

2. **Route** with `references/niche-routing.md`. Dentists go to the dentist Actor, trades (including HVAC) to the contractor Actor, restaurants and cafes to the restaurant Actor, everything else to Google Maps. Yellow Pages USA or Canada is a second source or the user's explicit choice. BBB is only for trust checks. When the user wants emails for a niche that Google Maps alone cannot provide (any niche outside dentists and trades), say that Google Maps returns no emails, and offer Yellow Pages as the second source or the email validator on emails the user already has.

3. **Size the run.** Set `maxResults` to the requested count. One Google Maps search lists about 120 places, so for more than 100 rows in one city split by neighbourhood or ZIP, and for a region propose the main cities. The contractor Actor takes one trade and one city per run; start one run per city, up to 3 at a time. On a free Apify plan the Google Maps Actor stops at 20 places per run; say so when the user asks for more.

4. **Estimate the cost** with the price table in `references/actor-contracts.md`. If the estimate is USD 0.50 or less, run now and report the cost afterwards. Above USD 0.50, show the estimate as one line ("about USD 1.25 for 25 contractor leads with emails, billed to your Apify account") and run only after the user agrees. For a list over 200 rows, suggest a 20-row sample first so the user can check quality.

5. **Run the Actors.** Call each tool with the input shape from the contract file; never add fields the contract does not list. Pass `callOptions.maxTotalChargeUsd: 1` (or the estimate, whichever is higher) for the contractor Actor, which refuses lower limits. Pass `callOptions.timeout: 600` for restaurant runs over 40 rows and `900` for Yellow Pages Canada runs with `extractDetails`.

6. **Wait for every run to finish.** A tool response with status `RUNNING` or `READY` is not a result. Call `get-actor-run` with the `runId` and `waitSecs: 45`, repeat until the status is `SUCCEEDED`, `FAILED`, `ABORTED` or `TIMED-OUT`, up to 8 times. Never report a run as done while it is still running, and never invent rows.

7. **Fetch the results**, whatever the run status was:
   - `get-dataset-items` with the run's default `datasetId` and `limit` equal to `maxResults` (at least 20);
   - `get-key-value-store-record` with the run's default `keyValueStoreId` and the record named in the contract (`OUTPUT` for Google Maps, contractor, dentist, restaurant, Yellow Pages USA, BBB and the validator; Yellow Pages Canada writes none, so use the run's status message).
   A run can end `SUCCEEDED` with fewer rows than requested. The record's `stopReason`, counts and `warnings` explain the shortfall: Google listed fewer places, the free-plan cap applied, the user's maximum cost per run was reached, or requests were blocked.

8. **Normalize and deduplicate** with `references/lead-schema.md`. Merge rows from several runs or sources into one list, fill gaps across sources, and count duplicates removed.

9. **Answer** with one table and a short cost line, as described below. Then offer the natural next step: validate the emails and export a CSV (`lead-list-cleanup` skill), check BBB ratings for the shortlist (`business-trust-check` skill), or add a second source.

## Answer format

Give one table with these columns: Name, Phone, Website, Email (only when the source returns emails), Address, Rating (with review count), Notes. Sort by rating and review count unless the user asked otherwise. Keep it to the first 25 rows in chat and say how many more are in the CSV. Then one line: "Apify cost: about USD X for N rows in M runs, billed to your Apify account." When fewer rows came back than requested, say why in one sentence using the summary record.

Show the data as the source gives it. Do not paste raw JSON. Do not show management or owner names from BBB rows unless the user asks who runs the business. Mark an email as "unverified" when `email_verification_status` is `unknown` or `unverified`.

## Responsible use

These are business listings published for customers to find the business. Use the lists for business-to-business outreach, market research and directories. Remind the user, once per conversation and in one sentence, that outreach must follow the laws that apply to them (for example CAN-SPAM in the US, CASL in Canada, PECR and GDPR in the UK and EU) and the recipient's opt-out requests. Do not build lists of private individuals; if the user asks for home addresses or personal emails of people rather than businesses, decline that part and offer the business listing instead.

## When things go wrong

- The connector is not connected or the tool call returns an authorization error: tell the user to open the plugin's Connectors tab and connect Apify (a free account works), then retry.
- The run reports `charge_limit`, `chargeLimitReached`, `REJECTED` or "maximum cost per run": the user's Apify spending limit per run stopped it; report how many rows were delivered and suggest raising the limit in Apify Console or splitting the request.
- The record says `BLOCKED`, `blocked` or every request failed: Google or the directory refused the lookups; nothing was charged for missing rows. Offer to retry once, or switch source (Yellow Pages for US and Canada, Google Maps for everything else).
- The contractor Actor refuses to start with a message about the maximum charge: pass `callOptions.maxTotalChargeUsd: 1` or higher.
- Google Maps rows have `null` phone or website: Google did not show them. Offer Yellow Pages as a second source to fill the gaps.
