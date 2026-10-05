---
name: local-trades-talent-scout
description: "Map a trade's local operators and the named standout staff in their reviews, then optionally source individual performers self-promoting on Instagram and TikTok — all with direct contact. Use when the user asks something like \"Source the best barber businesses and staff in Austin, Texas with their contact info\". Runs on the API Direct MCP tools (Google Maps, Instagram and TikTok data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Recruiting & Talent"
  platforms: "Google Maps, Instagram, TikTok"
  mcp-server: "https://apidirect.io/mcp"
---

# Local Trades Talent Scout

Google Maps lists local operators with owner names and contact channels, and fresh reviews routinely name the standout employee; together they reveal both business owners worth poaching and individual performers worth recruiting. Optionally widen the search to Instagram and TikTok, where individual trades talent post portfolios and booking contacts directly — surfacing high-skill performers who never appear in a Maps listing.

**Who it's for:** Franchise developers, local employers, and trades staffing agencies.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_details`, `place_reviews`, `search_instagram_users`, `instagram_user_profile`, `search_tiktok_users`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `trade` | yes | Trade or local business type to source talent from. | barber |
| `city` | yes | City to search within. | Austin, Texas |
| `platforms` | no | Comma-separated list of platforms to run — options: places, instagram, tiktok. Omit to run all of them; name specific platforms to limit the run. | instagram, tiktok |
| `pages` | no | How many result pages to pull per search — higher means broader coverage at the cost of more API usage. Defaults to 3. | 3 |
| `review_sort` | no | How to order each business's reviews when hunting for named staff. One of newest, most_relevant, highest_ranking or lowest_ranking. Defaults to newest to surface current employees. | newest |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{trade} {city}", pages={pages})
   ```

   Build the universe of local operators; keep open businesses with strong ratings and meaningful review counts.

2. **`place_details`**

   ```
   place_details(place_id=<place_id>)
   ```

   Pull owner_name, owner_link, emails and phone numbers for direct outreach to each operator.

3. **`place_reviews`**

   ```
   place_reviews(place_id=<place_id>, sort_by={review_sort}, get_sentiment=true)
   ```

   Read reviews to find named standout staff (e.g. 'ask for Maria') and gauge sentiment about the workplace; default sort newest surfaces employees who are still there.

4. **`search_instagram_users`**

   ```
   search_instagram_users(query="{trade} {city}")
   ```

   Only run if {platforms} includes instagram: find individual {trade} performers who self-promote with portfolio accounts, surfacing high-skill talent that never appears in a Maps listing.

5. **`instagram_user_profile`**

   ```
   instagram_user_profile(username=<username>)
   ```

   Only run if {platforms} includes instagram: pull the booking email, bio links and follower count from each promising performer's profile for direct recruiting outreach.

6. **`search_tiktok_users`**

   ```
   search_tiktok_users(query="{trade} {city}", pages={pages})
   ```

   Only run if {platforms} includes tiktok: find {trade} performers building a personal brand on TikTok — high-skill individuals worth recruiting, with their handle and bio contact.

## Deliver

A sourcing list of local {trade} operators with owner contact details and named high-performing staff pulled from recent reviews, plus — when enabled — individual {trade} performers found self-promoting on Instagram and TikTok with their public contact handles, booking emails and follower counts.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Source the best barber businesses and staff in Austin, Texas with their contact info
