# trawl — API reference (LLM-readable)

Premium data APIs for developers. Plain HTTPS: send a request with an API key,
get typed JSON back. This file is the complete reference, maintained for
agents. Human docs: https://trawl.dev/docs · Agent skill:
https://trawl.dev/agent-setup/SKILL.md

## Authentication

- Every request needs an API key in the `x-api-key` header (header names are
  case-insensitive). Keys look like `sk_live_…` and are account-wide: one key
  works on every trawl API.
- Users create keys at https://trawl.dev/console/keys (account signup:
  https://trawl.dev/signup — email code, no credit card).
- NEVER invent, guess, or hard-code a key. Keep it in an environment variable
  and call the API from a backend — never from client-side code, and never in
  a query string (query strings end up in logs and Referer headers).

## Usage & limits

- Every plan includes a monthly request allowance shared across ALL trawl APIs
  — one pool. Plans: https://trawl.dev/pricing
- Only successful (2xx) responses count against the allowance. Errors are free.
- The window resets on the first of each calendar month (UTC).
- Every response reports standing via headers: `X-RateLimit-Limit`,
  `X-RateLimit-Remaining`, `X-RateLimit-Reset` (Unix timestamp).
- When the allowance is spent, requests return 429 until the reset. Do not
  retry-loop a 429 — surface it to the user (upgrades:
  https://trawl.dev/console/billing).

## Errors

Non-2xx responses return JSON with a single field:

```json
{ "error": "min_price must be <= max_price" }
```

| Status | Meaning |
| --- | --- |
| 400 | A parameter failed validation — the message names the field and rule. |
| 403 | Missing, invalid, or deleted API key. |
| 404 | Nothing found — an unknown path or resource. |
| 429 | Monthly allowance spent. Wait for X-RateLimit-Reset or upgrade. |
| 500 | Internal error on trawl's side. |
| 503 | Search backend temporarily unavailable — safe to retry with backoff. |

## eBay API

Base URL: `https://api.trawl.dev/ebay/v1`

Sold-listings data: search 80+ million real completed eBay sales across the US
and UK marketplaces — final price, sale date, condition and shipping for every
item that actually sold. eBay itself only exposes ~90 days of sold history;
this keeps the history and adds query controls.

### GET /search

Finds sold listings whose title contains EVERY word in `query`, in any order
(eBay's own matching semantics). Results are always newest-first; there is no
sort parameter.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| query | string | yes | Words that must all appear in the listing title, in any order. |
| site | string | no | Marketplace: EBAY_US or EBAY_GB. Default EBAY_US. |
| exclude | string | no | Words that must NOT appear in the title. Must not overlap query. |
| category | number | no | Numeric leaf categoryId — find ids via /categories. Ids differ per marketplace. |
| min_price | number | no | Minimum sale price. Must be <= max_price when both set. |
| max_price | number | no | Maximum sale price. |
| condition | string | no | Comma-separated buckets: new, new_other, open_box, used, refurbished, parts, unknown. |
| date_from | string | no | Earliest sale date, inclusive, YYYY-MM-DD. |
| date_to | string | no | Latest sale date, inclusive, YYYY-MM-DD. |
| limit | number | no | Results per page, 1–240. Default 240. |
| page | number | no | Page number, 1–50. page × limit is capped at 15,000. |

Example:

```bash
curl "https://api.trawl.dev/ebay/v1/search?query=iphone+15+pro+256gb&condition=used&limit=240" \
  -H "x-api-key: $TRAWL_KEY"
```

Response shape (one result shown):

```json
{
  "site": "EBAY_US",
  "currency": "USD",
  "query": ["iphone", "15", "pro"],
  "filters": { "condition": ["used"] },
  "page": 1,
  "count": 240,
  "took_ms": 41,
  "results": [
    {
      "item_id": "256637082114",
      "title": "Apple iPhone 15 Pro 256GB Unlocked",
      "sale_price": 525.00,
      "shipping_price": 0,
      "currency": "$",
      "condition": "Pre-Owned",
      "condition_normalized": "used",
      "buying_format": "Buy It Now",
      "date_sold": "2026-07-18T00:00:00.000Z",
      "item_link": "https://www.ebay.com/itm/256637082114",
      "image_url": "https://i.ebayimg.com/images/g/abc/s-l500.webp",
      "location": "United States",
      "seller_name": "techresale",
      "seller_feedback_count": 1842,
      "seller_feedback_percent": 99.6,
      "categoryId": "9355"
    }
  ]
}
```

### GET /categories

Look up eBay leaf categories by name, busiest first — use the returned
`categoryId` as the `category` filter on /search. Category ids differ per
marketplace, so pass the same `site` you will search with.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| query | string | yes | Category name to search for. Case- and accent-insensitive. |
| site | string | no | Marketplace: EBAY_US or EBAY_GB. Default EBAY_US. |

```json
{
  "site": "EBAY_US",
  "total": 3,
  "count": 3,
  "categories": [
    { "categoryId": "183454", "name": "Pokémon TCG Cards", "group": "Toys & Hobbies" },
    { "categoryId": "2611", "name": "Pokémon Mixed Card Lots", "group": "Toys & Hobbies" },
    { "categoryId": "183466", "name": "Pokémon Sealed Boosters", "group": "Toys & Hobbies" }
  ]
}
```

## More APIs

More APIs land under the same base URL, key, and request pool — this file is
updated as they ship. Users can request and vote on new data sources in the
roadmap section at https://trawl.dev.
