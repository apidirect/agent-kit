---
name: turnaround-acquisition-target-finder
description: "Find established local businesses with proven demand but failing management — the owner's direct line, plus cross-platform proof the reputation is broken. Use when the user asks something like \"Find underperforming HVAC contractor businesses around Phoenix, Arizona I could buy and turn around, with the owner's contact details and what's broken\". Runs on the API Direct MCP tools (Google Maps, web forums, Reddit and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Investing & Deal Sourcing"
  platforms: "Google Maps, web forums, Reddit, X"
  mcp-server: "https://apidirect.io/mcp"
---

# Turnaround Acquisition Target Finder

A low star rating sitting on a high review count is the classic turnaround signal: customers keep coming despite the experience, so demand is real and the problem is operational and fixable. This pulls those targets straight from Google Maps with owner contacts attached, then corroborates the reputation gap across forums, Reddit, and X — with user-tunable rating and review-count thresholds so you control exactly which targets qualify.

**Who it's for:** Search funds, SMB acquirers, and operator-investors.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_details`, `place_reviews`, `search_forums`, `search_reddit`, `search_twitter`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `vertical` | yes | The business type to hunt for. | HVAC contractor |
| `metro` | yes | The metro/geography to search within. | Phoenix, Arizona |
| `platforms` | no | Comma-separated list of platforms to run — options: places, forums, reddit, twitter. Omit to run all of them; name specific platforms to limit the run. | forums, reddit, twitter |
| `max_rating` | no | Upper Google rating bound that qualifies a target as a turnaround candidate (weak execution). Defaults to 3.7. | 3.7 |
| `min_reviews` | no | Minimum review_count required to prove real, durable demand. Defaults to 100. | 100 |
| `country` | no | ISO country code for non-US metros; passed to the Places and forums calls for accurate geo and language. Defaults to us. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{vertical} {metro}", pages=10, country={country})
   ```

   Keep only business_status==OPERATIONAL with rating<={max_rating} (default 3.7) AND review_count>={min_reviews} (default 100), i.e. proven demand but weak execution. country is optional for non-US metros.

2. **`place_details`**

   ```
   place_details(place_id=<place_id>, country={country})
   ```

   Extract owner_name, owner_link, and emails_and_contacts for a direct off-market approach.

3. **`place_reviews`**

   ```
   place_reviews(place_id=<place_id>, sort_by=lowest_ranking, get_sentiment=true, country={country})
   ```

   Mine the angriest, anger/frustration reviews for the specific fixable failures (wait times, staffing, billing).

4. **`search_forums`**

   ```
   search_forums(query="{vertical} {metro} reviews", time=year, get_sentiment=true, country={country})
   ```

   Only run if {platforms} includes forums: corroborate the reputation gap and surface complaints not visible on Maps.

5. **`search_reddit`**

   ```
   search_reddit(query="{vertical} {metro} bad experience OR avoid OR worst", sort_by=relevance, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: surface local-subreddit threads naming specific operators and recurring operational failures, then match named businesses back to the Maps shortlist.

6. **`search_twitter`**

   ```
   search_twitter(query="{vertical} {metro} terrible OR avoid OR never again", sort_by=relevance, get_sentiment=true)
   ```

   Only run if {platforms} includes twitter: catch public gripes and named-business callouts that never reach Google Maps, adding a third independent confirmation of a broken reputation.

## Deliver

A ranked shortlist of operational-turnaround acquisition targets, each with owner contact, a demand-vs-execution gap score, and a concrete fix thesis — backed by complaint evidence triangulated across Google Maps reviews, forums, Reddit, and X.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find underperforming HVAC contractor businesses around Phoenix, Arizona I could buy and turn around, with the owner's contact details and what's broken.
