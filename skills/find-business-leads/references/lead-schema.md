# Normalized lead schema

Every source is mapped to these columns so lists from different Actors can be merged, deduplicated and exported the same way. Keep the source's original row available for the user on request, but present and export the normalized columns.

| Column | Type | Google Maps | Contractor / HVAC | Dentist | Restaurant | Yellow Pages US / CA | BBB |
|---|---|---|---|---|---|---|---|
| `name` | string | `title` | `business_name` | `business_name` | `restaurant_name` | `name` / `businessName` | `business_name` |
| `category` | string | `type` | `category` or `business_type` | `category` | `cuisine_type` | `primaryCategory` | first of `categories` |
| `phone` | string | `phoneNumber` | `phone` | `phone` | `phone` | `phone` | `phone` |
| `email` | string | null | `email` | `email` | null | `email` | `email` |
| `emailStatus` | string | null | `email_verification_status` | null | null | null | null |
| `website` | string | `website` | `website` | `website` | `website` | `website` | `website` |
| `address` | string | `address` | `address` | `address` + city, state, postal | `address` + city, state, postal | `addressFormatted` | `address` |
| `city` | string | `city` | parsed from `address` | `city` | `city` | `address.city` | parsed |
| `postalCode` | string | `postalCode` | parsed | `postal_code` | `postal_code` | `address.postalCode` | parsed |
| `country` | string | `countryCode` | US unless stated | `country` | `country` | US / CA | `country` |
| `rating` | number | `rating` | `rating` | `rating` | `rating` | `rating` | `review_rating` |
| `reviewCount` | integer | `ratingCount` | `review_count` | `reviews_count` | `review_count` | `reviewCount` | `review_count` |
| `hours` | string | `openingHours` flattened | `hours` | `opening_hours` | `opening_hours` flattened | `hours` flattened | `hours` flattened |
| `socials` | string | null | `facebook`, `instagram`, `linkedin`, `twitter` joined | null | null | `socialLinks` joined | null |
| `trust` | string | `isClaimed` | `email_verified` | `accepts_new_patients` | `is_open` | `claimed`, `bbbAccredited`, `yearsInBusiness` | `bbb_rating`, `is_accredited`, `complaint_count` |
| `sourceUrl` | string | `googleUrl` | `google_maps_url` | `google_maps_url` | `google_maps_url` | `profileUrl` | `profile_url` |
| `source` | string | google-maps | contractor / hvac | dentist | restaurant | yellow-pages-us / ca | bbb |
| `placeId` | string | `placeId` | `place_id` | null | `place_id` | `yellowPagesId` | null |

## Deduplication

Two rows are the same business when any of these match after normalization:
1. same `placeId` (Google sources only);
2. same phone after stripping everything except digits (compare the last 10 digits);
3. same website host (strip `http(s)://`, `www.`, trailing slash and paths);
4. same name after lowercasing and removing punctuation, within the same city.

When merging duplicates, keep the row with the most filled columns, and fill gaps from the others (a Yellow Pages email can complete a Google Maps row). Record which sources contributed in `source` as "google-maps+yellow-pages-us".

## Export

CSV columns in this order: `name`, `category`, `phone`, `email`, `emailStatus`, `website`, `address`, `city`, `postalCode`, `country`, `rating`, `reviewCount`, `hours`, `socials`, `trust`, `sourceUrl`, `source`. When you can write files (a coding agent or a desktop workspace), write `<niche>-<city>-leads-<YYYY-MM-DD>.csv` in the working folder; in a chat-only app give a downloadable file, or a code block if that is not possible. For HubSpot or Salesforce imports, rename columns to `Company name`, `Phone number`, `Email`, `Website URL`, `Street address`, `City`, `Postal code`, `Country/Region`, or ask the contractor and HVAC Actors for `outputFormat: "hubspot"` or `"salesforce"` directly.
