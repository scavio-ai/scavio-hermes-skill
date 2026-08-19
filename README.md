# Scavio for Hermes Agent

Structured web data across 31 platforms plus an Extract endpoint for any URL,
exposed as MCP tools and ready-to-use skills for
[Hermes Agent](https://hermes-agent.nousresearch.com/).

Built on the open [Agent Skills](https://agentskills.io) standard, so the same
skills also run in Claude, Cursor, Goose, OpenHands, and other compatible
clients.

## What you get

- A drop-in MCP server config connecting Hermes to `mcp.scavio.dev`
- 5 production-grade skills:
  - `product-deep-research` aggregate Amazon, Walmart, YouTube reviews, and
    Reddit threads into a buying brief
  - `competitive-intel-brief` Google, Reddit, and YouTube scan of a competitor
  - `price-drop-watch` Amazon vs Walmart side-by-side price diff
  - `reddit-sentiment-pulse` cross-subreddit sentiment aggregation
  - `youtube-research-digest` YouTube search plus transcript summarization

## Coverage

31 platforms plus Extract, all behind one API key.

| Group | Platforms |
|---|---|
| Search | Google (SERP, News, Maps, Shopping, Trends, Flights, Hotels, AI Mode) |
| Video and social | YouTube, TikTok, Instagram, Threads, X, LinkedIn, Reddit, Kuaishou |
| Commerce | Amazon, Walmart, eBay, Target, Home Depot, TikTok Shop |
| Travel and local | Booking.com, Airbnb, Tripadvisor, Yelp |
| Real estate | Zillow, Redfin |
| Jobs | Indeed, Glassdoor |
| Apps and software | App Store, Google Play, G2, Capterra |
| Ads and filings | Google Ads Transparency, Meta Ads, SEC EDGAR, UK Companies House |
| Any URL | Extract (HTML, markdown, or plain text) |

The hosted MCP server exposes 191 tools in total. The default surface registers
106 tools across 11 platforms, which is already a lot of context to carry in
every session, so the config snippet in this repo narrows that further. See
[Tool surface](#tool-surface) below.

## Install

The fastest path, once you have a Scavio API key:

```sh
# 1. Get a key (free: 50 credits on signup, one time, no credit card)
open "https://dashboard.scavio.dev/sign-up?utm_source=hermes"

# 2. Store it where Hermes can read it
mkdir -p ~/.hermes
echo "SCAVIO_API_KEY=sk_live_xxxxxxxxxxxx" >> ~/.hermes/.env
chmod 600 ~/.hermes/.env

# 3. Install this repo as a Hermes plugin
hermes plugins install scavio-ai/hermes-agent
```

Prefer to wire it up by hand, or want to tune which tools load? See
[INSTALL.md](INSTALL.md) for the manual path, the config snippet, and
troubleshooting.

## Try it

Start a session:

```sh
hermes chat
```

Then ask:

> Research the Sony WH-1000XM5 across Amazon, Walmart, YouTube reviews, and
> Reddit. Give me a buying brief with prices, review consensus, and red flags.

Hermes activates `product-deep-research`, fans out search across all four
platforms via Scavio, and returns a structured markdown brief.

## Tool surface

The Scavio MCP server exposes 191 tools. Loading all of them writes roughly
70k tokens into every session before you have typed anything, and exceeds the
tool limit some clients enforce. Two knobs control this, and the snippet in
`config/hermes.config.snippet.yaml` uses both:

- `x-scavio-platforms` on the MCP connection picks which platforms the server
  registers at all. It is additive against the keyword `default`, so
  `default,zillow,sec` means the default 11 platforms plus two more, and `all`
  means all 191 tools.
- `tools.include` in the Hermes server entry filters that down client-side to
  the exact tools you want in the picker.

The shipped snippet requests every platform these five skills actually call, then
includes 18 tools. Widen either list when you want more of the catalog.

## Pricing

Scavio is credit-metered. Cost varies by platform and by endpoint, so budget
against the table in the [docs](https://scavio.dev/docs?utm_source=hermes)
rather than assuming a flat rate.

Verified examples: Google, Amazon, Walmart, eBay, Target, Airbnb, Zillow,
Redfin, SEC EDGAR, UK Companies House and Meta Ads cost 1 credit; Threads,
Yelp, Tripadvisor, Indeed, Capterra, Google Play and Home Depot cost 2; G2
costs 5; Kuaishou ranges from 1 to 40 depending on the endpoint. A few
platforms price per endpoint rather than per platform, YouTube among them.

Plans:

- Free 50 credits on signup, one time, no credit card, no monthly refill
- Project $30/month 7,000 credits
- Bootstrap $100/month 28,000 credits
- Startup $250/month 85,000 credits
- Growth $500/month 200,000 credits

A single `product-deep-research` run typically makes 8 to 15 API calls. The
credit total depends on which platforms those calls land on.

[See plans](https://scavio.dev/pricing?utm_source=hermes)

## License

MIT. See [LICENSE](LICENSE).

## Support

- Issues and feature requests: [GitHub issues](https://github.com/scavio-ai/hermes-agent/issues)
- Scavio API docs: [scavio.dev/docs](https://scavio.dev/docs?utm_source=hermes)
- Hermes Agent: [hermes-agent.nousresearch.com](https://hermes-agent.nousresearch.com/)
