---
name: high-reach-detractor-radar
description: "Catch only the angry brand mentions across X, Reddit and Facebook and rank them by the size of the audience that could see them. Use when the user asks something like \"Scan X for angry posts about Ryanair and rank them by how many people could see them\". Runs on the API Direct MCP tools (X, Reddit and Facebook data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Brand & Social Listening"
  platforms: "X, Reddit, Facebook"
  mcp-server: "https://apidirect.io/mcp"
---

# High-Reach Detractor Radar

Most negative posts are harmless; the dangerous ones reach big audiences. Filtering brand mentions to anger/disgust across X, Reddit and Facebook — then weighting each by author follower count (X) or thread engagement (Reddit/Facebook) — turns raw backlash into a single prioritized blast-radius triage list.

**Who it's for:** PR and social crisis teams who need to triage backlash fast.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_twitter`, `twitter_user_profile`, `twitter_tweet_details`, `twitter_tweet_comments`, `search_reddit`, `search_facebook_posts`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `brand` | yes | Brand, product, or handle to monitor for backlash. | Ryanair |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, reddit, facebook. Omit to run all of them; name specific platforms to limit the run. | twitter, reddit, facebook |
| `result_depth` | no | How many pages of results to pull per platform (maps to the pages param on the X and Facebook searches). Higher = wider net, more cost. Defaults to 10. | 10 |
| `min_followers` | no | Reach threshold for X detractors: drop authors whose follower count is below this so the triage list focuses on accounts that can actually do damage. Omit to keep all. | 5000 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_twitter`**

   ```
   search_twitter(query={brand}, pages={result_depth}, sort_by=most_recent, get_sentiment=true)
   ```

   Core X scan (always run): keep only tweets with negative polarity or dominant_emotion in anger/disgust. pages defaults to 10 if {result_depth} is omitted.

2. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<angry_author>)
   ```

   For each angry author, pull followers_count and verified status to estimate blast radius. Drop any author below {min_followers} if that input is set.

3. **`twitter_tweet_details`**

   ```
   twitter_tweet_details(tweet_id=<angry_tweet_id>, get_sentiment=true)
   ```

   Confirm the top high-reach complaints' engagement (likes, retweets) and emotional intensity before they reach the triage list.

4. **`twitter_tweet_comments`**

   ```
   twitter_tweet_comments(tweet_id=<angry_tweet_id>, get_sentiment=true)
   ```

   Check whether the reply thread is piling on or defending, then score each X mention by followers x engagement.

5. **`search_reddit`**

   ```
   search_reddit(query={brand}, sort_by=top, get_sentiment=true)
   ```

   Only run if {platforms} includes reddit: surface the highest-upvoted angry threads about {brand}, keep negative/anger ones, and use upvotes plus comment count as the reach proxy (Reddit has no follower count). Merge into the triage list.

6. **`search_facebook_posts`**

   ```
   search_facebook_posts(query={brand}, pages={result_depth}, get_sentiment=true)
   ```

   Only run if {platforms} includes facebook: pull public posts mentioning {brand}, keep negative/anger ones, and rank by post engagement as the reach proxy. Output one merged, reach-ranked crisis-triage list across every selected platform.

## Deliver

A prioritized crisis-triage list of negative brand mentions across X — and optionally Reddit and Facebook — ranked by audience reach and pile-on risk, with each X author's follower count and verification status and each Reddit/Facebook thread's engagement as a reach proxy.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Scan X for angry posts about Ryanair and rank them by how many people could see them
