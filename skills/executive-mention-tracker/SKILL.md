---
name: executive-mention-tracker
description: "Catch every mention of a named leader across LinkedIn, X, news, Reddit, and optionally YouTube and Facebook, and split the wins from the reputation risks. Use when the user asks something like \"Track everything being said about Jane Okafor (https://www.linkedin.com/in/janeokafor) this week and flag any reputation risks before they spread\". Runs on the API Direct MCP tools (LinkedIn, X, news, Reddit, YouTube and Facebook data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "PR, Reputation & Crisis"
  platforms: "LinkedIn, X, news, Reddit, YouTube, Facebook"
  mcp-server: "https://apidirect.io/mcp"
---

# Executive Mention Tracker

An executive's reputation is shaped by what others say, not what they post. This watches mentions of a named leader across up to six networks at once and sorts them into wins to amplify and risks to get ahead of. You choose which surfaces to scan and how far back to look.

**Who it's for:** Comms teams and chiefs of staff protecting a named executive.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin`, `search_twitter`, `search_news`, `search_reddit`, `search_youtube`, `search_facebook_posts`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `exec_name` | yes | Full name of the executive to monitor. | Jane Okafor |
| `exec_linkedin_url` | yes | LinkedIn profile URL or slug used to match LinkedIn mentions of the person. | https://www.linkedin.com/in/janeokafor |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, twitter, news, reddit, youtube, facebook. Omit to run all of them; name specific platforms to limit the run. | youtube, facebook |
| `news_window` | no | How far back to pull press mentions, mapped to search_news time_published. One of 1h, 1d, 7d, 1y, anytime. Defaults to 7d for a weekly brief; widen for a deeper retrospective. | 7d |
| `pages` | no | Result depth (number of pages) to pull per paginated social surface (X, YouTube, Facebook). Higher values catch more low-engagement chatter at the cost of speed. Defaults to 4. | 4 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin`**

   ```
   search_linkedin(mentions_member={exec_linkedin_url}, get_sentiment=true, sort_by=most_recent)
   ```

   Pull posts that tag the exec and split positive endorsements (wins) from negative posts (risk).

2. **`search_twitter`**

   ```
   search_twitter(query="{exec_name}", get_sentiment=true, sort_by=most_recent, pages={pages})
   ```

   Capture public chatter and flag anger/negative items with high engagement for review. Tune {pages} to control how deep the sweep goes.

3. **`search_news`**

   ```
   search_news(query="{exec_name}", time_published={news_window}, limit=30)
   ```

   Surface earned/press mentions over the chosen {news_window} (default 7d) and note outlet tone and reach.

4. **`search_reddit`**

   ```
   search_reddit(query="{exec_name}", get_sentiment=true, sort_by=most_recent)
   ```

   Pull candid community threads and flag negative or anger-dominant posts as emerging risks.

5. **`search_youtube`**

   ```
   search_youtube(query="{exec_name}", get_sentiment=true, upload_date=this_week)
   ```

   Only run if {platforms} includes youtube: catch interviews, conference talks, commentary and criticism videos that name the exec, and flag negative, high-view videos as reputation risks.

6. **`search_facebook_posts`**

   ```
   search_facebook_posts(query="{exec_name}", get_sentiment=true, pages={pages})
   ```

   Only run if {platforms} includes facebook: surface public Facebook chatter about the exec and flag negative posts as emerging risks.

## Deliver

A weekly executive-reputation brief that sorts every mention across the chosen networks into wins, neutral, and risks, with the highest-reach negatives (including YouTube videos and Facebook posts when enabled) flagged for response.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Track everything being said about Jane Okafor (https://www.linkedin.com/in/janeokafor) this week and flag any reputation risks before they spread
