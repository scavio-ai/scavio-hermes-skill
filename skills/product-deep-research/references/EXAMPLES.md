# Worked example: product-deep-research

> **Placeholder data only.** Nothing below is a real result. Every price,
> rating, review count, quote and verdict is written as a `<PLACEHOLDER>`
> token on purpose, so that no figure in this file can be mistaken for
> something Scavio returned. This file shows the *shape* of a run. Fill it
> from live responses, and never copy a value out of it into a brief.

## Input

> Should I buy the `<BRAND> <MODEL>`?

## Trace

1. **Normalize.** `<BRAND> <MODEL>` -- manufacturer plus model number, already
   canonical. If the user had said only `<BRAND> headphones`, ask which model
   generation before spending credits.
2. **Retail fan-out.** Amazon and Walmart on the canonical string. Category is
   consumer electronics, so add eBay for street price on units past launch;
   skip Home Depot and Target, which do not carry the category meaningfully.
3. **Price cross-check.** One Google Shopping query on the same string to
   catch retailers outside that set. Confirm anything it surfaces on the
   retailer's own listing before quoting it.
4. **Video.** YouTube search `<BRAND> <MODEL> review`. Rank by view count,
   then flag titles that read contrarian.
5. **Social.** Reddit search on the model number, weighted to the
   product-specific and buying-advice subreddits it appears in. Category has
   an active review scene on X, so add one X search for launch-window
   complaints.
6. **Primary source.** Extract the manufacturer's spec page to confirm the
   claimed specification independently of any retailer's copy.
7. **Synthesize.**

## Output (abbreviated, placeholders)

```markdown
# <BRAND> <MODEL> buying brief

## Best price right now
- Amazon: $<AMAZON_PRICE> (<STARS> stars, <REVIEW_COUNT> reviews) <AMAZON_URL>
- Walmart: $<WALMART_PRICE> (<AVAILABILITY>) <WALMART_URL>
- eBay: $<EBAY_PRICE> (<CONDITION>: new / open-box / refurbished) <EBAY_URL>
- Verdict: buy at <RETAILER> for $<DIFF> savings

## Review consensus
- Strengths: <STRENGTH_1> <CITATION_URL>, <STRENGTH_2> <CITATION_URL>
- Weaknesses: <WEAKNESS_1> <CITATION_URL>, <WEAKNESS_2> <CITATION_URL>
- Overall: <STARS> stars / <YOUTUBE_SENTIMENT> / <REDDIT_SENTIMENT>

## Red flags
- <RECURRING_COMPLAINT> reported across <N> threads in r/<SUBREDDIT> <THREAD_URL>
- <ISSUE_RAISED_IN_VIDEO> -- <CHANNEL_NAME>, <VIDEO_URL>

## Top sources
- <AMAZON_URL>
- <YOUTUBE_URL_1>, <YOUTUBE_URL_2>
- <REDDIT_URL_1>, <REDDIT_URL_2>

## Recommendation
<one paragraph, grounded only in the cited sources above: buy / consider
alternative / wait>
```

## Attribution rule

The `<CHANNEL_NAME>` slot takes the channel exactly as it came back in the
YouTube response. Do not attribute a verdict to a well-known reviewer from
memory, and do not paraphrase a review you did not retrieve. An unattributed
"several reviewers noted" with no URL behind it is not a finding -- cut it.

## Software variant

For a SaaS product or a mobile app, steps 2 and 3 are replaced: G2 and
Capterra for B2B software, App Store and Google Play for consumer apps. The
brief keeps the same headings, with "Best price right now" becoming the plan
or tier comparison and the red flags drawn from the low-star reviews.

## Cost

Cost depends on which platforms the run touched, not on how many calls it
made -- per the cost table in SKILL.md, platforms are priced differently. The
electronics run sketched above is cheap: Amazon, Walmart, eBay and Google are
all 1-credit platforms, so the total is roughly the number of calls. Swap in
the software variant and the same call count costs several times more, since
G2 alone is 5 credits per call and Capterra and Google Play are 2.

Before quoting a run cost to a user, check the per-endpoint cost at
https://scavio.dev/docs rather than assuming a flat rate.
