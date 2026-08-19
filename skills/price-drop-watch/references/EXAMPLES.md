# Worked example: price-drop-watch

> **Illustrative only.** Every value below is a placeholder showing the
> shape of the output. No price, stock state, seller or URL here came from
> a live API response, and none should be repeated to a user as fact. In a
> real run, every cell is filled from the Scavio response for that request.

## Example 1: default two-source run

### Input

> Quick price check on <PRODUCT> (e.g. a specific 65-inch OLED TV model).

### Output

```markdown
# <PRODUCT_TITLE> price check

| Retailer | Price | Stock | Seller | Shipping |
| --- | --- | --- | --- | --- |
| Amazon | <AMAZON_PRICE> | <stock> | <seller> | <shipping> |
| Walmart | <WALMART_PRICE> | <stock> | <seller> | <shipping> |

**Cheaper at <RETAILER> by <DIFF> (<PCT>%).**

- Amazon: <AMAZON_URL>
- Walmart: <WALMART_URL>

Both listings are the bare unit, no bundle. <- caveat only if the responses
actually confirm this; drop the line otherwise.
```

### Cost

2 credits: one Amazon search (1) plus one Walmart search (1).

## Example 2: widened run with a condition mismatch

Shows when to add a third source and how to label a listing that is not
comparable like-for-like.

### Input

> Cheapest I can get <PRODUCT>, refurb is fine.

### Output

```markdown
# <PRODUCT_TITLE> price check

| Retailer | Price | Stock | Seller | Shipping |
| --- | --- | --- | --- | --- |
| Amazon | <AMAZON_PRICE> | <stock> | <seller> | <shipping> |
| Walmart | not listed | - | - | - |
| eBay | <EBAY_PRICE> | <stock> | <third-party seller> | <shipping> |

**Cheaper at eBay by <DIFF> (<PCT>%).**

- Amazon: <AMAZON_URL>
- eBay: <EBAY_URL>

The eBay listing is <condition, e.g. refurbished> from a third-party
seller; the Amazon listing is new. Not a like-for-like price.
```

### Cost

3 credits: Amazon (1) plus Walmart (1) plus eBay (1). Credit cost is per
platform, not flat - a Home Depot search in place of one of these would
cost 2.

## Rules this example is demonstrating

- Never invent a price, a review count, a seller name or a URL. If the
  response did not return a field, write "not returned" and note it.
- Label condition and bundle differences before declaring a winner.
- Prefer an in-stock listing over a cheaper out-of-stock one.
