---
name: tweet-amplifier-outreach-list
description: "Turn one viral tweet — plus optional Instagram and LinkedIn creators on the same topic — into a reach-ranked amplifier list for your launch. Use when the user asks something like \"Pull a reach-ranked list of everyone who amplified tweet 1750112233445566778 so I can line up launch boosters above 5000 followers\". Runs on the API Direct MCP tools (X, Instagram and LinkedIn data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Content & Influencer"
  platforms: "X, Instagram, LinkedIn"
  mcp-server: "https://apidirect.io/mcp"
---

# Amplifier Outreach List

A single viral tweet is a pre-qualified list of people who already love your message. This reconstructs its retweet, quote, and reply graph, keeps the friendly-sentiment branches, enriches engagers, and reach-ranks them into an outreach list. Optionally widen the pool with enthusiastic Instagram and LinkedIn creators posting about the same topic, choose which platforms run, and tune how deep the crawl goes.

**Who it's for:** Founders and growth marketers planning a launch.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `twitter_tweet_retweets`, `twitter_tweet_quotes`, `twitter_tweet_comments`, `twitter_user_profile`, `search_instagram`, `search_linkedin`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `tweet_id` | yes | Numeric id of the viral tweet to mine for amplifiers. | 1750112233445566778 |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, instagram, linkedin. Omit to run all of them; name specific platforms to limit the run. | twitter, instagram, linkedin |
| `topic` | no | Keyword or theme of the tweet/launch, used to find on-topic amplifiers on Instagram and LinkedIn. Required only if those platforms are enabled. | AI meeting notetaker |
| `min_followers` | no | Minimum follower/reach count for an amplifier to make the final list. | 5000 |
| `depth` | no | How many pages to crawl per source (higher = more candidates, slower). Defaults to a shallow crawl. Applies to the Twitter retweet/quote/comment crawl and the Instagram search; LinkedIn search returns a single page. | 10 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`twitter_tweet_retweets`**

   ```
   twitter_tweet_retweets(tweet_id={tweet_id}, pages={depth})
   ```

   Collect every account that retweeted the post as raw amplifier candidates.

2. **`twitter_tweet_quotes`**

   ```
   twitter_tweet_quotes(tweet_id={tweet_id}, pages={depth}, get_sentiment=true)
   ```

   Add quote-tweeters, keeping only positive or neutral polarity, and union them with the retweeters while deduping.

3. **`twitter_tweet_comments`**

   ```
   twitter_tweet_comments(tweet_id={tweet_id}, pages={depth}, get_sentiment=true)
   ```

   Fold in the most engaged, favorably-toned repliers as warm amplifier prospects.

4. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<amplifier_username>)
   ```

   Enrich each amplifier with followers_count and verified status, drop anyone below {min_followers}, and rank the list by reach.

5. **`search_instagram`**

   ```
   search_instagram(query={topic}, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes instagram: widen the pool with Instagram creators posting positively about {topic}, keep positive/neutral sentiment, dedupe against names already found, and rank by author reach.

6. **`search_linkedin`**

   ```
   search_linkedin(query={topic}, get_sentiment=true, sort_by=relevance, page=1)
   ```

   Only run if {platforms} includes linkedin: add high-engagement LinkedIn voices posting favorably about {topic} as B2B amplifiers, keep positive sentiment, dedupe, and merge into the single reach-ranked list.

## Deliver

A reach-ranked, sentiment-filtered, deduped list of launch amplifiers — built from the tweet's engagement graph and, optionally, on-topic Instagram and LinkedIn creators.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Pull a reach-ranked list of everyone who amplified tweet 1750112233445566778 so I can line up launch boosters above 5000 followers.
