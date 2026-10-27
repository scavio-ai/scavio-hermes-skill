---
name: price-drop-watch
description: Compare the current price of a specific product across major retailers (Amazon, Walmart, eBay, Target, Home Depot) plus Google Shopping, side-by-side, and flag the better deal. Use when the user asks "where is X cheapest", "Amazon vs Walmart price for X", "find the lowest price on X", or wants a quick price diff between retailers before buying. Returns a compact comparison table, not a full review.
license: MIT
metadata:
  author: scavio
  version: "1.1"
  requires: scavio-mcp
  homepage: https://github.com/scavio-ai/scavio-hermes-skill
---

# Price drop watch

Fast price comparison for a single product across the retailers Scavio
covers. No reviews, no sentiment, just price plus availability plus a clear
verdict.

If the user wants reviews and sentiment in addition to price, route to
`product-deep-research` instead. For travel pricing (flights, hotels,
stays) this skill is the wrong shape - Scavio's Google Flights, Google
Hotels, Booking.com and Airbnb endpoints have their own date and occupancy
inputs.

## When to use

- "Is X cheaper on Amazon or Walmart?"
- "Find the lowest price on X."
- "Quick price check for X."
- "Where should I buy X?"

## Inputs

- **Product**: name or SKU. Be specific. "iPhone 16 Pro 256GB Natural
  Titanium" beats "iPhone".
- **Retailers** (optional): defaults to Amazon and Walmart. Widen when the
  category warrants it - see the table under Procedure step 1.

## Retailer coverage

Pick sources by category rather than querying everything. Each extra source
costs credits and adds a normalization step.

| Source | Good for | Credits per request |
| --- | --- | --- |
| Amazon | general, default | 1 |
| Walmart | general, groceries, default | 1 |
| eBay | refurbished, discontinued, open-box | 1 |
| Target | household, apparel, toys | 1 |
| Home Depot | tools, appliances, building materials | 2 |
| Google Shopping | merchants none of the above carry | 1 |

If the product sells mainly on a merchant Scavio has no dedicated endpoint
for, fall back to Google Shopping to find the listing, then use the Extract
endpoint on that product URL to pull the price off the page. Extract is
tier-priced rather than flat - check the docs at https://scavio.dev/docs
before budgeting a run around it.

## Procedure

1. **Choose sources.** Default to Amazon and Walmart. Add from the table
   above when the category clearly lives elsewhere (power tools, refurbs,
   niche merchants). Two sources is usually enough for a verdict.

2. **Search each source** via Scavio for the same product string. Capture:
   title, price, availability, rating and review count where the source
   returns them, seller (first-party vs third-party), shipping, URL. Take
   the top first-party listing where available.

3. **Normalize.** Verify the listings are the same product configuration
   (same model, color, capacity, bundle vs bare unit, new vs refurbished).
   If they differ, flag the mismatch and pick the closest comparable on
   each side.

4. **Compute the diff** against the cheapest in-stock listing and output
   the table below.

## Output format

```markdown
# <Product> price check

| Retailer | Price | Stock | Seller | Shipping |
| --- | --- | --- | --- | --- |
| <retailer> | <price> | <in stock / out of stock> | <seller> | <shipping> |
| <retailer> | <price> | <in stock / out of stock> | <seller> | <shipping> |

**Cheaper at <retailer> by <diff> (<pct>%).**

- <retailer>: <url>
- <retailer>: <url>

<one-line caveat if seller / config / condition differs>
```

Every cell comes from the API response. Do not fill a gap with a
remembered or estimated price - if a source returned no price, write "not
returned" and say so in the caveat.

## Edge cases

- **Product not listed on one of the sources.** Show the sources that have
  it and mark the others "not listed". Consider adding Google Shopping
  before concluding the product is unavailable.
- **Third-party-only Amazon or eBay listing.** Flag it; third-party prices
  fluctuate and may include markups.
- **Condition mismatch.** eBay results are often refurbished or open-box.
  Never compare a refurbished price against a new one without labelling
  both.
- **Bundle mismatch** (e.g., one listing includes a case, the other
  doesn't). Normalize to bare unit if possible, otherwise flag.
- **Stock outage.** "In stock at <price>" beats "out of stock at a lower
  price". Recommend the available one even if nominally pricier.
- **Tool not registered.** The hosted MCP server at
  https://mcp.scavio.dev/mcp exposes a default subset of its 191 tools. If
  a retailer's tool is missing from the session, widen the surface with the
  `x-scavio-platforms` header (or the `SCAVIO_PLATFORMS` env var locally).
  It is additive - `default,<platform>` keeps the default set and adds the
  named one. Check https://scavio.dev/docs for the platform identifiers;
  `all` registers all 191 tools but ships a large payload into every
  session.

## Cost

Credit cost is per request and **varies by platform** - it is not a flat
rate across Scavio. The per-request costs for the sources this skill uses
are in the Retailer coverage table above.

A typical two-source run (Amazon plus Walmart) is 2 credits, plus 1 per
extra search if the first pass needs a retry or a follow-up lookup. Adding
Home Depot costs 2 rather than 1.

New accounts get 50 credits on signup, one-time, no credit card required -
enough for roughly 25 default two-source runs. Sign up at
https://dashboard.scavio.dev/sign-up.
