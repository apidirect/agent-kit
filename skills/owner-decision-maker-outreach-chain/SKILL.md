---
name: owner-decision-maker-outreach-chain
description: "Go from a map pin to the decision-maker, then open with a genuine hook pulled from their latest LinkedIn post, Google review, press mention, or Instagram. Use when the user asks something like \"For every boutique fitness studio in Denver, Colorado, find the owner's LinkedIn and draft a personalized opener from their recent posts\". Runs on the API Direct MCP tools (Google Maps, LinkedIn, news and Instagram data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Local & Places"
  platforms: "Google Maps, LinkedIn, news, Instagram"
  mcp-server: "https://apidirect.io/mcp"
---

# Owner Decision-Maker Outreach Chain

Google Maps increasingly exposes the owner's name and LinkedIn. Chaining a map pin to the owner's identity, then pulling a hook from whichever channel they are actually active on (LinkedIn posts, the freshest Google reviews, recent press, or Instagram), lets you skip the gatekeeper and open with something genuinely personal instead of a generic blast. You choose which sources to run.

**Who it's for:** Founders and AEs running high-touch 1:1 local outbound.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_details`, `linkedin_person_posts`, `place_reviews`, `search_news`, `search_instagram`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `business_type` | yes | Type of local business to target. | boutique fitness studio |
| `city` | yes | City and region to search. | Denver, Colorado |
| `platforms` | no | Comma-separated list of platforms to run — options: places, linkedin, news, instagram. Omit to run all of them; name specific platforms to limit the run. | linkedin, places, news, instagram |
| `country` | no | Two-letter region code to scope Maps results, reviews, and press to the right market. | us |
| `news_recency` | no | How fresh press mentions must be; maps to the news time window (1h, 1d, 7d, 1y, anytime). Default: `7d`. | 7d |
| `max_pages` | no | How many pages of map results to pull when seeding the outreach list (result depth). Defaults to 10. | 10 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{business_type} {city}", pages={max_pages}, country={country})
   ```

   Base step, always runs. Gather OPERATIONAL pins with place_id and website to seed the outreach list. country and max_pages are optional tuners (default pages=10).

2. **`place_details`**

   ```
   place_details(place_id=<place_id>, country={country})
   ```

   Base step, always runs. Extract business_name, owner_name, owner_link, and emails_and_contacts.linkedin, keeping only pins where an owner name or LinkedIn URL is exposed. These derived values feed every hook source below.

3. **`linkedin_person_posts`**

   ```
   linkedin_person_posts(url=<owner_linkedin_url>, page=1, get_sentiment=true)
   ```

   Only run if {platforms} includes linkedin (the default): read the owner's most recent posts to find a genuine hook (a milestone, an opinion, a frustration) and draft a one-line personalized opener.

4. **`place_reviews`**

   ```
   place_reviews(place_id=<place_id>, sort_by="newest", get_sentiment=true, country={country})
   ```

   Only run if {platforms} includes places: pull the freshest customer reviews so you can open with a specific, genuine reference (e.g. a recently praised class or a complaint to acknowledge) even when the owner has no LinkedIn presence. Near-universal coverage for local businesses.

5. **`search_news`**

   ```
   search_news(query="<owner_name> <business_name>", time_published={news_recency}, country={country})
   ```

   Only run if {platforms} includes news: surface recent press about the owner or business (an award, expansion, feature) as a congratulatory opener. news_recency defaults to a recent window so stale stories are filtered out.

6. **`search_instagram`**

   ```
   search_instagram(query="<business_name>", get_sentiment=true, pages=2)
   ```

   Only run if {platforms} includes instagram: read the studio's most recent Instagram content for a timely, on-brand hook. Strongest source for consumer-local owners (fitness, salons, restaurants) who live on IG rather than LinkedIn.

## Deliver

A per-owner dossier pairing each {business_type}'s decision-maker with a tailored cold-opener line, sourced from whichever channels the user enabled, LinkedIn posts, the freshest Google reviews, recent press, or latest Instagram content.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> For every boutique fitness studio in Denver, Colorado, find the owner's LinkedIn and draft a personalized opener from their recent posts
