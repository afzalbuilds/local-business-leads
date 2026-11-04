# Niche and country routing

Pick the Actor from the user's niche and country with the first matching row. "Actor" names the connector tool to call.

| # | User asks for | Country | Actor | Why |
|---|---|---|---|---|
| 1 | dentists, dental clinics, orthodontists, pediatric or cosmetic dentists, dental implants | any | `worldwide-dentist-scraper` | practice emails, specialties, new-patient flag; several cities per run |
| 2 | HVAC, air conditioning, AC repair, heating, furnace, duct cleaning, refrigeration | US and Canada (best), others with nulls | `contractor-lead-scraper` with `businessType: "HVAC Company"`, or `Custom` plus a query for a sub-niche | website emails with mailbox check, social links; only charged for leads with a contact detail |
| 3 | plumbers, roofers, electricians, general contractors, landscapers, painters, pest control, or any other trade (`Custom`) | US and Canada (best) | `contractor-lead-scraper` | same enrichment; needs `callOptions.maxTotalChargeUsd` of at least 1.00 |
| 4 | restaurants, cafes, coffee shops, bars, bakeries, any cuisine, or "with reviews" | any | `google-maps-restaurant-scraper` | cheapest source with review text, rating breakdown, price level, menu URL |
| 5 | any other business type, or the user wants the cheapest list, or more than 12 countries | `us`, `gb`, `ca`, `au`, `de`, `fr`, `es`, `it`, `jp`, `br`, `in`, `mx` | `google-maps-scraper` | USD 0.004 per place, several queries per run, no emails |
| 6 | a second source, "also check Yellow Pages", listings Google misses, or the user wants category, services, years in business and payment methods | US | `yellow-pages-us-scraper` | directory data with some emails, `bbbAccredited` flag |
| 7 | same as 6 | Canada | `yellow-pages-ca-scraper` | set `extractDetails: true` for hours, services and social links |
| 8 | BBB rating, accreditation, complaints, "is this company legit", trust check on a shortlist | US, Canada | `bbb-business-scraper` | by category plus city, by company name, or by profile URL |
| 9 | validate, verify, clean or dedupe emails before outreach | any | `email-address-validator` | USD 0.0015 per address; never sends mail |

## Rules

- A niche in rows 1 to 4 that the user wants **without emails and at the lowest price** can also go to `google-maps-scraper`: say the trade-off (USD 0.004 per place without emails, USD 0.05 per lead with emails) and let the user choose when the list is over 20 rows.
- Rows 2 and 3 use the same Actor, which takes one trade and one city per run. For several cities, start one run per city (up to 3 at a time) and merge.
- Row 5 on a free Apify plan returns at most 20 places per run. If the user needs more and the summary record says `freePlanLimitApplied`, tell them.
- Australia: use `google-maps-scraper` with `countryCode: "au"` (dentists can use the dentist Actor).
- Countries outside the Google Maps `countryCode` list: use the dentist Actor for dentists, otherwise `google-maps-scraper` with the closest `countryCode` and the full place name in the query, and say that results may be thinner.
- When the user gives a country-wide request ("all plumbers in Texas"), do not run one huge search. Propose the 5 to 10 biggest cities, estimate the cost, and run city by city.
- Yellow Pages rows with `isAd: true` are advertisers shown out of order; keep them but mark them, or drop them if the user wants organic listings only.

## Country and location formats

| Source | Location format | Examples |
|---|---|---|
| Google Maps (any niche) | place inside the query | "plumbers in Chicago IL", "cafes near Shibuya Tokyo" |
| Contractor (all trades including HVAC) | `location`: "City, ST" or ZIP | "Dallas, TX", "85001" |
| Dentists | `locations[]`: city, "City, ST", ZIP with city, or country | "Miami, FL", "London, UK", "Tokyo, Japan" |
| Restaurants | `location`: city, neighbourhood or region | "Austin, TX", "Manhattan, NY", "London, UK" |
| Yellow Pages USA | `locations[]`: "City, ST" or 5-digit ZIP | "Boston, MA", "90210" |
| Yellow Pages Canada | `locations[]`: "City, Province" | "Toronto, ON", "Vancouver, BC" |
| BBB | `location`: city, state, ZIP or postal code, plus `country` `US` or `CA` | "Phoenix, AZ", "Calgary, AB" |
