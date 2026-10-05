---
name: trend-to-founder-to-inbox
description: "Catch a rising topic, find the B2B founders posting about it on LinkedIn and X, then surface their contact channel. Use when the user asks something like \"Find what's trending around AI voice agents, the founders posting about it, and how I can reach them\". Runs on the API Direct MCP tools (X, LinkedIn and Google Search data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Cross-Platform Power Plays"
  platforms: "X, LinkedIn, Google Search"
  mcp-server: "https://apidirect.io/mcp"
---

# Trend-to-Founder-to-Inbox

A six-step chain across X, LinkedIn and the web: spot what's trending, find the founders riding it on LinkedIn and/or X (your choice), qualify them, dedupe, and resolve how to reach them — signal to inbox in one flow.

**Who it's for:** BD, partnerships, investors and founder-led sales.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `twitter_trends`, `search_twitter`, `search_linkedin`, `linkedin_person_posts`, `search_twitter_users`, `search_web`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `topic` | yes | The theme/space you care about. | AI voice agents |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, linkedin, web. Omit to run all of them; name specific platforms to limit the run. | linkedin, twitter |
| `region_woeid` | no | WOEID for the X trends scan (1 = worldwide). | 1 |
| `founder_sort` | no | Sort order for the LinkedIn founder search: most_recent (freshest riders) or relevance. Maps to search_linkedin sort_by. Default most_recent. | most_recent |
| `depth` | no | How many result pages to pull from the X and web searches — higher casts a wider net. Maps to the pages param. Default 2. | 2 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`twitter_trends`**

   ```
   twitter_trends(woeid={region_woeid})
   ```

   Pull live X trends; pick the ones adjacent to {topic}. Optional opener — skip if the topic is already clearly hot.

2. **`search_twitter`**

   ```
   search_twitter(query="{topic}", get_sentiment=true, sort_by=relevance, pages={depth})
   ```

   Confirm the topic is active and commerce-shaped; note the loudest voices — often founders themselves.

3. **`search_linkedin`**

   ```
   search_linkedin(author_title="Founder", query="{topic}", sort_by={founder_sort})
   ```

   Only run if {platforms} includes linkedin: find founders posting about the topic. Collect author_url for each.

4. **`linkedin_person_posts`**

   ```
   linkedin_person_posts(url=<author_url>)
   ```

   Only run if {platforms} includes linkedin: qualify each LinkedIn founder's relevance and cadence.

5. **`search_twitter_users`**

   ```
   search_twitter_users(query="{topic}", pages={depth})
   ```

   Only run if {platforms} includes twitter: find founders/builders on X whose bios match the topic; capture name + company from each profile.

6. **`search_web`**

   ```
   search_web(query="\"<founder name>\" <company> email", pages={depth})
   ```

   Resolve a contact channel for every founder gathered in steps 3-5 — first dedupe by name+company so the same founder found on both LinkedIn and X is only looked up once. For founders with a storefront, use place_details emails_and_contacts instead.

## Deliver

A deduped, ready-to-action list: trend → founders riding it (from LinkedIn and/or X) → qualification notes → a way to reach each.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find what's trending around AI voice agents, the founders posting about it, and how I can reach them.
