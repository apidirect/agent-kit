---
name: paid-creative-reverse-engineer
description: "Teardown a competitor's best-performing video hooks and angles across Facebook, TikTok and YouTube. Use when the user asks something like \"Reverse-engineer Gymshark's best-performing video ads and hooks\". Runs on the API Direct MCP tools (Facebook, TikTok and YouTube data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Competitive Intelligence"
  platforms: "Facebook, TikTok, YouTube"
  mcp-server: "https://apidirect.io/mcp"
---

# Paid-Creative Reverse Engineer

A rival's top reels and videos reveal which hooks, formats, and CTAs actually convert. Ranking Facebook reels by play count and TikTok by most-liked strips away vanity content, while an optional YouTube pass extends the teardown to video ads, pre-roll, and long-form hooks. A platforms input lets you choose exactly which cross-platform surfaces to run, and region, recency, and depth controls tune the pull.

**Who it's for:** Growth marketers, paid social teams, and creative strategists.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_facebook_pages`, `facebook_page_details`, `facebook_page_reels`, `facebook_page_videos`, `search_tiktok`, `search_youtube`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `competitor` | yes | The rival brand whose video creative you want to reverse-engineer. | Gymshark |
| `platforms` | no | Comma-separated list of platforms to run — options: facebook, tiktok, youtube. Omit to run all of them; name specific platforms to limit the run. | facebook, tiktok, youtube |
| `region` | no | Two-letter region code applied to the TikTok search to localize which creative is winning. | us |
| `recency_days` | no | Freshness window in days for TikTok results (one of 1, 7, 30, 90, 180) to focus on current creative trends. Default: `30`. | 30 |
| `depth` | no | How many pages of results to pull per platform (more pages = a deeper teardown); applies to the reels, videos, TikTok, and YouTube steps. Default: `2`. | 2 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_facebook_pages`**

   ```
   search_facebook_pages(query={competitor})
   ```

   Locate the brand's official Facebook page url.

2. **`facebook_page_details`**

   ```
   facebook_page_details(url=<page_url>)
   ```

   Pull the reels_page_id and delegate_page_id needed for the media endpoints.

3. **`facebook_page_reels`**

   ```
   facebook_page_reels(reels_page_id=<reels_page_id>, get_sentiment=true, pages={depth})
   ```

   Rank reels by play_count to surface their highest-performing hooks and openers. Facebook is the always-on anchor of the teardown.

4. **`facebook_page_videos`**

   ```
   facebook_page_videos(delegate_page_id=<delegate_page_id>, get_sentiment=true, pages={depth})
   ```

   Sort by views to extract recurring video angles, story structures, and CTAs.

5. **`search_tiktok`**

   ```
   search_tiktok(query={competitor}, sort_by=most_liked, region={region}, publish_time={recency_days}, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes tiktok: Cross-check which creative concepts win on TikTok; flag is_ad items and the trending sounds used.

6. **`search_youtube`**

   ```
   search_youtube(query={competitor}, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes youtube: Surface the brand's YouTube long-form video hooks, formats, and CTAs to complete the cross-platform creative teardown.

## Deliver

A cross-platform creative teardown of the rival's top-performing video hooks, formats, and angles — with the sounds and CTAs to replicate — across Facebook, TikTok, and (optionally) YouTube, for whichever platforms you choose to run.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Reverse-engineer Gymshark's best-performing video ads and hooks
