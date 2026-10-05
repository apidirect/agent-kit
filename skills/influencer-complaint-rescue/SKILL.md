---
name: influencer-complaint-rescue
description: "Catch a high-follower complaint on Instagram, X, or TikTok and get a private contact to fix it before it spreads. Use when the user asks something like \"Surface influential Instagram users trashing Glossier so we can reach out and fix it privately\". Runs on the API Direct MCP tools (Instagram, X and TikTok data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Product & Customer Insights"
  platforms: "Instagram, X, TikTok"
  mcp-server: "https://apidirect.io/mcp"
---

# Influencer Complaint Rescue

A negative post from a 50k-follower account does more damage than a hundred from nobodies. This skill finds angry posts about your brand across Instagram, X, and TikTok, ranks the authors by reach, and pulls their public contact (email, bio link, or website) so support can resolve it privately before it goes viral.

**Who it's for:** Brand/social support and reputation teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_instagram`, `instagram_user_profile`, `search_twitter`, `twitter_user_profile`, `search_tiktok`, `search_tiktok_users`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `brand` | yes | Brand name to monitor for influential negative posts (no @ or # prefix). | Glossier |
| `platforms` | no | Comma-separated list of platforms to run — options: instagram, twitter, tiktok. Omit to run all of them; name specific platforms to limit the run. | twitter, tiktok |
| `min_followers` | no | Optional minimum follower count. Complainers below this reach are dropped so the team focuses on accounts that can actually do damage. | 10000 |
| `region` | no | Optional TikTok region code to localize the complaint search to a single market. Applies only to the TikTok step. | US |
| `depth` | no | Optional number of result pages to pull per search platform (Instagram, X, and TikTok). Raise to widen the net on a noisy brand. Default: `2`. | 2 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_instagram`**

   ```
   search_instagram(query='{brand} disappointed OR refund OR scam OR broken OR "worst" OR "never again"', get_sentiment=true, pages={depth})
   ```

   Core complaint sweep. One combined query covers both mild and serious complaint language. Keep only posts with polarity==negative or dominant_emotion in anger/sadness. {depth} defaults to 1 page if not set.

2. **`instagram_user_profile`**

   ```
   instagram_user_profile(username=<top_negative_ig_author>)
   ```

   Fetch followers, category, public_email and external_url to rank complainers by reach and find a private contact channel. If {min_followers} is set, keep only authors at or above that follower count.

3. **`search_twitter`**

   ```
   search_twitter(query='{brand} disappointed OR refund OR scam OR "worst" OR "never again"', get_sentiment=true, sort_by='most_recent', pages={depth})
   ```

   Only run if {platforms} includes twitter: catch fresh angry tweets about the brand. Keep only polarity==negative posts. {depth} defaults to 1 page if not set.

4. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<top_negative_tweet_author>)
   ```

   Only run if {platforms} includes twitter: pull follower count and bio/website link to rank the loudest complainer by reach and find a contact channel. Apply the {min_followers} threshold if set.

5. **`search_tiktok`**

   ```
   search_tiktok(query='{brand} disappointed OR refund OR scam OR "worst"', get_sentiment=true, sort_by='most_recent', publish_time=30, pages={depth}, region={region})
   ```

   Only run if {platforms} includes tiktok: surface complaint videos from the last 30 days. Keep only polarity==negative. Pass region={region} only if set, otherwise omit for a global search. {depth} defaults to 1 page if not set.

6. **`search_tiktok_users`**

   ```
   search_tiktok_users(query=<top_negative_tiktok_author>)
   ```

   Only run if {platforms} includes tiktok: look up the complainer's TikTok profile for follower count and bio contact. Apply the {min_followers} threshold if set.

## Deliver

A cross-platform triage list of negative posts (Instagram by default, plus optional X and TikTok) ranked by author reach, each paired with the complainer's public email or bio contact channel so support can reach out and resolve privately before it spreads.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Surface influential Instagram users trashing Glossier so we can reach out and fix it privately.
