---
name: rising-creator-early-detector
description: "Find undervalued creators across TikTok, Instagram and YouTube by engagement-to-follower ratio before they blow up. Use when the user asks something like \"Find up-and-coming home cooking gadgets creators on TikTok who get way more engagement than their follower count suggests\". Runs on the API Direct MCP tools (TikTok, Instagram and YouTube data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Content & Influencer"
  platforms: "TikTok, Instagram, YouTube"
  mcp-server: "https://apidirect.io/mcp"
---

# Rising-Creator Early Detector

Surfaces the most-engaged recent content in a niche across TikTok (with optional Instagram and YouTube), divides engagement by each creator's follower or subscriber count, and flags those punching far above their size — cheap partnerships before prices rise. You choose the platforms, freshness window, search depth, and follower ceiling.

**Who it's for:** Influencer marketers, talent scouts and brand partnerships.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_tiktok`, `search_tiktok_users`, `search_instagram`, `search_instagram_users`, `search_youtube`, `search_youtube_channels`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `niche` | yes | The content niche to scan. | home cooking gadgets |
| `platforms` | no | Comma-separated list of platforms to run — options: tiktok, instagram, youtube. Omit to run all of them; name specific platforms to limit the run. | tiktok, instagram, youtube |
| `region` | no | 2-letter region code to scope the scan to. Applies to the TikTok search (the only search tool with a region param). | us |
| `freshness_days` | no | How recent the content must be, in days. Maps directly to TikTok's publish_time (allowed values: 0, 1, 7, 30, 90, 180) and to the nearest YouTube upload window (today/this_week/this_month/this_year). Default: 30. | 30 |
| `pages` | no | How many pages of results to pull per search — more pages = a wider net of creators. Default: 2. | 2 |
| `max_followers` | no | Ceiling on follower/subscriber count, applied in the ratio step, so only smaller, still-undervalued creators surface. Default: no cap. | 100000 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_tiktok`**

   ```
   search_tiktok(query="{niche}", sort_by=most_liked, publish_time={freshness_days}, region="{region}", pages={pages})
   ```

   Base scan (always runs). Pull the most-liked TikTok videos in the niche from your freshness window. Record each author handle and the video's like count.

2. **`search_tiktok_users`**

   ```
   search_tiktok_users(query=<tiktok_author>, pages={pages})
   ```

   For each TikTok author, get follower count and video_count, then compute likes ÷ followers. Flag creators with a high ratio but a following at or below {max_followers} (no cap if unset) — rising creators to lock in early.

3. **`search_instagram`**

   ```
   search_instagram(query="{niche}", pages={pages})
   ```

   Only run if {platforms} includes instagram: pull recent niche posts on Instagram. Record each creator's username and the post's like/comment engagement.

4. **`search_instagram_users`**

   ```
   search_instagram_users(query=<instagram_username>)
   ```

   Only run if {platforms} includes instagram: look up each creator's follower count, compute engagement ÷ followers, and flag high-ratio accounts at or below {max_followers}. (Note: search_instagram_users takes only query — no pages param.)

5. **`search_youtube`**

   ```
   search_youtube(query="{niche}", upload_date=this_month, pages={pages})
   ```

   Only run if {platforms} includes youtube: pull top recent niche videos. Record each channel name and the video's view count. Adjust upload_date to match {freshness_days}: today / this_week / this_month / this_year.

6. **`search_youtube_channels`**

   ```
   search_youtube_channels(query=<youtube_channel>, pages={pages})
   ```

   Only run if {platforms} includes youtube: pull each channel's subscriber count and compute views ÷ subscribers. Then merge all platforms into one shortlist, dedupe creators that appear on more than one network, drop anyone above {max_followers}, and rank by engagement-to-follower ratio (best first), tagging each with its platform.

## Deliver

A deduped, cross-platform shortlist of undervalued creators, each tagged with its platform, follower/subscriber count, and engagement-to-follower ratio, ranked best-first for contact-worthiness.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find up-and-coming home cooking gadgets creators on TikTok who get way more engagement than their follower count suggests.
