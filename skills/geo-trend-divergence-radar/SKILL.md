---
name: geo-trend-divergence-radar
description: "Catch a topic breaking in one region before it spreads — diff live X trend lists across markets, then confirm the surge on TikTok and in regional news. Use when the user asks something like \"Find topics breaking in 23424977 (United States) that haven't hit 23424975 (United Kingdom) yet and tell me which to jump on\". Runs on the API Direct MCP tools (X, TikTok and news data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Market Research & Trends"
  platforms: "X, TikTok, news"
  mcp-server: "https://apidirect.io/mcp"
---

# Geo-Trend Divergence Radar

Trends surface in one geography first. By set-differencing two regional X trend lists and reading the driver posts, you spot a topic surging in market A that hasn't hit market B yet. Optionally confirm each divergent topic on TikTok (region-scoped) and in the lead market's regional news to separate X-only blips from genuine cross-platform breakouts — a verified early-mover window.

**Who it's for:** Trend forecasters, social strategists, and content teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `twitter_trends`, `search_twitter`, `search_tiktok`, `search_news`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `region_a_woeid` | yes | WOEID of the lead market to scan for emerging trends. | 23424977 (United States) |
| `region_b_woeid` | yes | WOEID of the comparison market to diff against. | 23424975 (United Kingdom) |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, tiktok, news. Omit to run all of them; name specific platforms to limit the run. | twitter, tiktok, news |
| `region_a_code` | no | Region code for the lead market (region A), used to scope the optional TikTok breakout check to the same geography as region_a_woeid. Maps to search_tiktok's region param. | US |
| `region_a_country` | no | Country code for the lead market (region A), used to scope the optional regional-news breakout check. Maps to search_news's country param. | us |
| `tiktok_window` | no | Recency window for the optional TikTok check, as a publish_time value: 1, 7, 30, 90, or 180 days (0 = no time filter). Tighter windows catch the freshest surges. Default: `7`. | 7 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`twitter_trends`**

   ```
   twitter_trends(woeid={region_a_woeid})
   ```

   Pull the live trend list for the lead market and capture each trend name and tweet volume.

2. **`twitter_trends`**

   ```
   twitter_trends(woeid={region_b_woeid})
   ```

   Pull the comparison market's list and set-difference it to isolate trends present in A but absent in B.

3. **`search_twitter`**

   ```
   search_twitter(query=<divergent trend>, sort_by=most_recent, pages=3, get_sentiment=true)
   ```

   Read the freshest posts behind each divergent trend to identify the driver and whether sentiment is positive momentum or backlash.

4. **`search_twitter`**

   ```
   search_twitter(query=<divergent trend>, sort_by=relevance, pages=2)
   ```

   Pull the highest-engagement posts to estimate amplification potential and rank which divergent topics to act on first.

5. **`search_tiktok`**

   ```
   search_tiktok(query=<divergent trend>, region={region_a_code}, publish_time={tiktok_window}, sort_by=most_recent, get_sentiment=true)
   ```

   Only run if {platforms} includes tiktok: confirm the divergent topic is genuinely surging on TikTok in the lead market (region A) and not just an X-only blip; capture recent post velocity and sentiment to validate cross-platform momentum.

6. **`search_news`**

   ```
   search_news(query=<divergent trend>, country={region_a_country}, time_published=1d)
   ```

   Only run if {platforms} includes news: check whether the topic has broken into the lead market's regional news in the last 24h — a strong sign it is crossing from social into mainstream and worth front-running before the comparison market catches up.

## Deliver

A ranked shortlist of topics trending in the lead region but not yet in the comparison region, each with its driver, sentiment, an optional cross-platform confirmation (TikTok post velocity and regional-news pickup), and an early-mover recommendation.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find topics breaking in 23424977 (United States) that haven't hit 23424975 (United Kingdom) yet and tell me which to jump on
