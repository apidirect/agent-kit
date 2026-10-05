---
name: creator-contact-extractor
description: "Turn any niche into a contact sheet of reachable creators — with public emails and links — across Instagram, TikTok, YouTube, and X. Use when the user asks something like \"Build me an outreach contact sheet of Instagram creators in vegan meal prep with at least 10000 followers and a public email or link\". Runs on the API Direct MCP tools (Instagram, TikTok, YouTube and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Content & Influencer"
  platforms: "Instagram, TikTok, YouTube, X"
  mcp-server: "https://apidirect.io/mcp"
---

# Creator Contact Extractor

Most niche creators hide a real email or link in their bio. This sweeps Instagram, TikTok, YouTube, and X for active accounts in the niche, then reads profiles to harvest the public_email and external_url that outreach teams need, deduping everything into one sheet.

**Who it's for:** Influencer marketers and brand outreach teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_instagram_users`, `instagram_user_profile`, `search_tiktok_users`, `search_youtube_channels`, `search_twitter_users`, `twitter_user_profile`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `niche` | yes | The creator niche or topic to source contacts from. | vegan meal prep |
| `platforms` | no | Comma-separated list of platforms to run — options: instagram, tiktok, youtube, twitter. Omit to run all of them; name specific platforms to limit the run. | tiktok, youtube, twitter |
| `min_followers` | no | Minimum follower/subscriber count required to keep a creator on the sheet, applied on any platform that exposes a count. | 10000 |
| `result_depth` | no | How many pages of results to pull per optional-platform search (maps to the pages param). Higher = more creators but slower. Defaults to 2. | 2 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_instagram_users`**

   ```
   search_instagram_users(query={niche})
   ```

   Core default (always runs): shortlist verified or creator/business Instagram accounts whose handle or name fits the niche.

2. **`instagram_user_profile`**

   ```
   instagram_user_profile(username=<creator_handle>)
   ```

   Read each shortlisted Instagram handle; keep only profiles exposing a public_email or external_url and above {min_followers}, capturing category and follower count for the sheet.

3. **`search_tiktok_users`**

   ```
   search_tiktok_users(query={niche}, pages={result_depth})
   ```

   Only run if {platforms} includes tiktok: pull TikTok creators in the niche and harvest the email/link many expose in their bio, deduping against the Instagram sheet and filtering by {min_followers} where a count is present.

4. **`search_youtube_channels`**

   ```
   search_youtube_channels(query={niche}, pages={result_depth})
   ```

   Only run if {platforms} includes youtube: list YouTube channels in the niche and extract the business email/links surfaced in the channel description, filtering by {min_followers} (subscriber count) where present and deduping into the master sheet.

5. **`search_twitter_users`**

   ```
   search_twitter_users(query={niche}, pages={result_depth})
   ```

   Only run if {platforms} includes twitter: shortlist X/Twitter creators whose handle, name, or bio fits the niche, to enrich in the next step.

6. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<twitter_handle>)
   ```

   Only run if {platforms} includes twitter: read each shortlisted X profile and keep those whose bio exposes a website/contact link and clear {min_followers}, capturing follower count and merging into the deduped master sheet.

## Deliver

An outreach-ready, deduped contact sheet of niche creators across the chosen platforms (Instagram by default, plus optional TikTok, YouTube, and X) with public email/link, category, and follower/subscriber count.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me an outreach contact sheet of Instagram creators in vegan meal prep with at least 10000 followers and a public email or link.
