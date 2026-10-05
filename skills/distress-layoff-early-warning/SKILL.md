---
name: distress-layoff-early-warning
description: "Detect competitor instability from a surge of 'open to work' employees, cross-checked against X, Reddit, layoff forums and the news wire. Use when the user asks something like \"Is there any sign Acme Corp is in trouble or doing layoffs right now?\". Runs on the API Direct MCP tools (LinkedIn, X, Reddit, web forums and news data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Competitive Intelligence"
  platforms: "LinkedIn, X, Reddit, web forums, news"
  mcp-server: "https://apidirect.io/mcp"
---

# Distress & Layoff Early-Warning

Anchors on a rival's LinkedIn employee 'open to work'/layoff posts, then corroborates across your choice of X, Reddit, layoff forums and the news wire — useful for displacement, poaching, or de-risking a deal.

**Who it's for:** Competitive intel, recruiters, and investors tracking a watchlist.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_companies`, `search_linkedin`, `search_twitter`, `search_reddit`, `search_forums`, `search_news`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `competitor` | yes | The company to watch for distress. | Acme Corp |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, twitter, reddit, forums, news. Omit to run all of them; name specific platforms to limit the run. | twitter, reddit, forums, news |
| `news_window` | no | How far back to pull news. One of: 1h, 1d, 7d, 1y, anytime. Defaults to 7d. | 7d |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query="{competitor}")
   ```

   Always run: resolve the company_id.

2. **`search_linkedin`**

   ```
   search_linkedin(author_company=<company_id>, query="\"open to work\" OR \"impacted\" OR \"laid off\"", get_sentiment=true, sort_by=most_recent)
   ```

   Always run (core signal): count employees posting departure/availability signals; a spike + sadness/anger sentiment is the tell.

3. **`search_twitter`**

   ```
   search_twitter(query="{competitor} layoffs", get_sentiment=true, sort_by=most_recent)
   ```

   Only run if {platforms} includes twitter: corroborate with public X chatter and gauge mood.

4. **`search_reddit`**

   ```
   search_reddit(query="{competitor} layoffs OR laid off", get_sentiment=true, sort_by=most_recent)
   ```

   Only run if {platforms} includes reddit: r/layoffs and company-specific subreddits are where employees vent first; sentiment confirms severity.

5. **`search_forums`**

   ```
   search_forums(query="{competitor} layoffs", get_sentiment=true, time=month)
   ```

   Only run if {platforms} includes forums: layoff-discussion forums often carry insider threads before the news wire picks it up.

6. **`search_news`**

   ```
   search_news(query="{competitor} layoffs", time_published="{news_window}")
   ```

   Only run if {platforms} includes news: confirm against reporting; lookback defaults to the last 7 days via {news_window}.

## Deliver

A short risk readout: signal count per platform, trend vs baseline, representative posts/threads/articles, and a confidence call.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Is there any sign Acme Corp is in trouble or doing layoffs right now?
