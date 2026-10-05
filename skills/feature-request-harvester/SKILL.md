---
name: feature-request-harvester
description: "Turn scattered \"I wish it could\" chatter across Reddit, forums, X, YouTube and Facebook into a ranked, evidence-backed feature backlog. Use when the user asks something like \"Build me a ranked feature backlog of everything users wish Notion could do\". Runs on the API Direct MCP tools (Reddit, web forums, X, YouTube and Facebook data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Product & Customer Insights"
  platforms: "Reddit, web forums, X, YouTube, Facebook"
  mcp-server: "https://apidirect.io/mcp"
---

# Feature Request Harvester

Users rarely file feature requests — they vent them as wishes in passing. This skill mines wish-phrasing across up to five communities (Reddit posts and comments, niche forums, X, YouTube wishlist/review videos, and Facebook user-group posts) and ranks each desired capability by how often it recurs and how strongly people feel about its absence. Use the optional platforms input to choose which sources run.

**Who it's for:** Product managers and founders building a roadmap.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_reddit`, `search_reddit_comments`, `search_forums`, `search_twitter`, `search_youtube`, `search_facebook_posts`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `product` | yes | Product or brand name to mine for unmet feature wishes. | Notion |
| `platforms` | no | Comma-separated list of platforms to run — options: reddit, forums, twitter, youtube, facebook. Omit to run all of them; name specific platforms to limit the run. | reddit, forums, twitter, youtube, facebook |
| `recency` | no | Freshness window for the forums search. One of: any, day, hour, month, week, year. Defaults to year so evergreen power-user threads still surface. | year |
| `depth` | no | How many result pages to pull per source on the comment, X, YouTube and Facebook scans — raise for broader coverage, lower for a fast scan. Default: `2`. | 2 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_reddit`**

   ```
   search_reddit(query='{product} "i wish" OR "would love" OR "needs a"', sort_by=top, get_sentiment=true)
   ```

   Anchor source — always runs. Capture top wish-phrased posts; prioritize ones whose dominant_emotion is anticipation or sadness (felt gaps).

2. **`search_reddit_comments`**

   ```
   search_reddit_comments(query='{product} lacks OR "doesnt support" OR workaround', sort_by=top, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes reddit: Surface buried in-thread complaints and the workarounds people resort to as proof of a missing capability.

3. **`search_forums`**

   ```
   search_forums(query='{product} feature request OR missing OR "no way to"', time={recency}, get_sentiment=true)
   ```

   Only run if {platforms} includes forums: Pull niche power-user threads that name specific missing capabilities; {recency} controls the freshness window.

4. **`search_twitter`**

   ```
   search_twitter(query='{product} "wish it could" OR "why cant {product}" OR "needs to add"', sort_by=relevance, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes twitter: Add public X chatter and treat the most-engaged wishes as the strongest demand signal.

5. **`search_youtube`**

   ```
   search_youtube(query='{product} "needs to add" OR missing OR "wish it had" OR review', upload_date=this_year, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes youtube: Mine creator wishlist and review videos ('things [product] needs to fix') where missing features are called out in detail.

6. **`search_facebook_posts`**

   ```
   search_facebook_posts(query='{product} "wish it had" OR "no way to" OR missing OR "cant"', get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes facebook: Mine public posts and product-user-group chatter for 'wish it had / no way to' gaps, then cluster every collected source into themes and rank by cross-platform mention frequency and emotional intensity.

## Deliver

A ranked feature backlog where each requested capability is scored by mention frequency across whichever selected platforms ran and by emotional intensity, with linked source quotes. Cluster all collected items into themes before ranking, and flag capabilities that recur on three or more platforms as highest-conviction.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me a ranked feature backlog of everything users wish Notion could do.
