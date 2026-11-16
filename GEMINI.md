# Local Business Leads: Google Maps, Yellow Pages & BBB

This extension builds local business lead lists from Google Maps, Yellow Pages, BBB and niche directories with Imzlab Technologies' Actors on Apify, through the `business-leads` MCP server. The full instructions live in the extension's skills. Activate the matching skill before you call any of the server's tools:

- `find-business-leads`: any niche and place, routed to the right source.
- `lead-list-cleanup`: merging, deduplicating, email validation and CSV export.
- `business-trust-check`: BBB rating, accreditation and complaints for a shortlist.
- `leads-cost-estimate`: the Apify cost of a list before running it.

Every tool call on `business-leads` starts a paid Actor run on the user's own Apify account. Follow the skill's cost rules: estimate from the price table in its references, ask before large batches, and never quote a cost from a run's usage figures. Wait for each run to finish, account for every requested item, and never invent a result.

If the listing tools are missing, the server is not signed in yet: ask the user to run `/mcp auth business-leads` and approve Apify's sign-in page in the browser.

Commands: `/bbb-check`, `/leads-report`, `/leads`. Subagent for unattended runs that leave a report on disk: `lead-list-builder`.
