---
name: viral-tiktok-pattern-miner
description: "Reverse-engineer the week's winning short-form hooks, sounds, and formats across TikTok, YouTube, and Instagram before they saturate. Use when the user asks something like \"Decode the winning TikTok formats and sounds for home espresso in us over the last week so I can brief my editor\". Runs on the API Direct MCP tools (TikTok, YouTube and Instagram data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Content & Influencer"
  platforms: "TikTok, YouTube, Instagram"
  mcp-server: "https://apidirect.io/mcp"
---

# Viral TikTok Pattern Miner

Sorting TikTok by most_liked over a recent window surfaces proven winners, while the most_recent feed reveals which formats and sounds are still fresh enough to ride rather than already played out; optionally cross-checking YouTube Shorts and Instagram Reels confirms which hooks actually travel across platforms versus being TikTok-only flukes.

**Who it's for:** Short-form content strategists and TikTok editors.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_tiktok`, `search_tiktok_users`, `search_youtube`, `search_instagram`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `niche` | yes | The content niche to mine for winning patterns. | home espresso |
| `region` | no | Two-letter region code to scope the TikTok feed (TikTok only; YouTube and Instagram passes are global). | us |
| `platforms` | no | Comma-separated list of platforms to run — options: tiktok, youtube, instagram. Omit to run all of them; name specific platforms to limit the run. | tiktok, youtube, instagram |
| `window` | no | Lookback window in days for the TikTok feed, mapping to publish_time (valid values 7, 30, 90, 180). Smaller catches patterns earlier; larger builds a deeper winners list. Defaults to 7. | 7 |
| `depth` | no | How many result pages to pull per platform search; higher means a deeper scan and more candidate videos. Defaults to a shallow pass. | 3 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_tiktok`**

   ```
   search_tiktok(query={niche}, sort_by=most_liked, publish_time={window}, region={region}, get_sentiment=true, pages={depth})
   ```

   Take the top-liked videos of the window and log each one's music_title/author, hook, and format, keeping the positively-received winners.

2. **`search_tiktok`**

   ```
   search_tiktok(query={niche}, sort_by=most_recent, publish_time={window}, region={region}, pages={depth})
   ```

   Compare against the freshest uploads to flag which winning formats and sounds are still emerging versus already saturated.

3. **`search_tiktok_users`**

   ```
   search_tiktok_users(query=<winning_creator>, pages=2)
   ```

   Resolve the creators behind the top videos and capture follower and video counts to identify who to model or partner with.

4. **`search_youtube`**

   ```
   search_youtube(query={niche}, upload_date=this_week, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes youtube: Pull this week's short-form YouTube videos for the niche and log the hooks, title patterns, and formats of the high-view, positively-received winners to confirm which TikTok patterns also win as YouTube Shorts.

5. **`search_instagram`**

   ```
   search_instagram(query={niche}, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes instagram: Capture the same niche on Instagram to see which hooks and Reel formats are also winning there, reinforcing the cross-platform patterns worth briefing your editor over the TikTok-only flukes.

## Deliver

A content playbook of the period's winning hooks, sounds, and formats plus the creators driving them, with each pattern flagged as confirmed across TikTok, YouTube, and Instagram versus emerging or already saturated.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Decode the winning TikTok formats and sounds for home espresso in us over the last week so I can brief my editor.
