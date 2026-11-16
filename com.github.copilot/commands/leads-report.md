---
description: "Build a multi-city lead list with deduplication, email validation and a CSV file"
argument-hint: "<niche> in <city 1>, <city 2>, ... [count per city] [with emails]"
---

Build a multi-city lead list for: $ARGUMENTS

Use the `find-business-leads` skill to plan one run per city (up to 3 at a time) and show the run plan with the estimated Apify cost before starting. After the runs finish, use the `lead-list-cleanup` skill to merge the cities, remove duplicates, validate any emails (ask first when validation costs more than USD 0.50), and write `<niche>-leads-<YYYY-MM-DD>.csv` in the working folder (or as a downloadable file in chat). Finish with the summary line, the needs-attention table and the file path.
