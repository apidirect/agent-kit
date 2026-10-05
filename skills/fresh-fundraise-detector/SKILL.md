---
name: fresh-fundraise-detector
description: "Catch founders the day they announce a round across X, LinkedIn and Reddit, rank them by real traction, and corroborate against fresh press before everyone else calls. Use when the user asks something like \"Catch startups that just announced a Series A round in climate tech this week and rank the founders by real traction\". Runs on the API Direct MCP tools (X, LinkedIn, Reddit and news data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Investing & Deal Sourcing"
  platforms: "X, LinkedIn, Reddit, news"
  mcp-server: "https://apidirect.io/mcp"
---

# Fresh Fundraise Detector

Founders broadcast closings on X, LinkedIn and Reddit before press fully lands. Cross-matching 'we raised' posts across these networks, optionally corroborating each raise against fresh news, then checking the founder's X following separates signal from noise for warm, time-sensitive intros (co-invest, follow-on, or services). A platforms toggle plus a min-followers cutoff let you choose how wide and how strict to run.

**Who it's for:** Crossover funds, co-investors, and founder-facing service providers.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_twitter`, `search_linkedin`, `search_reddit`, `search_news`, `twitter_user_profile`, `search_linkedin_companies`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `round_keywords` | yes | The round phrasing to track across every platform. | Series A |
| `sector` | no | Sector keyword to narrow the search across every platform. | climate tech |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, linkedin, reddit, news. Omit to run all of them; name specific platforms to limit the run. | twitter, linkedin, reddit, news |
| `news_window` | no | Freshness window for the optional news corroboration step (maps to search_news time_published). One of 1h, 1d, 7d, 1y, anytime. Defaults to 7d. | 7d |
| `min_followers` | no | Minimum X follower count to keep a founder when ranking by reach; founders below this are dropped. Leave blank to keep all. | 5000 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_twitter`**

   ```
   search_twitter(query="\"we raised\" {round_keywords} {sector}", sort_by=most_recent, get_sentiment=true)
   ```

   Core X scan (always runs). Keep celebratory, positive-polarity announcement tweets from the last few days and capture the founder handle.

2. **`search_linkedin`**

   ```
   search_linkedin(query="thrilled to announce {round_keywords} {sector}", author_title="Founder", sort_by=most_recent, get_sentiment=true)
   ```

   Core LinkedIn corroboration (always runs). Confirm each raise from the founder's own post, keep positive-tone announcements, and pull the company name and profile url.

3. **`search_reddit`**

   ```
   search_reddit(query="\"we raised\" {round_keywords} {sector}", sort_by=most_recent, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: Catch founders posting their raise in startup subreddits; keep positive announcement threads and the OP handle as extra leads.

4. **`search_news`**

   ```
   search_news(query="{round_keywords} {sector} funding raised", time_published={news_window})
   ```

   Only run if {platforms} includes news: Cross-confirm each social raise against fresh press to filter out vaporware and capture the official amount and lead investor.

5. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<founder_handle>)
   ```

   Read followers_count and verified to rank founders by reach and legitimacy; drop anyone below {min_followers} when it is set.

6. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query=<company_name>)
   ```

   Resolve each company to its canonical LinkedIn id/url for a clean CRM hand-off.

## Deliver

A same-week list of just-funded startups with founder handles, traction signals, optional press corroboration, and resolved company profiles ready for outreach.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Catch startups that just announced a Series A round in climate tech this week and rank the founders by real traction.
