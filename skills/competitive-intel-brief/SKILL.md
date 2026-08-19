---
name: competitive-intel-brief
description: Build a competitor intelligence brief from public web signals - Google search and news, Reddit discussion, YouTube coverage, plus optional deep passes over review sites (G2, Capterra), hiring boards (LinkedIn, Indeed, Glassdoor), app stores, social, ad transparency archives, and company filings. Use when the user asks "what is X up to", "research competitor X", "give me a brief on X", or wants a one-page snapshot of a company's positioning, recent news, customer sentiment, hiring, and content strategy.
license: MIT
metadata:
  author: scavio
  version: "1.1"
  requires: scavio-mcp
  homepage: https://github.com/scavio-ai/hermes-agent
---

# Competitive intel brief

Generate a one-page competitor snapshot from public web signals. Useful for
sales prep, market research, partnership scoping, and weekly competitor
tracking.

## When to use

- "Give me a brief on Acme Corp."
- "What's the latest on <competitor>?"
- "Research <company> for our pitch tomorrow."
- "Pull together a competitor scan for the pricing meeting."

Do not use for personal-name research (people search has different sourcing
and privacy considerations).

## Inputs

- **Target**: company, product, or brand name.
- **Optional**: time window (default: last 90 days), focus area (pricing,
  product, hiring, sentiment), target type (B2B software, consumer app,
  consumer brand, public company) - this drives which optional passes below
  are worth running.

## Procedure

Run the core pass first. It is cheap and works for every target type. Then
add only the optional passes that match the target and the user's focus
area - each one costs credits and most targets do not need all of them.

### Core pass (always)

1. **Google search** via Scavio for the target name. Capture the canonical
   homepage and the top 5 to 10 organic results.

2. **Google News** via Scavio for the target name, constrained to the
   requested time window. This is the source for the "Recent news" section -
   do not infer news items from the organic results.

3. **Reddit search** via Scavio for the target name. Capture top threads
   from the time window. Sort by relevance, then re-sort the top set by
   score. Note the dominant subreddits (industry-specific subs are higher
   signal than r/all).

4. **YouTube search** via Scavio for the target. Look specifically for:
   official channel uploads (positioning), demos and tutorials by third
   parties (developer mindshare), and complaint or comparison videos
   (weaknesses).

5. **Extract** the target's own pages (pricing, changelog, customers) to
   markdown when positioning or pricing is in scope. This is the only
   reliable way to quote a competitor's own copy rather than paraphrasing
   a search snippet.

### Optional passes (pick by target type and focus)

- **B2B software** - G2 and Capterra for structured review sentiment,
  ratings, and named alternatives. G2 is the most expensive call in this
  skill (see Cost); run it once on the target, not on every competitor in
  a comparison set.
- **Hiring and org signals** - LinkedIn (company profile, job postings),
  Indeed, Glassdoor. Job posts are the highest-signal public leak of
  roadmap and go-to-market direction; Glassdoor covers employee sentiment
  and headcount churn.
- **Consumer app** - App Store and Google Play for ratings, review volume,
  version history, and complaint themes.
- **Consumer brand** - Instagram, TikTok, X, Threads for audience size,
  posting cadence, and campaign messaging.
- **Paid positioning** - Google Ads Transparency and Meta Ads for the ad
  creative a competitor is actually running. This shows the claims they are
  willing to pay for, which often differs from the homepage.
- **Demand trajectory** - Google Trends for the target name versus the
  category and versus one or two named rivals.
- **Financials and legal identity** - SEC EDGAR for US public companies,
  UK Companies House for UK-registered entities (directors, filing history,
  accounts). Use these instead of guessing at revenue or funding.
- **Marketplace and retail presence** - Amazon, Walmart, eBay, Target, Home
  Depot when the target sells physical goods; Booking.com, Airbnb,
  Tripadvisor, Yelp, Zillow, Redfin when the target operates in travel,
  hospitality, or real estate.

### Synthesis

6. **Synthesize** into the brief format below. Every fact must cite a URL
   that appeared in an actual Scavio response from this run. If a section
   has no supporting result, write "no signal found" - never fill the gap
   from memory, and never invent a funding round, a headline, a review
   count, a rating, or a named person's quote.

## Output format

```markdown
# <Target> competitive brief

## Snapshot
- Category: <inferred from sources>
- Homepage: <url>
- Top story in window: <one-line summary> <url>

## Positioning
<2 to 4 sentences from the homepage (Extract) and top organic results>

## Customer sentiment
- Reddit: dominant subreddits <list>, sentiment positive / mixed / negative
- Review sites (if run): <rating and review count as returned by G2 /
  Capterra / App Store / Google Play, each with its source URL>
- Top themes: <bullets with citations>

## Content footprint (YouTube)
- Official channel: <url, subscriber count if returned>
- Third-party coverage: <high / medium / low>
- Notable critical takes: <bullets with citations>

## Hiring and org signals (if run)
<open roles by function, notable new leadership, with citations>

## Paid positioning (if run)
<claims and offers seen in Google Ads Transparency / Meta Ads, with citations>

## Recent news (in window)
<bulleted list of news items with URLs>

## So what
<one paragraph: implications for the user's situation, if context given>
```

## Edge cases

- **Common-word target name** (e.g., "Apple" the company vs the fruit).
  Disambiguate by adding category or domain. Ask the user to confirm if
  ambiguous.
- **Stealth or pre-launch company.** Reddit, YouTube, and review sites may
  be empty. Note this; lean on Google plus Companies House or EDGAR for
  what little signal exists.
- **Very large company.** Constrain searches by time window and topic to
  avoid drowning in results.
- **Private company.** EDGAR and Companies House will not have US private
  entities. Do not substitute a press-reported valuation for a filing and
  do not present it as one.
- **Platform not registered in the MCP session.** The default Scavio MCP
  surface registers 106 tools across 11 platforms; the full catalog is 191
  tools across 31 platforms plus Extract. If a pass above has no matching
  tool, widen the session with the `x-scavio-platforms` header (or the
  `SCAVIO_PLATFORMS` env var locally). It is additive - for example
  `x-scavio-platforms: default,g2,sec`. `all` registers all 191 tools and
  ships a large payload in every session, so prefer naming the platforms
  you need.
- **No MCP available.** Call the REST API directly: base URL
  `https://api.scavio.dev`, header `Authorization: Bearer $SCAVIO_API_KEY`,
  every endpoint is POST with a JSON body. Docs: https://scavio.dev/docs

## Cost

Credit cost varies by platform - there is no flat per-request price.
Verified costs for the platforms this skill uses:

| Platform | Credits per request |
| --- | --- |
| Google (SERP, News, Trends, Maps, Shopping, AI Mode), Amazon, Walmart, eBay, Target, Airbnb, Zillow, Redfin, SEC EDGAR, UK Companies House, Meta Ads | 1 |
| Threads, Yelp, Tripadvisor, Indeed, Capterra, Google Play, Home Depot | 2 |
| G2 | 5 |

For any platform not listed here, check the per-endpoint cost in the docs
rather than assuming 1 credit.

Typical runs:

- **Core pass only** (Google + Google News + Reddit + YouTube + 1 Extract):
  roughly 5 to 8 credits depending on follow-ups.
- **B2B software deep pass** (core + G2 + Capterra + LinkedIn + Indeed):
  add roughly 10 to 15 credits, most of it G2.
- **Consumer app deep pass** (core + App Store + Google Play + social):
  add roughly 5 to 10 credits.

New accounts get 50 credits on signup, one-time, no credit card required -
enough for several core-pass briefs. Sign up at
https://dashboard.scavio.dev/sign-up
