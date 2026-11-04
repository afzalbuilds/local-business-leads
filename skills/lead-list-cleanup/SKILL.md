---
name: lead-list-cleanup
description: Clean, deduplicate, email-validate and export a lead list. Use when the user says "clean this list", "dedupe", "remove duplicates", "validate these emails", "verify emails", "which emails are bad", "export to CSV", "prepare for HubSpot", "Salesforce import", "merge these lists", "combine the Google Maps and Yellow Pages results", or hands over a CSV, spreadsheet or pasted list of businesses or email addresses.
---

# Lead list cleanup

Builds on the `find-business-leads` skill: read its `references/actor-contracts.md` (email validator section) and `references/lead-schema.md` first. This skill takes lists you built in this conversation or lists the user supplies, and returns one clean, exportable list.

## Steps

1. **Load the lists.** From results already in the conversation, or from a pasted list, CSV or spreadsheet. Map each list's columns onto the normalized schema in `lead-schema.md` (name, phone, email, website, address, city, postal code, country, rating, review count, source). Keep every extra column the user had so it survives the export.

2. **Deduplicate** with the rules in `lead-schema.md` (place ID, phone digits, website host, name plus city). Merge duplicates into the fullest row and record the combined `source`. Report how many were merged.

3. **Validate emails** when the list has an email column and the user wants outreach or asked for validation. Collect the unique addresses, estimate the cost (USD 0.0015 each, about USD 1.50 per 1,000, plus USD 0.006 per run) and, above USD 0.50, confirm first. Run `muhammadafzal--email-address-validator` with `emails` (up to 10,000 per run; split larger lists), wait with `get-actor-run`, read the dataset and the `OUTPUT` record. Join results back by `normalizedEmail`. Set `emailStatus` to `valid`, `risky` or `invalid` and add the first reason to Notes. Keep `risky` rows (consumer-domain and role addresses are normal for small businesses) and flag them; move `invalid` rows to a separate "needs attention" group rather than deleting them.

4. **Flag quality issues** without running anything: missing phone and website, Yellow Pages `isAd` rows, rows whose website host does not match the email domain (a possible stale email), and businesses with a rating under 3 if the user cares about reputation.

5. **Export.** Write the CSV with the column order in `lead-schema.md`. In Claude Code or Cowork, write it next to the input file or in the working folder as `<name>-clean-<YYYY-MM-DD>.csv`; in chat, give a downloadable file, or a code block if that is not possible. For HubSpot or Salesforce, rename the columns as described in `lead-schema.md` and say which columns the CRM will still ask to map.

6. **Report** in this order: a summary line (rows in, duplicates merged, rows out, emails valid/risky/invalid, Apify cost), a short "needs attention" table (invalid emails, missing contact details), then the file.

## Rules

- Never fabricate a validation result. Addresses that were not checked stay "not checked".
- Validation never sends an email and never contacts the person; say so if the user asks.
- Keep personal data minimal: do not add columns the sources did not have, and do not look up individuals.
