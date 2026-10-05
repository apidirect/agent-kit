---
name: city-market-entry-scan
description: "Get a 360-degree read on one city before you expand — local chatter on Facebook and Reddit, trending hooks, competitor density, and hiring momentum — running only the lenses you choose. Use when the user asks something like \"Should I open a boutique fitness studio in Nashville, Tennessee, United States? Run a full market-entry scan\". Runs on the API Direct MCP tools (Facebook, X, Google Maps, LinkedIn and Reddit data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Cross-Platform Power Plays"
  platforms: "Facebook, X, Google Maps, LinkedIn, Reddit"
  mcp-server: "https://apidirect.io/mcp"
---

# City Market-Entry Scan

Expanding into a new metro is guesswork until you layer independent lenses: what locals post on Facebook, what they discuss on Reddit, what's trending on Twitter, how saturated the map is with competitors on Google Places, and whether rivals are staffing up there on LinkedIn. Each lens resolves a different unknown, and you decide which to run via the platforms input — from a fast two-lens spot-check to the full five-lens scan.

**Who it's for:** Operators and franchise or expansion teams evaluating a new city.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_facebook_locations`, `search_facebook_posts`, `twitter_trends`, `search_places`, `search_linkedin_jobs`, `search_reddit`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `niche` | yes | Your business category. | boutique fitness studio |
| `city` | yes | The target metro to evaluate. | Nashville, Tennessee, United States |
| `platforms` | no | Comma-separated list of platforms to run — options: facebook, twitter, places, linkedin, reddit. Omit to run all of them; name specific platforms to limit the run. | facebook, places, reddit |
| `woeid` | no | Twitter WOEID for the metro, used to pull its local trends. Required only when 'twitter' is included in platforms. | 2457170 |
| `job_window` | no | How recent LinkedIn job posts must be when gauging hiring momentum. One of 1h, 24h, 7d, 30d. Defaults to 30d. | 7d |
| `result_depth` | no | How many pages of Google Places results to map for competitor density. Higher = wider coverage. Defaults to 10. | 20 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_facebook_locations`**

   ```
   search_facebook_locations(query="{city}")
   ```

   Only run if {platforms} includes facebook: Resolve the city to a numeric location_id so the Facebook post search is geo-scoped to that metro.

2. **`search_facebook_posts`**

   ```
   search_facebook_posts(query="{niche}", location_id=<location_id>, get_sentiment=true)
   ```

   Only run if {platforms} includes facebook: Read how locals actually talk about this category in-market — demand, complaints, and gaps surfaced by real residents.

3. **`twitter_trends`**

   ```
   twitter_trends(woeid={woeid})
   ```

   Only run if {platforms} includes twitter: Pull what's trending in the metro right now to spot local events, sentiment, and hooks to localize the launch. Requires the woeid input.

4. **`search_places`**

   ```
   search_places(query="{niche} {city}", zoom=12, pages={result_depth})
   ```

   Only run if {platforms} includes places: Map competitor density and ratings to judge saturation and find under-served pockets of the city. Set result_depth higher for wider coverage (default 10).

5. **`search_linkedin_jobs`**

   ```
   search_linkedin_jobs(query="{niche} {city}", sort_by=most_recent, posted_ago={job_window})
   ```

   Only run if {platforms} includes linkedin: Gauge whether rivals are staffing up locally; active job posts signal a growing, competitive market and reveal pay benchmarks. Tighten job_window (default 30d) for only the freshest reqs.

6. **`search_reddit`**

   ```
   search_reddit(query="{niche} {city}", get_sentiment=true, sort_by=relevance)
   ```

   Only run if {platforms} includes reddit: Surface organic local demand from city and niche threads — recommendation requests, complaints about incumbents, and unmet needs — a second, unprompted read on whether residents actually want this category.

## Deliver

A multi-lens market-entry brief for the city — local demand sentiment (Facebook and Reddit), trending hooks, competitor saturation, and hiring momentum — limited to the lenses you selected and capped with a go / no-go recommendation.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Should I open a boutique fitness studio in Nashville, Tennessee, United States? Run a full market-entry scan.
