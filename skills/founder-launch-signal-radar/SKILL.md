---
name: founder-launch-signal-radar
description: "Surface founders announcing launches across LinkedIn, X, and Reddit — filtered to founders, then graded for conviction and traction. Use when the user asks something like \"Show me founders who just announced a launch in AI agents, with how much traction each seems to have\". Runs on the API Direct MCP tools (LinkedIn, X and Reddit data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Investing & Deal Sourcing"
  platforms: "LinkedIn, X, Reddit"
  mcp-server: "https://apidirect.io/mcp"
---

# Founder Launch-Signal Radar

Starts with the canonical LinkedIn author_title=Founder filter plus launch language, then fans the same intent out to X and Reddit, where founders also announce shipping. Confirms each poster really is a founder, grades conviction sentiment, and pulls platform-native traction signals — LinkedIn posting cadence, X follower reach, Reddit engagement. The 'author param' play, now multi-platform and user-tunable.

**Who it's for:** Seed investors, BD, and anyone who wants to reach founders at launch.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin`, `search_twitter`, `search_reddit`, `linkedin_person_posts`, `linkedin_post_details`, `twitter_user_profile`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `topic` | no | Optional space/keyword to focus on. Applied to every platform's search. | AI agents |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, twitter, reddit. Omit to run all of them; name specific platforms to limit the run. | linkedin, twitter, reddit |
| `sort` | no | Sort order for the discovery searches: most_recent (default, best for a fresh radar) or relevance. Valid on all three search tools. | most_recent |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin`**

   ```
   search_linkedin(author_title="Founder", query="{topic} (\"just launched\" OR \"introducing\" OR \"we shipped\")", sort_by={sort})
   ```

   Core play: only posts authored by Founders announcing something. Runs by default; skip only if {platforms} is set and excludes linkedin. Collect each author_url and post_url.

2. **`search_twitter`**

   ```
   search_twitter(query="{topic} (\"just launched\" OR introducing OR \"we shipped\" OR \"go live\")", sort_by={sort}, get_sentiment=true)
   ```

   Only run if {platforms} includes twitter: founders announce launches on X even more than on LinkedIn. Collect each tweet's author username and tweet URL. X has no author-title filter, so founder status is confirmed in step 6.

3. **`search_reddit`**

   ```
   search_reddit(query="{topic} (\"just launched\" OR \"I built\" OR introducing OR \"feedback on my\")", sort_by={sort}, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: catches founders self-posting launches in startup subreddits (r/SaaS, r/startups, r/SideProject). get_sentiment grades excitement; upvotes and comment counts are the traction signal.

4. **`linkedin_person_posts`**

   ```
   linkedin_person_posts(url=<author_url>)
   ```

   Only for LinkedIn hits from step 1: check the founder's posting cadence and any traction signals.

5. **`linkedin_post_details`**

   ```
   linkedin_post_details(url=<post_url>, get_sentiment=true)
   ```

   Only for LinkedIn hits from step 1: grade conviction/excitement and confirm it's an original post, not a repost.

6. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<author_username>)
   ```

   Only run if {platforms} includes twitter: pull the poster's bio to confirm they're actually a founder (replacing the missing author-title filter) and their follower count as the X traction signal.

## Deliver

A cross-platform, newest-first feed of founders who just launched — name, company, what they shipped, a conviction grade, and platform-native traction signals (LinkedIn cadence, X follower reach, Reddit upvotes/comments).

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Show me founders who just announced a launch in AI agents, with how much traction each seems to have.
