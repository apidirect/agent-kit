---
name: storefront-to-human-contact-bridge
description: "Turn a business listing into the owner's emails, phones and every linked social — Instagram, X, Facebook and LinkedIn — resolved into one person. Use when the user asks something like \"Get me the owner's contact details and socials for Blue Bottle Coffee, Oakland\". Runs on the API Direct MCP tools (Google Maps, Instagram, X, Facebook and LinkedIn data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "OSINT & Due Diligence"
  platforms: "Google Maps, Instagram, X, Facebook, LinkedIn"
  mcp-server: "https://apidirect.io/mcp"
---

# Storefront-to-Human Contact Bridge

Bridges a map pin to a real person: scrape the business's contact block and every linked social (Instagram, X, Facebook, LinkedIn) that place_details returns, then enrich the chosen handles into one profile of the human behind it. Use {platforms} to pick which networks to run and {country} to lock the right region.

**Who it's for:** Sales, recruiters, journalists and diligence teams.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_places`, `place_details`, `instagram_user_profile`, `twitter_user_profile`, `facebook_page_details`, `linkedin_company_details`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `business` | yes | The business name (and city) to resolve. | Blue Bottle Coffee, Oakland |
| `platforms` | no | Comma-separated list of platforms to run — options: places, instagram, twitter, facebook, linkedin. Omit to run all of them; name specific platforms to limit the run. | instagram, twitter, facebook, linkedin |
| `country` | no | Two-letter country code to disambiguate which region's business to resolve (maps to the country param on search_places/place_details). | us |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_places`**

   ```
   search_places(query="{business}", country="{country}")
   ```

   Resolve the business to a place_id (take the best match). {country} is optional and only disambiguates region when supplied.

2. **`place_details`**

   ```
   place_details(place_id=<place_id>, country="{country}")
   ```

   Read emails_and_contacts: emails, phone_numbers, and the linked instagram/twitter/linkedin/facebook profiles — these links feed every enrichment step below.

3. **`instagram_user_profile`**

   ```
   instagram_user_profile(url=<instagram_link>)
   ```

   Only run if {platforms} includes instagram (and an instagram_link exists): enrich the linked Instagram — bio, external_url, public_email — to identify the person.

4. **`twitter_user_profile`**

   ```
   twitter_user_profile(username=<twitter_handle>)
   ```

   Only run if {platforms} includes twitter (and a twitter_handle exists): do the same on X to cross-confirm the same human and gather more context.

5. **`facebook_page_details`**

   ```
   facebook_page_details(url=<facebook_link>)
   ```

   Only run if {platforms} includes facebook (and a facebook_link exists): pull the linked Facebook page's about/contact block — email, phone, website — often the richest owner-contact source for a local business.

6. **`linkedin_company_details`**

   ```
   linkedin_company_details(url=<linkedin_link>)
   ```

   Only run if {platforms} includes linkedin (and a linkedin_link exists): enrich the linked LinkedIn company to confirm the entity and surface a named founder/owner — turning 'a business' into a named decision-maker.

## Deliver

A contact dossier — emails, phones, every linked social profile across Instagram, X, Facebook and LinkedIn, and the likely named person behind the business.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Get me the owner's contact details and socials for Blue Bottle Coffee, Oakland.
