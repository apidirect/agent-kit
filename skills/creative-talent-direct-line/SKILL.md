---
name: creative-talent-direct-line
description: "Find portfolio-grade creative talent across Instagram, TikTok, YouTube and X — and pull their public emails and portfolio links in one pass. Use when the user asks something like \"Find me brand identity designer talent near Lisbon and grab their portfolio links and emails\". Runs on the API Direct MCP tools (Instagram, TikTok, YouTube and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Recruiting & Talent"
  platforms: "Instagram, TikTok, YouTube, X"
  mcp-server: "https://apidirect.io/mcp"
---

# Creative Talent Direct Line

Creatives showcase their best work on Instagram, TikTok, YouTube and X, and many expose a business email or portfolio link right in their profile. Searching by craft on each chosen platform — then reading the Instagram profile for a public email — collapses discovery and contact into a single pass the user can scope to the networks that matter.

**Who it's for:** Creative directors, agencies, and studios hiring visual talent.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_instagram`, `search_instagram_users`, `instagram_user_profile`, `search_tiktok_users`, `search_youtube_channels`, `search_twitter_users`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `craft` | yes | The creative skill or portfolio type to search for. | brand identity designer |
| `location` | no | City or region to bias the search toward; folded into the query on every platform that runs. | Lisbon |
| `platforms` | no | Comma-separated list of platforms to run — options: instagram, tiktok, youtube, twitter. Omit to run all of them; name specific platforms to limit the run. | tiktok, youtube, twitter |
| `result_depth` | no | How many pages of results to pull per search (higher = more candidates, slower). Maps to the pages param. Defaults to 3 if omitted. | 3 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_instagram`**

   ```
   search_instagram(query="{craft} portfolio {location}", pages={result_depth})
   ```

   Base platform (always runs): surface creators showcasing the craft; keep posts with strong engagement and a clear portfolio focus.

2. **`search_instagram_users`**

   ```
   search_instagram_users(query="{craft} {location}")
   ```

   Base platform (always runs): expand the candidate pool with accounts whose handle or bio directly matches the craft.

3. **`instagram_user_profile`**

   ```
   instagram_user_profile(username=<creator_username>)
   ```

   Base platform (always runs): read bio, follower count, external_url and public_email to capture each creator's portfolio link and a direct contact.

4. **`search_tiktok_users`**

   ```
   search_tiktok_users(query="{craft} {location}", pages={result_depth})
   ```

   Only run if {platforms} includes tiktok: surface video editors, motion designers and short-form creators whose accounts match the craft; capture handles and any bio links.

5. **`search_youtube_channels`**

   ```
   search_youtube_channels(query="{craft} {location}", pages={result_depth})
   ```

   Only run if {platforms} includes youtube: find filmmakers, editors and animators by channel; many publish a business email and portfolio link in their About section.

6. **`search_twitter_users`**

   ```
   search_twitter_users(query="{craft} {location}", pages={result_depth})
   ```

   Only run if {platforms} includes twitter: surface illustrators and designers whose bio names the craft and links a portfolio or contact.

## Deliver

A contact-ready list of {craft} creatives across the chosen platforms — with portfolio links, social handles, and public emails (richest on Instagram via the profile read; often present in YouTube About sections) where available.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find me brand identity designer talent near Lisbon and grab their portfolio links and emails
