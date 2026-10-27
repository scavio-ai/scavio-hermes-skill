---
name: product-deep-research
description: Aggregate prices, listings, reviews, and sentiment for a product across retail (Amazon, Walmart, eBay, Target, Home Depot), search (Google Shopping and SERP), video and social (YouTube, Reddit, TikTok, X, Threads), and software review sites (G2, Capterra, App Store, Google Play) via Scavio, then synthesize a buying brief. Use when the user asks for thorough buying advice, deep product evaluation, "should I buy", competitive shopping research, or wants a consolidated brief across multiple sources.
license: MIT
metadata:
  author: scavio
  version: "1.1"
  requires: scavio-mcp
  homepage: https://github.com/scavio-ai/scavio-hermes-skill
---

# Product deep research

Produce a structured buying brief for a single product by fanning out across
retail listings, search, video reviews, and social discussion via Scavio, then
synthesizing the results.

## When to use

The user asks any of:

- "Should I buy X?"
- "Research X for me."
- "Help me decide between X and Y." (run twice and compare)
- "What do people think of X?"
- "Find me the best price on X."

Works for physical goods and for software or apps -- the source set changes,
the procedure does not.

## Inputs

- **Product**: name, model number, or SKU. If ambiguous, ask one clarifying
  question (e.g., "Which model generation do you mean?").
- **Optional context**: budget, intended use, must-have features, country or
  marketplace, preferred retailers.

## Access

Scavio is reachable two ways. Use whichever the agent is already wired for.

- **Hosted MCP**: `https://mcp.scavio.dev/mcp` over streamable HTTP, auth
  header `x-api-key`.
- **REST**: base URL `https://api.scavio.dev`, auth header
  `Authorization: Bearer $SCAVIO_API_KEY`. Every endpoint is POST with a JSON
  body.

The default MCP surface registers 106 tools across 11 platforms out of 191
total. If a platform this skill calls for is not exposed in the session, widen
it with the `x-scavio-platforms` header (or the `SCAVIO_PLATFORMS` env var
locally). It is additive, e.g. `default,ebay,target`. `all` registers all 191
tools and ships a large payload in every session -- prefer naming the handful
you actually need.

Endpoint reference: https://scavio.dev/docs

## Procedure

1. **Normalize the product name.** Resolve to a canonical product string.
   For electronics, prefer manufacturer + model number. For apparel or
   household goods, brand + model + variant. For software, the vendor's
   product name as it appears on its own site.

2. **Fan out across retail.** Always search Amazon and Walmart. Add the
   retailers that actually carry the category, and skip the ones that do not:

   - **eBay** -- discontinued, refurbished, or open-box units; also the best
     signal on real street price once a product is past launch.
   - **Target** -- general merchandise, home, apparel.
   - **Home Depot** -- tools, appliances, building and garden.

   Capture per listing: title, price, rating, review count, seller, and URL.

3. **Cross-check price with Google Shopping.** One Google Shopping query on
   the canonical string gives an aggregate view and catches retailers outside
   the set above. Treat it as a pointer, not as the quoted price -- confirm
   the number on the retailer listing before putting it in the brief.

4. **Search YouTube** for `<product> review`. Capture the top 3 to 5 review
   videos with channel name, view count, and any titles that suggest
   contrarian takes ("don't buy", "pros and cons", "honest review"). If
   transcript fetching is available, summarize each video's verdict in one
   sentence and attribute it to the channel, never to a person you did not
   see named in the transcript.

5. **Search social for lived-in sentiment.** Reddit is the primary source:
   capture the top 3 to 5 threads with subreddit, post title, score, and
   dominant sentiment, weighting product-specific and buying-advice
   subreddits. Add, where the category warrants it:

   - **X** -- launch-window complaints and support escalations.
   - **TikTok** -- consumer goods, beauty, kitchen, fashion; often the only
     place a defect trend surfaces early.
   - **Threads** -- supplementary; thin for most categories.

6. **For software or apps, swap the retail leg** for the review platforms:
   **G2** and **Capterra** for B2B software, **App Store** and **Google Play**
   for consumer apps. Capture rating, review count, and the recurring
   complaints in the low-star reviews.

7. **Pull primary sources with Extract.** Use the Extract endpoint on any URL
   that matters and is not a supported platform -- the manufacturer spec page,
   a recall or safety notice, a long-form review on an independent site -- to
   get clean markdown you can quote from.

8. **Synthesize** into the brief format below. Cite every claim with a URL.
   Anything you could not verify does not go in the brief.

## Output format

```markdown
# <Canonical Product Name> buying brief

## Best price right now
- <Retailer>: $<price> (<rating>, <review count>) <URL>
- <Retailer>: $<price> (<availability>) <URL>
- Verdict: buy at <retailer> for $<diff> savings

## Review consensus
- Strengths: <bullet list with citations>
- Weaknesses: <bullet list with citations>
- Overall rating across sources: <retail stars> / <video sentiment> / <social sentiment>

## Red flags
<list with citations, or "none found">

## Top sources
- <retail listings>
- <video reviews, ranked>
- <social threads, ranked>

## Recommendation
<one paragraph: buy / consider alternative / wait>
```

Every figure in the brief must trace to a response you actually received. If a
retailer returned nothing, write "not found" -- do not carry a price over from
another retailer, and do not estimate.

## Edge cases

- **No listing found at a given retailer.** Note this prominently. The product
  may be retailer-exclusive, regional, or out of stock. Try a Google SERP
  search for the manufacturer's own store page, then Extract it.
- **Conflicting prices across listings.** Show the lowest verified price from
  a first-party seller. Flag third-party-only and marketplace listings, and
  flag eBay prices as used or refurbished where that is what they are.
- **Sponsored video content.** Note when the top results are clearly sponsored
  ("paid promotion" disclosures or affiliate-heavy channels).
- **Thin social signal.** If fewer than 3 substantive threads exist, say so
  and lower confidence in the consensus section.
- **Named-reviewer verdicts.** Only attribute an opinion to a named reviewer
  or channel if that attribution came back in a response. Never reconstruct a
  well-known reviewer's take from memory.
- **Regional pricing.** Prices and availability differ by marketplace. State
  which country the brief covers.

## Cost

Credit cost varies by platform -- it is not a flat rate per request. Verified
examples: Google, Amazon, Walmart, eBay, Target, Airbnb, Zillow, Redfin, SEC
EDGAR, UK Companies House and Meta Ads cost 1 credit; Threads, Yelp,
Tripadvisor, Indeed, Capterra, Google Play and Home Depot cost 2; G2 costs 5;
Kuaishou ranges from 1 to 40 depending on the endpoint. Check
https://scavio.dev/docs for the current cost of any endpoint before quoting a
number to the user.

So budget by source mix, not by request count. A lean physical-goods run
(Amazon, Walmart, Google Shopping, plus a couple of follow-ups) sits in the
single digits. A wide run that adds Home Depot, G2 or Capterra, and several
follow-up reads costs meaningfully more, because those platforms are priced
higher per call.

New accounts get 50 credits on signup. One-time, not a monthly allowance, no
credit card required: https://dashboard.scavio.dev/sign-up

## See also

- [references/EXAMPLES.md](references/EXAMPLES.md) for a worked example.
