---
name: hiring-signal-account-builder
description: "Turn fresh job posts that imply a tooling gap into ranked accounts, the internal budget owner, and a cross-platform talking point. Use when the user asks something like \"Find companies that posted a Revenue Operations Manager job in the last week, then find the VP Sales at each one\". Runs on the API Direct MCP tools (LinkedIn, news and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Lead Generation & Sales"
  platforms: "LinkedIn, news, X"
  mcp-server: "https://apidirect.io/mcp"
---

# Hiring-Signal Account Builder

A company hiring for a role often signals a need your product fills. This finds those fresh roles, enriches and ranks each company by ICP fit, then surfaces the function head who owns the budget. Optionally it corroborates the signal with recent company news and the buyer's latest X posts for a sharper, warmer opener.

**Who it's for:** B2B sales and demand-gen teams that sell into a specific function.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_jobs`, `linkedin_company_details`, `search_linkedin`, `search_news`, `search_twitter`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `trigger_role` | yes | A role whose hiring implies need for your product. | Revenue Operations Manager |
| `budget_owner_title` | yes | The title of the person who'd own the purchase. | VP Sales |
| `location` | no | Geo to scope to (resolve a location_id from the Job Location IDs doc). | United States |
| `posted_ago` | no | How fresh the job posts must be — maps to the job search recency window. One of 1h, 24h, 7d, 30d. Defaults to 7d. | 7d |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, news, twitter. Omit to run all of them; name specific platforms to limit the run. | news, twitter |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_jobs`**

   ```
   search_linkedin_jobs(query="{trigger_role}", posted_ago={posted_ago}, location_id=<location_id>, sort_by=most_recent)
   ```

   Get companies that posted this role within the chosen {posted_ago} window (default 7d). Collect each jobs[].company_id, company_url and company_name.

2. **`linkedin_company_details`**

   ```
   linkedin_company_details(url=<company_url>)
   ```

   Enrich each account — size, industry, specialities — and rank by ICP fit.

3. **`search_linkedin`**

   ```
   search_linkedin(author_company=<company_id>, author_title="{budget_owner_title}", sort_by=most_recent)
   ```

   Find the budget owner at each account via their own posts, giving you a warm, current talking point. Collect their name as <budget_owner_name>.

4. **`search_news`**

   ```
   search_news(query="<company_name>", time_published=7d, limit=5)
   ```

   Only run if {platforms} includes news: pull recent news on the account (funding, expansion, product launch) to corroborate the hiring signal, sharpen ICP ranking, and add a timely outreach hook.

5. **`search_twitter`**

   ```
   search_twitter(query="<budget_owner_name>", sort_by=most_recent)
   ```

   Only run if {platforms} includes twitter: pull the budget owner's recent X posts for an additional, often more candid, talking point when their LinkedIn is quiet. Note: this is a name keyword search, so disambiguate against the company/role before using a post as a hook.

## Deliver

A ranked list of accounts (with fit notes) each paired with a named budget owner and a recent post to reference — optionally enriched with corroborating company news and the buyer's latest X activity.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find companies that posted a Revenue Operations Manager job in the last week, then find the VP Sales at each one.
