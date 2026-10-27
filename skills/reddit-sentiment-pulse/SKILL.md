---
name: reddit-sentiment-pulse
description: Aggregate Reddit sentiment on a topic, product, company, or person across multiple relevant subreddits and produce a one-page pulse report. Optionally cross-check the read against other social platforms. Use when the user asks "what does Reddit think of X", "what's the buzz on X", "find the dominant sentiment on X", or wants community sentiment specifically (not Google or YouTube). Distinct from product-deep-research which spans multiple platforms.
license: MIT
metadata:
  author: scavio
  version: "1.1"
  requires: scavio-mcp
  homepage: https://github.com/scavio-ai/scavio-hermes-skill
---

# Reddit sentiment pulse

Cross-subreddit sentiment scan for a topic. Returns a structured pulse
report with dominant sentiment, top threads, and surfaced themes.

## When to use

- "What does Reddit think of X?"
- "Pull the Reddit pulse on X."
- "Is X getting hate or love online?"
- "Summarize the latest Reddit discussion on X."

For multi-platform research (Amazon + YouTube + Reddit), use
`product-deep-research`. For competitor research, use
`competitive-intel-brief`.

## Inputs

- **Topic**: a product, company, person, event, or concept.
- **Optional**: time window (default: 90 days), specific subreddits to focus on.
- **Optional**: `cross_check` - whether to sanity-check the Reddit read
  against another platform (see step 8). Default: off.

## Access

Reddit tools are served by the Scavio MCP server at
`https://mcp.scavio.dev/mcp` (streamable HTTP, auth header `x-api-key`).
The default tool surface registers 106 tools across 11 platforms; if a
platform this skill wants is not in that surface, widen it with the
`x-scavio-platforms` header (or `SCAVIO_PLATFORMS` locally). The header is
additive, so pass `default,<platform>` rather than the platform alone.
Avoid `all` - it registers all 191 tools and inflates every session.

Without MCP, the same data is available over REST at
`https://api.scavio.dev` with `Authorization: Bearer $SCAVIO_API_KEY`.
Every endpoint is POST with a JSON body. Docs: https://scavio.dev/docs

## Procedure

1. **Initial Reddit search** via Scavio for the topic. Capture the top 20
   threads. Note which subreddits surface most.

2. **Identify the 3 to 5 highest-signal subreddits.** Filter for topical
   relevance over volume. r/all and meme subs are noise unless the user
   explicitly wants meme sentiment.

3. **Per-subreddit re-search** via Scavio with the topic constrained to
   each high-signal subreddit. Capture the top 3 to 5 threads per sub.

4. **Read sampled threads.** For each top thread, read the post body and
   the top 3 to 5 comments to extract sentiment and themes. Use Scavio's
   thread fetch tool if available, otherwise the search snippet plus URL.

5. **Classify sentiment** at the thread level: positive / mixed / negative
   / neutral. Compute a weighted aggregate using thread score as weight.

6. **Extract themes.** Common praise points, common complaints, recurring
   memes, notable contrarian takes.

7. **Synthesize** the pulse report.

8. **Optional cross-check** (only when `cross_check` is on, or when the
   Reddit sample is thin or looks captured by one sub). Reddit is a single
   community with its own skew; one adjacent source is usually enough to
   tell a real signal from a subreddit in-joke. Pick the one that matches
   the topic, do not run them all:

   - **X or Threads** - fast-moving reaction to news, launches, and
     controversies; the closest analogue to Reddit discussion.
   - **YouTube comments** - for hardware, games, and anything with a
     heavy review-video culture.
   - **G2 or Capterra** - for B2B software, where Reddit skews toward
     practitioners and away from buyers.
   - **Amazon or Walmart reviews** - for consumer products, where Reddit
     skews toward enthusiasts and away from average owners.
   - **Glassdoor or Indeed** - only when the topic is a company as an
     employer.

   Report cross-check findings in their own section. Never merge them into
   the Reddit weighted aggregate - the sampling is not comparable.

## Output format

```markdown
# <Topic> Reddit pulse

## Aggregate sentiment
**<positive / mixed / negative>** (n=<thread count>, weighted by score)

## Top subreddits
- r/<sub> (<n threads>): <one-line take>
- r/<sub> (<n threads>): <one-line take>
- r/<sub> (<n threads>): <one-line take>

## Top threads
1. [<title>](<url>) r/<sub>, <score>, <sentiment>
2. ...
3. ...

## Themes
- **Praise**: <bullets>
- **Complaints**: <bullets>
- **Contrarian / niche takes**: <bullets>

## Notable quotes
> "<short quote>" r/<sub>

## Cross-check (only if run)
<platform>: <does it agree with the Reddit read, and where does it diverge>

## Caveats
<note any sample bias, brigading signals, or low-volume confidence issues>
```

Every value in the report must come from a fetched response. Quotes are
copied verbatim from real comments with the subreddit attached, never
paraphrased into a sharper line and never composed to illustrate a theme.
If a field is unknown, write "unknown" rather than filling it in.

## Edge cases

- **Brigading.** If one thread's comments look coordinated (uniform
  sentiment, low-karma accounts), flag it and exclude from weighting.
- **Astroturfing.** Be skeptical of unanimously positive threads in
  product-specific subs run by the company.
- **Stale topic.** If most threads are over 6 months old, note that
  sentiment may not reflect the current state.
- **Low signal.** Fewer than 5 substantive threads -> output a "low
  confidence" pulse and recommend the user broaden the search, and
  consider a step 8 cross-check rather than padding with weak threads.
- **Wrong-platform topic.** Some topics barely exist on Reddit (regional
  services, most B2B software, non-English markets). Say so and move to
  the platform that actually holds the discussion instead of stretching a
  four-thread sample into a "pulse".

## Cost

A typical run is 5 to 10 Scavio requests: one initial search, 3 to 5
per-subreddit searches, and a few thread fetches. An optional cross-check
adds 1 to 3 more.

Credits are **not** flat across the API - the per-request cost depends on
the platform and the endpoint. Most search endpoints (Google, Amazon,
Walmart, eBay, Target, Airbnb, Zillow, Redfin, SEC EDGAR, UK Companies
House, Meta Ads) are 1 credit; Threads, Yelp, Tripadvisor, Indeed,
Capterra, Google Play, and Home Depot are 2; G2 is 5; Kuaishou ranges from
1 to 40 by endpoint. Price a run against the live table at
https://scavio.dev/docs before quoting a number to the user, and prefer
quoting request counts over credits when the exact price is not to hand.

New accounts get 50 credits on signup - one time, no credit card required:
https://dashboard.scavio.dev/sign-up
