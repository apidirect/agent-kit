---
name: reviewer-footprint-tracker
description: "Trail one Google reviewer across nearby venues — and optionally the wider web and X — to infer their home turf and daily routine. Use when the user asks something like \"Trace the Google reviewer Marco D. who reviewed Blue Bottle Coffee in Oakland, California and map where else they go\". Runs on the API Direct MCP tools (Google Maps, Google Search and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "OSINT & Due Diligence"
  platforms: "Google Maps, Google Search, X"
  mcp-server: "https://apidirect.io/mcp"
---

# Reviewer Footprint Tracker

A single Google review leaks more than people realize. By anchoring on one venue, capturing the reviewer's author id, and re-scanning nearby businesses for the same author, you reconstruct a geographic footprint from public reviews alone. Optional platform toggles extend the trail to other review sites via web search and to location-tagged posts on X, while scan_depth, review_sort, and country let the operator dial reach, sort order, and localization.

**Who it's for:** Skip tracers, investigators, and physical-security teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_reviews`, `search_web`, `search_twitter`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `business_name` | yes | A venue the target is known to have reviewed. | Blue Bottle Coffee |
| `city` | yes | The city/area the seed venue is in. | Oakland, California |
| `reviewer_name` | yes | The target's Google reviewer display name. | Marco D. |
| `platforms` | no | Comma-separated list of platforms to run — options: places, web, twitter. Omit to run all of them; name specific platforms to limit the run. | web, twitter |
| `scan_depth` | no | How thoroughly to sweep — sets the number of result pages for the nearby-venue enumeration and the per-venue review re-scan. Higher = wider footprint, slower run. Default: `10`. | 10 |
| `review_sort` | no | Sort order for the per-venue review re-scan: highest_ranking, lowest_ranking, most_relevant, or newest. Defaults to newest for recency. | newest |
| `country` | no | Country to localize Google Places and web results for non-US targets. | United States |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{business_name} {city}", pages=2, country={country})
   ```

   Resolve the seed venue's place_id and record its lat/lng as the anchor point. Pass {country} for non-US targets.

2. **`place_reviews`**

   ```
   place_reviews(place_id=<seed_place_id>, sort_by=newest, pages=10, get_sentiment=true, country={country})
   ```

   Find {reviewer_name}'s review and capture their author_id, author_reviews_link, and the tone/emotion of their writing.

3. **`search_places`**

   ```
   search_places(query="restaurants cafes bars", lat=<venue_lat>, lng=<venue_lng>, zoom=14, pages={scan_depth})
   ```

   Enumerate venues clustered tightly around the anchor point, the reviewer's likely home or work radius. {scan_depth} controls how wide the venue sweep goes (default 5).

4. **`place_reviews`**

   ```
   place_reviews(place_id=<nearby_place_id>, sort_by={review_sort}, pages={scan_depth}, country={country})
   ```

   Re-scan each nearby venue for the same author_id to assemble the reviewer's repeat-visit footprint and infer their routine. {review_sort} defaults to newest; {scan_depth} sets how deep to read each venue's reviews (default 10).

5. **`search_web`**

   ```
   search_web(query="\"{reviewer_name}\" reviews {city}", country={country}, pages=3)
   ```

   Only run if {platforms} includes web: Sweep other review sites (Yelp, TripAdvisor, OpenTable) and public pages for the same named reviewer to extend the footprint beyond Google and surface a fuller name or handle that sharpens cross-platform matching.

6. **`search_twitter`**

   ```
   search_twitter(query="{reviewer_name} {city}", sort_by=most_recent, pages=3, get_sentiment=true)
   ```

   Only run if {platforms} includes twitter: Surface the same person's location-tagged posts and check-ins on X to add venues they mention publicly and enrich the pattern-of-life map.

## Deliver

A geographic footprint of the target reviewer mapping the venues they frequent, with an inferred home/work radius and routine — built from matching Google author ids and, when enabled, corroborating mentions from other review sites and X check-ins.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Trace the Google reviewer Marco D. who reviewed Blue Bottle Coffee in Oakland, California and map where else they go.
