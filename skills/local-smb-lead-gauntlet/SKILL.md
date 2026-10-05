---
name: local-smb-lead-gauntlet
description: "Turn a map full of local businesses into a contact-rich, multi-platform lead dossier for every place. Use when the user asks something like \"Build me a contact dossier for every med spas in Austin, Texas, United States so my team can start outreach tomorrow\". Runs on the API Direct MCP tools (Google Maps, Instagram, Facebook, X and LinkedIn data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Cross-Platform Power Plays"
  platforms: "Google Maps, Instagram, Facebook, X, LinkedIn"
  mcp-server: "https://apidirect.io/mcp"
---

# Local SMB Lead Gauntlet

Google Maps gives you the business and its website, but place_details quietly returns scraped emails, phones, and social handles. Fan those handles out across the networks you choose — Instagram, Facebook, Twitter, and LinkedIn for the owner's professional channel — and each lead becomes a complete, verified contact card.

**Who it's for:** Agencies and B2B sales teams prospecting local SMBs.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_details`, `instagram_user_profile`, `facebook_page_details`, `twitter_user_profile`, `search_linkedin_companies`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `niche` | yes | The type of local business to prospect. | med spas |
| `city` | yes | City or metro to search in. | Austin, Texas, United States |
| `platforms` | no | Comma-separated list of platforms to run — options: places, instagram, facebook, twitter, linkedin. Omit to run all of them; name specific platforms to limit the run. | instagram, facebook, twitter, linkedin |
| `depth` | no | How many pages of Google Maps results to scan (maps to search_places pages; more pages = more leads). Defaults to 6. | 6 |
| `country` | no | 2-letter country code to region-bias the Maps search and contact scrape (maps to the country param on search_places and place_details). Defaults to the country in {city}. | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{niche} {city}", pages={depth}, country={country})
   ```

   Collect place_id, business name, website, rating, and review_count for each business and keep the ones with a website or phone as the working lead list.

2. **`place_details`**

   ```
   place_details(place_id=<place_id>, country={country})
   ```

   Pull emails_and_contacts (emails, phone_numbers, instagram/facebook/twitter handles) and owner_name to seed each lead's dossier.

3. **`instagram_user_profile`**

   ```
   instagram_user_profile(username=<instagram_handle>)
   ```

   Only run if {platforms} includes instagram: enrich with bio, follower count, public_email, external_url, and category to gauge size and capture a second reachable email.

4. **`facebook_page_details`**

   ```
   facebook_page_details(url=<facebook_url>)
   ```

   Only run if {platforms} includes facebook: grab official page contact info (email/phone) and confirm the business is active before adding it to outreach.

5. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<twitter_handle>)
   ```

   Only run if {platforms} includes twitter: pull followers_count and verified status to prioritize the most credible, reachable owners.

6. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query="<business_name> {city}", page=1)
   ```

   Only run if {platforms} includes linkedin: find the business's LinkedIn company page to extend the dossier to the B2B professional channel. Match on name and city to avoid false hits.

## Deliver

A ranked lead sheet of local businesses, each with verified emails, phones, owner name, and the social profiles you chose to enrich (Instagram, Facebook, Twitter, and optionally a LinkedIn company page) — a ready-to-work contact dossier per place.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Build me a contact dossier for every med spas in Austin, Texas, United States so my team can start outreach tomorrow.
