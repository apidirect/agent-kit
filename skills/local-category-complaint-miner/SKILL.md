---
name: local-category-complaint-miner
description: "Sweep one-star reviews — plus optional Reddit, X, and Facebook chatter — across a metro's category to surface the unmet needs nobody is solving. Use when the user asks something like \"Mine one-star reviews for dog grooming businesses across Austin, Texas and cluster the unmet needs I could build around\". Runs on the API Direct MCP tools (Google Maps, Reddit, X and Facebook data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Market Research & Trends"
  platforms: "Google Maps, Reddit, X, Facebook"
  mcp-server: "https://apidirect.io/mcp"
---

# Local Category Complaint Miner

A metro's lowest-rated reviews are a free, honest backlog of unmet needs. Sentiment-filter the angriest Google reviews across many local players, then optionally triangulate against Reddit, X, and Facebook complaints to confirm which pains are real, recurring, and still live — so you can productize or out-execute them.

**Who it's for:** Founders, local-market entrants, and product strategists.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_reviews`, `search_reddit`, `search_twitter`, `search_facebook_posts`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `category` | yes | The business category to investigate. | dog grooming |
| `metro` | yes | City or metro to sweep. | Austin, Texas |
| `platforms` | no | Comma-separated list of platforms to run — options: places, reddit, twitter, facebook. Omit to run all of them; name specific platforms to limit the run. | reddit, twitter, facebook |
| `country` | no | ISO country code to disambiguate Google Places results for non-US metros. Applied to all Places calls; leave blank to auto-infer from the metro. | us |
| `depth` | no | How many pages to pull per source (harshest reviews and social posts). Maps to the pages param; higher = deeper, more complete sweep but slower. Defaults to 3. | 3 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{category} {metro}", country={country}, pages=5)
   ```

   Collect every local provider in the category with its place_id, rating, and review_count; prioritize those with enough reviews to mine. Pass {country} only for non-US metros, otherwise omit it.

2. **`place_reviews`**

   ```
   place_reviews(place_id=<place_id>, sort_by=lowest_ranking, pages={depth}, get_sentiment=true, country={country})
   ```

   For each provider pull the harshest reviews and keep only items with negative polarity or anger/disgust as the dominant emotion; capture review dates so you can flag complaints that are still live. {depth} defaults to 3.

3. **`place_reviews`**

   ```
   place_reviews(place_id=<place_id>, sort_by=newest, pages=2, get_sentiment=true, country={country})
   ```

   Cross-check recent reviews to confirm the complaint is still live and not a fixed legacy issue; drop themes that only appear in old reviews.

4. **`search_reddit`**

   ```
   search_reddit(query="{category} {metro}", get_sentiment=true, sort_by=top)
   ```

   Only run if {platforms} includes reddit: mine city-subreddit threads and recommendation posts for the same category; keep negative-sentiment comments that name a recurring failure, and treat any theme that ALSO appears in the Google reviews as high-confidence.

5. **`search_twitter`**

   ```
   search_twitter(query="{category} {metro}", get_sentiment=true, pages={depth}, sort_by=most_recent)
   ```

   Only run if {platforms} includes twitter: capture fresh, geo-relevant complaints to confirm the pain is still live right now; keep only negative-sentiment posts and fold them into the same complaint clusters.

6. **`search_facebook_posts`**

   ```
   search_facebook_posts(query="{category} {metro}", get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes facebook: pull local-group and community posts where residents vent about or seek replacements for category providers; keep negative-sentiment items and merge recurring pains into the existing clusters.

## Deliver

A clustered list of recurring unmet needs across the metro's category, ranked by frequency and emotional intensity, with the worst-performing incumbents named. Themes that recur in BOTH Google reviews and the optional Reddit/X/Facebook sweeps are flagged as highest-confidence opportunities.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Mine one-star reviews for dog grooming businesses across Austin, Texas and cluster the unmet needs I could build around
