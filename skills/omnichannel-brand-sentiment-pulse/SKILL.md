---
name: omnichannel-brand-sentiment-pulse
description: "Track one brand keyword across the social networks you choose and roll it into a single share-of-positive scoreboard. Use when the user asks something like \"Build me an omnichannel sentiment pulse for Notion and scope the TikTok read to us\". Runs on the API Direct MCP tools (X, Reddit, YouTube, TikTok, Instagram and web forums data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Brand & Social Listening"
  platforms: "X, Reddit, YouTube, TikTok, Instagram, web forums"
  mcp-server: "https://apidirect.io/mcp"
---

# Omnichannel Brand Sentiment Pulse

Each network has a different mood; running the same query with get_sentiment across up to six social surfaces — X, Reddit, YouTube, TikTok, Instagram and niche forums — and normalizing the polarity splits turns scattered chatter into one comparable reputation score you can scope to just the channels that matter.

**Who it's for:** Brand and comms leads who need one number for how a brand is perceived.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_twitter`, `search_reddit`, `search_youtube`, `search_tiktok`, `search_instagram`, `search_forums`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `brand` | yes | Brand or product name to monitor. | Notion |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, reddit, youtube, tiktok, instagram, forums. Omit to run all of them; name specific platforms to limit the run. | twitter, reddit, instagram |
| `region` | no | 2-letter country code to scope TikTok results and niche-forum results. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_twitter`**

   ```
   search_twitter(query={brand}, pages=10, sort_by=most_recent, get_sentiment=true)
   ```

   Only run if {platforms} is unset or includes twitter: Tally positive/negative/neutral polarity and the dominant emotion for the X channel.

2. **`search_reddit`**

   ```
   search_reddit(query={brand}, page=5, sort_by=top, get_sentiment=true)
   ```

   Only run if {platforms} is unset or includes reddit: Capture the Reddit polarity split and pull the most-upvoted complaint threads.

3. **`search_youtube`**

   ```
   search_youtube(query={brand}, pages=3, upload_date=this_month, get_sentiment=true)
   ```

   Only run if {platforms} is unset or includes youtube: Score this month's video coverage sentiment and note the loudest creators.

4. **`search_tiktok`**

   ```
   search_tiktok(query={brand}, pages=3, region={region}, publish_time=30, sort_by=most_liked, get_sentiment=true)
   ```

   Only run if {platforms} is unset or includes tiktok: Gauge short-video buzz and the dominant emotion on the most-liked recent clips, scoped to {region} when provided.

5. **`search_instagram`**

   ```
   search_instagram(query={brand}, pages=3, get_sentiment=true)
   ```

   Only run if {platforms} is unset or includes instagram: Score caption and post sentiment and the dominant emotion on recent brand posts to capture the visual/creator channel.

6. **`search_forums`**

   ```
   search_forums(query={brand}, time=month, country={region}, get_sentiment=true)
   ```

   Only run if {platforms} is unset or includes forums: Add niche-community sentiment (scoped to {region} when provided), then normalize whichever channels were run into one weighted share-of-positive scoreboard.

## Deliver

A single-page scoreboard showing share-of-positive, dominant emotion, and top quotes for the brand on each selected network, plus a blended cross-channel sentiment score.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me an omnichannel sentiment pulse for Notion and scope the TikTok read to us
