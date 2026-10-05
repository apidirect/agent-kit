---
name: new-location-expansion-signal-detector
description: "Catch a brand opening in a new metro weeks early by triangulating its local hiring cluster, Maps footprint, local press, and resident chatter. Use when the user asks something like \"Tell me if Sweetgreen is about to open a location in Miami, Florida based on their LinkedIn hiring and Google Maps footprint\". Runs on the API Direct MCP tools (LinkedIn, Google Maps, news and Reddit data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Local & Places"
  platforms: "LinkedIn, Google Maps, news, Reddit"
  mcp-server: "https://apidirect.io/mcp"
---

# New-Location Expansion Signal Detector

Brands staff a new location weeks before opening. A cluster of metro-specific operational job posts on LinkedIn is the earliest public signal, and a Maps cross-check confirms whether the storefront is already listed. Optionally fold in local press and city-subreddit chatter to confirm a lease or opening date, so you can reach them before the doors open.

**Who it's for:** Suppliers, commercial realtors, and vendors who want first contact.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_companies`, `search_linkedin_jobs`, `linkedin_job_details`, `search_places`, `search_news`, `search_reddit`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `brand` | yes | The company you suspect is expanding. | Sweetgreen |
| `target_metro` | yes | The metro you want expansion signals for. | Miami, Florida |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, places, news, reddit. Omit to run all of them; name specific platforms to limit the run. | news, reddit |
| `lookback` | no | How recent the hiring posts and news must be. Maps to posted_ago on search_linkedin_jobs (allowed: 1h, 24h, 7d, 30d) and time_published on search_news (allowed: 1h, 1d, 7d, 1y, anytime). Use a value valid for each tool's own enum: 1h and 7d work for both. News has no month-level option (it jumps from 7d straight to 1y), so map LinkedIn 24h→News 1d and LinkedIn 30d→News 7d (30d and 24h are NOT valid News values, and 1d/1y are NOT valid LinkedIn values). Default 7d keeps both tight. | 7d |
| `country` | no | Two-letter country code to scope the Maps and News lookups to the right market. Maps to the country param on search_places and search_news. | US |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query="{brand}")
   ```

   Resolve the brand to its numeric company_id so job results are scoped to the real employer.

2. **`search_linkedin_jobs`**

   ```
   search_linkedin_jobs(query="{brand} {target_metro}", company_ids=<company_id>, posted_ago={lookback}, sort_by=most_recent)
   ```

   Surface brand-new postings in the metro within {lookback}; a cluster of local ops roles (GM, store/shift manager) is the expansion signal.

3. **`linkedin_job_details`**

   ```
   linkedin_job_details(url=<job_url>)
   ```

   Open each posting to confirm the work location and read the role; multiple local operational roles equals a confirmed buildout.

4. **`search_places`**

   ```
   search_places(query="{brand} {target_metro}", pages=2, country={country})
   ```

   Cross-check Google Maps for a recently added or not-yet-open {brand} location to time outreach precisely.

5. **`search_news`**

   ```
   search_news(query="{brand} {target_metro} opening", time_published={lookback}, country={country})
   ```

   Only run if {platforms} includes news: Catch local press, commercial-real-estate, and 'coming soon' coverage that often confirms a lease or opening date before the store ever appears on Maps. Use a time_published value valid for News (1h, 1d, 7d, 1y, anytime); if {lookback} is 30d/24h, use the nearest News equivalent (7d/1d).

6. **`search_reddit`**

   ```
   search_reddit(query="{brand} {target_metro}", sort_by=most_recent)
   ```

   Only run if {platforms} includes reddit: Sweep recent reddit chatter (including the metro's city subreddit) for residents spotting construction, signage, or asking when {brand} opens — the earliest ground-level confirmation.

## Deliver

An early-warning brief on {brand} opening in {target_metro}, combining the LinkedIn hiring cluster and role details, the Maps footprint, and (optionally) local press and resident chatter, so you can pitch before launch.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Tell me if Sweetgreen is about to open a location in Miami, Florida based on their LinkedIn hiring and Google Maps footprint
