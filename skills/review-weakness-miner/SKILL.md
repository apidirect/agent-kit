---
name: review-weakness-miner
description: "Surface a rival's recurring failures from their angriest reviews and complaint threads across Google, Facebook, and Reddit. Use when the user asks something like \"Find Equinox's biggest recurring complaints in Austin from their worst reviews\". Runs on the API Direct MCP tools (Google Maps, Facebook and Reddit data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Competitive Intelligence"
  platforms: "Google Maps, Facebook, Reddit"
  mcp-server: "https://apidirect.io/mcp"
---

# Review Weakness Miner

A competitor's worst reviews are a free, honest product-gap audit. Sorting to the lowest-ranked reviews and corroborating across review sites plus unprompted Reddit complaint threads isolates the failures that repeat, not the one-off rants. Use the platforms input to choose which sources beyond Google to run, and depth to control how much evidence to pull.

**Who it's for:** Product teams, local marketers, and competitive positioning leads.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_reviews`, `search_facebook_pages`, `facebook_page_details`, `facebook_page_reviews`, `search_reddit`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `competitor` | yes | The rival business/brand name to locate on Google Maps, Facebook, and Reddit. | Equinox |
| `city` | yes | The city or market to scope the place search to. | Austin |
| `platforms` | no | Comma-separated list of platforms to run — options: places, facebook, reddit. Omit to run all of them; name specific platforms to limit the run. | facebook, reddit |
| `depth` | no | How many pages of reviews/threads to pull per source (maps to the pages param on the review tools and the page param on search_reddit). Higher means more evidence and more cost; 1-2 is usually enough. Default: `2`. | 2 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{competitor} {city}")
   ```

   Always runs (Google is the anchor source). Grab the place_id of the rival's location(s) with the most reviews.

2. **`place_reviews`**

   ```
   place_reviews(place_id=<place_id>, sort_by=lowest_ranking, get_sentiment=true, pages={depth})
   ```

   Always runs. Keep 1-2 star reviews and cluster recurring failure themes (wait times, billing, staff). pages={depth} is optional; omit to pull the default depth.

3. **`search_facebook_pages`**

   ```
   search_facebook_pages(query={competitor})
   ```

   Only run if {platforms} includes facebook: find the rival's official Facebook page url for a second review source.

4. **`facebook_page_details`**

   ```
   facebook_page_details(url=<page_url>)
   ```

   Only run if {platforms} includes facebook: resolve the page_id required for the reviews endpoint.

5. **`facebook_page_reviews`**

   ```
   facebook_page_reviews(page_id=<page_id>, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes facebook: keep recommend=false reviews and confirm which weaknesses repeat across Google and Facebook. pages={depth} is optional.

6. **`search_reddit`**

   ```
   search_reddit(query="{competitor} complaints", get_sentiment=true, sort_by=top, page={depth})
   ```

   Only run if {platforms} includes reddit: pull the most-upvoted complaint threads to corroborate which failures users repeat unprompted, beyond formal reviews. page={depth} is optional (note search_reddit uses the singular page param).

## Deliver

A ranked weakness map of the rival's top recurring complaints, evidenced by quoted 1-star reviews from Google and Facebook and, when enabled, the most-upvoted Reddit complaint threads.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find Equinox's biggest recurring complaints in Austin from their worst reviews
