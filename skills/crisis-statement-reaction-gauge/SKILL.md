---
name: crisis-statement-reaction-gauge
description: "Measure whether your crisis statement calmed the room or poured gas on it — on X and, optionally, across Reddit, news, and forums. Use when the user asks something like \"Did our apology tweet 1789012345678901234 calm people down or make it worse? Break down the reaction and flag the loudest critics\". Runs on the API Direct MCP tools (X, Reddit, news and web forums data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "PR, Reputation & Crisis"
  platforms: "X, Reddit, news, web forums"
  mcp-server: "https://apidirect.io/mcp"
---

# Crisis-Statement Reaction Gauge

Replies and quote-tweets carry the first verdict on a public apology, but the room is bigger than one thread. Score the sentiment of the statement's own comments and quotes, then optionally fold in fresh Reddit, news, and forum reaction to learn within hours whether the wider audience is cooling off or escalating — and where the loudest detractors are concentrated.

**Who it's for:** Comms leads and PR teams managing a live incident.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `twitter_tweet_details`, `twitter_tweet_comments`, `twitter_tweet_quotes`, `search_reddit_comments`, `search_news`, `search_forums`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `tweet_id` | yes | Numeric ID of the apology or crisis-statement tweet to evaluate. | 1789012345678901234 |
| `statement_topic` | no | Brand name or short topic phrase describing the incident, used as the search query for the optional cross-platform reaction steps. Required if you enable any platform other than twitter. | Acme data breach apology |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, reddit, news, forums. Omit to run all of them; name specific platforms to limit the run. | twitter, reddit, news, forums |
| `news_window` | no | Freshness window for the news sweep, mapped to search_news time_published. Use a tight window for a live incident. Accepts 1h, 1d, 7d, 1y, or anytime. Default: `1d`. | 1d |
| `country` | no | Optional ISO country code to scope the news and forum sweeps to a single market; omit for all regions. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`twitter_tweet_details`**

   ```
   twitter_tweet_details(tweet_id={tweet_id}, get_sentiment=true)
   ```

   Record the statement's impressions, retweets, and likes to size the audience you are measuring against.

2. **`twitter_tweet_comments`**

   ```
   twitter_tweet_comments(tweet_id={tweet_id}, get_sentiment=true, pages=5)
   ```

   Compute the positive/negative/neutral split of replies and flag every reply whose dominant_emotion is anger or disgust.

3. **`twitter_tweet_quotes`**

   ```
   twitter_tweet_quotes(tweet_id={tweet_id}, get_sentiment=true, pages=5)
   ```

   Quotes signal stronger conviction, so surface the high-reach negative quote-tweets as the loudest detractors.

4. **`search_reddit_comments`**

   ```
   search_reddit_comments(query={statement_topic}, get_sentiment=true, pages=3, sort_by=most_recent)
   ```

   Only run if {platforms} includes reddit and {statement_topic} is set: pull the freshest Reddit comments reacting to the incident, add their positive/negative split to the cross-platform rollup, and quote the most upvoted critical take.

5. **`search_news`**

   ```
   search_news(query={statement_topic}, time_published={news_window}, country={country}, limit=25)
   ```

   Only run if {platforms} includes news and {statement_topic} is set: scan how the press is framing the statement (no sentiment param here, so read the headline tone) — a rising share of skeptical or negative headlines means the room is still inflamed despite a calm reply thread.

6. **`search_forums`**

   ```
   search_forums(query={statement_topic}, get_sentiment=true, time=week, country={country}, page=1)
   ```

   Only run if {platforms} includes forums and {statement_topic} is set: capture niche industry/enthusiast forum discussion and fold its sentiment into the cross-platform negative-share percentage.

## Deliver

A calmed-vs-inflamed verdict that combines the tweet's own reply and quote sentiment with optional Reddit, news, and forum reaction — including the cross-platform negative-share percentage, the top high-reach detractors, and a recommended next move.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Did our apology tweet 1789012345678901234 calm people down or make it worse? Break down the reaction and flag the loudest critics
