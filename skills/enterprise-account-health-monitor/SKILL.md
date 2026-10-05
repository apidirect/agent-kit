---
name: enterprise-account-health-monitor
description: "Score a key B2B account's churn risk from employee posts, rival-tool job reqs, risk-event news, and public switching chatter. Use when the user asks something like \"Is Globex at risk of churning? Check their employees' posts and whether they're hiring for Salesforce\". Runs on the API Direct MCP tools (LinkedIn, news, X and Reddit data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Product & Customer Insights"
  platforms: "LinkedIn, news, X, Reddit"
  mcp-server: "https://apidirect.io/mcp"
---

# Enterprise Account Health Monitor

Reads an account's churn risk from employee sentiment and competitor-tool job reqs on LinkedIn, then optionally layers in risk-event news and public switching chatter on X and Reddit — an early warning no CRM gives you.

**Who it's for:** Customer success and account management on enterprise books.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_companies`, `search_linkedin`, `search_linkedin_jobs`, `search_news`, `search_twitter`, `search_reddit`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `account` | yes | The customer account to monitor. | Globex |
| `competitor_tool` | no | A rival tool whose mention in job posts or public chatter signals evaluation. | Salesforce |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, news, twitter, reddit. Omit to run all of them; name specific platforms to limit the run. | news, twitter, reddit |
| `news_window` | no | Freshness window for the account risk-event news scan (maps to search_news time_published; allowed: 1h, 1d, 7d, 1y, anytime). Defaults to 7d. | 7d |
| `job_window` | no | How far back to scan job posts for competitor-tool reqs (maps to search_linkedin_jobs posted_ago; allowed: 1h, 24h, 7d, 30d). Defaults to 30d. | 30d |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query="{account}")
   ```

   Resolve the account's company_id.

2. **`search_linkedin`**

   ```
   search_linkedin(author_company=<company_id>, get_sentiment=true, sort_by=most_recent)
   ```

   Scan employee posts for frustration or 'evaluating alternatives' language (negative polarity).

3. **`search_linkedin_jobs`**

   ```
   search_linkedin_jobs(query="{competitor_tool}", company_ids=<company_id>, posted_ago={job_window})
   ```

   Job posts requiring a competitor's tool are a strong switch signal. Defaults to posted_ago=30d if job_window not set (allowed: 1h, 24h, 7d, 30d).

4. **`search_news`**

   ```
   search_news(query="{account}", time_published={news_window})
   ```

   Only run if {platforms} includes news: catch account-level risk events (layoffs, M&A, new CFO/CIO, budget cuts, restructuring) that predict churn. Defaults to time_published=7d if news_window not set.

5. **`search_twitter`**

   ```
   search_twitter(query="{account} {competitor_tool}", get_sentiment=true, sort_by=most_recent)
   ```

   Only run if {platforms} includes twitter: surface public frustration or switching chatter tying the account to the competitor (negative polarity). Falls back to just {account} if no competitor_tool.

6. **`search_reddit`**

   ```
   search_reddit(query="{account} {competitor_tool}", get_sentiment=true, sort_by=most_recent)
   ```

   Only run if {platforms} includes reddit: find threads where the account's people discuss evaluating alternatives or pain with the current tool. Blend all signals into one evidence-cited churn-risk score.

## Deliver

A health readout per account: employee-sentiment signal, competitor-tool job reqs, risk-event news, public switching chatter, and a blended churn-risk rating with evidence cited per signal.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Is Globex at risk of churning? Check their employees' posts and whether they're hiring for Salesforce.
