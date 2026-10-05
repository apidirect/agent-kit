---
name: competitor-employee-poacher
description: "List a named competitor's employees in a target role across LinkedIn, X and Instagram — sourced from their own profiles, not the company page. Use when the user asks something like \"Find Staff Engineers who work at Stripe and post on LinkedIn, with a personalized angle for each\". Runs on the API Direct MCP tools (LinkedIn, X and Instagram data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Recruiting & Talent"
  platforms: "LinkedIn, X, Instagram"
  mcp-server: "https://apidirect.io/mcp"
---

# Competitor Employee Poacher

Uses LinkedIn author_company + author_title to surface the people who actually work at a rival in the exact role you're hiring, then optionally widens to X and Instagram bios to catch in-role talent that's quiet on LinkedIn — a live, qualifiable, multi-channel sourcing list with a per-person outreach hook.

**Who it's for:** Recruiters and founders sourcing senior or hard-to-find talent.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_companies`, `search_linkedin`, `linkedin_person_posts`, `search_twitter_users`, `twitter_user_profile`, `search_instagram_users`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `competitor` | yes | The company to source talent from. | Stripe |
| `target_role` | yes | The role/title you're hiring for. | Staff Engineer |
| `platforms` | no | Comma-separated list of platforms to run — options: linkedin, twitter, instagram. Omit to run all of them; name specific platforms to limit the run. | linkedin, twitter |
| `industry` | no | Optional LinkedIn industry filter (maps to author_industry) to disambiguate a generic or multi-vertical competitor name and tighten results. | Software Development |
| `sort_order` | no | Order for the LinkedIn employee search (maps to sort_by). Use most_recent to favor currently-active employees or relevance for best role match. Defaults to most_recent. | most_recent |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query="{competitor}")
   ```

   Resolve the competitor to its numeric company_id.

2. **`search_linkedin`**

   ```
   search_linkedin(author_company=<company_id>, author_title="{target_role}", author_industry="{industry}", sort_by={sort_order})
   ```

   Surface employees in that exact role who post publicly. Pass {industry} only if provided; sort_by defaults to most_recent. Collect each author and author_url.

3. **`linkedin_person_posts`**

   ```
   linkedin_person_posts(url=<author_url>)
   ```

   Read each person's recent posts to gauge tenure, interests and an opener before reaching out.

4. **`search_twitter_users`**

   ```
   search_twitter_users(query="{competitor} {target_role}")
   ```

   Only run if {platforms} includes twitter: find people who name {competitor} and {target_role} in their X bio — a sourcing channel where recruiter InMail is saturated. Collect each username.

5. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<username>)
   ```

   Only run if {platforms} includes twitter: pull each profile's bio, location and any link to confirm they still work at {competitor}, capture an outreach channel, and dedupe against the LinkedIn names already found.

6. **`search_instagram_users`**

   ```
   search_instagram_users(query="{competitor} {target_role}")
   ```

   Only run if {platforms} includes instagram: for creative/design/video roles, surface competitor employees who name {competitor} in their bio, with portfolio links that double as the personalized hook.

## Deliver

A deduped, multi-channel sourcing list — name, profile/handle URLs, role signals and a personalized hook — for in-role talent at the competitor, with the best outreach channel (LinkedIn, X or Instagram) attached to each person.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find Staff Engineers who work at Stripe and post on LinkedIn, with a personalized angle for each.
