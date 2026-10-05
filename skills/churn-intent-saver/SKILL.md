---
name: churn-intent-saver
description: "Catch 'I'm cancelling / switching' posts across X, Reddit and forums and prioritize the loudest accounts for a save. Use when the user asks something like \"Who's threatening to cancel or switch away from API Direct right now, and which ones have the biggest audiences?\". Runs on the API Direct MCP tools (X, Reddit and web forums data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Product & Customer Insights"
  platforms: "X, Reddit, web forums"
  mcp-server: "https://apidirect.io/mcp"
---

# Churn-Intent Saver

Monitors X — and optionally Reddit and product forums — for explicit churn intent about your brand, sizes each X author's reach, and pulls context so you can intervene before they're gone — biggest megaphones first.

**Who it's for:** Support, success and social teams doing proactive saves.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_twitter`, `twitter_user_profile`, `twitter_user_tweets`, `search_reddit`, `search_forums`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `brand` | yes | Your brand/product to monitor for churn and switching intent. | API Direct |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, reddit, forums. Omit to run all of them; name specific platforms to limit the run. | twitter, reddit, forums |
| `depth` | no | How many result pages to pull per platform (maps to each tool's pages/page param). Higher = more candidates, more cost. Default: `2`. | 2 |
| `time_window` | no | Freshness filter for the forums sweep (maps to the forums time param). Keep recent so saves are still actionable. Default: `week`. | week |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_twitter`**

   ```
   search_twitter(query="(cancelling OR \"switching from\" OR \"done with\") {brand}", get_sentiment=true, sort_by=most_recent, pages={depth})
   ```

   Core (always runs): surface explicit churn-intent posts about {brand}; keep negative-polarity ones. Use {depth} to control how many pages to pull.

2. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<author>)
   ```

   For each churning X author from step 1, pull followers_count and verification to size each account's blast radius.

3. **`twitter_user_tweets`**

   ```
   twitter_user_tweets(username=<author>, get_sentiment=true)
   ```

   Read <author>'s recent tweets for context on why they're leaving so you can draft a tailored save.

4. **`search_reddit`**

   ```
   search_reddit(query="{brand} (cancelling OR \"switching from\" OR \"done with\")", get_sentiment=true, sort_by=most_recent, page={depth})
   ```

   Only run if {platforms} includes reddit: catch the same cancellation/switching intent in subreddit threads; keep negative-polarity posts and note the subreddit as added context for the save.

5. **`search_forums`**

   ```
   search_forums(query="{brand} cancel OR switching OR leaving", get_sentiment=true, time={time_window}, page={depth})
   ```

   Only run if {platforms} includes forums: surface churn intent on niche/product forums; use {time_window} (e.g. week) to keep results fresh and actionable.

## Deliver

A prioritized save queue across X, Reddit and forums — handle/source, reach where available, reason, and a suggested response — highest-reach churners first.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Who's threatening to cancel or switch away from API Direct right now, and which ones have the biggest audiences?
