---
name: layoff-wave-interceptor
description: "Catch freshly laid-off talent across LinkedIn, X and Reddit within days of the announcement, before every other recruiter circles back. Use when the user asks something like \"Find people just laid off in fintech who are open to work as a Backend Engineer and give me a shortlist to reach out to today\". Runs on the API Direct MCP tools (news, LinkedIn, X and Reddit data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Recruiting & Talent"
  platforms: "news, LinkedIn, X, Reddit"
  mcp-server: "https://apidirect.io/mcp"
---

# Layoff Wave Interceptor

News breaks layoffs faster than social does; by resolving each affected company to its numeric LinkedIn company_id and scanning fresh 'open to work' / 'impacted' posts -- and optionally extending the sweep to X and Reddit, where many people announce availability before updating LinkedIn -- you reach displaced candidates while they are still on the market and before the inbound rush.

**Who it's for:** In-house recruiters and staffing agencies racing to place displaced talent.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_news`, `search_linkedin_companies`, `search_linkedin`, `linkedin_person_posts`, `search_twitter`, `search_reddit`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `sector` | yes | Industry or sector to monitor for layoff announcements. | fintech |
| `role` | yes | Target job title to prioritize among the impacted employees. | Backend Engineer |
| `platforms` | no | Comma-separated list of platforms to run — options: news, linkedin, twitter, reddit. Omit to run all of them; name specific platforms to limit the run. | linkedin, twitter, reddit |
| `freshness` | no | How far back to scan layoff news, mapped to search_news time_published. One of 1h, 1d, 7d, 1y, anytime. Defaults to 7d. | 7d |
| `country` | no | Optional ISO country code to geo-narrow the layoff news scan (search_news country param) for region-specific recruiting. Omit for global. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_news`**

   ```
   search_news(query="{sector} layoffs", time_published={freshness}, country={country}, limit=50)
   ```

   Identify companies that announced workforce reductions within {freshness} and extract each affected company name. Pass {country} only when a region is specified; default time_published to 7d if {freshness} is omitted.

2. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query=<company name>)
   ```

   Resolve each affected company from the news into its numeric LinkedIn company_id.

3. **`search_linkedin`**

   ```
   search_linkedin(query="open to work", author_company=<company_id>, author_title={role}, get_sentiment=true, sort_by=most_recent)
   ```

   Surface impacted employees who just announced availability; keep the most recent posts matching the target role.

4. **`linkedin_person_posts`**

   ```
   linkedin_person_posts(url=<person_url>, get_sentiment=true)
   ```

   Confirm the person is genuinely displaced and currently active before adding them to the outreach list.

5. **`search_twitter`**

   ```
   search_twitter(query="{sector} laid off open to work {role}", sort_by=most_recent, get_sentiment=true)
   ```

   Only run if {platforms} includes twitter: catch displaced {role} talent announcing layoffs or availability on X, where many post days before updating LinkedIn. Keep the most recent, negatively-tilted-but-active posts.

6. **`search_reddit`**

   ```
   search_reddit(query="laid off {sector} {role} open to work", sort_by=most_recent, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: surface freshly laid-off {role} candidates posting in layoff and career communities for an extra sourcing channel beyond LinkedIn.

## Deliver

A time-ranked shortlist of freshly displaced {role} candidates from companies that just announced {sector} layoffs, drawn from LinkedIn and (optionally) X and Reddit, with profile/post links and recency signals.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find people just laid off in fintech who are open to work as a Backend Engineer and give me a shortlist to reach out to today
