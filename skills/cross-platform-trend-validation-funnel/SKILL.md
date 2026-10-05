---
name: cross-platform-trend-validation-funnel
description: "Promote a topic only when it surges at once across X, Reddit, YouTube, TikTok, Instagram and the news — killing one-platform fads. Use when the user asks something like \"Validate whether tinned fish recipes is genuinely surging across platforms before I build content around it\". Runs on the API Direct MCP tools (X, Reddit, YouTube, TikTok, Instagram and news data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Market Research & Trends"
  platforms: "X, Reddit, YouTube, TikTok, Instagram, news"
  mcp-server: "https://apidirect.io/mcp"
---

# Cross-Platform Trend Validation Funnel

Single-platform spikes are often noise. Requiring fresh momentum across up to six independent surfaces — live X trends, hot Reddit, this-week YouTube supply, 7-day TikTok traction, fresh Instagram posts, and independent news coverage — filters real surges from manufactured ones. You choose which platforms count, and the verdict promotes a topic only when a clear majority agree.

**Who it's for:** Content strategists, trend investors, and brand marketers.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `twitter_trends`, `search_reddit`, `search_youtube`, `search_tiktok`, `search_instagram`, `search_news`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `topic` | yes | The topic, hashtag concept, or keyword to validate. | tinned fish recipes |
| `trend_woeid` | no | WOEID for the live X trend check (1 = Worldwide). | 1 |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, reddit, youtube, tiktok, instagram, news. Omit to run all of them; name specific platforms to limit the run. | reddit, youtube, tiktok, instagram, news |
| `region` | no | Country/region code used to scope TikTok results and news coverage. Leave blank for global. | US |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`twitter_trends`**

   ```
   twitter_trends(woeid={trend_woeid})
   ```

   Anchor gate (always runs): check whether {topic} or a close variant appears in the live X trend list and capture its tweet volume as the first momentum signal.

2. **`search_reddit`**

   ```
   search_reddit(query={topic}, sort_by=hot, page=1, get_sentiment=true)
   ```

   Only run if {platforms} is unset or includes reddit: confirm Reddit communities are actively surfacing {topic} in hot right now and gauge whether chatter is positive or skeptical.

3. **`search_youtube`**

   ```
   search_youtube(query={topic}, upload_date=this_week, get_sentiment=true)
   ```

   Only run if {platforms} is unset or includes youtube: measure fresh creator supply this week; a rising count of new uploads signals real production interest, not a one-off.

4. **`search_tiktok`**

   ```
   search_tiktok(query={topic}, publish_time=7, sort_by=most_liked, region={region}, get_sentiment=true)
   ```

   Only run if {platforms} is unset or includes tiktok: confirm short-form traction in the last 7 days (optionally scoped to {region}) and keep only if top videos show recent high-like velocity.

5. **`search_instagram`**

   ```
   search_instagram(query={topic}, get_sentiment=true)
   ```

   Only run if {platforms} is unset or includes instagram: check whether {topic} is generating fresh posts on the major visual platform — strong corroboration for consumer, food, and lifestyle trends — and read the sentiment.

6. **`search_news`**

   ```
   search_news(query={topic}, time_published=7d, country={region})
   ```

   Only run if {platforms} is unset or includes news: confirm independent editorial coverage in the last 7 days (optionally scoped to {region}) — the hardest signal to manufacture — so the verdict isn't built on social hype alone.

## Deliver

A go/no-go verdict that promotes {topic} only when a clear majority of the selected platforms — spanning social, video, visual, and editorial media — show fresh, rising momentum in the same window, not a single isolated spike.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Validate whether tinned fish recipes is genuinely surging across platforms before I build content around it
