---
name: ugc-advocate-creator-finder
description: "Surface the happiest creators posting about your brand across Instagram, TikTok and X — handing back follower counts and contact emails. Use when the user asks something like \"Find me Instagram creators who love Glossier and give me their follower counts and emails\". Runs on the API Direct MCP tools (Instagram, TikTok and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Brand & Social Listening"
  platforms: "Instagram, TikTok, X"
  mcp-server: "https://apidirect.io/mcp"
---

# UGC Advocate & Creator Finder

Filtering branded posts to positive/joy/trust sentiment isolates genuine advocates; enriching each Instagram author's profile turns praise into a contactable, reach-ranked outreach list, while optional TikTok and X passes widen the advocate pool onto the networks where UGC actually lives.

**Who it's for:** Influencer and community marketers building a UGC and creator pipeline.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_instagram`, `instagram_user_profile`, `search_instagram_users`, `search_tiktok`, `search_twitter`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `brand` | yes | Brand name or campaign keyword (no @ or # prefix). | Glossier |
| `platforms` | no | Comma-separated list of platforms to run — options: instagram, tiktok, twitter. Omit to run all of them; name specific platforms to limit the run. | instagram, tiktok, twitter |
| `result_depth` | no | How many pages to scan per search (1-5). Higher surfaces more candidates at the cost of more calls. Maps to the pages param on every search step. Defaults to 5. | 5 |
| `region` | no | Optional country/region code to focus the TikTok advocate pass on a target market; omit for global. Only affects the TikTok step. | US |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_instagram`**

   ```
   search_instagram(query={brand}, pages={result_depth}, get_sentiment=true)
   ```

   Keep only posts whose polarity is positive or dominant_emotion is joy/trust, and collect the author usernames.

2. **`instagram_user_profile`**

   ```
   instagram_user_profile(username=<positive_author>)
   ```

   Enrich each advocate with followers, category, public_email, and external_url.

3. **`search_instagram_users`**

   ```
   search_instagram_users(query={brand})
   ```

   Find fan, affiliate, and lookalike accounts the post search missed and add their handles to the pool.

4. **`instagram_user_profile`**

   ```
   instagram_user_profile(url=<fan_account_url>)
   ```

   Enrich those accounts too, then rank everyone by followers and flag the ones with a public_email as outreach-ready.

5. **`search_tiktok`**

   ```
   search_tiktok(query={brand}, pages={result_depth}, get_sentiment=true, region={region}, sort_by=most_liked)
   ```

   Only run if {platforms} includes tiktok: pull TikTok posts about the brand, keep posts whose sentiment is positive or joy/trust, collect the creator handles, and rank them by likes as a reach proxy. Pass region only if the user supplied one, otherwise omit it for a global search.

6. **`search_twitter`**

   ```
   search_twitter(query={brand}, pages={result_depth}, get_sentiment=true, sort_by=relevance)
   ```

   Only run if {platforms} includes twitter: pull X posts about the brand, keep the positive ones, collect the highest-engagement accounts plus any bio link in their results, and merge them into the final reach-ranked outreach shortlist.

## Deliver

A reach-ranked, cross-platform shortlist of positive brand creators (Instagram plus optional TikTok and X) with follower counts, category, and a public email or bio link for direct outreach.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find me Instagram creators who love Glossier and give me their follower counts and emails
