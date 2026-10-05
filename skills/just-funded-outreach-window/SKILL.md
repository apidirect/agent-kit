---
name: just-funded-outreach-window
description: "Catch companies the week they raise — when budgets are fresh and buyers are saying yes. Use when the user asks something like \"Find fintech companies that raised a Series A this week and surface the VP of Engineering I should reach out to at each one\". Runs on the API Direct MCP tools (news, Google Search, X and LinkedIn data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Lead Generation & Sales"
  platforms: "news, Google Search, X, LinkedIn"
  mcp-server: "https://apidirect.io/mcp"
---

# Just-Funded Outreach Window

A new funding round means a freshly-approved budget and a 90-day spending window. This skill scans this-week funding signals across news (and optionally web and X) and turns them into named decision-makers with a congrats-on-the-raise hook before competitors notice. You choose which discovery channels run and how far back to look.

**Who it's for:** B2B founders and AEs selling into venture-backed companies.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_news`, `search_web`, `search_twitter`, `search_linkedin_companies`, `search_linkedin`, `linkedin_company_posts`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `sector` | yes | Your target market or vertical to watch for raises. | fintech |
| `funding_stage` | no | The funding round to monitor. | Series A |
| `persona_title` | yes | Job title of the budget-holding buyer to surface at each funded company. | VP of Engineering |
| `platforms` | no | Comma-separated list of platforms to run — options: news, web, twitter, linkedin. Omit to run all of them; name specific platforms to limit the run. | news, web, twitter |
| `freshness` | no | Optional. How far back to scan funding news (maps to the news time_published window). Defaults to 7d. | 7d |
| `country` | no | Optional. Restrict discovery to a single market (maps to the news and web country param). Omit for global coverage. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_news`**

   ```
   search_news(query="raises {funding_stage} {sector}", time_published={freshness}, country={country}, limit=50)
   ```

   Always run. Collect every company that publicly announced a raise in the funding news index and extract each company name. Defaults: freshness 7d; omit country for global coverage.

2. **`search_web`**

   ```
   search_web(query="{sector} raises {funding_stage} funding", time=week, country={country}, include_ai_overview=true)
   ```

   Only run if {platforms} includes web: Catch raises the news index misses (Crunchbase, TechCrunch, PR wires) and extract additional just-funded company names; align time with your freshness window and use the AI overview to dedupe against the news list.

3. **`search_twitter`**

   ```
   search_twitter(query="{sector} \"we raised\" {funding_stage}", sort_by=most_recent, pages=2)
   ```

   Only run if {platforms} includes twitter: Catch founders announcing their raise directly on X — often hours before press — and extract the company name. Feeds the same buyer-persona flow below, so it complements rather than duplicates founder-centric sibling skills.

4. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query=<company_name>)
   ```

   Resolve each freshly-funded company name gathered from the active discovery channels to its numeric company_id and LinkedIn URL, deduping companies that appeared on more than one channel.

5. **`search_linkedin`**

   ```
   search_linkedin(author_company=<company_id>, author_title={persona_title}, sort_by=most_recent)
   ```

   Surface the budget-holding {persona_title} at the funded account and what they are posting about right now.

6. **`linkedin_company_posts`**

   ```
   linkedin_company_posts(url=<company_url>)
   ```

   Pull the company's own recent posts to craft a funding-anchored opener tied to their stated roadmap.

## Deliver

A deduped, ranked list of just-funded accounts in {sector} (assembled from your chosen discovery channels), each with the named decision-maker and a tailored, funding-anchored opening line.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find fintech companies that raised a Series A this week and surface the VP of Engineering I should reach out to at each one
