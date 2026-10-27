---
name: youtube-research-digest
description: Search YouTube for a topic and produce a digest of the top videos, including channel, view count, and a one-line takeaway per video. Use when the user asks "what's on YouTube about X", "summarize YouTube discussion of X", "find the best videos on X", or wants to triage a topic without watching hours of content. If transcripts are accessible, includes summarized verdicts.
license: MIT
metadata:
  author: scavio
  version: "1.1"
  requires: scavio-mcp
  homepage: https://github.com/scavio-ai/scavio-hermes-skill
---

# YouTube research digest

Triage a topic on YouTube without watching the videos. Surfaces the top
results with a one-line takeaway each, ranked by signal not view count
alone.

## When to use

- "Find me good YouTube videos on X."
- "Summarize what YouTube is saying about X."
- "I don't want to watch 10 videos, just give me the gist."
- "What are the top reviews of X on YouTube?"

For shopping research that needs prices and Reddit sentiment too, use
`product-deep-research`. For Reddit-only sentiment, use
`reddit-sentiment-pulse`.

## Access

YouTube tools come from the Scavio MCP server at
`https://mcp.scavio.dev/mcp` (streamable HTTP, auth header `x-api-key`).
Callers not using MCP hit the REST API directly: every endpoint is a POST
with a JSON body to `https://api.scavio.dev`, auth header
`Authorization: Bearer $SCAVIO_API_KEY`. Endpoint reference:
https://scavio.dev/docs

## Inputs

- **Topic**: any searchable topic, product, person, or event.
- **Optional**: angle ("review", "tutorial", "explainer", "comparison").
- **Optional**: cross-check sources (see step 6). Off by default; each one
  costs extra credits.

## Procedure

1. **Search YouTube** via Scavio for the topic. Capture the top 10 to 15
   results.

2. **Rank by signal**, not view count. Signal proxies, in priority order:
   high view count from a non-promotional channel, recency for time-
   sensitive topics, comparison or "vs" titles for product topics, channels
   with topic expertise.

3. **Filter sponsored content.** If the title or description is heavy with
   affiliate disclosures or "sponsored by", note it but downrank.

4. **For the top 5 results**: capture title, channel, view count, upload
   date, URL, and a one-line takeaway. Derive the takeaway from the title
   plus description plus the transcript if Scavio's transcript tool is
   available. Without transcripts, infer cautiously and mark inferred
   takeaways with `(inferred)`.

5. **Identify contrarian takes.** At least one video in the digest should
   be a critical or skeptical perspective if any exist in the result set.

6. **Optional cross-checks.** Scavio covers 31 platforms plus a generic
   Extract endpoint, so a YouTube-only digest can be corroborated when the
   topic warrants it. Only pull what the topic actually needs, and say in
   the output which sources were used:

   - **TikTok** for short-form coverage of the same topic. Often surfaces
     complaints and hands-on clips that never get a long-form video.
   - **Reddit** for written sentiment behind the video consensus, and to
     catch cases where creators and owners disagree.
   - **Google** (SERP or News) to check whether written coverage says the
     same thing as the video consensus.
   - **X** for reaction to a launch or an event in the first 48 hours,
     when view counts are still meaningless.
   - **Product topics**: Amazon, Walmart, Target, eBay for real listing
     prices and review counts, instead of quoting a price a creator said
     on camera months ago.
   - **Software topics**: G2 or Capterra for buyer reviews.
   - **App topics**: App Store or Google Play for store ratings and
     reviews.
   - **Extract** for any article or spec page linked in a video
     description, converted to markdown for reading.

   Never merge a cross-check value into a video's verdict as if the
   creator said it. Attribute every number to the source it came from.

7. **Synthesize** into the digest format below.

## Output format

```markdown
# <Topic> YouTube digest

## Top videos

### 1. <title>
- Channel: <name>, <subs if available>
- Views: <n>, uploaded <date>
- Verdict: <one-line takeaway>
- <url>

### 2. <title>
...

### 3. <title>
...

## Contrarian / critical
### <title>
- One-line counter-take
- <url>

## Consensus
<2 to 3 sentences summarizing the dominant view across the top 5>

## Cross-checks (only if step 6 ran)
<source: what it confirmed or contradicted>
```

Every field above is filled from data returned by the API in this run.
Never carry a value over from an example, a memory, or a guess. If a
field is missing from the response, write `not available` rather than
supplying a plausible-looking substitute.

## Edge cases

- **Sponsored saturation.** If 4 of 5 top videos are clearly sponsored,
  flag it as a soft signal of marketing push and broaden the search to
  include "honest review" or critical-keyword queries.
- **No transcripts available.** Mark all takeaways as `(inferred)` and
  recommend the user watch the top 1 to 2 directly.
- **Very recent topic.** View counts are unreliable in the first 48 hours.
  Lean on channel reputation, and consider the X cross-check in step 6.
- **A cross-check platform's tools are not registered.** The MCP server
  exposes 191 tools in total, but the default surface registers 106 tools
  across 11 platforms. Widen it with the `x-scavio-platforms` header, or
  `SCAVIO_PLATFORMS` locally. It is additive, so pass
  `default,<platform>` (for example `default,zillow,sec`). Avoid `all`:
  it registers all 191 and is a large payload in every session.

## Cost

Credit cost is per endpoint and varies by platform, so do not budget a
flat rate per call. The published per-endpoint costs are at
https://scavio.dev/docs and in the dashboard playground; check them
before quoting a number to the user.

A run costs, in order: one YouTube search, plus one transcript fetch per
video you pull a verdict for, plus one call per optional cross-check in
step 6. Transcripts are the dominant term, so cap them at the top 3 to 5
videos unless the user asks for more.

Cross-check calls are priced differently from each other. Verified
examples: Google, Amazon, Walmart, eBay, Target, Airbnb, Zillow, Redfin,
SEC EDGAR, UK Companies House and Meta Ads are 1 credit; Threads, Yelp,
Tripadvisor, Indeed, Capterra, Google Play and Home Depot are 2; G2 is 5;
Kuaishou ranges from 1 to 40 depending on the endpoint.

New accounts get 50 credits on signup. One time, not a monthly refill, no
credit card required: https://dashboard.scavio.dev/sign-up
