# Actor contracts

Prices are Apify Store pay-per-event prices on the FREE tier as of 2026-11-04; paid Apify plans are 2.5 to 20 percent lower. A record is charged when it is written to the dataset (exceptions are noted per Actor). Every run also charges one start event per started GB of the Actor's default memory. Schemas were read from the Actors' live default builds on 2026-11-04 and the output shapes were checked against live runs on the same day (see the note at the end).

All Actors belong to Apify user `muhammadafzal`. On the **business-leads** MCP server the tool name is the Actor ID with `/` replaced by `--`, for example `muhammadafzal--google-maps-scraper`. Some apps add a server prefix, turn hyphens into underscores, or shorten a long name and add a suffix; match on the start of the Actor slug. Each tool returns the run (`runId`, `status`, `storages.datasets.default.id`, `storages.keyValueStores.default.id`), not the rows. If a tool response already includes dataset items, use them; otherwise fetch them with `get-dataset-items`.

Run options (`memory`, `timeout`, `maxTotalChargeUsd`) go in the tool's `callOptions`, never inside `input`. Send only the input fields listed below. Every Actor handles its own network routing internally, so nothing else is needed.

## Shared rules

- `maxResults` is the spending control. Always set it to the number the user asked for, never leave the default.
- Google Maps lists about 120 places for one search. Asking for more from one search rarely adds rows; split the area into several cities, ZIP codes or neighbourhoods instead.
- A search that finds nothing charges only the start event. Blocked requests never charge a record.
- Always compute a cost from the prices in this file. A run response can include usage or cost fields (for example `usageTotalUsd`); those are Apify's internal platform usage and are far lower than the billed price, so never report them as the cost.
- The key-value record named per Actor explains the run: how many rows were delivered, why it stopped, and warnings. Read it whenever fewer rows than requested came back.

---

## Google Maps, any niche: `google-maps-scraper`

Price: USD 0.004 per place, start USD 0.006251 (256 MB, one event). Up to 200 places per query, several queries per run. **Free Apify plans are capped at 20 places per run**; paid plans use the full limit. Typical duration: 11 seconds for 20 places.

Input:
```json
{
  "searchQueries": ["dentists in Boulder CO", "orthodontists in Boulder CO"],
  "maxResults": 50,
  "countryCode": "us",
  "languageCode": "en",
  "includePhotos": false,
  "minRating": 0
}
```
- `searchQueries`: each entry is one search, written as "<business type> in <place>". `maxResults` applies per query.
- `countryCode`: `us`, `gb`, `ca`, `au`, `de`, `fr`, `es`, `it`, `jp`, `br`, `in`, `mx`. `languageCode`: `en`, `es`, `fr`, `de`, `it`, `pt`, `ja`, `ko`, `zh-CN`.
- `locationCoords`: optional "lat,lng" string; when set, keep only the business type in the query for exact targeting.
- `minRating`: 0 to 5; use 4 when the user wants only well-rated businesses.
- `includePhotos: false` keeps rows small.

Dataset row (camelCase): `placeId`, `title`, `address`, `city`, `postalCode`, `countryCode`, `phoneNumber`, `website`, `rating`, `ratingCount`, `latitude`, `longitude`, `type`, `types[]`, `priceLevel`, `openingHours` (object of day to hours, or null), `description`, `thumbnailUrl`, `cid`, `fid`, `googleUrl`, `isClaimed`, `searchQuery`, `scrapedAt`. No emails and no review text.

Key-value record `OUTPUT`: `status`, `stopReason`, `delivered`, `queries[]` (per query: `query`, `found`, `delivered`, `duplicates`, `filteredByRating`, `outcome`), `skippedQueries` (count), `settings` (includes `freePlanLimitApplied`), `finishedAt`. When `settings.freePlanLimitApplied` is true, tell the user the free plan capped the run at 20 places.

---

## Contractor trades: `contractor-lead-scraper`

Price: USD 0.05 per enriched lead, start USD 0.00005 (512 MB). One trade and one location per run, up to about 120 places; `maxResults` 1 to 500 (default 50). **A lead is charged only when it has a phone, website or email**; leads with only a name are saved free. Apify requires `callOptions.maxTotalChargeUsd` of at least 1.00 for this Actor; a lower limit is rejected before the run starts. About 14 seconds for 3 leads with website checks; allow 3 to 5 seconds per lead beyond that.

Input:
```json
{
  "businessType": "Roofer",
  "location": "Tampa, FL",
  "maxResults": 25,
  "enrichEmails": true,
  "verifyEmails": true,
  "enrichSocials": true,
  "outputFormat": "full"
}
```
- `businessType`: `HVAC Company`, `Plumber`, `Roofer`, `Electrician`, `General Contractor`, `Landscaper`, `Painter`, `Pest Control`, or `Custom`. With `Custom`, also send `searchQuery`, a full Google Maps query such as "concrete contractors in Tampa, FL" or "AC repair service in Phoenix, AZ". HVAC sub-niches (AC repair, heating, furnace, duct cleaning, commercial HVAC) go through `Custom` with such a query.
- `location`: "City, ST" works best; ZIP codes work. Any text Google Maps understands is accepted, but runs on record use US cities.
- `enrichEmails` opens the business website (home page plus up to two contact or about pages). `verifyEmails` asks the domain's mail server whether the mailbox exists without sending mail. Turn both off for a faster phone-only list; the price is the same.
- `outputFormat`: `full` for the fields below; `hubspot` or `salesforce` return columns named for those CRMs' contact imports.

Dataset row (snake_case): `business_name`, `business_type`, `category`, `phone`, `email`, `email_verified`, `email_verification_status` (`verified` | `unverified` | `unknown` | null), `email_verification_note`, `all_emails[]` of `{address, verified, status, verificationNote}`, `website`, `address`, `rating`, `review_count`, `hours` (one string, "Monday: 7 AM–7 PM; Tuesday: ..."), `facebook`, `instagram`, `linkedin`, `twitter`, `google_maps_url`, `place_id`, `latitude`, `longitude`, `scraped_at`, `source_url`. A `twitter` value of `https://twitter.com/intent/tweet?text=` is a share button, not a profile; show it as empty. An `email` on a consumer domain (gmail.com and similar) was published on the business website; `email_verification_status` is often `unknown` for such domains because their mail servers do not answer mailbox checks.

Key-value record `OUTPUT`: `status` (`SUCCEEDED` | `BLOCKED` | `FAILED`), `searchQuery`, `leadsSaved`, `leadsCharged`, `leadsWithoutContactNotCharged`, `stopReason` (`maxResults reached` | `all Google Maps results processed` | `no matching businesses`), `error`, `finishedAt`. The run ends FAILED when Google blocked the search; nothing is charged then.

---

## Dentists, worldwide: `worldwide-dentist-scraper`

Price: USD 0.05 per listing, start USD 0.005 (256 MB). Several locations per run; each location is one Google Maps search of about 120 places; `maxResults` 1 to 5000 applies to the whole run. **Charged whether or not an email was found.** Duplicates across searches are delivered once. Free and paid plans have the same limits. Allow 2 to 4 seconds per listing with email extraction on.

Input:
```json
{
  "searchQuery": "dentist",
  "locations": ["Denver, CO", "Boulder, CO"],
  "maxResults": 40,
  "language": "en",
  "includeEmails": true
}
```
- `searchQuery`: `dentist`, `orthodontist`, `pediatric dentist`, `cosmetic dentist`, `dental implants` and similar.
- Use `locations` for one or more places; `location` (single string) is ignored when `locations` is set. Works in any country; `language` picks the text language (`en`, `es`, `fr`, `de`, `pt`, `it`, `ja`, `ko`, `zh-CN`, `ar`, `hi`, `ru`, `nl`, `pl`, `tr`).
- `startUrls`: optional Google Maps search URLs (not single-place URLs).
- `includeEmails: false` is faster but costs the same.

Dataset row (snake_case): `business_name`, `category`, `specialties`, `address`, `city`, `state`, `postal_code`, `country`, `phone`, `email`, `website`, `google_maps_url`, `rating`, `reviews_count`, `latitude`, `longitude`, `opening_hours`, `accepts_new_patients`, `scraped_at`, `source_url`.

Key-value record `OUTPUT`: `status`, `delivered`, `requestedMaxResults`, `searches`, `failedSearches`, `stopReason`, `warnings[]`, `emails` (stats, or null when extraction was off), `finishedAt`.

---

## Restaurants: `google-maps-restaurant-scraper`

Price: USD 0.004001 per restaurant, start USD 0.006251 (256 MB). One search per run, `maxResults` 1 to 120. Default run timeout is 180 seconds; pass `callOptions.timeout: 600` for more than 40 restaurants with reviews. Blocked or empty searches charge nothing.

Input:
```json
{
  "searchQuery": "pizza",
  "location": "Boulder, CO",
  "maxResults": 40,
  "scrapeReviews": true,
  "maxReviewsPerPlace": 5,
  "scrapePhotos": false,
  "language": "en"
}
```
- `searchQuery`: cuisine, type or name ("sushi", "vegan restaurants", "coffee shops"). `location`: city, neighbourhood or region; for a ZIP code include the city.
- Google shows at most 5 reviews per place to logged-out visitors; values above 5 return 5.

Dataset row (snake_case): `restaurant_name`, `place_id`, `address` (may start with the restaurant name), `city`, `state`, `postal_code`, `country`, `latitude`, `longitude`, `phone`, `website`, `menu_url`, `rating`, `review_count`, `price_level`, `cuisine_type`, `opening_hours` (object), `is_open`, `photos[]`, `description`, `google_maps_url`, `rating_distribution` (1 to 5 star counts), `reviews[]` of `{author, rating, date, text, ownerReply}`, `scraped_at`, `source_url`. No emails.

Key-value record `OUTPUT`: `status` (`DATA` | `EMPTY` | `BLOCKED` | `REJECTED` | `FAILED`), `message`, `query`, `requestedResults`, `effectiveLimit`, `resultsFound`, `resultsDelivered`, `reviewsDelivered`, `placesWithReviews`, `detailsFailed`, `warnings[]`, `finishedAt`. `REJECTED` means the user's maximum cost per run allows fewer than one result. `resultsFound` above `resultsDelivered` means more restaurants exist than were requested.

---

## Yellow Pages USA: `yellow-pages-us-scraper`

Price: USD 0.0112 per listing, start USD 0.018836 (256 MB). Prices changed on 2026-11-04. Several terms and locations per run (every term is crossed with every location); `maxResults` caps the whole run (default 20, 0 means up to 10,000, same on free plans); `maxPagesPerSearch` 1 to 50 with about 30 listings per page. Fast: a few seconds per page.

Input:
```json
{
  "searchTerms": ["plumber"],
  "locations": ["Boston, MA"],
  "maxResults": 60,
  "maxPagesPerSearch": 3,
  "sortBy": "default",
  "includeReviews": false
}
```
- `locations`: "City, ST" or 5-digit ZIP. `startUrls`: yellowpages.com search, city-category or profile URLs instead of terms.
- `sortBy`: `default`, `distance`, `name`, `rating`. `includeReviews: true` opens each profile (slower) and adds `reviews[]`.

Dataset row (camelCase): `name`, `phone`, `email`, `website`, `address` (object) and `addressFormatted`, `coordinates`, `categories[]`, `primaryCategory`, `services[]`, `paymentMethods[]`, `hours` (object such as `{"mo-su": "open 24 hours"}`), `rating`, `reviewCount`, `reviews`, `socialLinks` (object with facebook, instagram, linkedin, twitter, youtube), `logo`, `photos[]`, `claimed`, `bbbAccredited`, `yearsInBusiness`, `description`, `tagline`, `isAd`, `yellowPagesId`, `profileUrl`, `sourceUrl`, `searchTerm`, `searchLocation`, `scrapedAt`. Drop rows where `isAd` is true unless the user wants advertisers.

Key-value record `OUTPUT`: `ok`, `status` (`succeeded` | `empty` | `blocked` | `invalid_input`), `message`, `deliveredCount`, `chargedCount`, `stopReason` (`completed` | `result_limit` | `charge_limit` | `time_limit` | `aborted`), `pagesFetched`, `warnings[]`, `finishedAt`. Blocked or invalid runs end FAILED with no record charges.

---

## Yellow Pages Canada: `yellow-pages-ca-scraper`

Price: USD 0.005001 per listing, start USD 0.006251 per GB (2 GB default, so USD 0.0125). Several terms and locations per run; `maxResults` caps the run (default 20, 0 means no cap up to 10,000 page requests); `maxPagesPerSearch` 1 to 100, about 50 listings per page. Default timeout 300 seconds; pass `callOptions.timeout: 900` for more than 200 listings or when `extractDetails` is on.

Input:
```json
{
  "searchTerms": ["plumber"],
  "locations": ["Toronto, ON"],
  "maxResults": 60,
  "maxPagesPerSearch": 3,
  "extractDetails": false,
  "sortBy": "default"
}
```
- `locations`: "City, Province" such as "Vancouver, BC". When empty the Actor searches Toronto, ON, so always set it.
- `extractDetails: true` opens each listing page and adds coordinates, hours, services, payment methods, social links and logo; `includeReviews` needs it too.

Dataset row (camelCase): `businessName`, `phone`, `email`, `website`, `address` (`{street, city, province, postalCode}`), `addressFormatted`, `coordinates`, `categories[]`, `primaryCategory`, `rating`, `reviewCount`, `hours`, `socialLinks`, `logo`, `photos[]`, `services[]`, `paymentMethods[]`, `yearsInBusiness`, `claimed`, `profileUrl`, `yellowPagesId`, `searchTerm`, `searchLocation`, `sourceUrl`, `scrapedAt`, `reviews[]`.

Key-value record: none. Use the dataset count and the run's `statusMessage` ("Saved N Canadian business listings ...; stop reason: ...; charged N business-scraped events").

---

## BBB profiles: `bbb-business-scraper`

Price: USD 0.025 per profile, start USD 0.005 per GB (2 GB default, so USD 0.01). `maxResults` 1 to 1,000 (default 10); `maxPagesPerSearch` 1 to 25. About 35 seconds for a 3-profile keyword search, 20 to 30 seconds per profile URL. Pass `callOptions.timeout: 900` for more than 15 profiles. One row per business even when BBB lists several locations.

Input, keyword search:
```json
{
  "searchQuery": "plumber",
  "location": "Phoenix, AZ",
  "country": "US",
  "maxResults": 10,
  "maxPagesPerSearch": 2,
  "includeReviews": false,
  "includeComplaints": false
}
```
Input, known businesses: `startUrls` with bbb.org `/us/.../profile/...` or `/ca/.../profile/...` URLs, or a `searchQuery` set to the company name, phone number, website or email with `location` left blank. `country`: `US` or `CA`.

Dataset row (snake_case): `business_name`, `profile_url`, `country`, `address`, `phone`, `email`, `website`, `categories[]`, `description`, `bbb_rating` (letter grade such as `A+`, or null when BBB shows none), `is_accredited`, `years_in_business`, `bbb_file_opened`, `business_started`, `entity_type`, `local_bbb`, `review_rating`, `review_count`, `complaint_count`, `reviews[]`, `complaints[]`, `management_contacts[]` (strings such as "Mr. J. Smith, Owner"; show them only when the user asks who runs the business), `additional_websites[]`, `payment_methods[]`, `hours`, `scraped_at`, `source_url`.

Key-value record `OUTPUT`: `status` (`SUCCEEDED` | `BLOCKED`), `stop_reason`, `records_emitted`, `records_charged`, `search_pages_visited`, `profile_urls_discovered`, `failed_requests`, `completed_at`. If every request fails the run ends FAILED with no charges.

---

## Email validation: `email-address-validator`

Price: USD 0.001501 per address checked (about USD 1.50 per 1,000), start USD 0.006251 (one event). Up to 10,000 unique addresses per run (`maxEmails`, default 1,000). Every processed address is charged, including invalid ones. A 3-address run took about 2 seconds; allow about 0.5 seconds per address. It never sends mail and never logs in anywhere.

Input:
```json
{
  "emails": ["info@example.com", "sales@example.org"],
  "checkDns": true,
  "checkMx": true,
  "allowDisposable": false,
  "allowRoleAccounts": true,
  "outputInvalidOnly": false,
  "maxEmails": 1000
}
```
- `emailText`: optional pasted text; every address found in it is added. Duplicates are removed case-insensitively.
- `outputInvalidOnly: true` writes only invalid rows (all are still charged).

Dataset row (camelCase): `email`, `normalizedEmail`, `isValid`, `status` (`valid` | `risky` | `invalid`), `confidence` (0 to 1), `syntaxValid`, `domain`, `domainAscii`, `localPart`, `dnsChecked`, `domainHasDns`, `mxChecked`, `mxRecords[]`, `hasMx`, `isDisposable`, `isRoleAccount`, `isFreeProvider`, `reasons[]`, `checkedAt`.

Status meaning: `invalid` fails a hard check (syntax, no DNS, no MX, disposable domain). `risky` is usable but lower quality: a consumer free-provider domain (gmail.com, outlook.com, yahoo.com always come back `risky` with confidence 0.65), a role account when `allowRoleAccounts` is false, or a run with DNS or MX checks off. `valid` passed every enabled check. For business lists, keep `valid` and `risky` rows and flag `risky`; drop `invalid`.

Key-value record `OUTPUT`: `summary` (`total`, `valid`, `risky`, `invalid`, `chargedEvents`, `estimatedPpeCostUsd`, `stopReason`, `notStored[]`, `checkedAt`), `datasetUrl`.

---

## Cost formula

cost = runs × start fee + charged rows × per-row price.

Examples on the FREE tier: 20 Google Maps places ≈ USD 0.086; 100 Google Maps places in 2 queries ≈ USD 0.41; 25 contractor or HVAC leads ≈ USD 1.25; 30 dentists ≈ USD 1.505; 40 restaurants with reviews ≈ USD 0.166; 60 Yellow Pages US listings ≈ USD 0.69; 60 Yellow Pages Canada listings ≈ USD 0.31; 10 BBB profiles ≈ USD 0.26; validating 200 emails ≈ USD 0.31.

Quote costs as "about": contractor and HVAC leads without any contact detail are free, empty searches charge only the start fee, and the user's Apify plan tier may be discounted.

## Verification note

Live runs on 2026-11-04 (2 to 4 rows each, minimal input) confirmed the dataset fields and key-value records above for all eight Actors: Google Maps (3 dentists in Boulder, `OUTPUT` as listed), contractor (3 roofers in Tampa, emails and `OUTPUT` as listed), dentist (3 in Boulder, `OUTPUT` as listed), restaurant (3 pizza places in Boulder with 6 reviews, `OUTPUT.status` was `DATA`), Yellow Pages USA (3 plumbers in Boston, `OUTPUT` as listed), Yellow Pages Canada (3 plumbers in Toronto with `extractDetails`, no record), BBB (2 plumbers in Phoenix, `OUTPUT` as listed) and the email validator (4 addresses, `OUTPUT` as listed).

Two sibling Actors were tested the same day and left out of this plugin until they are repaired: `hvac-lead-scraper` (rows came back without phone, website, address or email for most places) and `yellow-pages-au-scraper` (a plumber search in Sydney returned unrelated nationwide listings). Route HVAC requests to the contractor Actor and Australian requests to Google Maps with `countryCode: "au"`.
