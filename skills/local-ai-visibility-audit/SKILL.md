---
name: local-ai-visibility-audit
description: "Reveal which local businesses get recommended for 'best X in city' across Google's AI and the community threads it cites — and which strong, well-reviewed ones stay invisible. Use when the user asks something like \"Audit who Google's AI recommends for best wedding photographer in Nashville, Tennessee, then list strong businesses it ignores and how to reach them\". Runs on the API Direct MCP tools (Google Search, Reddit, web forums and Google Maps data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Local & Places"
  platforms: "Google Search, Reddit, web forums, Google Maps"
  mcp-server: "https://apidirect.io/mcp"
---

# Local AI Visibility Audit

Google's AI Overviews and AI Mode now decide who gets recommended for 'best X in city,' and they pull heavily from Reddit and forum threads. Optionally union those community sources with the AI-cited set, diff the whole recommendation landscape against the real high-rated businesses on Maps, and you surface strong companies that are invisible to AI — a clean sales wedge. Which community platforms to run, the country, and how deep to pull Maps results are all tunable.

**Who it's for:** Local SEO and AI-visibility agencies prospecting for work.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `google_ai_mode`, `search_web`, `search_reddit`, `search_forums`, `search_places`, `place_details`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `category` | yes | Business category to audit. | wedding photographer |
| `city` | yes | City and region to audit. | Nashville, Tennessee |
| `platforms` | no | Comma-separated list of platforms to run — options: web, reddit, forums, places. Omit to run all of them; name specific platforms to limit the run. | reddit, forums |
| `country` | no | Two-letter country code to localize all Google results (AI Mode, AI Overview, Maps, place details). Omit to let Google infer region. | us |
| `result_pages` | no | How many pages of Maps results to pull for the ground-truth set (default 10). | 10 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`google_ai_mode`**

   ```
   google_ai_mode(prompt="best {category} in {city}", country={country})
   ```

   Capture reply_parts and reference_links to record exactly which businesses Google's AI Mode names and cites. country is optional — omit it to let Google infer the region.

2. **`search_web`**

   ```
   search_web(query="best {category} {city}", include_ai_overview=true, location="{city}", country={country})
   ```

   Pull ai_overview.text_parts and reference_links. location="{city}" simulates a searcher inside the metro so the AI Overview reflects true local results. Union with step 1 to build the AI-recommended set.

3. **`search_reddit`**

   ```
   search_reddit(query="best {category} {city}", sort_by=top)
   ```

   Only run if {platforms} includes reddit: surface the businesses the community recommends in 'best {category} {city}' threads — a major source AI Overviews cite. Add these as a second recommendation set.

4. **`search_forums`**

   ```
   search_forums(query="best {category} {city}", time=year)
   ```

   Only run if {platforms} includes forums: capture business names recommended in independent forum threads over the past year to widen the community-recommendation set beyond Reddit.

5. **`search_places`**

   ```
   search_places(query="{category} {city}", pages={result_pages}, country={country})
   ```

   List every OPERATIONAL business with strong rating and review_count — the ground-truth set that deserves to rank. pages={result_pages} (default 10) controls how deep to pull.

6. **`place_details`**

   ```
   place_details(place_id=<place_id>, country={country})
   ```

   For high-rated businesses absent from BOTH the AI set and any community set, pull contact info (phone, website, email if present) to pitch an AI-visibility fix.

## Deliver

A gap report of strong local {category} businesses absent from Google's AI (and, if selected, the Reddit/forum threads it cites), each with contacts for an AI-visibility pitch.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Audit who Google's AI recommends for best wedding photographer in Nashville, Tennessee, then list strong businesses it ignores and how to reach them
