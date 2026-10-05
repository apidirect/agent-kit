---
name: switcher-harvesting-engine
description: "Find the people actively shopping away from a rival across Reddit, X, forums and LinkedIn, and intercept them mid-defection. Use when the user asks something like \"Find people switching away from Mailchimp in email marketing and qualify the highest-reach ones for me\". Runs on the API Direct MCP tools (Reddit, X, web forums and LinkedIn data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Lead Generation & Sales"
  platforms: "Reddit, X, web forums, LinkedIn"
  mcp-server: "https://apidirect.io/mcp"
---

# Switcher Harvesting Engine

Frustrated customers announce their exit publicly before they pick a replacement. This skill harvests those defectors across Reddit, X, community forums and LinkedIn, filters for genuine negative intent, and vets each one so you reach out while they're still deciding. An optional platforms input lets you choose which channels to sweep.

**Who it's for:** Competitive-displacement sales and growth teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_reddit`, `search_reddit_users`, `search_twitter`, `twitter_user_profile`, `search_forums`, `search_linkedin`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `competitor` | yes | The rival product your prospects are leaving. | Mailchimp |
| `category` | no | Product category to keep the search on-topic. | email marketing |
| `platforms` | no | Comma-separated list of platforms to run — options: reddit, twitter, forums, linkedin. Omit to run all of them; name specific platforms to limit the run. | reddit, twitter, forums, linkedin |
| `recency` | no | Freshness window for the forums sweep (maps to the forums time param). Defaults to month if unset. | month |
| `min_followers` | no | Minimum X follower count to keep a switcher in the reach-scored list, used to set the influence bar during vetting. | 1000 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_reddit`**

   ```
   search_reddit(query="alternative to {competitor}", sort_by=hot, get_sentiment=true)
   ```

   Keep negative-polarity posts where someone is actively seeking a {category} replacement, and grab each poster's username.

2. **`search_reddit_users`**

   ```
   search_reddit_users(query=<reddit_username>)
   ```

   Vet karma and account age to drop throwaway and bot accounts before outreach.

3. **`search_twitter`**

   ```
   search_twitter(query="switching from {competitor}", pages=3, sort_by=most_recent, get_sentiment=true)
   ```

   Capture live defection tweets and keep ones with negative polarity directed at {competitor}.

4. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<twitter_handle>)
   ```

   Pull followers_count and verified status to prioritize high-influence, on-ICP switchers; drop anyone below {min_followers} when it is set.

5. **`search_forums`**

   ```
   search_forums(query="alternative to {competitor}", get_sentiment=true, time={recency})
   ```

   Only run if {platforms} includes forums: sweep niche community forums for users seeking a {category} replacement for {competitor}; keep negative-polarity threads and default the time window to month if {recency} is unset.

6. **`search_linkedin`**

   ```
   search_linkedin(query="switching off {competitor}", mentions_company={competitor}, get_sentiment=true, sort_by=most_recent)
   ```

   Only run if {platforms} includes linkedin: capture B2B buyers publicly announcing a move off {competitor}; keep negative-sentiment posts and prioritize authors whose company fits your ICP for warm, named outreach.

## Deliver

A vetted, reach-scored list of high-intent switchers leaving {competitor} across your chosen channels (Reddit, X, forums, LinkedIn), each with the source post link for warm outreach.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find people switching away from Mailchimp in email marketing and qualify the highest-reach ones for me
