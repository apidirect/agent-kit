---
name: ai-serp-reputation-audit
description: "See what Google's AI tells the public about your brand — then trace that narrative back to the Reddit, news, and forum sources you can actually fix. Use when the user asks something like \"What does Google's AI say about API Direct, and which sources is it pulling that from?\". Runs on the API Direct MCP tools (Google Search, Reddit, news and web forums data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "PR, Reputation & Crisis"
  platforms: "Google Search, Reddit, news, web forums"
  mcp-server: "https://apidirect.io/mcp"
---

# AI-SERP Reputation Audit

Captures Google's AI Overview and AI Mode answer about your brand, then traces the narrative back to the cited domains and — optionally — the Reddit threads, news articles, and forum posts feeding it (with sentiment), so you can fix the actual inputs. Choose which source platforms to trace and which market to audit.

**Who it's for:** PR, brand and SEO teams managing how AI describes them.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_web`, `google_ai_mode`, `search_reddit`, `search_news`, `search_forums`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `brand` | yes | The brand or person to audit. | API Direct |
| `platforms` | no | Comma-separated list of platforms to run — options: web, reddit, news, forums. Omit to run all of them; name specific platforms to limit the run. | reddit, news, forums |
| `country` | no | Optional. Two-letter country code to localize the AI answer and source searches — AI Overviews differ by market. Omit to use the default market. | us |
| `freshness` | no | Optional. How far back to pull cited news sources (maps to news time_published). Omit for all-time. Default: `7d`. | 7d |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_web`**

   ```
   search_web(query="{brand} reviews", include_ai_overview=true, country={country})
   ```

   Read ai_overview.text_parts (the public summary) and collect ai_overview.reference_links. Pass {country} to localize the Overview if the user set it.

2. **`google_ai_mode`**

   ```
   google_ai_mode(prompt="What is the reputation of {brand}? Note complaints and praise.", country={country})
   ```

   Get the fuller conversational answer plus its reference_links citations. Merge these citation domains with the Overview's into one <cited_domains> set.

3. **`search_web`**

   ```
   search_web(query="site:<cited_domain> {brand}")
   ```

   For each domain in <cited_domains>, open the exact pages driving the AI's narrative — these are the pages to fix or pitch.

4. **`search_reddit`**

   ```
   search_reddit(query="{brand}", get_sentiment=true, sort_by=relevance)
   ```

   Only run if {platforms} includes reddit: Google's AI Overview frequently cites Reddit on reputation queries — pull the actual threads shaping (or about to shape) the narrative, with sentiment, even if they weren't in this week's citations.

5. **`search_news`**

   ```
   search_news(query="{brand}", country={country}, time_published={freshness})
   ```

   Only run if {platforms} includes news: Surface the news articles the AI cites or is likely to cite next. Use {freshness} (e.g. 7d) to bound recency; omit for anytime.

6. **`search_forums`**

   ```
   search_forums(query="{brand}", get_sentiment=true, country={country}, time=month)
   ```

   Only run if {platforms} includes forums: Pull forum and review-site threads commonly cited in AI answers, with sentiment, to find feeding inputs beyond the cited domains.

## Deliver

A report: what Google's AI says about the brand, the sources it cites, and the specific pages to fix or pitch — optionally enriched with the Reddit threads, news articles, and forum posts driving the narrative, scored for sentiment.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> What does Google's AI say about API Direct, and which sources is it pulling that from?
