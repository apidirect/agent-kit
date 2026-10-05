---
name: competitor-conquest-radar
description: "Intercept people publicly complaining about a competitor — on LinkedIn, X and Reddit — the moment they post. Use when the user asks something like \"Find people on LinkedIn complaining about Notion this month and tell me who they are and what they're frustrated about\". Runs on the API Direct MCP tools (LinkedIn, X and Reddit data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Lead Generation & Sales"
  platforms: "LinkedIn, X, Reddit"
  mcp-server: "https://apidirect.io/mcp"
---

# Competitor Conquest Radar

Turns a competitor's name into a live feed of their unhappy users — on LinkedIn via the mentions_company filter, and optionally on X and Reddit via keyword search — using AI sentiment to keep only the negative posts, then qualifies each person before you reach out.

**Who it's for:** SDRs, founders and growth teams running competitive-displacement plays.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_companies`, `search_linkedin`, `linkedin_person_posts`, `search_twitter`, `search_reddit`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `competitor` | yes | The rival brand/company to monitor. | Notion |
| `your_product` | no | Your product, so the agent can judge fit. | Coda |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, twitter, reddit. Omit to run all of them; name specific platforms to limit the run. | linkedin, twitter, reddit |
| `sort` | no | Result ordering applied to every search step (maps to each tool's sort_by). most_recent surfaces the freshest complaints; relevance surfaces the closest keyword matches. Defaults to most_recent. | most_recent |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query="{competitor}")
   ```

   Run if {platforms} includes linkedin (the default). Resolve the competitor to its numeric LinkedIn company_id (take the best-matching result).

2. **`search_linkedin`**

   ```
   search_linkedin(mentions_company=<company_id>, get_sentiment=true, sort_by={sort})
   ```

   Only run if {platforms} includes linkedin: Pull posts that mention the competitor. Keep only posts where sentiment.polarity == "negative" (dominant_emotion anger/disgust/sadness are the hottest).

3. **`linkedin_person_posts`**

   ```
   linkedin_person_posts(url=<author_url>)
   ```

   Only run if {platforms} includes linkedin: For each promising LinkedIn complainer, read their recent posts to confirm they fit your ICP before outreach.

4. **`search_twitter`**

   ```
   search_twitter(query="{competitor}", get_sentiment=true, sort_by={sort})
   ```

   Only run if {platforms} includes twitter: Search X for public complaints about {competitor} (try variants like "{competitor} switching OR cancel OR frustrated"). Keep only negative-sentiment posts; the author handle is your prospect, and the tweet text is your why-they-fit context.

5. **`search_reddit`**

   ```
   search_reddit(query="{competitor}", get_sentiment=true, sort_by={sort})
   ```

   Only run if {platforms} includes reddit: Search Reddit for complaints about {competitor}. Keep only negative-sentiment posts; capture the username and subreddit as ICP context, and dedupe anyone already found on LinkedIn or X.

## Deliver

A ranked list of qualified prospects across LinkedIn (and optionally X/Twitter and Reddit) — name or handle, profile/post URL, what they complained about, and why they fit — newest and angriest first.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find people on LinkedIn complaining about Notion this month and tell me who they are and what they're frustrated about.
