---
name: cross-platform-video-share-of-voice
description: "Find the creators dominating your topic across YouTube, TikTok, and Instagram and rank them by reach and sentiment. Use when the user asks something like \"Show me who owns the video conversation about AI note-taking apps on YouTube and TikTok in us\". Runs on the API Direct MCP tools (YouTube, TikTok and Instagram data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Brand & Social Listening"
  platforms: "YouTube, TikTok, Instagram"
  mcp-server: "https://apidirect.io/mcp"
---

# Cross-Platform Video Share-of-Voice

Video is where category narratives form; pairing each platform's content search with its creator-resolver gives true subscriber and follower counts, so you can rank who actually owns the conversation rather than who posted most. It spans YouTube, TikTok, and Instagram Reels for full short-form coverage, with the user choosing exactly which platforms execute.

**Who it's for:** Content and partnerships teams scouting the video landscape of a category.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_youtube`, `search_youtube_channels`, `search_tiktok`, `search_tiktok_users`, `search_instagram`, `search_instagram_users`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `topic` | yes | Topic, product category, or branded keyword to track on video. | AI note-taking apps |
| `region` | no | 2-letter country code to scope TikTok results. | us |
| `platforms` | no | Comma-separated list of platforms to run — options: youtube, tiktok, instagram. Omit to run all of them; name specific platforms to limit the run. | youtube, tiktok, instagram |
| `depth` | no | How many pages of content to pull per platform (controls result volume and cost). Defaults to 5. | 5 |
| `tiktok_sort` | no | How to rank the TikTok pull: most_liked (reach), most_recent (freshness), or relevance. Defaults to most_liked. | most_liked |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_youtube`**

   ```
   search_youtube(query={topic}, pages={depth}, upload_date=this_month, get_sentiment=true)
   ```

   Only run if {platforms} includes youtube: collect this month's videos, the channels behind them, and per-video sentiment.

2. **`search_youtube_channels`**

   ```
   search_youtube_channels(query={topic}, pages=2)
   ```

   Only run if {platforms} includes youtube: resolve subscriber_count for each channel to size its real YouTube reach.

3. **`search_tiktok`**

   ```
   search_tiktok(query={topic}, pages={depth}, region={region}, publish_time=30, sort_by={tiktok_sort}, get_sentiment=true)
   ```

   Only run if {platforms} includes tiktok: pull recent TikToks on the topic (most_liked by default) with sentiment, scoped to {region}.

4. **`search_tiktok_users`**

   ```
   search_tiktok_users(query={topic}, pages=2)
   ```

   Only run if {platforms} includes tiktok: resolve TikTok follower and video counts, then merge every platform run so far into one share-of-voice ranking by reach times positive sentiment.

5. **`search_instagram`**

   ```
   search_instagram(query={topic}, pages={depth}, get_sentiment=true)
   ```

   Only run if {platforms} includes instagram: pull recent Instagram Reels and posts on the topic with per-post sentiment to capture short-form video reach.

6. **`search_instagram_users`**

   ```
   search_instagram_users(query={topic})
   ```

   Only run if {platforms} includes instagram: resolve each creator's Instagram follower count, then fold Instagram into the unified reach-times-sentiment leaderboard alongside YouTube and TikTok, tagging each creator's platform skew.

## Deliver

A unified video share-of-voice leaderboard of the top YouTube, TikTok, and (optionally) Instagram creators on your topic, ranked by reach times positive sentiment, with each creator's platform skew.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Show me who owns the video conversation about AI note-taking apps on YouTube and TikTok in us
