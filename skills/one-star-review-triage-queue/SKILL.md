---
name: one-star-review-triage-queue
description: "Surface the angriest reviews and public complaints nobody has answered yet, ranked so your team replies in the order that protects your reputation most. Use when the user asks something like \"Build me a triage queue of the angriest unanswered reviews for Bluebird Coffee in Austin, Texas, including our Facebook page https://www.facebook.com/bluebirdcoffee\". Runs on the API Direct MCP tools (Google Maps, Facebook, X and Reddit data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "PR, Reputation & Crisis"
  platforms: "Google Maps, Facebook, X, Reddit"
  mcp-server: "https://apidirect.io/mcp"
---

# One-Star Review Triage Queue

Negative reviews and complaints with no response do the most lasting damage, yet they hide at the bottom of the pile. This skill pulls the lowest-ranked, angriest, still-unanswered reviews from Google Maps and Facebook — and, when enabled, unanswered negative brand mentions on X and Reddit — into one prioritized response queue. A platforms toggle lets you choose exactly which surfaces to scan, and a depth control sets how far down each source you go.

**Who it's for:** Reputation and CX managers for local or multi-location brands.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_reviews`, `facebook_page_details`, `facebook_page_reviews`, `search_twitter`, `search_reddit`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `brand_name` | yes | The business name to monitor reviews and mentions for. | Bluebird Coffee |
| `location` | yes | City/area to disambiguate the Google Maps listing. | Austin, Texas |
| `facebook_page_url` | no | The brand's Facebook page URL to also pull recommendations from (required if platforms includes facebook). | https://www.facebook.com/bluebirdcoffee |
| `platforms` | no | Comma-separated list of platforms to run — options: places, facebook, twitter, reddit. Omit to run all of them; name specific platforms to limit the run. | facebook, twitter, reddit |
| `depth` | no | How many pages to pull per source (higher = deeper scan, slower). Maps to the pages param on the review and search tools. Defaults to 4 if unset. | 4 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{brand_name} {location}")
   ```

   Always run (Google is the anchor). Match the brand and grab the place_id of the correct listing (verify by name and business_status).

2. **`place_reviews`**

   ```
   place_reviews(place_id=<place_id>, sort_by=lowest_ranking, get_sentiment=true, pages={depth})
   ```

   Always run. Keep 1-2 star reviews with no owner response whose dominant_emotion is anger or disgust, hardest-hitting first. {depth} defaults to 4 if unset.

3. **`facebook_page_details`**

   ```
   facebook_page_details(url={facebook_page_url})
   ```

   Only run if {platforms} includes facebook and {facebook_page_url} is provided: resolve the page_id needed to read the page's recommendations.

4. **`facebook_page_reviews`**

   ```
   facebook_page_reviews(page_id=<page_id>, get_sentiment=true, pages={depth})
   ```

   Only run if {platforms} includes facebook: keep recommend=false items with negative sentiment and no page reply, then interleave with the Google queue by severity.

5. **`search_twitter`**

   ```
   search_twitter(query="{brand_name}", get_sentiment=true, sort_by=most_recent, pages={depth})
   ```

   Only run if {platforms} includes twitter: keep negative-sentiment posts mentioning the brand that have no reply from the brand's own handle, newest first; fold into the queue by severity.

6. **`search_reddit`**

   ```
   search_reddit(query="{brand_name}", get_sentiment=true, sort_by=most_recent)
   ```

   Only run if {platforms} includes reddit: keep negative-sentiment posts/threads naming the brand that the brand has not responded to, then merge into the single triage queue by sentiment intensity. (search_reddit paginates via single-page `page`, so {depth} is not applied here.)

## Deliver

A single prioritized triage queue of the angriest unanswered complaints — Google and Facebook reviews plus optional X and Reddit mentions — ordered by sentiment intensity for the fastest reputation-saving replies.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me a triage queue of the angriest unanswered reviews for Bluebird Coffee in Austin, Texas, including our Facebook page https://www.facebook.com/bluebirdcoffee
