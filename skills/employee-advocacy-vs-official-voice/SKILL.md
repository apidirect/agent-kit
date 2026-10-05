---
name: employee-advocacy-vs-official-voice
description: "Measure whether your employees — and the wider public — out-amplify your own company page on LinkedIn. Use when the user asks something like \"Tell me whether HubSpot employees amplify the brand more than its official LinkedIn page\". Runs on the API Direct MCP tools (LinkedIn data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Brand & Social Listening"
  platforms: "LinkedIn"
  mcp-server: "https://apidirect.io/mcp"
---

# Employee Advocacy vs Official Voice

LinkedIn uniquely splits posts BY the company page (from_company) from posts BY its employees (author_company) and posts that merely mention it (mentions_company); comparing these three voices by volume, sentiment, and engagement reveals whether your real brand reach is the page, your people, or the wider market. Optional topic, seniority (author_title), and sort controls let you focus the read on a campaign or a specific rank of employee.

**Who it's for:** Employer brand, internal comms, and social leads benchmarking advocacy.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_linkedin_companies`, `search_linkedin`, `linkedin_company_details`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `company` | yes | Company name to analyze. | HubSpot |
| `topic` | no | Optional. Narrow every voice (page, employees, mentions) to posts about a specific campaign, product, or theme. Maps to the search_linkedin query param. Omit to read all posts. | AI product launch |
| `focus_title` | no | Optional. Restrict the employee-voice capture to a specific role or seniority to see which ranks actually amplify. Maps to the search_linkedin author_title param. Omit to include all employees. | VP |
| `sort_by` | no | Optional. How to rank captured posts. Maps to the search_linkedin sort_by enum (most_recent or relevance). Defaults to most_recent. | most_recent |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_linkedin_companies`**

   ```
   search_linkedin_companies(query={company})
   ```

   Resolve the brand to its numeric company_id and LinkedIn company url.

2. **`search_linkedin`**

   ```
   search_linkedin(from_company=<company_id>, query={topic}, sort_by={sort_by}, get_sentiment=true)
   ```

   Capture the official brand-page posts as the engagement and sentiment baseline. {topic} and {sort_by} are optional — omit {topic} to read all posts; sort_by defaults to most_recent.

3. **`search_linkedin`**

   ```
   search_linkedin(author_company=<company_id>, author_title={focus_title}, query={topic}, sort_by={sort_by}, get_sentiment=true)
   ```

   Capture posts written by employees of that company and their sentiment, noting the most influential voices. Optional {focus_title} narrows to a seniority or role (e.g. VP, Engineer) to see which ranks amplify; omit it to include all employees.

4. **`search_linkedin`**

   ```
   search_linkedin(mentions_company=<company_id>, query={topic}, sort_by={sort_by}, get_sentiment=true)
   ```

   Capture the wider 'earned voice' — posts that mention the brand from people who are neither the page nor its employees; net out the page and employee posts to isolate genuine third-party advocacy.

5. **`linkedin_company_details`**

   ```
   linkedin_company_details(url=<company_url>)
   ```

   Pull employee count to normalize employee-post volume, then compute the advocacy amplification ratio of employee reach versus official reach (and versus earned third-party reach).

## Deliver

An advocacy report comparing three voices — official brand-page posts, employee posts, and earned third-party mentions — by volume, sentiment, and engagement, normalized by headcount into an amplification ratio, with your top employee voices and an optional breakdown by seniority or campaign topic.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Tell me whether HubSpot employees amplify the brand more than its official LinkedIn page
