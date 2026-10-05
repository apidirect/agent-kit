---
name: adverse-media-sweep
description: "Scan every public surface — press, web, social, video, and forums — for lawsuits, fraud claims, and negative chatter before you onboard a person or company. Use when the user asks something like \"Run an adverse-media sweep on Acme Capital LLC (crypto fund Miami) before we approve the account\". Runs on the API Direct MCP tools (news, Google Search, Reddit, X, YouTube and web forums data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "OSINT & Due Diligence"
  platforms: "news, Google Search, Reddit, X, YouTube, web forums"
  mcp-server: "https://apidirect.io/mcp"
---

# Adverse Media Sweep

KYC/EDD teams need negative-coverage evidence, not vibes. This chains dated press wires with legal-keyword web queries and sentiment-filtered Reddit, X, YouTube, and forum chatter — and lets you toggle which surfaces run and pin a jurisdiction — so red flags surface even when they never made the news.

**Who it's for:** Compliance, KYC/AML, and investor due-diligence teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_news`, `search_web`, `search_reddit`, `search_twitter`, `search_youtube`, `search_forums`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `subject_name` | yes | The person or legal entity being screened. | Acme Capital LLC |
| `context_terms` | no | Disambiguating context (industry, city, or aliases) to avoid same-name false positives. | crypto fund Miami |
| `platforms` | no | Comma-separated list of platforms to run — options: news, web, reddit, twitter, youtube, forums. Omit to run all of them; name specific platforms to limit the run. | reddit, twitter, youtube, forums |
| `country` | no | ISO country code to focus jurisdiction-specific news, web, and forum results (maps to the real country param on those tools). Omit for a global sweep. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_news`**

   ```
   search_news(query="{subject_name} {context_terms}", time_published=1y, limit=50, country={country})
   ```

   Backbone (always run): pull the last year of press coverage and keep articles framed around legal, regulatory, or financial-misconduct language. Pass country={country} only when screening a specific jurisdiction.

2. **`search_web`**

   ```
   search_web(query="{subject_name} lawsuit OR fraud OR investigation OR settlement OR scam", time=year, country={country})
   ```

   Backbone (always run): surface court filings, regulator notices, and complaint sites that never reach mainstream wires. Pass country={country} only when screening a specific jurisdiction.

3. **`search_reddit`**

   ```
   search_reddit(query="{subject_name}", sort_by=top, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: keep top threads whose polarity is negative or dominant_emotion is anger/disgust to catch customer or investor grievances.

4. **`search_twitter`**

   ```
   search_twitter(query="{subject_name}", sort_by=most_recent, pages=5, get_sentiment=true)
   ```

   Only run if {platforms} includes twitter: flag recent negatively-polarized mentions and complaint threads, noting accounts repeating the same allegation.

5. **`search_youtube`**

   ```
   search_youtube(query="{subject_name} scam OR fraud OR lawsuit OR exposed", upload_date=this_year, get_sentiment=true)
   ```

   Only run if {platforms} includes youtube: capture investigative or expose videos and scam warnings; retain negatively-toned clips and note channels that show corroborating documents or filings.

6. **`search_forums`**

   ```
   search_forums(query="{subject_name}", time=year, get_sentiment=true, country={country})
   ```

   Only run if {platforms} includes forums: capture niche-forum scam reports and warnings, retaining only negative-polarity posts with corroborating detail. Pass country={country} only when screening a specific jurisdiction.

## Deliver

A consolidated adverse-media memo ranking negative findings by severity, each tied to a dated source link (press, web, social, or video) for the KYC/EDD file.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Run an adverse-media sweep on Acme Capital LLC (crypto fund Miami) before we approve the account.
