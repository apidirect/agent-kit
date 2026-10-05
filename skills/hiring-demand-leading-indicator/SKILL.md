---
name: hiring-demand-leading-indicator
description: "Turn fresh job-posting velocity for an emerging skill into a forward demand signal — triangulated with news and practitioner chatter — and a map of who is investing. Use when the user asks something like \"Track weekly hiring velocity for RAG engineer in 103644278 (United States) and tell me which companies are investing ahead of the market\". Runs on the API Direct MCP tools (LinkedIn, news and Reddit data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Market Research & Trends"
  platforms: "LinkedIn, news, Reddit"
  mcp-server: "https://apidirect.io/mcp"
---

# Hiring-as-Demand Leading Indicator

Companies hire ahead of building. Counting fresh, last-7-day LinkedIn job posts for an emerging skill against a 30-day baseline reveals hiring velocity, and profiling the posting companies maps who is betting before the market notices. Optional news and Reddit passes confirm the same demand from the corporate-investment and practitioner angles, so you act on accelerating trends, not single-platform noise.

**Who it's for:** VCs, market analysts, and competitive-intel teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_jobs`, `search_linkedin_companies`, `linkedin_company_details`, `search_news`, `search_reddit`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `emerging_skill` | yes | The skill, role, or technology to track as a demand proxy. | RAG engineer |
| `location_id` | no | Numeric LinkedIn location id to scope the hiring market (resolve via the linkedin-job-locations doc). | 103644278 (United States) |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, news, reddit. Omit to run all of them; name specific platforms to limit the run. | news, reddit |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_jobs`**

   ```
   search_linkedin_jobs(query={emerging_skill}, posted_ago=7d, sort_by=most_recent, location_id={location_id})
   ```

   Core: count fresh postings this week and capture the hiring companies as the leading-edge demand signal.

2. **`search_linkedin_jobs`**

   ```
   search_linkedin_jobs(query={emerging_skill}, posted_ago=30d, sort_by=most_recent, location_id={location_id})
   ```

   Core: pull the 30-day window as a baseline so the 7-day count becomes a velocity / acceleration ratio.

3. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query=<top hiring company name>)
   ```

   Core: resolve each leading employer's name from the job results to its numeric company id and company url.

4. **`linkedin_company_details`**

   ```
   linkedin_company_details(url=<company url>)
   ```

   Core: profile the biggest investors in the skill (size, founded year, specialities, similar_companies) to map who is betting and which adjacent firms to watch.

5. **`search_news`**

   ```
   search_news(query={emerging_skill}, time_published=7d)
   ```

   Only run if {platforms} includes news: corroborate the corporate-investment axis — surface this week's funding rounds, hiring announcements, and product launches around the skill to confirm which companies are betting and whether momentum is building.

6. **`search_reddit`**

   ```
   search_reddit(query={emerging_skill}, sort_by=most_recent, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: capture the grassroots practitioner-demand axis — 'who's hiring', adoption, and learning threads with sentiment — as an independent confirmation that demand is accelerating beyond corporate hiring.

## Deliver

A weekly demand-velocity readout for the skill, a ranked list of the companies hiring hardest against it, and the adjacent firms likely to follow — plus, when enabled, corroborating corporate-investment (news) and practitioner-demand (Reddit) signals confirming whether the trend is real and accelerating.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Track weekly hiring velocity for RAG engineer in 103644278 (United States) and tell me which companies are investing ahead of the market
