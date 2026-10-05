---
name: local-buying-intent-capture
description: "Catch locals the moment they post — on Facebook, Reddit, or X — that they need exactly what you sell. Use when the user asks something like \"Find fresh leads in Austin, Texas who are posting that they're looking for a realtor\". Runs on the API Direct MCP tools (Facebook, Reddit and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Lead Generation & Sales"
  platforms: "Facebook, Reddit, X"
  mcp-server: "https://apidirect.io/mcp"
---

# Local Buying-Intent Lead Capture

People announce life events and service needs in real time — in geo-tagged Facebook posts, in their city subreddit, and on X — before they ever Google a provider. This skill geofences a metro and mines those channels for that live intent, and lets you choose which platforms to run.

**Who it's for:** Realtors, local service businesses, and territory reps.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_facebook_locations`, `search_facebook_posts`, `search_reddit`, `search_twitter`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `city` | yes | Target metro or city to geofence. | Austin, Texas |
| `intent_phrase` | yes | The buying-intent phrase a local would post. | looking for a realtor |
| `platforms` | no | Comma-separated list of platforms to run — options: facebook, reddit, twitter. Omit to run all of them; name specific platforms to limit the run. | facebook, reddit, twitter |
| `result_depth` | no | How many pages of results to pull per platform where supported (maps to the pages param). Higher returns more leads but runs slower; defaults to a shallow first page. | 3 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_facebook_locations`**

   ```
   search_facebook_locations(query={city})
   ```

   Only run if {platforms} includes facebook: resolve the metro to its numeric location_id for geo-scoped searching.

2. **`search_facebook_posts`**

   ```
   search_facebook_posts(query="{intent_phrase}", location_id=<location_id>, sort_by=most_recent, get_sentiment=true, pages={result_depth})
   ```

   Only run if {platforms} includes facebook: surface fresh geo-tagged public posts signaling active need and keep ones expressing clear intent.

3. **`search_reddit`**

   ```
   search_reddit(query="{intent_phrase} {city}", sort_by=most_recent, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: catch locals posting recommendation requests in their city subreddit — the top organic channel for 'who should I hire' asks — scoped by the city term in the query.

4. **`search_twitter`**

   ```
   search_twitter(query="{intent_phrase} {city}", sort_by=most_recent, get_sentiment=true, pages={result_depth})
   ```

   Only run if {platforms} includes twitter: capture fresh public posts where locals ask for or announce the need, scoped to the metro by the city term in the query.

## Deliver

A live, deduped feed of local residents in {city} signaling active intent across Facebook, Reddit, and X — each with the post link and sentiment for direct outreach.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find fresh leads in Austin, Texas who are posting that they're looking for a realtor
