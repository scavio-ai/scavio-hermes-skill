# Worked example: youtube-research-digest

## Read this first

Everything below is a SHAPE ILLUSTRATION, not a research result. Channel
names, view counts, upload dates, verdicts and consensus text are
placeholders written as `<...>` on purpose. There is no real product,
no real creator, and no real opinion anywhere on this page.

Do not copy any value from this file into a digest. Do not treat a
placeholder as a default. Every field in a real run is filled from what
the API returned in that run, and a field missing from the response is
written as `not available`. Presenting an invented channel, number or
verdict as a finding is the single worst failure mode of this skill.

## Input

> Give me a YouTube digest on <topic>.

## Output shape

```markdown
# <Topic> YouTube digest

## Top videos

### 1. <video title, verbatim from the API>
- Channel: <channel name>, <subscriber count if returned>
- Views: <view count>, uploaded <upload date>
- Verdict: <one line, from the transcript, or marked (inferred)>
- <video url>

### 2. <video title>
- Channel: <channel name>, <subscriber count if returned>
- Views: <view count>, uploaded <upload date>
- Verdict: <one line> (inferred)
- <video url>

### 3. <video title>
- Channel: <channel name>, <subscriber count if returned>
- Views: <view count>, uploaded <upload date>
- Verdict: <one line>
- <video url>

## Contrarian / critical
### <video title of the most skeptical result in the set>
- Channel: <channel name>
- Verdict: <the counter-take, in one line>
- <video url>

## Consensus
<2 to 3 sentences describing where the top 5 agree and where they split.
Written only from the verdicts above. If the top 5 do not actually agree,
say that instead of manufacturing a consensus.>

## Cross-checks (only if step 6 ran)
<platform: what it confirmed or contradicted, attributed to that platform
and not folded into any creator's verdict>
```

## Notes on filling it in

- **Verdicts.** A verdict is a compression of what the video says, drawn
  from its transcript. Without a transcript, infer from title and
  description only, and append `(inferred)`. Never quote a creator.
  Paraphrase what the transcript contains.
- **Numbers.** View counts, subscriber counts and dates are copied from
  the response. If the response omits one, write `not available`. Do not
  round a missing number into existence.
- **Prices.** They do not belong in a YouTube verdict. A price a creator
  read out on camera is stale by the time the digest runs. Pull it from a
  retail cross-check (step 6) and label it with its source.

## Cost

Cost is per endpoint and varies by platform, so this example states no
total. A run is one YouTube search, plus one transcript fetch per video
you pull a verdict for, plus one call per cross-check. Look up the
current per-endpoint costs at https://scavio.dev/docs or in the dashboard
playground, then multiply.

New accounts get 50 credits on signup. One time, not a monthly refill, no
credit card required: https://dashboard.scavio.dev/sign-up
