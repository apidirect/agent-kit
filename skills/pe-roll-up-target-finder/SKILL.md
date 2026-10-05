---
name: pe-roll-up-target-finder
description: "Harvest fragmented local operators across Google, Facebook, LinkedIn and news — with owner contacts plus live sale and succession signals — for acquisition outreach. Use when the user asks something like \"Build me an acquisition list of HVAC contractors in Phoenix, AZ with owner contact details\". Runs on the API Direct MCP tools (Google Maps, Facebook, news and LinkedIn data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Investing & Deal Sourcing"
  platforms: "Google Maps, Facebook, news, LinkedIn"
  mcp-server: "https://apidirect.io/mcp"
---

# PE Roll-Up Target Finder

Maps every operator in a fragmented local vertical across Google Maps and, optionally, Facebook; qualifies them as real established businesses via reviews and LinkedIn company size; scrapes owner contacts; and flags sale/retirement/distress signals from news — producing a ranked, ready acquisition pipeline you control by platform.

**Who it's for:** Search funds, PE roll-ups and acquisition entrepreneurs.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_details`, `place_reviews`, `search_facebook_pages`, `search_news`, `search_linkedin_companies`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `vertical` | yes | The local business category to roll up. | HVAC contractors |
| `metro` | yes | The metro/area to target. | Phoenix, AZ |
| `platforms` | no | Comma-separated list of platforms to run — options: places, facebook, news, linkedin. Omit to run all of them; name specific platforms to limit the run. | facebook, news, linkedin |
| `search_depth` | no | How many result pages to pull when discovering operators (maps to the pages param on search_places and search_facebook_pages). Higher = more targets, slower. Default: `20`. | 20 |
| `country` | no | Two-letter country code to scope Places, place details, and news results to the right market. | us |
| `news_window` | no | How far back to scan for sale/retirement/distress signals (maps to search_news time_published; allowed values 1h, 1d, 7d, 1y, anytime). Use a wider window for slow-moving succession news. Default: `1y`. | 1y |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{vertical} {metro}", pages={search_depth}, country={country})
   ```

   Core, always runs. Pull up to ~200 operators (default pages=20). Keep business_status=OPERATIONAL with a real review_count to filter out shells and dead listings.

2. **`place_details`**

   ```
   place_details(place_id=<place_id>, country={country})
   ```

   Core, always runs. Read emails_and_contacts (emails, phones, socials) and owner_name/owner_link for each target — this is the contact backbone of the acquisition list.

3. **`place_reviews`**

   ```
   place_reviews(place_id=<place_id>, sort_by=lowest_ranking, get_sentiment=true)
   ```

   Optional analysis (Places). Skim the worst reviews to judge operational quality, owner fatigue, and succession risk — strong tells that an owner may be ready to exit.

4. **`search_facebook_pages`**

   ```
   search_facebook_pages(query="{vertical} {metro}", pages={search_depth})
   ```

   Only run if {platforms} includes facebook: discover additional local operators with Facebook pages that are thin or absent on Google Maps, then merge into the same target list and dedupe against the Places results by name. (For full phone/website, follow up per page with facebook_page_details if needed.)

5. **`search_news`**

   ```
   search_news(query="<business_name> for sale OR owner retiring OR acquired OR closing", time_published={news_window}, country={country})
   ```

   Only run if {platforms} includes news: scan recent local news for each top target for explicit sale, retirement, succession, or distress signals (default news_window=1y). Use hits to rank targets by seller readiness, not just contactability.

6. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query="<business_name>", page=1)
   ```

   Only run if {platforms} includes linkedin: confirm the operator is a real, established company and read its size to qualify roll-up fit (skip one-person gigs), and use the company page as a route to the named owner/decision-maker. (Pull linkedin_company_details on the matched company if employee_count is not present in search results.)

## Deliver

A CRM-ready table — business, owner, email, phone, rating, review count, plus a sale/succession-signal flag and (optional) LinkedIn company size — ranked by acquisition attractiveness and seller readiness.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me an acquisition list of HVAC contractors in Phoenix, AZ with owner contact details.
