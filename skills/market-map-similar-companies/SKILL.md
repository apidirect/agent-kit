---
name: market-map-similar-companies
description: "Crawl one seed's similar-companies graph on LinkedIn, then cross-fill the gaps with AI competitor lists and funding news into a full, sized market map. Use when the user asks something like \"Build me a map of Ramp's competitive landscape by expanding similar companies two hops out\". Runs on the API Direct MCP tools (LinkedIn, Google Search and news data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Competitive Intelligence"
  platforms: "LinkedIn, Google Search, news"
  mcp-server: "https://apidirect.io/mcp"
---

# Market Map via Similar-Companies Crawl

Walks LinkedIn's similar_companies edges breadth-first from one seed, enriching each with size, founding year and specialities. Optionally cross-fills the graph's blind spots with an AI competitor list (web) and a category funding-news scan (news), deduping everything into one sized landscape — with the user choosing which platforms run.

**Who it's for:** Strategy, corp-dev, competitive-intel and founders mapping a space.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_companies`, `linkedin_company_details`, `google_ai_mode`, `search_news`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `seed_company` | yes | A representative company to start the crawl from. | Ramp |
| `depth` | no | How many hops to expand the LinkedIn similar_companies graph (2 is usually plenty). | 2 |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, web, news. Omit to run all of them; name specific platforms to limit the run. | linkedin, web, news |
| `region` | no | Optional country focus for the AI competitor list and news scan (maps to the country param). Leave blank for global. | us |
| `news_window` | no | How far back to scan funding/entrant news (maps to time_published: 1h, 1d, 7d, 1y, anytime). Default: `1y`. | 1y |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query="{seed_company}", page=1)
   ```

   Resolve the seed to a company page URL.

2. **`linkedin_company_details`**

   ```
   linkedin_company_details(url=<seed_company_url>)
   ```

   Read similar_companies[], employee count, founded_year and specialities. Capture the seed's category/specialities to steer the later web and news expansion. Queue each similar company you haven't seen.

3. **`linkedin_company_details`**

   ```
   linkedin_company_details(url=<next_similar_company_url>)
   ```

   Repeat breadth-first up to {depth} hops, deduping by company. Stop when no new companies appear.

4. **`google_ai_mode`**

   ```
   google_ai_mode(prompt="List companies that compete with or are similar to {seed_company} in <seed_category>", country="{region}")
   ```

   Only run if {platforms} includes web: surface private, regional and non-LinkedIn-active players the similar_companies graph misses. For each new company name, feed it back through steps 1-3 to resolve its URL and enrich it (tag source = AI list).

5. **`search_news`**

   ```
   search_news(query="<seed_category> startup OR {seed_company} competitor funding", country="{region}", time_published="{news_window}", limit=50)
   ```

   Only run if {platforms} includes news: catch newly funded or emerging entrants in the category. Add any new company to the map flagged as a new entrant and enrich via steps 1-3 (tag source = news).

## Deliver

A deduped market table — company, size, founding year, specialities, and source (LinkedIn graph / AI list / news) — grouped into clusters, with newly funded or non-LinkedIn entrants flagged.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me a map of Ramp's competitive landscape by expanding similar companies two hops out.
