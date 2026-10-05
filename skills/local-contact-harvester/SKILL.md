---
name: local-contact-harvester
description: "Turn a niche and a city into a deduped, CRM-ready list of every local business — across Google Maps and Facebook — with phones, emails, and social handles. Use when the user asks something like \"Build me a CRM-ready contact list of every med spa in Austin, Texas, enriched with emails and Instagram handles\". Runs on the API Direct MCP tools (Google Maps, Instagram and Facebook data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Local & Places"
  platforms: "Google Maps, Instagram, Facebook"
  mcp-server: "https://apidirect.io/mcp"
---

# Local Contact Harvester

Google Maps exposes phones, websites, and social links for nearly every local business, but Facebook-only SMBs slip through the cracks. One broad Maps sweep plus per-pin detail enrichment turns a map into an import-ready contact database; an optional Facebook Pages sweep catches operators that never surfaced on Maps and merges them in, and an optional Instagram pass verifies and adds public emails. The user chooses which extra platforms run, how deep to sweep, and which region to scope.

**Who it's for:** Agencies, SDRs, and local service sellers building outbound lists.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_details`, `instagram_user_profile`, `search_facebook_pages`, `facebook_page_details`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `niche` | yes | The business category to harvest. | med spa |
| `city` | yes | City and region to search. | Austin, Texas |
| `platforms` | no | Comma-separated list of platforms to run — options: places, instagram, facebook. Omit to run all of them; name specific platforms to limit the run. | facebook, instagram |
| `depth` | no | How many result pages to sweep on each search (maps to the pages param on search_places and search_facebook_pages). Higher pulls more businesses; defaults to 20. | 20 |
| `country` | no | ISO country code to scope the Google Maps search and detail lookups for non-US harvests (maps to the country param on search_places and place_details). | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{niche} {city}", pages={depth}, country={country})
   ```

   Core sweep: pull up to ~200 pins (set by {depth}, default 20 pages; scope with {country} if set), keep only business_status==OPERATIONAL, and collect each place_id, name, and website.

2. **`place_details`**

   ```
   place_details(place_id=<place_id>, country={country})
   ```

   For each pin extract emails_and_contacts (emails, phone_numbers, instagram/facebook/linkedin) and owner_name, then dedupe rows by phone number and domain.

3. **`instagram_user_profile`**

   ```
   instagram_user_profile(url=<instagram_url>)
   ```

   Only run if {platforms} includes instagram: for pins that exposed an Instagram link, pull public_email, followers, and category to enrich and validate the contact record.

4. **`search_facebook_pages`**

   ```
   search_facebook_pages(query="{niche} {city}", pages={depth})
   ```

   Only run if {platforms} includes facebook: discover local business Facebook Pages for the same niche+city — including operators that never surfaced on Google Maps — and collect each page URL and name.

5. **`facebook_page_details`**

   ```
   facebook_page_details(url=<facebook_page_url>)
   ```

   Only run if {platforms} includes facebook: pull phone, email, website, and address from each Page, then merge into the master list and dedupe against the Maps rows by phone number and domain so no business is double-counted.

## Deliver

A deduped, CRM-ready spreadsheet of every {niche} in {city} with name, phone, email, website, owner, and social handles — merged across Google Maps and (optionally) Facebook per the platforms you enable.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me a CRM-ready contact list of every med spa in Austin, Texas, enriched with emails and Instagram handles
