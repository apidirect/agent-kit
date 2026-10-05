---
name: crisis-amplification-tracer
description: "For a damaging tweet, map who's spreading it and how angry the room is — then check if the crisis has jumped to Reddit, the press, and forums. Use when the user asks something like \"This tweet is going viral about us — who's spreading it, how big are they, and how angry is everyone?\". Runs on the API Direct MCP tools (X, Reddit, news and web forums data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "PR, Reputation & Crisis"
  platforms: "X, Reddit, news, web forums"
  mcp-server: "https://apidirect.io/mcp"
---

# Crisis Amplification Tracer

Decomposes a viral negative tweet into its amplifiers (ranked by followers) and the sentiment of its quotes and replies, then optionally checks whether the same crisis has spilled onto Reddit, into the press, and across forums — so you can decide whether to engage, escalate, or wait it out.

**Who it's for:** Comms and PR teams triaging a live incident.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `twitter_tweet_retweets`, `twitter_tweet_quotes`, `twitter_tweet_comments`, `search_reddit`, `search_news`, `search_forums`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `tweet_id` | yes | The numeric ID of the tweet in question. | 1788888888888888888 |
| `topic` | no | The brand name or crisis keyword to trace beyond Twitter (required to enable any cross-platform spillover step). Without it, only the Twitter steps run. | Acme Corp data breach |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, reddit, news, forums. Omit to run all of them; name specific platforms to limit the run. | twitter, reddit, news, forums |
| `news_recency` | no | How far back the press check looks (maps to the news time_published enum: 1h, 1d, 7d, 1y, or anytime). Use a tight window for a live incident. Default: `1d`. | 1d |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`twitter_tweet_retweets`**

   ```
   twitter_tweet_retweets(tweet_id={tweet_id}, pages=5)
   ```

   Core step (always runs): list who amplified it; rank by followers_count to find the highest-reach spreaders.

2. **`twitter_tweet_quotes`**

   ```
   twitter_tweet_quotes(tweet_id={tweet_id}, get_sentiment=true)
   ```

   Core step (always runs): read how quote-tweets reframe it and the polarity of each.

3. **`twitter_tweet_comments`**

   ```
   twitter_tweet_comments(tweet_id={tweet_id}, get_sentiment=true)
   ```

   Core step (always runs): gauge the reply crowd's mood to decide escalate vs ignore.

4. **`search_reddit`**

   ```
   search_reddit(query={topic}, get_sentiment=true, sort_by=most_recent)
   ```

   Only run if {platforms} includes reddit and {topic} is set: check whether the crisis has jumped to Reddit threads and read the mood there, freshest first.

5. **`search_news`**

   ```
   search_news(query={topic}, time_published={news_recency})
   ```

   Only run if {platforms} includes news and {topic} is set: detect whether the press has picked up the story within {news_recency} — the single biggest escalation signal that the incident has left social media.

6. **`search_forums`**

   ```
   search_forums(query={topic}, get_sentiment=true, time=day)
   ```

   Only run if {platforms} includes forums and {topic} is set: catch niche-community spillover in the last day and its sentiment.

## Deliver

A triage brief: top amplifiers by reach, quote/reply sentiment split, cross-platform spillover status (Reddit thread mood, press pickup, forum chatter), and an escalate-or-monitor recommendation.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> This tweet is going viral about us — who's spreading it, how big are they, and how angry is everyone?
