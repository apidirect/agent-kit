---
name: pre-call-account-dossier
description: "Walk into every discovery call already knowing the company, its latest moves across web and news, and the buyer's hot buttons. Use when the user asks something like \"Build me a pre-call dossier on Acme Corp before my discovery call with their Head of Growth\". Runs on the API Direct MCP tools (Google Search, news and LinkedIn data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Lead Generation & Sales"
  platforms: "Google Search, news, LinkedIn"
  mcp-server: "https://apidirect.io/mcp"
---

# Pre-Call Account Dossier

A great discovery call is won in the prep. This skill chains a cited AI brief, recent strategic moves from both web and news, the company's own LinkedIn posts, and the buyer's personal LinkedIn activity into a one-page dossier with ready-made rapport hooks. Pick which sources run and how far back to look.

**Who it's for:** AEs and founders prepping for discovery and demo calls.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `google_ai_mode`, `search_web`, `search_news`, `search_linkedin_companies`, `linkedin_company_posts`, `search_linkedin`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `company` | yes | The account you're about to call. | Acme Corp |
| `persona_title` | no | Title of the person you'll be on the call with; used to filter the buyer's personal LinkedIn posts. | Head of Growth |
| `platforms` | no | Comma-separated list of platforms to run — options: web, news, linkedin. Omit to run all of them; name specific platforms to limit the run. | web, news, linkedin |
| `freshness` | no | How far back the web moves search looks (maps to search_web time). Defaults to month. | month |
| `country` | no | Two-letter country code to localize the AI brief, web, and news results for non-US accounts. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`google_ai_mode`**

   ```
   google_ai_mode(prompt="Summarize {company}: business model, recent news, and likely pain points", country={country})
   ```

   Always run — the synthesis backbone. Capture a cited brief from reply_parts and reference_links as the dossier spine, localized via {country}.

2. **`search_web`**

   ```
   search_web(query="{company} funding OR partnership OR product launch", include_ai_overview=true, time={freshness}, country={country})
   ```

   Only run if {platforms} includes web: pull recent strategic moves plus the AI overview citations for fresh talking points, scoped to {freshness}.

3. **`search_news`**

   ```
   search_news(query="{company} funding OR partnership OR launch OR executive", time_published=7d, country={country})
   ```

   Only run if {platforms} includes news: pull authoritative recent press for citeable, dateable moves that confirm or sharpen the web findings.

4. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query="{company}")
   ```

   Only run if {platforms} includes linkedin: resolve the account to its numeric company_id and LinkedIn URL for the next two steps.

5. **`linkedin_company_posts`**

   ```
   linkedin_company_posts(url=<company_url>)
   ```

   Only run if {platforms} includes linkedin: read the company's own recent posts for the priorities and language they use about themselves.

6. **`search_linkedin`**

   ```
   search_linkedin(author_company=<company_id>, author_title={persona_title}, sort_by=most_recent, get_sentiment=true)
   ```

   Only run if {platforms} includes linkedin and a {persona_title} is given: surface what your buyer is personally posting and feeling to mine specific rapport hooks.

## Deliver

A one-page pre-call dossier covering the company's model, recent moves (web + news), stated priorities, and personalized rapport hooks for your {persona_title} — scoped to the sources you choose via {platforms}.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me a pre-call dossier on Acme Corp before my discovery call with their Head of Growth
