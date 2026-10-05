---
name: unhappy-customer-winback-miner
description: "Find local businesses whose own 1-star reviews — on Google and (optionally) Facebook — name the exact pain your product fixes, then hand you the owner to pitch. Use when the user asks something like \"Find dentist in Phoenix, Arizona whose worst reviews complain about wait time, and get me the owner's contact for each\". Runs on the API Direct MCP tools (Google Maps and Facebook data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Local & Places"
  platforms: "Google Maps, Facebook"
  mcp-server: "https://apidirect.io/mcp"
---

# Unhappy-Customer Win-Back Miner

A business's worst reviews name the exact problem you solve. Sentiment-filtering the lowest-rated reviews across Google Places and Facebook Pages for anger plus your keyword surfaces warm prospects and gives you their verbatim complaint to lead the pitch with — wherever they vent.

**Who it's for:** Founders selling a fix for a specific operational pain.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_reviews`, `place_details`, `search_facebook_pages`, `facebook_page_reviews`, `facebook_page_details`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `category` | yes | Type of business to scan. | dentist |
| `city` | yes | City and region to search. | Phoenix, Arizona |
| `pain_keyword` | yes | The pain your product solves, as customers phrase it. | wait time |
| `platforms` | no | Comma-separated list of platforms to run — options: places, facebook. Omit to run all of them; name specific platforms to limit the run. | places, facebook |
| `scan_depth` | no | Optional. How many pages to pull per search and per review lookup, controlling result volume vs. speed. Maps to the pages param on every search/review step. Defaults to 3-5. | 5 |
| `country` | no | Optional. Two-letter country code passed to the Places search, reviews, and details calls to localize results for non-US markets. Defaults to us. | us |
| `translate_reviews` | no | Optional. When true, translates non-English Google reviews so the anger + pain_keyword filter works in any market. Maps to place_reviews.translate_reviews. Defaults to false. | true |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{category} {city}", pages={scan_depth}, country={country})
   ```

   Collect place_id, rating, and review_count for OPERATIONAL {category} businesses that have enough reviews to mine. Prioritize places with a low overall rating AND a high review_count, since those are the warmest, most-evidenced win-back targets.

2. **`place_reviews`**

   ```
   place_reviews(place_id=<place_id>, sort_by=lowest_ranking, pages={scan_depth}, get_sentiment=true, country={country}, translate_reviews={translate_reviews})
   ```

   Keep reviews where dominant_emotion is anger or disgust and the text mentions {pain_keyword}, capturing the exact quote and date to lead the pitch with.

3. **`place_details`**

   ```
   place_details(place_id=<place_id>, country={country})
   ```

   For businesses with matching painful reviews, pull owner_name, owner_link, and emails_and_contacts so you can send a tailored 'I can fix this' message.

4. **`search_facebook_pages`**

   ```
   search_facebook_pages(query="{category} {city}", pages={scan_depth})
   ```

   Only run if {platforms} includes facebook: Find the official Facebook Pages for {category} businesses in {city}, capturing each page_id and page url. This widens the net to businesses that gather more complaints on Facebook than on Google.

5. **`facebook_page_reviews`**

   ```
   facebook_page_reviews(page_id=<page_id>, get_sentiment=true, pages={scan_depth})
   ```

   Only run if {platforms} includes facebook: Keep reviews/recommendations whose dominant_emotion is anger or disgust and whose text mentions {pain_keyword}, capturing the exact quote and date. Dedupe businesses already flagged via Google so each prospect appears once with the strongest complaint.

6. **`facebook_page_details`**

   ```
   facebook_page_details(url=<page_url>)
   ```

   Only run if {platforms} includes facebook: For pages with matching painful reviews, pull the listed phone, email, and website so you can reach the owner with the same tailored 'I can fix this' pitch.

## Deliver

A deduped prospect list of {category} businesses in {city} with a verbatim complaint matching {pain_keyword} — mined from Google reviews and, when enabled, Facebook Page reviews — plus the owner's contact for a win-back pitch.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find dentist in Phoenix, Arizona whose worst reviews complain about wait time, and get me the owner's contact for each
