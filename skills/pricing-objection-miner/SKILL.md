---
name: pricing-objection-miner
description: "Harvest authentic, sentiment-scored pricing complaints about a rival across the platforms you choose to arm your sales battlecard. Use when the user asks something like \"Build me a pricing battlecard against HubSpot from real customer complaints\". Runs on the API Direct MCP tools (Reddit, web forums, X, YouTube and LinkedIn data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Competitive Intelligence"
  platforms: "Reddit, web forums, X, YouTube, LinkedIn"
  mcp-server: "https://apidirect.io/mcp"
---

# Pricing Objection Miner

Buyers vent about a competitor's pricing in public long before they tell a salesperson. Pulling verbatim, negatively-charged gripes across Reddit, forums, X, YouTube, and LinkedIn — on whichever platforms you select — gives you the exact language to disarm objections. LinkedIn and YouTube add the B2B and review-video voice that pure social search misses.

**Who it's for:** Sales enablement, product marketing, and competitive battlecard owners.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_reddit`, `search_forums`, `search_twitter`, `search_youtube`, `search_linkedin`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `competitor` | yes | The rival product or company whose pricing complaints you want to mine. | HubSpot |
| `platforms` | no | Comma-separated list of platforms to run — options: reddit, forums, twitter, youtube, linkedin. Omit to run all of them; name specific platforms to limit the run. | reddit, forums, twitter, youtube, linkedin |
| `time_window` | no | Forum lookback window to keep complaints fresh. One of any, day, hour, month, week, year. Defaults to year. | year |
| `depth` | no | How many result pages to pull on platforms that paginate (X and YouTube). Higher = more verbatim quotes, slower. Defaults to 2. | 2 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_reddit`**

   ```
   search_reddit(query="{competitor} pricing", sort_by=top, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: keep posts where dominant_emotion is anger or disgust and capture the why behind the cost frustration.

2. **`search_forums`**

   ```
   search_forums(query="{competitor} expensive", time={time_window}, get_sentiment=true)
   ```

   Only run if {platforms} includes forums: pull long-form niche threads about hidden fees, seat costs, and surprise renewals; keep polarity=negative.

3. **`search_twitter`**

   ```
   search_twitter(query="{competitor} too expensive", sort_by=most_recent, pages={depth}, get_sentiment=true)
   ```

   Only run if {platforms} includes twitter: grab recent public gripes to confirm the complaints are current, not stale.

4. **`search_reddit`**

   ```
   search_reddit(query="{competitor} cancel OR switched OR alternative", sort_by=top, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: isolate switching triggers tied to price for the battlecard's 'why they leave' section.

5. **`search_youtube`**

   ```
   search_youtube(query="{competitor} pricing review", upload_date=this_year, pages={depth}, get_sentiment=true)
   ```

   Only run if {platforms} includes youtube: surface 'why I cancelled' and pricing-review videos; keep negative sentiment to capture the objection narrative buyers actually watch before they push back.

6. **`search_linkedin`**

   ```
   search_linkedin(query="{competitor} pricing", sort_by=relevance, get_sentiment=true)
   ```

   Only run if {platforms} includes linkedin: B2B buyers and consultants publicly critique seat costs and renewal hikes here — keep negative posts and capture the business rationale, which is the highest-value language for a B2B battlecard.

## Deliver

A battlecard section of verbatim, sentiment-scored pricing objections grouped by theme (seat costs, hidden fees, lock-in) with the switching triggers, drawn from whichever of Reddit, forums, X, YouTube, and LinkedIn you choose to run.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me a pricing battlecard against HubSpot from real customer complaints
