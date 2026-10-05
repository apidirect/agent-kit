---
name: niche-youtube-creator-leaderboard
description: "See who's actually winning a niche by momentum, not follower vanity — YouTube core, optionally extended to TikTok and Instagram. Use when the user asks something like \"Rank the YouTube creators actually winning in AI productivity tools right now by view-velocity versus their subscriber base\". Runs on the API Direct MCP tools (YouTube, TikTok and Instagram data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Content & Influencer"
  platforms: "YouTube, TikTok, Instagram"
  mcp-server: "https://apidirect.io/mcp"
---

# Niche YouTube Creator Leaderboard

Subscriber/follower counts are lagging vanity metrics. Pairing the niche creator roster with this month's and this week's real engagement exposes the over-indexers truly winning right now. YouTube is the always-on core; TikTok and Instagram can be toggled on to find momentum creators wherever they post, applying the same velocity-over-vanity logic.

**Who it's for:** Brand sponsorship and creator-partnership scouts.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_youtube_channels`, `search_youtube`, `search_tiktok`, `search_instagram`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `niche` | yes | The niche to rank momentum creators within. | AI productivity tools |
| `platforms` | no | Comma-separated list of platforms to run — options: youtube, tiktok, instagram. Omit to run all of them; name specific platforms to limit the run. | tiktok, instagram |
| `depth` | no | Optional. How many pages to pull per platform — more pages means a deeper roster and more videos/posts for accurate velocity attribution. Defaults to 3. | 3 |
| `region` | no | Optional. Region code to scope the TikTok momentum search to a specific market when scouting region-specific creators. Only affects the TikTok step. | US |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_youtube_channels`**

   ```
   search_youtube_channels(query={niche}, pages={depth})
   ```

   Core (always runs): build the candidate roster of niche YouTube channels and record each subscriber_count as the vanity baseline.

2. **`search_youtube`**

   ```
   search_youtube(query={niche}, upload_date=this_month, get_sentiment=true, pages={depth})
   ```

   Core: pull this month's uploads, attribute view counts back to channels, and flag which are drawing positive reception via sentiment.

3. **`search_youtube`**

   ```
   search_youtube(query={niche}, upload_date=this_week, pages={depth})
   ```

   Core: layer in this week's breakout videos and rank channels by view-velocity relative to subscriber base to surface the momentum over-indexers.

4. **`search_tiktok`**

   ```
   search_tiktok(query={niche}, publish_time=7, sort_by=most_liked, get_sentiment=true, region={region}, pages={depth})
   ```

   Only run if {platforms} includes tiktok: pull the week's most-liked niche TikToks (optionally scoped to {region}), attribute like-velocity to creators, and rank the TikTok momentum risers by engagement velocity, not follower vanity.

5. **`search_instagram`**

   ```
   search_instagram(query={niche}, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes instagram: pull recent niche Instagram posts, attribute engagement back to creators, and surface the accounts over-indexing on recent engagement velocity for the same momentum leaderboard.

## Deliver

A live, optionally cross-platform leaderboard of the niche's top creators ranked by momentum (view/engagement velocity versus follower base) rather than subscriber count alone — YouTube by default, with optional TikTok and Instagram extensions under the user's control.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Rank the YouTube creators actually winning in AI productivity tools right now by view-velocity versus their subscriber base.
