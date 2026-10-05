---
name: reddit-deep-expertise-sourcer
description: "Surface passive technical experts across Reddit, forums and X by the depth of the answers they give, not the resumes they wrote. Use when the user asks something like \"Find me passive experts who clearly know how to solve debugging Kubernetes etcd quorum loss and check they are credible, established accounts\". Runs on the API Direct MCP tools (Reddit, web forums and X data)."
license: MIT
compatibility: "Needs the API Direct MCP server (https://apidirect.io/mcp) connected with an API Direct API key."
metadata:
  author: "API Direct"
  category: "Recruiting & Talent"
  platforms: "Reddit, web forums, X"
  mcp-server: "https://apidirect.io/mcp"
---

# Reddit Deep-Expertise Sourcer

The best engineers rarely job-hunt, but they answer hard questions in public. Mining the top answers to a deep technical problem across Reddit, specialized forums and X reveals demonstrated competence, and per-platform vetting filters down to credible, established accounts worth a cold approach.

**Who it's for:** Technical recruiters and founders hunting passive senior or specialist talent.

## Before you start

This skill calls tools on the [API Direct](https://apidirect.io) MCP server: `search_reddit_comments`, `search_reddit_users`, `search_forums`, `search_twitter`, `search_twitter_users`.

- If those tools are not available, the server is not connected. Tell the user to install the API Direct plugin or extension, or add `https://apidirect.io/mcp` as a remote MCP server with an `X-API-Key` header (setup: https://github.com/apidirect/agent-kit#setup).
- A tool error saying `No API key provided` or `Invalid API key` means the key is missing or wrong. Keys come from https://apidirect.io/dashboard/keys; new accounts sign up at https://apidirect.io/signup?utm_source=agent-kit.

## Inputs

| Input | Required | What it is | Example |
|---|---|---|---|
| `problem` | yes | A deep, specific technical problem only a real expert would answer well. | debugging Kubernetes etcd quorum loss |
| `skill_area` | no | Broader skill area to widen the expert pool and spot recurring names. | Kubernetes |
| `platforms` | no | Comma-separated list of platforms to run — options: reddit, forums, twitter. Omit to run all of them; name specific platforms to limit the run. | reddit, forums, twitter |
| `time_window` | no | Freshness window for the forums sweep, to bias toward currently active experts (maps to the forums time param). Default: `year`. | year |
| `result_depth` | no | How many pages to pull per Reddit/X search to widen the candidate pool (maps to the pages param). Default: `3`. | 3 |

Ask the user for any required input they have not given. Use an optional input's default when the user doesn't set it, and leave optional filters such as `country` out of the call entirely when they have no value.

## Choosing platforms

A step marked "Only run if {platforms} includes X" runs only when the user named X among their platforms, or named no platforms at all. Every other step always runs, whichever platforms the user picked.

## Steps

Call the tools in this order. `{name}` is an input from the table above; `<name>` is a value you take from an earlier step's results.

1. **`search_reddit_comments`**

   ```
   search_reddit_comments(query={problem}, sort_by=top, get_sentiment=true, pages={result_depth})
   ```

   Gather the most upvoted, substantive Reddit answers to the hard problem and note the authors who show first-hand expertise. Raise {result_depth} to widen the candidate pool.

2. **`search_reddit_comments`**

   ```
   search_reddit_comments(query={skill_area}, sort_by=top)
   ```

   Widen to the broader skill area to spot authors who recur across multiple deep threads, signaling genuine depth.

3. **`search_reddit_users`**

   ```
   search_reddit_users(query=<comment_author>)
   ```

   Vet each recurring Reddit author's karma and account age to keep only established, credible experts and drop throwaway accounts.

4. **`search_forums`**

   ```
   search_forums(query={problem}, get_sentiment=true, time={time_window})
   ```

   Only run if {platforms} includes forums: sweep specialized technical forums — where the deepest written Q&A lives — for first-hand answers to the same problem, optionally bounded to {time_window} to favor currently active experts.

5. **`search_twitter`**

   ```
   search_twitter(query={problem}, sort_by=relevance, get_sentiment=true, pages={result_depth})
   ```

   Only run if {platforms} includes twitter: surface experts publicly answering the hard problem in threads and replies, ranked by relevance, and note the handles that recur.

6. **`search_twitter_users`**

   ```
   search_twitter_users(query=<twitter_handle>)
   ```

   Only run if {platforms} includes twitter: vet each recurring X author by follower count and verification to keep only credible, established voices before a cold approach.

## Deliver

A vetted, deduped list of passive subject-matter experts who publicly solve {problem}-class issues — sourced from Reddit and, when enabled, technical forums and X — ranked by demonstrated depth and account credibility.

## Cost

Each tool call is billed at its normal API Direct price, shown in the tool's description; `get_sentiment` adds a small surcharge per page. Every account gets 50 free requests per endpoint each month. Page through results only as far as the deliverable needs.

## Example request

> Find me passive experts who clearly know how to solve debugging Kubernetes etcd quorum loss and check they are credible, established accounts
