# Worked example: reddit-sentiment-pulse

> **This page is a shape reference, not a result.** Every value below is a
> placeholder in angle brackets. Nothing here was fetched from Reddit, so
> nothing here may be copied into a real report, quoted to a user, or used
> as a sanity check on what "normal" numbers look like. Fill each slot from
> an actual Scavio response or leave it as "unknown".

## Input

> What's Reddit's pulse on <TOPIC>?

## Output (abbreviated, all values illustrative)

```markdown
# <TOPIC> Reddit pulse

## Aggregate sentiment
**<positive / mixed / negative / neutral>** (n=<thread count>, weighted by score)

## Top subreddits
- r/<primary-sub> (<n> threads): <one-line take on this sub's read>
- r/<broader-sub> (<n> threads): <one-line take, note if it diverges>
- r/<niche-sub> (<n> threads): <one-line take>

## Top threads
1. [<thread title>](<url>) r/<sub>, <score>, <sentiment>
2. [<thread title>](<url>) r/<sub>, <score>, <sentiment>
3. [<thread title>](<url>) r/<sub>, <score>, <sentiment>

## Themes
- **Praise**: <recurring positive point>, <recurring positive point>
- **Complaints**: <recurring negative point>, <recurring negative point>
- **Contrarian / niche takes**: <minority position and roughly how common it is>

## Notable quotes
> "<verbatim comment text>" r/<sub>

## Cross-check (only if run)
<platform>: <agrees / diverges, and on what>

## Caveats
<which subs skew the sample, e.g. a product's own sub skews enthusiast;
which sub better reflects the broader market; any brigading or staleness>
```

## Cost

Depends on how many subreddits and threads the run touches, and on the
per-platform, per-endpoint credit price - the API is not flat-rate. See the
Cost section of `SKILL.md` and the live table: https://scavio.dev/docs

## Anti-pattern: do not invent the example

The most damaging failure mode for this skill is a report that reads as
researched but was written from memory. Specifically, never:

- name a real product and attach thread counts, upvote scores, or
  subreddit breakdowns that were not fetched;
- assert a factual product claim ("the hinge issue was fixed in the second
  revision") that no fetched thread states;
- attribute a quote to a named person, reviewer, or account without the
  verbatim text and a link from a real response;
- fill a "score" or "n=" slot with a plausible-looking number to make the
  report feel complete.

If the data is not in a response you fetched, the honest output is a
low-confidence pulse with the gaps named. Free credits to try a real run
(50 on signup, one time, no credit card): https://dashboard.scavio.dev/sign-up
