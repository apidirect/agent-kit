---
name: support-deflection-gap-finder
description: "Find the questions users keep asking that no doc, video, or AI answer deflects — across forums, Reddit, X and YouTube. Use when the user asks something like \"Find the Airtable questions people keep asking that we have no tutorial for\". Runs on the API Direct MCP tools (web forums, Reddit, X, YouTube and Google Search data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Product & Customer Insights"
  platforms: "web forums, Reddit, X, YouTube, Google Search"
  mcp-server: "https://apidirect.io/mcp"
---

# Support Deflection Gap-Finder

Every repeated "how do I" with no good doc, video, or search answer is a ticket you'll keep paying for. This skill mines high-frequency help-seeking across forums, Reddit and X, then cross-checks coverage on YouTube and the open web (including Google's AI overview) to expose where a single piece of content would deflect the most support volume. Use the optional platforms input to choose which surfaces run, and tune the demand window, region, and result depth.

**Who it's for:** Support and knowledge-base content teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_forums`, `search_reddit_comments`, `search_twitter`, `search_youtube`, `search_web`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `product` | yes | Product whose help-content gaps you want to find. | Airtable |
| `platforms` | no | Comma-separated list of platforms to run — options: forums, reddit, twitter, youtube, web. Omit to run all of them; name specific platforms to limit the run. | forums, reddit, twitter, youtube, web |
| `demand_window` | no | Recency window for help-seeking demand and web coverage checks (maps to the forums and web time param). Defaults to year. | year |
| `region` | no | Optional country code to localize demand and coverage for region-specific products (maps to forums and web country). Leave blank for global. | us |
| `depth` | no | How many result pages to scan per surface (maps to the pages param on reddit/twitter/youtube). Higher = more questions and tutorials sampled. Default: `3`. | 3 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_forums`**

   ```
   search_forums(query='{product} "how do i" OR "how to" OR "cant figure out"', time={demand_window}, country={region}, get_sentiment=true)
   ```

   Collect recurring how-to questions; flag ones tagged confusion/fear as the most painful tasks. Set {demand_window} (default year) and optional {region} to localize.

2. **`search_reddit_comments`**

   ```
   search_reddit_comments(query='how do i {product}', sort_by=top, pages={depth}, get_sentiment=true)
   ```

   Add Reddit help-seeking comments and cluster everything by the underlying task. {depth} controls how many pages to scan.

3. **`search_twitter`**

   ```
   search_twitter(query='{product} "how do i" OR "how do you" OR "how to"', sort_by=relevance, pages={depth}, get_sentiment=true)
   ```

   Only run if {platforms} includes twitter: capture real-time help-seeking tweets and merge them into the task clusters; negative sentiment marks the most painful gaps.

4. **`search_youtube`**

   ```
   search_youtube(query='{product} tutorial how to', pages={depth})
   ```

   Count existing tutorials per clustered task; tasks with few or zero quality videos are content gaps.

5. **`search_web`**

   ```
   search_web(query='<top_demand_task> {product} how to', time={demand_window}, country={region}, include_ai_overview=true)
   ```

   Only run if {platforms} includes web: check whether a good doc, blog, or help-center page already ranks — and whether Google's AI overview already answers it. Strong existing coverage means lower deflection ROI; thin or missing coverage confirms the gap.

6. **`search_youtube`**

   ```
   search_youtube(query='<top_demand_task> {product}')
   ```

   Confirm the single highest-demand question genuinely has no good video answer before prioritizing it; combine with the web coverage read to rank by deflection potential.

## Deliver

A prioritized list of doc/video topics where user demand is high (forums, Reddit, and X) but tutorial and search coverage is missing or thin (YouTube plus open web and Google's AI overview), ranked by deflection potential.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find the Airtable questions people keep asking that we have no tutorial for.
