---
name: cross-platform-identity-resolver
description: "Pin one real person to all of their social handles and confirm the match with shared bio links and emails. Use when the user asks something like \"Find and confirm every social account belonging to Jane Rivera, who sometimes goes by janerivera\". Runs on the API Direct MCP tools (X, Instagram, Reddit, YouTube and TikTok data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "OSINT & Due Diligence"
  platforms: "X, Instagram, Reddit, YouTube, TikTok"
  mcp-server: "https://apidirect.io/mcp"
---

# Cross-Platform Identity Resolver

Anyone can claim a name; the trick is cross-confirmation. This finds candidate accounts across X, Instagram, Reddit, YouTube, and TikTok — running only the platforms you choose — then pulls a full profile to prove they're the same human via reused usernames, shared external links, and a public email.

**Who it's for:** Investigators, recruiters vetting candidates, and trust-and-safety teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_twitter_users`, `search_instagram_users`, `search_reddit_users`, `search_youtube_channels`, `search_tiktok_users`, `instagram_user_profile`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `full_name` | yes | The person's real or display name to search across platforms. | Jane Rivera |
| `known_handle` | no | A username seed they're known to reuse, used to match reused identities across platforms. | janerivera |
| `platforms` | no | Comma-separated list of platforms to run — options: twitter, instagram, reddit, youtube, tiktok. Omit to run all of them; name specific platforms to limit the run. | twitter, instagram, reddit, tiktok |
| `result_depth` | no | How many pages of candidates to scan on the platforms that support paging (X, YouTube, TikTok). Higher means more candidate accounts but slower. Defaults to 2-3 pages when unset. | 3 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_twitter_users`**

   ```
   search_twitter_users(query="{full_name}", pages={result_depth})
   ```

   Only run if {platforms} is unset or includes twitter: collect candidate X accounts, recording each handle, verified status, bio, and any linked URL. Defaults to 3 pages when {result_depth} is unset.

2. **`search_instagram_users`**

   ```
   search_instagram_users(query="{full_name}")
   ```

   Only run if {platforms} is unset or includes instagram: list candidate Instagram handles whose display name or bio overlaps the X candidates.

3. **`search_reddit_users`**

   ```
   search_reddit_users(query="{known_handle}")
   ```

   Only run if {platforms} is unset or includes reddit: match a reused username, capturing karma and account age to gauge whether the account is established or throwaway. Fall back to {full_name} as the query when no {known_handle} seed is provided.

4. **`search_youtube_channels`**

   ```
   search_youtube_channels(query="{full_name}", pages={result_depth})
   ```

   Only run if {platforms} is unset or includes youtube: find channels whose description links back to the same website or handles seen on other platforms. Defaults to 2 pages when {result_depth} is unset.

5. **`search_tiktok_users`**

   ```
   search_tiktok_users(query="{full_name}", pages={result_depth})
   ```

   Only run if {platforms} is unset or includes tiktok: surface TikTok handles matching the name or the reused {known_handle}, then compare each account's display name and bio link against the candidates already collected from the other platforms. Defaults to 2 pages when {result_depth} is unset.

6. **`instagram_user_profile`**

   ```
   instagram_user_profile(username=<candidate_handle>)
   ```

   Pull the strongest candidate's bio, external_url, and public_email to cross-confirm it is the same person behind all the handles; the shared link or email is the decisive proof tying the X, Instagram, Reddit, YouTube, and TikTok accounts together.

## Deliver

A confirmed identity map linking the person's X, Instagram, Reddit, YouTube, and TikTok accounts with the shared link/email evidence that ties them together.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find and confirm every social account belonging to Jane Rivera, who sometimes goes by janerivera.
