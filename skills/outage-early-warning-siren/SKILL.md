---
name: outage-early-warning-siren
description: "Detect an incident from a corroborated cross-platform complaint surge minutes before the support queue floods. Use when the user asks something like \"Watch X and alert me the moment Figma looks like it's having an outage\". Runs on the API Direct MCP tools (X, Reddit, web forums and Google Search data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Product & Customer Insights"
  platforms: "X, Reddit, web forums, Google Search"
  mcp-server: "https://apidirect.io/mcp"
---

# Outage Early-Warning Siren

Customers complain publicly long before they open a ticket. This skill watches real-time complaint volume on X — and optionally Reddit, forums, and the open web (Downdetector-style trackers and status pages) — against normal background noise, weights it by complainer reach, and fires the siren only when a corroborated surge across the sources you enabled looks like a genuine outage rather than one-off gripes.

**Who it's for:** Support leads and on-call/reliability teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_twitter`, `search_reddit`, `search_forums`, `search_web`, `twitter_user_profile`, `twitter_tweet_comments`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `brand` | yes | Brand, app, or service name people would mention when it breaks. | Figma |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, reddit, forums, web. Omit to run all of them; name specific platforms to limit the run. | reddit, forums, web |
| `depth` | no | How many result pages to pull per source. Higher means broader coverage and a more reliable volume read, but slower. Maps to the pages/page param on each search. Default: `10`. | 10 |
| `time_window` | no | Freshness window for the forums and web sweeps (the only two sources that accept a time filter). Use a tight window like 'hour' for a true early-warning siren. Allowed values: hour, day, week, month, year, any. Default: `hour`. | hour |
| `country` | no | Optional region to narrow the forums and web sweeps when you suspect a geo-localized outage (e.g. a single cloud region). Leave blank for a global read. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_twitter`**

   ```
   search_twitter(query='{brand} down OR broken OR "not working" OR outage', sort_by=most_recent, pages={depth}, get_sentiment=true)
   ```

   Always-on real-time base. Count fresh complaints and keep only items with polarity==negative or dominant_emotion in anger/fear. Judge the rate against normal background chatter for {brand} so isolated gripes don't trip the siren, and capture the earliest complaint timestamps plus the loudest tweet_id and complainant_username for later steps.

2. **`search_reddit`**

   ```
   search_reddit(query='{brand} down OR outage OR "not working"', sort_by=most_recent, page={depth}, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: sweep fresh subreddit threads (the classic 'is {brand} down?' posts) and keep negative/anger items. A simultaneous spike here corroborating X is strong evidence of a real shared incident rather than noise.

3. **`search_forums`**

   ```
   search_forums(query='{brand} down OR outage OR "not working"', time={time_window}, country={country}, page={depth}, get_sentiment=true)
   ```

   Only run if {platforms} includes forums: pull forum threads within {time_window} reporting {brand} failures. Treat a cluster of fresh negative posts as independent corroboration of an active outage, especially for B2B/technical products whose users live on forums before they tweet.

4. **`search_web`**

   ```
   search_web(query='is {brand} down', time={time_window}, country={country}, include_ai_overview=true, pages={depth})
   ```

   Only run if {platforms} includes web: catch Downdetector-style outage trackers, third-party status reports, and aggregator pages for {brand} within {time_window}. The AI overview gives a fast human-readable 'is it down right now' verdict to gut-check the social signal.

5. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<loudest_complainant_username>)
   ```

   For the top recent complainers surfaced in step 1, pull followers_count and verified to weight blast radius by reach. A verified or high-follower complainer means a far larger audience already sees the problem, raising the alert severity.

6. **`twitter_tweet_comments`**

   ```
   twitter_tweet_comments(tweet_id=<loudest_complaint_id>, get_sentiment=true)
   ```

   Check the pile-on replies on the loudest tweet from step 1 to confirm a real shared incident (many users echoing the same failure) versus an isolated account issue, then emit the final go/no-go alert combining this with any corroborating sources you enabled.

## Deliver

A go/no-go outage alert with estimated blast radius (a cross-platform complaint surge weighted by complainer reach) plus links to the earliest complaints across every source you enabled, so on-call can confirm and respond before the support queue floods.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Watch X and alert me the moment Figma looks like it's having an outage.
