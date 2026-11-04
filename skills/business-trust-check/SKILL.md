---
name: business-trust-check
description: Check whether a business is trustworthy using its BBB profile and public ratings. Use when the user says "is this company legit", "BBB rating", "accredited", "complaints against", "check these contractors", "vet this vendor", "which of these should I hire", "reputation check", "verify this business", or wants trust signals for a shortlist of companies or for leads already found.
---

# Business trust check

Builds on the `find-business-leads` skill: read its `references/actor-contracts.md` (BBB section) first. BBB covers the United States and Canada. For other countries, use the Google Maps rating, review count, claimed status and years in business from the listing instead, and say that BBB does not cover that country.

## Steps

1. **Collect the businesses.** From the user's shortlist (names, websites, phone numbers, bbb.org URLs) or from a lead list in this conversation. Up to 10 at a time; for more, confirm the cost first (USD 0.025 per profile plus USD 0.01 per run).

2. **Look them up** with `muhammadafzal--bbb-business-scraper`:
   - bbb.org profile URLs: send them in `startUrls`, `maxResults` equal to the count;
   - a company name, phone, website or email: one run per business with `searchQuery` set to that value, `location` blank, `country` `US` or `CA`, `maxResults: 1`;
   - a category plus city ("roofers in Phoenix"): `searchQuery`, `location`, `country`, `maxResults` as requested.
   Add `includeComplaints: true` when the user asks about complaints and `includeReviews: true` for review snippets. Pass `callOptions.timeout: 900` for more than 15 profiles. Wait with `get-actor-run`, then read the dataset and the `OUTPUT` record.

3. **Match** each requested business to a row by name, phone digits or website host. A business with no BBB profile is reported as "no BBB profile", which is common and not a warning on its own.

4. **Score** each business in plain words, from these signals: `bbb_rating` (A+ to F, or none), `is_accredited`, `years_in_business`, `complaint_count` against size, `review_rating` and `review_count`, and the Google Maps rating and review count if the lead list has them. Suggested bands: "strong" (A or A+, accredited or 5+ years, few complaints), "fine" (A/B rating or no profile but good Google rating and 20+ reviews), "check further" (C or lower, many complaints, under 2 years, or very few reviews), "avoid" (F, unanswered complaint pattern, revoked accreditation).

5. **Answer** with one table: Business, BBB rating, Accredited, Years in business, Complaints, Google rating (reviews), Verdict, Profile link. Then one line with the Apify cost. Offer to pull complaint details for any "check further" row.

## Rules

- Never invent a rating. "No BBB profile" is a valid result.
- Explain once that BBB ratings reflect complaint handling and transparency, not work quality, and that accreditation is paid membership.
- Do not show `management_contacts` names unless the user asks who runs the company.
