---
name: cross-platform-demand-triangulation
description: "Prove a niche is not just hot but accelerating by triangulating fresh demand and buyer-intent signals across up to six independent platforms — and run only the ones you trust. Use when the user asks something like \"Is creatine gummies a growing demand or already saturated — triangulate it across platforms and give me a verdict\". Runs on the API Direct MCP tools (Google Search, YouTube, TikTok, Reddit, web forums and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Cross-Platform Power Plays"
  platforms: "Google Search, YouTube, TikTok, Reddit, web forums, X"
  mcp-server: "https://apidirect.io/mcp"
---

# Cross-Platform Demand Triangulation

A single platform lies — a trend can look red-hot on TikTok yet be dead in the forums where real buyers ask questions. Measuring recent volume and sentiment in parallel across web search, YouTube, TikTok, Reddit, forums, and X separates durable, buyer-backed demand from a flash in the pan. Pass a platforms list to run only the signals you care about, set a region to localize the scan, and the engine triangulates a single verdict from exactly the sources you chose.

**Who it's for:** Founders and product teams validating a niche before they build.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_web`, `search_youtube`, `search_tiktok`, `search_reddit`, `search_forums`, `search_twitter`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `niche` | yes | The topic or product niche to validate. | creatine gummies |
| `platforms` | no | Comma-separated list of platforms to run — options: web, youtube, tiktok, reddit, forums, twitter. Omit to run all of them; name specific platforms to limit the run. | web, youtube, tiktok, reddit |
| `region` | no | 2-letter country code to localize the scan — sets the TikTok region plus the web and forum country. Leave blank for a global read. | us |
| `time_window` | no | Recency window for the keyword-search demand scans (web and forums). One of day, week, month, or year. Defaults to month. | month |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_web`**

   ```
   search_web(query="{niche}", country={region}, time={time_window}, include_ai_overview=true)
   ```

   Only run if {platforms} includes web: Read the recent results and the AI overview to gauge how much organic search demand and buyer-intent the niche already commands — the hardest demand signal to fake and the strongest tell of real purchase consideration.

2. **`search_youtube`**

   ```
   search_youtube(query="{niche}", upload_date=this_month, get_sentiment=true)
   ```

   Only run if {platforms} includes youtube: Count fresh uploads this month and read dominant_emotion to see whether creators are actively producing and audiences are excited right now.

3. **`search_tiktok`**

   ```
   search_tiktok(query="{niche}", region={region}, publish_time=30, sort_by=most_liked, get_sentiment=true)
   ```

   Only run if {platforms} includes tiktok: Pull the most-liked clips from the last 30 days — high likes on recent posts confirm consumer pull, not just creator push.

4. **`search_reddit`**

   ```
   search_reddit(query="{niche}", sort_by=hot, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: Sample hot threads to see if communities are organically surfacing the topic right now — rising hot posts signal accelerating interest.

5. **`search_forums`**

   ```
   search_forums(query="{niche}", country={region}, time={time_window}, get_sentiment=true)
   ```

   Only run if {platforms} includes forums: Check whether real buyers are asking questions or complaining; forum demand is higher-intent and far harder to fake than video views.

6. **`search_twitter`**

   ```
   search_twitter(query="{niche}", sort_by=most_recent, get_sentiment=true)
   ```

   Only run if {platforms} includes twitter: Sample the most recent posts to gauge live, real-time chatter volume and mood — the fastest-moving of the six signals, useful for spotting a niche that is breaking this week.

## Deliver

A one-page demand verdict scoring the niche on volume and momentum across every platform you selected — web search, YouTube, TikTok, Reddit, forums, and X — with a clear hot / cooling / dead call.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Is creatine gummies a growing demand or already saturated — triangulate it across platforms and give me a verdict.
