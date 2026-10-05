---
name: dtc-traction-scout
description: "Catch consumer brands breaking out across TikTok, Reddit and YouTube before they raise, then surface the founder's inbox. Use when the user asks something like \"Surface men's skincare brands blowing up on TikTok in us right now and get me their founder contact details\". Runs on the API Direct MCP tools (TikTok, Reddit, YouTube and Instagram data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Investing & Deal Sourcing"
  platforms: "TikTok, Reddit, YouTube, Instagram"
  mcp-server: "https://apidirect.io/mcp"
---

# DTC Breakout Traction Scout

Breakout DTC traction shows up on TikTok months before it reaches a pitch deck. Sorting by most-liked surfaces the brand handle, and optional Reddit and YouTube passes widen the net to brands generating organic, pre-ad-spend buzz and video momentum. Instagram then hands you the founder's public email and store link for a pre-institutional intro.

**Who it's for:** Consumer and seed VCs hunting pre-institutional brands.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_tiktok`, `search_reddit`, `search_youtube`, `search_tiktok_users`, `instagram_user_profile`, `search_instagram`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `niche` | yes | The consumer category to scout. | men's skincare |
| `platforms` | no | Comma-separated list of platforms to run — options: tiktok, reddit, youtube, instagram. Omit to run all of them; name specific platforms to limit the run. | tiktok, instagram, reddit, youtube |
| `lookback` | no | How many days back to scan TikTok for breakout videos (maps to publish_time). Valid values: 7, 30, 90, 180. Defaults to 30 if omitted. | 30 |
| `region` | no | 2-letter region code for TikTok search. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_tiktok`**

   ```
   search_tiktok(query="{niche}", sort_by=most_liked, publish_time={lookback}, region={region}, get_sentiment=true)
   ```

   Core breakout signal: keep videos with outsized likes in the last {lookback} days (default 30) and pull the recurring brand handle driving them.

2. **`search_reddit`**

   ```
   search_reddit(query="{niche}", sort_by=top, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: surface brands getting organic, upvoted community buzz in the niche and add any recurring handles to the candidate set — pre-ad-spend demand a deck won't show yet.

3. **`search_youtube`**

   ```
   search_youtube(query="{niche}", upload_date=this_month, get_sentiment=true)
   ```

   Only run if {platforms} includes youtube: catch brands trending in hauls and review videos this month and add them as additional breakout candidates.

4. **`search_tiktok_users`**

   ```
   search_tiktok_users(query=<brand_name>)
   ```

   Resolve each candidate brand account and read follower/video counts to confirm sustained momentum, not a one-hit video.

5. **`instagram_user_profile`**

   ```
   instagram_user_profile(username=<brand_handle>)
   ```

   Contact engine: grab bio, followers, external_url, public_email, and category for direct founder outreach.

6. **`search_instagram`**

   ```
   search_instagram(query=<brand_name>, get_sentiment=true)
   ```

   Verify cross-platform pull and gauge audience sentiment before reaching out; drop brands with weak or negative sentiment.

## Deliver

A shortlist of pre-institutional DTC brands with TikTok (plus optional Reddit/YouTube) traction signals, Instagram metrics, and direct founder contact info.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Surface men's skincare brands blowing up on TikTok in us right now and get me their founder contact details.
