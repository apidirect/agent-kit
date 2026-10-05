# API Direct

This extension connects the API Direct MCP server (`https://apidirect.io/mcp`), which gives you read-only, real-time data tools for LinkedIn, X, Reddit, YouTube, TikTok, Instagram, Facebook, Threads, Bluesky, Truth Social, Google Maps, Amazon, Trustpilot, news, forums and web search. Every call is billed to the user's API Direct account at the price shown in the tool's description.

It also ships 63 skills in `skills/`: playbooks that chain these tools to reach an outcome such as finding leads, tracking a competitor's unhappy customers, monitoring a brand or mining reviews. When a request matches a skill, follow that skill's steps.

If an API Direct tool returns `No API key provided` or `Invalid API key`, the key is missing or wrong: the user can set it with `gemini extensions config api-direct`, and get a key at https://apidirect.io/dashboard/keys.
