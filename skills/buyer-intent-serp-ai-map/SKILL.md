---
name: buyer-intent-serp-ai-map
description: "Map the affiliate, comparison, AI-answer, and community landscape that owns a high-intent buying keyword. Use when the user asks something like \"Map who owns the buyer-intent landscape for best project management software 2026 across Google results and AI Overview\". Runs on the API Direct MCP tools (Google Search, Reddit and YouTube data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Market Research & Trends"
  platforms: "Google Search, Reddit, YouTube"
  mcp-server: "https://apidirect.io/mcp"
---

# Buyer-Intent SERP and AI-Overview Map

For a "best X" query, the ranking pages plus Google's AI Overview reveal who controls the purchase decision and which sources the AI trusts, and comparing classic SERP to AI Mode shows where consensus is consolidating. Optional Reddit and YouTube layers add the community-recommended and creator-reviewed picks buyers cross-check before they decide, completing the picture of who owns the buying keyword.

**Who it's for:** SEO, affiliate, and product-marketing teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_web`, `google_ai_mode`, `search_reddit`, `search_youtube`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `product` | yes | The product or category buyers are comparing. | project management software |
| `year` | no | Year to anchor the buying query for freshness. | 2026 |
| `platforms` | no | Comma-separated list of platforms to run — options: web, reddit, youtube. Omit to run all of them; name specific platforms to limit the run. | reddit, youtube |
| `region` | no | Optional two-letter country code to localize the SERP and AI answers to a specific buying market (applies to the web and AI Mode steps; Reddit and YouTube run globally). Omit for default localization. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_web`**

   ```
   search_web(query="best {product} {year}", pages=3, include_ai_overview=true, country={region})
   ```

   Core step, always runs. Capture the organic ranking domains plus ai_overview.text_parts and ai_overview.reference_links to see who owns the SERP and which sources feed the AI summary. Pass country={region} only if region is set.

2. **`search_web`**

   ```
   search_web(query="{product} alternatives", pages=2, include_ai_overview=true, country={region})
   ```

   Core step, always runs. Pull the alternatives landscape to map which competitors are positioned against each other and which review sites recur. Pass country={region} only if region is set.

3. **`google_ai_mode`**

   ```
   google_ai_mode(prompt="What is the best {product} and why?", country={region})
   ```

   Core step, always runs. Capture reply_parts and reference_links to get Google's conversational pick and compare its cited sources against the SERP reference_links. Pass country={region} only if region is set.

4. **`google_ai_mode`**

   ```
   google_ai_mode(prompt="Compare the top {product} options on price and features", country={region})
   ```

   Core step, always runs. Extract the head-to-head framing the AI uses and flag any vendor that appears in AI answers but not the organic top results as an emerging mover. Pass country={region} only if region is set.

5. **`search_reddit`**

   ```
   search_reddit(query="best {product}", sort_by="top", get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: pull the top community threads where real users recommend a 'best {product}'. Tally which vendors recur and their sentiment to see which picks the community trusts, then cross-check them against the SERP and AI Mode winners to spot consensus or divergence.

6. **`search_youtube`**

   ```
   search_youtube(query="best {product} {year}", upload_date="this_year", get_sentiment=true)
   ```

   Only run if {platforms} includes youtube: surface the creators owning the video review/comparison landscape for this buying keyword this year. Note which vendors the top reviews feature and the overall tone, and flag any creator-favored vendor that the organic SERP and AI answers underweight.

## Deliver

A landscape map of the domains, vendors, and — when enabled — the Reddit communities and YouTube creators that control a buying keyword across organic SERP, Google's AI answers, and community channels, plus the sources the AI consistently cites and any vendor that appears in AI/community answers but not the organic top results.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Map who owns the buyer-intent landscape for best project management software 2026 across Google results and AI Overview
