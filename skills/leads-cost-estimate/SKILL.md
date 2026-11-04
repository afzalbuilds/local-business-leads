---
name: leads-cost-estimate
description: Estimate what a lead list will cost on Apify before running anything. Use when the user asks "how much will it cost", "price per lead", "is it free", "how is this billed", "cheapest way to get", "cost of 500 leads", or wants to compare Google Maps, Yellow Pages, dentist, contractor, restaurant, BBB or email validation prices, or asks about Apify credits, plans or charges for this plugin.
---

# Lead list cost estimate

Prices are in `../find-business-leads/references/actor-contracts.md` (FREE-tier Apify Store prices, verified 2026-11-04). Use that file as the single source; do not quote other numbers.

## How billing works (two or three sentences when asked)

The plugin is free. Each listing Actor is a pay-per-event Actor on Apify Store, billed to the user's own Apify account: one event per row written to the dataset, plus a small start event per run. Empty searches and blocked requests charge no rows, and contractor leads without any contact detail are free. Free Apify accounts include monthly usage credit; paid plans get tiered discounts of up to 20 percent per row.

## Estimate

1. Route the request with `../find-business-leads/references/niche-routing.md` and count rows per Actor.
2. Runs per Actor: one per search for Google Maps (several queries fit in one run), one per trade and city for the contractor Actor, one per run for dentists (many cities fit), one per search for restaurants, one per run for Yellow Pages (several terms and cities fit), one per batch of up to 10,000 addresses for the validator.
3. Cost per Actor = runs × start fee + rows × per-row price.
4. Present a small table (Actor, rows, runs, per row, subtotal) and the total as "about USD X". Say that the real charge is lower when the source lists fewer businesses than requested.
5. When the user asks for the cheapest option: Google Maps at USD 0.004 per place (no emails) versus the dentist or contractor Actor at USD 0.05 per lead (with emails and checks); Yellow Pages USA at USD 0.0112 and Canada at USD 0.005 per listing with occasional emails. For 1,000 Google Maps places in 10 cities that is about USD 4.06; for 1,000 contractor leads with emails about USD 50.
6. Add the rough time: seconds for Google Maps and Yellow Pages, 3 to 10 seconds per lead for contractor and dentist runs with email extraction, 20 to 30 seconds per BBB profile.

Do not run any Actor from this skill. If the user then says to go ahead, hand over to `find-business-leads`, `lead-list-cleanup` or `business-trust-check`.
