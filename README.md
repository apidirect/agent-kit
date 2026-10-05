<p align="center">
  <a href="https://apidirect.io"><img src="assets/logo.png" alt="API Direct" width="80" height="80"></a>
</p>

<h1 align="center">API Direct Agent Kit</h1>

<p align="center">
  63 research skills for AI agents, running on <a href="https://apidirect.io">API Direct</a>'s hosted MCP server.<br>
  A Claude Code plugin, a Gemini CLI extension and a Cursor plugin in one repo.
</p>

Each skill is a playbook that chains API Direct's real-time data tools with the right filters to reach a business outcome: finding leads, catching a competitor's unhappy customers, monitoring a brand, mining reviews, sourcing creators or vetting a company. The data comes from LinkedIn, X, Reddit, YouTube, TikTok, Instagram, Facebook, Threads, Bluesky, Truth Social, Google Maps, Amazon, Trustpilot, news, forums and web search, all through one API key.

Installing the kit also connects the MCP server itself, so your agent can call any API Direct tool directly as well as run the skills. Every tool is read-only.

**Contents:** [Setup](#setup) · [Skills](#skills) · [How it works](#how-it-works) · [Links](#links)

## Setup

1. Sign up at [apidirect.io](https://apidirect.io/signup?utm_source=agent-kit). Every account gets 50 free requests per endpoint each month, with no card required.
2. Copy an API key from [Dashboard > API Keys](https://apidirect.io/dashboard/keys). Keys start with `ak_live_`.
3. Install the kit in your agent, below.

### Claude Code

```
/plugin marketplace add apidirect/agent-kit
/plugin install api-direct@api-direct
```

Claude Code asks for your API key when you enable the plugin and keeps it in secure storage. The skills appear as `/api-direct:<skill>`, or just describe what you want and Claude picks the right one.

### Gemini CLI

```
gemini extensions install https://github.com/apidirect/agent-kit
```

Gemini CLI asks for your API key during install and stores it as a sensitive setting. To change it later, run `gemini extensions config api-direct`.

### Cursor

Install **API Direct** from the [Cursor Marketplace](https://cursor.com/marketplace) and enter your API key when Cursor asks for the plugin's configuration.

### Any other agent

The skills follow the open [Agent Skills](https://agentskills.io) format, so any agent that reads `SKILL.md` folders can use them (Codex, GitHub Copilot, OpenCode and others):

```
npx skills add apidirect/agent-kit
```

Then connect the MCP server at `https://apidirect.io/mcp` with your key in an `X-API-Key` header. [apidirect/mcp-server](https://github.com/apidirect/mcp-server) has a ready-made config for every common client.

## Skills

### Lead Generation & Sales

| Skill | What it does |
|---|---|
| [Competitor Conquest Radar](skills/competitor-conquest-radar/SKILL.md) | Intercept people publicly complaining about a competitor — on LinkedIn, X and Reddit — the moment they post. |
| [Hiring-Signal Account Builder](skills/hiring-signal-account-builder/SKILL.md) | Turn fresh job posts that imply a tooling gap into ranked accounts, the internal budget owner, and a cross-platform talking point. |
| [Just-Funded Outreach Window](skills/just-funded-outreach-window/SKILL.md) | Catch companies the week they raise — when budgets are fresh and buyers are saying yes. |
| [Switcher Harvesting Engine](skills/switcher-harvesting-engine/SKILL.md) | Find the people actively shopping away from a rival across Reddit, X, forums and LinkedIn, and intercept them mid-defection. |
| [Local Buying-Intent Lead Capture](skills/local-buying-intent-capture/SKILL.md) | Catch locals the moment they post — on Facebook, Reddit, or X — that they need exactly what you sell. |
| [Pre-Call Account Dossier](skills/pre-call-account-dossier/SKILL.md) | Walk into every discovery call already knowing the company, its latest moves across web and news, and the buyer's hot buttons. |

### Recruiting & Talent

| Skill | What it does |
|---|---|
| [Competitor Employee Poacher](skills/competitor-employee-poacher/SKILL.md) | List a named competitor's employees in a target role across LinkedIn, X and Instagram — sourced from their own profiles, not the company page. |
| [Layoff Wave Interceptor](skills/layoff-wave-interceptor/SKILL.md) | Catch freshly laid-off talent across LinkedIn, X and Reddit within days of the announcement, before every other recruiter circles back. |
| [Reddit Deep-Expertise Sourcer](skills/reddit-deep-expertise-sourcer/SKILL.md) | Surface passive technical experts across Reddit, forums and X by the depth of the answers they give, not the resumes they wrote. |
| [Creative Talent Direct Line](skills/creative-talent-direct-line/SKILL.md) | Find portfolio-grade creative talent across Instagram, TikTok, YouTube and X — and pull their public emails and portfolio links in one pass. |
| [Local Trades Talent Scout](skills/local-trades-talent-scout/SKILL.md) | Map a trade's local operators and the named standout staff in their reviews, then optionally source individual performers self-promoting on Instagram and TikTok — all with direct contact. |

### Competitive Intelligence

| Skill | What it does |
|---|---|
| [Market Map via Similar-Companies Crawl](skills/market-map-similar-companies/SKILL.md) | Crawl one seed's similar-companies graph on LinkedIn, then cross-fill the gaps with AI competitor lists and funding news into a full, sized market map. |
| [Distress & Layoff Early-Warning](skills/distress-layoff-early-warning/SKILL.md) | Detect competitor instability from a surge of 'open to work' employees, cross-checked against X, Reddit, layoff forums and the news wire. |
| [Competitor Hiring Roadmap Decoder](skills/competitor-hiring-roadmap-decoder/SKILL.md) | Reverse-engineer a rival's unannounced roadmap from the roles they just opened — then cross-check it against the news and open web. |
| [Pricing Objection Miner](skills/pricing-objection-miner/SKILL.md) | Harvest authentic, sentiment-scored pricing complaints about a rival across the platforms you choose to arm your sales battlecard. |
| [Review Weakness Miner](skills/review-weakness-miner/SKILL.md) | Surface a rival's recurring failures from their angriest reviews and complaint threads across Google, Facebook, and Reddit. |
| [Paid-Creative Reverse Engineer](skills/paid-creative-reverse-engineer/SKILL.md) | Teardown a competitor's best-performing video hooks and angles across Facebook, TikTok and YouTube. |

### Investing & Deal Sourcing

| Skill | What it does |
|---|---|
| [Series-A Inflection Detector](skills/series-a-inflection-detector/SKILL.md) | Find startups hiring their first GTM/finance exec — then confirm the raise across news and X. |
| [PE Roll-Up Target Finder](skills/pe-roll-up-target-finder/SKILL.md) | Harvest fragmented local operators across Google, Facebook, LinkedIn and news — with owner contacts plus live sale and succession signals — for acquisition outreach. |
| [Founder Launch-Signal Radar](skills/founder-launch-signal-radar/SKILL.md) | Surface founders announcing launches across LinkedIn, X, and Reddit — filtered to founders, then graded for conviction and traction. |
| [Turnaround Acquisition Target Finder](skills/turnaround-acquisition-target-finder/SKILL.md) | Find established local businesses with proven demand but failing management — the owner's direct line, plus cross-platform proof the reputation is broken. |
| [DTC Breakout Traction Scout](skills/dtc-traction-scout/SKILL.md) | Catch consumer brands breaking out across TikTok, Reddit and YouTube before they raise, then surface the founder's inbox. |
| [Fresh Fundraise Detector](skills/fresh-fundraise-detector/SKILL.md) | Catch founders the day they announce a round across X, LinkedIn and Reddit, rank them by real traction, and corroborate against fresh press before everyone else calls. |

### PR, Reputation & Crisis

| Skill | What it does |
|---|---|
| [AI-SERP Reputation Audit](skills/ai-serp-reputation-audit/SKILL.md) | See what Google's AI tells the public about your brand — then trace that narrative back to the Reddit, news, and forum sources you can actually fix. |
| [Crisis Amplification Tracer](skills/crisis-amplification-tracer/SKILL.md) | For a damaging tweet, map who's spreading it and how angry the room is — then check if the crisis has jumped to Reddit, the press, and forums. |
| [One-Star Review Triage Queue](skills/one-star-review-triage-queue/SKILL.md) | Surface the angriest reviews and public complaints nobody has answered yet, ranked so your team replies in the order that protects your reputation most. |
| [Crisis-Statement Reaction Gauge](skills/crisis-statement-reaction-gauge/SKILL.md) | Measure whether your crisis statement calmed the room or poured gas on it — on X and, optionally, across Reddit, news, and forums. |
| [Executive Mention Tracker](skills/executive-mention-tracker/SKILL.md) | Catch every mention of a named leader across LinkedIn, X, news, Reddit, and optionally YouTube and Facebook, and split the wins from the reputation risks. |

### Product & Customer Insights

| Skill | What it does |
|---|---|
| [Enterprise Account Health Monitor](skills/enterprise-account-health-monitor/SKILL.md) | Score a key B2B account's churn risk from employee posts, rival-tool job reqs, risk-event news, and public switching chatter. |
| [Churn-Intent Saver](skills/churn-intent-saver/SKILL.md) | Catch 'I'm cancelling / switching' posts across X, Reddit and forums and prioritize the loudest accounts for a save. |
| [Feature Request Harvester](skills/feature-request-harvester/SKILL.md) | Turn scattered "I wish it could" chatter across Reddit, forums, X, YouTube and Facebook into a ranked, evidence-backed feature backlog. |
| [Outage Early-Warning Siren](skills/outage-early-warning-siren/SKILL.md) | Detect an incident from a corroborated cross-platform complaint surge minutes before the support queue floods. |
| [Support Deflection Gap-Finder](skills/support-deflection-gap-finder/SKILL.md) | Find the questions users keep asking that no doc, video, or AI answer deflects — across forums, Reddit, X and YouTube. |
| [Influencer Complaint Rescue](skills/influencer-complaint-rescue/SKILL.md) | Catch a high-follower complaint on Instagram, X, or TikTok and get a private contact to fix it before it spreads. |

### Content & Influencer

| Skill | What it does |
|---|---|
| [Rising-Creator Early Detector](skills/rising-creator-early-detector/SKILL.md) | Find undervalued creators across TikTok, Instagram and YouTube by engagement-to-follower ratio before they blow up. |
| [Creator Contact Extractor](skills/creator-contact-extractor/SKILL.md) | Turn any niche into a contact sheet of reachable creators — with public emails and links — across Instagram, TikTok, YouTube, and X. |
| [Viral TikTok Pattern Miner](skills/viral-tiktok-pattern-miner/SKILL.md) | Reverse-engineer the week's winning short-form hooks, sounds, and formats across TikTok, YouTube, and Instagram before they saturate. |
| [Amplifier Outreach List](skills/tweet-amplifier-outreach-list/SKILL.md) | Turn one viral tweet — plus optional Instagram and LinkedIn creators on the same topic — into a reach-ranked amplifier list for your launch. |
| [Niche YouTube Creator Leaderboard](skills/niche-youtube-creator-leaderboard/SKILL.md) | See who's actually winning a niche by momentum, not follower vanity — YouTube core, optionally extended to TikTok and Instagram. |

### OSINT & Due Diligence

| Skill | What it does |
|---|---|
| [Storefront-to-Human Contact Bridge](skills/storefront-to-human-contact-bridge/SKILL.md) | Turn a business listing into the owner's emails, phones and every linked social — Instagram, X, Facebook and LinkedIn — resolved into one person. |
| [Adverse Media Sweep](skills/adverse-media-sweep/SKILL.md) | Scan every public surface — press, web, social, video, and forums — for lawsuits, fraud claims, and negative chatter before you onboard a person or company. |
| [Cross-Platform Identity Resolver](skills/cross-platform-identity-resolver/SKILL.md) | Pin one real person to all of their social handles and confirm the match with shared bio links and emails. |
| [Cited Dossier Verifier](skills/cited-dossier-verifier/SKILL.md) | Generate a subject dossier, then independently re-verify every AI-stated claim against its primary source and optional first-hand social corroboration. |
| [Reviewer Footprint Tracker](skills/reviewer-footprint-tracker/SKILL.md) | Trail one Google reviewer across nearby venues — and optionally the wider web and X — to infer their home turf and daily routine. |

### Cross-Platform Power Plays

| Skill | What it does |
|---|---|
| [Trend-to-Founder-to-Inbox](skills/trend-to-founder-to-inbox/SKILL.md) | Catch a rising topic, find the B2B founders posting about it on LinkedIn and X, then surface their contact channel. |
| [Local SMB Lead Gauntlet](skills/local-smb-lead-gauntlet/SKILL.md) | Turn a map full of local businesses into a contact-rich, multi-platform lead dossier for every place. |
| [Cross-Platform Demand Triangulation](skills/cross-platform-demand-triangulation/SKILL.md) | Prove a niche is not just hot but accelerating by triangulating fresh demand and buyer-intent signals across up to six independent platforms — and run only the ones you trust. |
| [City Market-Entry Scan](skills/city-market-entry-scan/SKILL.md) | Get a 360-degree read on one city before you expand — local chatter on Facebook and Reddit, trending hooks, competitor density, and hiring momentum — running only the lenses you choose. |

### Brand & Social Listening

| Skill | What it does |
|---|---|
| [Omnichannel Brand Sentiment Pulse](skills/omnichannel-brand-sentiment-pulse/SKILL.md) | Track one brand keyword across the social networks you choose and roll it into a single share-of-positive scoreboard. |
| [UGC Advocate & Creator Finder](skills/ugc-advocate-creator-finder/SKILL.md) | Surface the happiest creators posting about your brand across Instagram, TikTok and X — handing back follower counts and contact emails. |
| [Cross-Platform Video Share-of-Voice](skills/cross-platform-video-share-of-voice/SKILL.md) | Find the creators dominating your topic across YouTube, TikTok, and Instagram and rank them by reach and sentiment. |
| [Employee Advocacy vs Official Voice](skills/employee-advocacy-vs-official-voice/SKILL.md) | Measure whether your employees — and the wider public — out-amplify your own company page on LinkedIn. |
| [High-Reach Detractor Radar](skills/high-reach-detractor-radar/SKILL.md) | Catch only the angry brand mentions across X, Reddit and Facebook and rank them by the size of the audience that could see them. |

### Market Research & Trends

| Skill | What it does |
|---|---|
| [Geo-Trend Divergence Radar](skills/geo-trend-divergence-radar/SKILL.md) | Catch a topic breaking in one region before it spreads — diff live X trend lists across markets, then confirm the surge on TikTok and in regional news. |
| [Local Category Complaint Miner](skills/local-category-complaint-miner/SKILL.md) | Sweep one-star reviews — plus optional Reddit, X, and Facebook chatter — across a metro's category to surface the unmet needs nobody is solving. |
| [Buyer-Intent SERP and AI-Overview Map](skills/buyer-intent-serp-ai-map/SKILL.md) | Map the affiliate, comparison, AI-answer, and community landscape that owns a high-intent buying keyword. |
| [Hiring-as-Demand Leading Indicator](skills/hiring-demand-leading-indicator/SKILL.md) | Turn fresh job-posting velocity for an emerging skill into a forward demand signal — triangulated with news and practitioner chatter — and a map of who is investing. |
| [Cross-Platform Trend Validation Funnel](skills/cross-platform-trend-validation-funnel/SKILL.md) | Promote a topic only when it surges at once across X, Reddit, YouTube, TikTok, Instagram and the news — killing one-platform fads. |

### Local & Places

| Skill | What it does |
|---|---|
| [Local Contact Harvester](skills/local-contact-harvester/SKILL.md) | Turn a niche and a city into a deduped, CRM-ready list of every local business — across Google Maps and Facebook — with phones, emails, and social handles. |
| [Unhappy-Customer Win-Back Miner](skills/unhappy-customer-winback-miner/SKILL.md) | Find local businesses whose own 1-star reviews — on Google and (optionally) Facebook — name the exact pain your product fixes, then hand you the owner to pitch. |
| [Owner Decision-Maker Outreach Chain](skills/owner-decision-maker-outreach-chain/SKILL.md) | Go from a map pin to the decision-maker, then open with a genuine hook pulled from their latest LinkedIn post, Google review, press mention, or Instagram. |
| [Local AI Visibility Audit](skills/local-ai-visibility-audit/SKILL.md) | Reveal which local businesses get recommended for 'best X in city' across Google's AI and the community threads it cites — and which strong, well-reviewed ones stay invisible. |
| [New-Location Expansion Signal Detector](skills/new-location-expansion-signal-detector/SKILL.md) | Catch a brand opening in a new metro weeks early by triangulating its local hiring cluster, Maps footprint, local press, and resident chatter. |

## How it works

- **Skills run on your agent, data comes from API Direct.** A skill is a set of instructions; your agent follows it by calling the API Direct MCP tools it names, in order, then writes up the result.
- **You pay per request.** Each tool call is billed at its endpoint price, from $0.001, shown in every tool's description. There are no subscriptions. See [pricing](https://apidirect.io/endpoints).
- **The same skills live on the server.** Any MCP client connected to `https://apidirect.io/mcp` can also list and run them through the `list_skills` and `get_skill` tools or as MCP prompts, without installing this kit.

## Links

- Docs: https://apidirect.io/docs
- MCP server: https://github.com/apidirect/mcp-server
- OpenAPI spec: https://apidirect.io/openapi.json
- Status: https://apidirect.io/status
- Support: support@apidirect.io

This repository is generated from API Direct's skill library (version 1.0.1). To report a problem with a skill, email support@apidirect.io or open an issue.

## License

The skills and configuration in this repository are MIT licensed. The API Direct service is subject to the [API Direct terms](https://apidirect.io/terms).
