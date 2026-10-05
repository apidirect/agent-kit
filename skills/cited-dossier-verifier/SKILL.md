---
name: cited-dossier-verifier
description: "Generate a subject dossier, then independently re-verify every AI-stated claim against its primary source and optional first-hand social corroboration. Use when the user asks something like \"Build me a source-verified dossier on Northwind Logistics Inc focused on regulatory actions and litigation\". Runs on the API Direct MCP tools (Google Search, news, Reddit and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "OSINT & Due Diligence"
  platforms: "Google Search, news, Reddit, X"
  mcp-server: "https://apidirect.io/mcp"
---

# Cited Dossier Verifier

AI summaries hallucinate, which is fatal for due diligence. This grabs an AI-mode dossier with its citations, then re-queries each cited domain and dated press coverage — and optionally Reddit and X for independent first-hand corroboration — to confirm or quarantine every claim before it lands in your file. You control which corroboration channels run, the subject's jurisdiction, and how far back the press window reaches.

**Who it's for:** Analysts and investigators who need defensible, source-traced dossiers.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `google_ai_mode`, `search_web`, `search_news`, `search_reddit`, `search_twitter`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `subject` | yes | The person or company the dossier is about. | Northwind Logistics Inc |
| `focus` | no | What to emphasize in the dossier. | regulatory actions and litigation |
| `platforms` | no | Comma-separated list of platforms to run — options: web, news, reddit, twitter. Omit to run all of them; name specific platforms to limit the run. | reddit, twitter |
| `country` | no | Two-letter country code to bias the AI dossier and news toward the subject's home jurisdiction (maps to the country param on google_ai_mode and search_news). Useful for jurisdiction-specific regulatory and legal claims. | US |
| `news_window` | no | How far back to pull dated press coverage for corroboration (maps to search_news time_published). One of 1h, 1d, 7d, 1y, anytime. Defaults to 1y. | 1y |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`google_ai_mode`**

   ```
   google_ai_mode(prompt="Summarize controversies, lawsuits, regulatory actions and key facts about {subject}, focused on {focus}", country={country})
   ```

   Capture each claim from reply_parts and the supporting reference_links so every assertion has a candidate citation. Pass {country} to bias the answer toward the subject's home jurisdiction when set.

2. **`search_web`**

   ```
   search_web(query="{subject}", include_ai_overview=true)
   ```

   Run a second independent AI overview plus organic results; compare its ai_overview.reference_links against the first pass to catch contradictions.

3. **`search_web`**

   ```
   search_web(query="site:<citation_domain> {subject}")
   ```

   For each cited domain from steps 1 and 2, query the original source directly to confirm the claim is actually stated there and not fabricated.

4. **`search_news`**

   ```
   search_news(query="{subject}", time_published={news_window}, limit=30, country={country})
   ```

   Corroborate or contradict the AI-stated claims against dated press coverage, flagging anything unsupported. {news_window} defaults to 1y; set {country} for jurisdiction-specific coverage.

5. **`search_reddit`**

   ```
   search_reddit(query="{subject}", sort_by=relevance)
   ```

   Only run if {platforms} includes reddit: independently corroborate or contradict each AI claim against community discussion, surfacing first-hand accounts and contradictions the press missed before promoting any claim.

6. **`search_twitter`**

   ```
   search_twitter(query="{subject}", sort_by=relevance)
   ```

   Only run if {platforms} includes twitter: cross-check claims against real-time public statements from the subject, accusers, and journalists; quarantine any claim no independent voice supports.

## Deliver

A fully-cited dossier where every claim is traced to and confirmed against its primary source plus any independent social corroboration you enabled, with unverifiable assertions clearly quarantined.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me a source-verified dossier on Northwind Logistics Inc focused on regulatory actions and litigation.
