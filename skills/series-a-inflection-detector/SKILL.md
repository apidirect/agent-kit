---
name: series-a-inflection-detector
description: "Find startups hiring their first GTM/finance exec — then confirm the raise across news and X. Use when the user asks something like \"Find startups that just posted their first VP Sales or Head of Finance role and look recently funded\". Runs on the API Direct MCP tools (LinkedIn, news and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Investing & Deal Sourcing"
  platforms: "LinkedIn, news, X"
  mcp-server: "https://apidirect.io/mcp"
---

# Series-A Inflection Detector

The first 'VP Sales' or 'Head of Finance' req is a reliable signal a startup just raised and is scaling. This finds those roles on LinkedIn, confirms the company is young and small, and (optionally) cross-confirms a fresh raise with funding news and the founder's own announcement on X.

**Who it's for:** VCs, growth investors and vendors who sell into newly-scaling startups.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_jobs`, `linkedin_company_details`, `linkedin_company_posts`, `search_news`, `search_twitter`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `first_exec_roles` | no | The first-exec titles that signal scaling. | "VP Sales" OR "Head of Finance" OR "first sales hire" |
| `location` | no | Geo to scope the job search to (resolve a location_id). | San Francisco Bay Area |
| `posted_ago` | no | How fresh the first-exec job posting must be. Maps to the search_linkedin_jobs posted_ago enum (1h, 24h, 7d, 30d). Tighten to 7d to catch reqs within days of a raise. Default: `30d`. | 30d |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, news, twitter. Omit to run all of them; name specific platforms to limit the run. | linkedin, news, twitter |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_jobs`**

   ```
   search_linkedin_jobs(query="{first_exec_roles}", posted_ago={posted_ago}, location_id=<location_id>, sort_by=most_recent)
   ```

   Core signal — find recent first-exec postings. Collect each company_id, company_url and company_name for the steps below.

2. **`linkedin_company_details`**

   ```
   linkedin_company_details(url=<company_url>)
   ```

   Keep only young, small companies (low employee count, recent founded_year) — that's the post-Series-A profile. Capture company_name for the news/X cross-checks.

3. **`linkedin_company_posts`**

   ```
   linkedin_company_posts(url=<company_url>)
   ```

   Skim recent company posts to confirm momentum / a funding mention.

4. **`search_news`**

   ```
   search_news(query="<company_name> (\"Series A\" OR funding OR raised)", time_published=7d, limit=10)
   ```

   Only run if {platforms} includes news: independently confirm a fresh raise via funding coverage. Attach the article as proof and to time the outreach window.

5. **`search_twitter`**

   ```
   search_twitter(query="<company_name> (\"Series A\" OR raised OR funding)", sort_by=most_recent, get_sentiment=true)
   ```

   Only run if {platforms} includes twitter: catch the founder's own funding/traction announcement on X to confirm the raise and supply a warm cold-open hook.

## Deliver

A shortlist of likely just-raised startups with the role that flagged them, quick firmographics, and — when news/X are enabled — a linked funding article or founder announcement that confirms the raise and times the outreach.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find startups that just posted their first VP Sales or Head of Finance role and look recently funded.
