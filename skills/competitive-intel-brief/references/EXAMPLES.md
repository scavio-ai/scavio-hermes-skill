# Worked example: competitive-intel-brief

Everything below is an ILLUSTRATIVE TEMPLATE, not a record of a real run.
The target is a made-up company and every value is a placeholder in angle
brackets. Do not copy any of it into a real brief, and do not present any
value here as a finding. Real briefs contain only values that came back in
an actual Scavio response, each with its source URL.

## Input

> Give me a brief on <TARGET>, the <category> tool.

(Example shape: a B2B software target, so the B2B optional pass applies.)

## Trace

1. Google: `<TARGET> <category> software` to disambiguate the name.
2. Google News: `<TARGET>` constrained to the last 90 days.
3. Reddit: `<TARGET>` filtered to the category subreddits that come back.
4. YouTube: `<TARGET> review` and `<TARGET> vs <RIVAL>`.
5. Extract: `<TARGET homepage>/pricing` to markdown, for the positioning
   and pricing lines.
6. B2B pass: G2 and Capterra for the target's rating and review count;
   LinkedIn and Indeed for open roles.

## Output shape (abbreviated)

```markdown
# <TARGET> competitive brief

## Snapshot
- Category: <category, inferred from the organic results>
- Homepage: <url returned by the Google search>
- Top story in window: <headline from Google News> <url>

## Positioning
<2 to 4 sentences drawn from the Extract of the target's own homepage and
pricing page, plus the top organic snippets. Quote their copy rather than
paraphrasing from memory.>

## Customer sentiment
- Reddit: dominant subreddits <subs returned by the search>, sentiment
  <positive / mixed / negative, with the threads that support the call>
- Review sites: <rating> from <review count> reviews on <G2 or Capterra>
  <url> - use the numbers in the response, never an estimate
- Top themes: <bullets, each citing a thread or review URL>

## Content footprint (YouTube)
- Official channel: <channel url>, <subscriber count if the response
  includes one; otherwise "not returned">
- Third-party coverage: <high / medium / low, justified by result count>
- Notable critical takes: <video titles and urls from the search>

## Hiring and org signals
<open roles by function from LinkedIn / Indeed, with citations>

## Recent news (last 90 days)
<bullets from Google News only, each with a URL and a date>

## So what
<one paragraph tying the above to the user's situation, if they gave one>
```

## Anti-example: what not to do

The following lines are the failure mode this skill exists to avoid. Each
one reads like a finding but is model output with no source behind it:

- "Series C funding round announcement" - no filing, no news URL.
- "Sentiment: strongly positive" with no threads cited.
- "4.6 stars from 1,200 reviews" when the response returned neither number.
- A named reviewer or creator credited with a verdict they never gave.

If a section has nothing behind it, write "no signal found". An empty
section is a usable brief; an invented one is a liability.

## Cost

Cost depends on which passes run and on the platforms involved - credit
cost varies by platform, so there is no single number for a brief. A
core-pass-only run of this shape is roughly 5 to 8 credits; adding the B2B
pass above adds roughly 10 to 15 more, most of it the single G2 call at 5
credits. See the Cost table in SKILL.md.
