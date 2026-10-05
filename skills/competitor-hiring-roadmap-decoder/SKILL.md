---
name: competitor-hiring-roadmap-decoder
description: "Reverse-engineer a rival's unannounced roadmap from the roles they just opened — then cross-check it against the news and open web. Use when the user asks something like \"Decode what Ramp is quietly building next quarter from their open roles\". Runs on the API Direct MCP tools (LinkedIn, news and Google Search data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Competitive Intelligence"
  platforms: "LinkedIn, news, Google Search"
  mcp-server: "https://apidirect.io/mcp"
---

# Competitor Hiring Roadmap Decoder

Companies telegraph their next 2-3 quarters through who they hire before they ship. By clustering a rival's freshest job posts by function and reading the JDs — then optionally cross-checking against news and the open web — you infer the initiatives behind the headcount and flag which are still hidden versus already surfacing publicly.

**Who it's for:** Founders, product leaders, and competitive strategy teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_companies`, `search_linkedin_jobs`, `linkedin_job_details`, `search_news`, `google_ai_mode`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `competitor` | yes | The rival company's name to resolve to a LinkedIn company page. | Ramp |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, news, web. Omit to run all of them; name specific platforms to limit the run. | linkedin, news, web |
| `posted_ago` | no | Freshness window for the 'net-new roles' job search (maps to the LinkedIn jobs posted_ago param). Defaults to 7d. One of 1h, 24h, 7d, 30d. | 7d |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query={competitor}, page=1)
   ```

   Take the top match's numeric company_id to use as the jobs filter.

2. **`search_linkedin_jobs`**

   ```
   search_linkedin_jobs(company_ids=<company_id>, posted_ago={posted_ago}, sort_by=most_recent)
   ```

   Capture net-new roles in the chosen freshness window (defaults to 7d if {posted_ago} is omitted) and group titles by function (eng, sales, ops, finance).

3. **`search_linkedin_jobs`**

   ```
   search_linkedin_jobs(company_ids=<company_id>, posted_ago=30d, sort_by=most_recent)
   ```

   Establish a 30-day baseline to see which functions are accelerating versus steady-state — this velocity diff is the core ranking signal.

4. **`linkedin_job_details`**

   ```
   linkedin_job_details(url=<job_url>)
   ```

   Read the responsibilities of each net-new role type to name the underlying product or GTM initiative behind the headcount.

5. **`search_news`**

   ```
   search_news(query={competitor}, time_published=7d)
   ```

   Only run if {platforms} includes news: pull the rival's last month of coverage (funding, launches, exec hires, partnerships) and tag each inferred initiative as still-hidden or already corroborated publicly.

6. **`google_ai_mode`**

   ```
   google_ai_mode(prompt=What new products, features, or markets is {competitor} building toward right now, and what is the evidence?)
   ```

   Only run if {platforms} includes web: cross-confirm and source-trace the inferred roadmap against blogs, changelogs, and public chatter, flagging any initiative the hiring signal missed.

## Deliver

A roadmap brief mapping the rival's fresh hires into 3-5 likely product and GTM initiatives, ranked by hiring velocity, with each initiative tagged as still-hidden or already corroborated by public news/web signals.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Decode what Ramp is quietly building next quarter from their open roles
