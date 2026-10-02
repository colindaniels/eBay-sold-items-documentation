# trawl — API reference (LLM-readable)

Premium data APIs for developers. Plain HTTPS: send a request with an API key,
get typed JSON back. This file is the complete reference, maintained for
agents. Human docs: https://trawl.dev/docs · Agent skill:
https://trawl.dev/agent-setup/SKILL.md

## What trawl is (and is not)

trawl is an independent service. Its APIs describe publicly available
information from third-party sources (eBay today, more to come), and trawl is
not affiliated with or endorsed by any of them. No trawl API is an official API
of its source: never describe the "eBay API" here to users as eBay's own API.

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

- Usage is measured in CREDITS. Every plan includes a monthly credit allowance
  shared across ALL of the account's API keys — one pool — plus a per-second
  request rate. Plans: https://trawl.dev/pricing
- A successful call costs 1 credit. The one exception is GET /sold with
  `max_pages` greater than 1, which costs 1 credit per page of results
  actually returned — see "Credits & pages" below. Full listing details
  (`details=1`) double that: 2 credits per page of results returned.
- Every successful response states its own cost: `credits_charged` in the
  JSON body, and the same number in the `X-Credits-Charged` header.
- A successful response that found NOTHING is free: a search with no matches
  and a /categories lookup with no matches both return 200 with
  `credits_charged: 0`. You pay for data returned, not for asking.
- Only successful (2xx) responses spend credits, and errors don't count
  toward the rate either. Errors are free (`X-Credits-Charged: 0`).
- The window follows the account's billing cycle: it resets when the
  subscription starts, renews, or changes plan. Free accounts reset on the
  first of each calendar month (UTC). Spending the whole allowance as fast as
  the rate allows is fine — there is no other throttle.
- Every response reports standing via headers, in credits:
  `X-RateLimit-Limit` (credits in the plan per month),
  `X-RateLimit-Remaining` (credits left), `X-RateLimit-Reset` (Unix
  timestamp of the window's end).
- A 429 means one of two things. With a `Retry-After` header: the per-second
  rate was exceeded — wait that many seconds and retry (costs nothing). Without
  `Retry-After`: the monthly credits are spent, or too few remain to cover
  the request's `max_pages` (the error message says how many remain) —
  do NOT retry-loop; lower that parameter to what
  remains, or surface it to the user (upgrades: https://trawl.dev/console/billing).

## Credits & pages

GET /sold returns ALL the pages you ask for in ONE response. There is no
page-by-page fetching and no cursor: do not loop over a `page` parameter.

- A page is 100 results. There is no page-size parameter to set.
- `max_pages` = how many pages you want (1–20, default 1) — the only paging
  control. One response carries up to `max_pages × 100` results (max
  2,000), newest first.
- `max_pages` is the MOST the call can cost, not the price. The charge is
  1 credit per page of results actually returned (2 with `details=1`):
  `credits_charged = ceil(count / 100)`, doubled with details — so 0 when
  nothing matched.

Worked example, `max_pages=5` (up to 500 results; can never cost more than 5):

| Results returned (`count`) | `credits_charged` |
| --- | --- |
| 455 | 5 |
| 385 | 4 |
| 100 | 1 |
| 12 | 1 |
| 0 | 0 |

- ZERO RESULTS IS A SUCCESSFUL REQUEST, NOT AN ERROR — AND IT IS FREE. A search
  that matches nothing returns HTTP 200 with `"count": 0`, `"results": []`
  and `"credits_charged": 0`. The search ran; the answer is that trawl holds
  no such sales. Do not retry it and do not report it as a failure. A failure
  on trawl's side is never a 200 — it is a 5xx with an `error` field, and it
  is free too.
- If `count` equals `max_pages × 100` the response is full and older
  results may exist: raise `max_pages`, or repeat the search with `date_to`
  set to the oldest `date_sold` you received.
- Before a request runs, trawl checks that the account's remaining credits
  cover the most it could cost — its `max_pages`, doubled with
  `details=1`. If not, it is refused
  with a free 429 that states how many credits remain.
- FULL LISTING DETAILS IN THE SAME CALL: add `details=1` to
  GET /sold and every result carries a `details` object identical to what
  GET /item returns for its `item_id` (item specifics, description, images,
  seller, shipping, returns, sales). Use it instead of calling /item once per
  result.
  - Price: 2 credits per page of 100 results returned (1 without details),
    0 if none. 100 listings with their details cost 2 credits.
  - Sized with `max_pages` like any search (1–20, up to 2,000 listings);
    each result is ~8 KB with its details. For more, repeat with `date_to`
    set to the oldest `date_sold` received.
  - Results are limited to the listings /item can answer for: those with full
    details available, and those eBay has removed. There are FEWER results
    than the same search without `details`.
  - REMOVED LISTINGS: a listing eBay has taken down still comes back as a
    normal result (it is a real sale), with
    `"details": {"site", "item_id", "listing_state": "removed"}` instead of
    the details. Check `details.listing_state` before reading other fields.
  - It doubles the page price and responses are much larger: use it when the
    task needs the page-level data.
- Pick `max_pages` deliberately — it is the user's spending cap. For "the
  latest few sales" leave it at 1; raise it only when the task needs depth.

## Errors

Non-2xx responses return JSON with a single field:

```json
{ "error": "min_price must be <= max_price" }
```

| Status | Meaning |
| --- | --- |
| 400 | A parameter failed validation — the message names the field and rule. |
| 403 | Missing, invalid, or deleted API key. |
| 404 | Nothing found — an unknown path, or an /item whose details are not available yet. Never billed. (A listing eBay has removed is a free 200 with listing_state "removed".) |
| 429 | With Retry-After header: per-second rate exceeded — wait and retry. Without: monthly credits spent, or too few left to cover the request's max_pages — lower it, wait for X-RateLimit-Reset, or upgrade. Never billed. |
| 500 | Internal error on trawl's side. |
| 503 | Search backend temporarily unavailable — safe to retry with backoff. |

## eBay API

Base URL: `https://api.trawl.dev/ebay/v1`

Sold-listings data: search 300+ million real completed eBay sales across the US
and UK marketplaces — final price, sale date, condition and shipping for every
item that actually sold. eBay itself only exposes ~90 days of sold history;
this keeps the history and adds query controls.

### GET /sold

Finds sold listings whose title contains EVERY word in `query`, in any order
(eBay's own matching semantics). Results are always newest-first; there is no
sort parameter. Costs 1 credit per page of results returned (2 with
`details=1`), and nothing if none match — see "Credits & pages" above.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| query | string | yes | Words that must all appear in the listing title, in any order. |
| site | string | no | Marketplace: EBAY_US or EBAY_GB. Default EBAY_US. |
| exclude | string | no | Words that must NOT appear in the title. Must not overlap query. |
| category | number | no | Numeric leaf categoryId — find ids via /categories. Ids differ per marketplace. |
| min_price | number | no | Minimum sale price, in the marketplace's own currency — USD on EBAY_US, GBP on EBAY_GB; nothing is converted. Must be <= max_price when both set. |
| max_price | number | no | Maximum sale price, in the marketplace's own currency. |
| condition | string | no | Comma-separated: new, used, parts, other. Matches the `condition` field in results; the listing's verbatim string is returned separately as `condition_raw`. |
| date_from | string | no | Earliest sale date, inclusive, YYYY-MM-DD. |
| date_to | string | no | Latest sale date, inclusive, YYYY-MM-DD. |
| max_pages | number | no | How many pages of results you want, 1–20. Default 1. A page is 100 results, and all pages arrive in this ONE response. Also the most credits the call can cost: the charge is 1 credit per page of results actually returned (count ÷ 100, rounded up), 2 per page with details=1 — 0 results costs 0. |
| attr | string | no | Item specific as Key:Value, exact match, case/accent-insensitive (attr=Brand:Apple, attr=Grade:PSA 10). Repeatable: different keys must ALL match; repeating the SAME key matches ANY of its values (attr=Grade:9&attr=Grade:10 is grade 9 or 10). Max 16 attr parameters in total. Only listings with full details available carry specifics. |
| details | boolean | no | Set to 1 to include each result's full listing details, for 2 credits per page instead of 1: a `details` object per result, identical to GET /item's response. Limits results to listings with full details available. See "Credits & pages". |

`attr` changes what matches, never the response shape. `details=1` adds one
field, `details`, to each result, and `"details": true` to the envelope.
Without `details=1` a result
is the sale only; the page data (specifics, description, images, seller) comes
from GET /item.

Example:

```bash
curl "https://api.trawl.dev/ebay/v1/sold?query=iphone+15+pro+256gb&condition=used&max_pages=5" \
  -H "x-api-key: $TRAWL_KEY"
```

Every result carries the fields below; `bids` is a number on auctions and
`null` otherwise, and `epid` (eBay's product id) is `null` when the listing
has none. If you omit `max_pages` you still get one page for 1 credit, but the
envelope reports `"page": 1` in place of `"max_pages"` — send `max_pages`
and ignore `page`.

Response shape (one result shown). Here 385 results came back, which is 4
pages of 100, so the call cost 4 of the 5 credits `max_pages=5` allowed:

```json
{
  "site": "EBAY_US",
  "currency": "USD",
  "query": ["iphone", "15", "pro"],
  "filters": { "condition": ["used"] },
  "max_pages": 5,
  "count": 385,
  "credits_charged": 4,
  "took_ms": 41,
  "results": [
    {
      "title": "Apple iPhone 15 Pro 256GB Unlocked",
      "sale_price": 525.00,
      "shipping_price": 0,
      "currency": "$",
      "condition": "used",
      "condition_raw": "Pre-Owned",
      "date_sold": "2026-07-18T00:00:00.000Z",
      "buying_format": "Buy It Now",
      "bids": null,
      "best_offer_available": true,
      "location": "United States",
      "item_id": "256637082114",
      "epid": "13051890211",
      "categoryId": "9355",
      "item_link": "https://www.ebay.com/itm/256637082114",
      "image_url": "https://i.ebayimg.com/images/g/abc/s-l500.webp"
    }
  ]
}
```

`condition_raw` is eBay's own wording and can be `null`: on older sales the
condition slot sometimes held seller advert text or a catalog attribute, and
anything that is not a condition is withheld. Newer sales pass eBay's text
through verbatim, including labels not listed here.

TRADING CARDS AND COINS: a /sold result says only "Pre-Owned" (or "New (Other)"
for a slab) — eBay's sold search carries nothing more. The card condition,
grader and grade are on the listing page: use `details=1` and read
`details.condition_raw` and `details.grading` (see GET /item). Do not infer
a card's condition from the top-level `condition_raw`.

### GET /item

One listing's page-level data: every item specific, the full description, all
images, seller detail, shipping and returns, plus that listing's recorded
sales. Costs 1 credit. Listing details become available a few minutes after a sale, so a
just-sold item answers 404 until then; a 404 is never billed — retry later
rather than treating it as final. A listing eBay has removed is different: it
answers a FREE 200, `{"site", "item_id", "listing_state": "removed",
"credits_charged": 0}`, with no other fields. That answer IS final — do not
retry it, and check `listing_state` before reading other fields.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| item_id | string | yes | The numeric eBay item id — the `item_id` of any /sold result. |
| site | string | no | Marketplace the item sold on: EBAY_US or EBAY_GB. Default EBAY_US. |

```bash
curl "https://api.trawl.dev/ebay/v1/item?item_id=256637082114" -H "x-api-key: $TRAWL_KEY"
```

```json
{
  "site": "EBAY_US",
  "item_id": "256637082114",
  "title": "Apple iPhone 13 Pro 256GB Graphite Unlocked",
  "condition": "used",
  "condition_raw": "Pre-Owned",
  "condition_description": "Light scratches on the frame, screen flawless.",
  "grading": null,
  "listing_state": "sold",
  "sold_at": "2026-07-18T21:14:00.000Z",
  "sale_price": 525.00,
  "currency": "US$",
  "buying_format": "Buy It Now",
  "returns_text": "30 days returns. Buyer pays for return shipping.",
  "seller_accepts_returns": true,
  "location": "Austin, Texas",
  "categoryId": "9355",
  "item_link": "https://www.ebay.com/itm/256637082114",
  "images": ["https://i.ebayimg.com/images/g/abc/s-l1600.webp"],
  "attributes": [
    { "key": "Brand", "value": "Apple", "values": ["Apple"] },
    { "key": "Storage Capacity", "value": "256 GB", "values": ["256 GB"] }
  ],
  "description_text": "Fully unlocked, battery health 91%. Comes with the original box.",
  "seller": { "username": "phone-depot", "feedback_percent": 99.6, "feedback_count": 6864 },
  "feedback": [{ "username": "b***y", "rating": "positive", "comment": "Exactly as described" }],
  "sales": [
    { "date_sold": "2026-07-18T00:00:00.000Z", "sale_price": 525.00, "shipping_price": 0, "currency": "$" }
  ],
  "credits_charged": 1
}
```

There is no shipping price on the listing itself: a listing page quotes shipping
to whoever is viewing it, which is not what the buyer paid. The shipping the
buyer paid is `shipping_price` on each entry of `sales` (and on every /sold
result).

`listing_state` is what the listing's page showed when its details were read:
`sold`, `active` or `ended_unsold`. `removed` means eBay has taken the
listing down: the response then has no other fields (see above).

AN `active` LISTING IS STILL A SOLD ITEM — READ `sales`. Every item_id trawl
holds sold at least once. `active` means the listing was still live when its
page was read: a multi-quantity listing with stock left (the usual case), or a
relist. A live page has no sold banner, so the page-level sale fields are
`null`: `sold_at`, `sale_price` and `best_offer_accepted`. There is no
top-level `date_sold` on /item at all. The sale dates and prices are in
`sales` — one entry per recorded sale, newest first, each with `date_sold`,
`sale_price`, `shipping_price` and `currency`. The same applies to
`ended_unsold`. Rule: for when and for how much, read `sales[]` (or the
/sold result); use the top-level `sold_at` / `sale_price` only when
`listing_state` is `sold`. Do not treat a null `sold_at` as "not sold".

`grading` is for trading cards and coins, `null` on everything else. eBay's
sold search says only "Pre-Owned" for these; the listing page says
"Graded - PSA 10" or "Ungraded - Near mint or better" (that string is
`condition_raw` here), and `grading` splits it:

```json
{ "graded": true,  "grader": "PSA", "grade": "10", "condition": null }
{ "graded": false, "grader": null,  "grade": null, "condition": "Near mint or better" }
```

`grading.condition` is the declared condition of an UNGRADED item only
("Near mint or better", "Lightly played (Excellent)", "Moderately played (Very
good)", "Heavily played"; coins: "Uncirculated" etc.) — a slab has none,
its grade is its condition. If a graded line has no trailing grade, the whole
tail is in `grader` and `grade` is null. `grade` is a string ("9.5",
"Authentic").

### GET /categories

Look up eBay leaf categories by name, busiest first — use the returned
`categoryId` as the `category` filter on /sold. Costs 1 credit, nothing if no
category matches. Category ids differ per
marketplace, so pass the same `site` you will search with.

Matching covers the category's full path, not just its own name — "collectible
card games" finds every CCG leaf. Note that eBay's taxonomy has NO per-brand or
per-sport categories: Pokémon or baseball cards live under generic categories
like `183454` (CCG Individual Cards) or `261328` (Trading Card Singles), with
the game/sport as a listing attribute. To narrow to a brand, combine the
generic `category` id with brand words in `query` on /sold.

| Parameter | Type | Required | Description |
| --- | --- | --- | --- |
| query | string | yes | Words to match against category names and their parent path — every word must appear, in any order. Case- and accent-insensitive. A numeric query is an id lookup instead: it returns that exact category (with a `leaf` flag — only leaf ids work as the /sold category filter). |
| site | string | no | Marketplace: EBAY_US or EBAY_GB. Default EBAY_US. |

```json
{
  "site": "EBAY_US",
  "total": 38,
  "count": 5,
  "categories": [
    { "categoryId": "261328", "name": "Trading Card Singles", "group": "Sports Mem, Cards & Fan Shop" },
    { "categoryId": "183050", "name": "Trading Card Singles", "group": "Collectibles" },
    { "categoryId": "261329", "name": "Trading Card Lots", "group": "Sports Mem, Cards & Fan Shop" },
    { "categoryId": "261332", "name": "Sealed Trading Card Boxes", "group": "Sports Mem, Cards & Fan Shop" },
    { "categoryId": "261330", "name": "Trading Card Sets", "group": "Sports Mem, Cards & Fan Shop" }
  ],
  "credits_charged": 1
}
```

## More APIs

More APIs land under the same base URL, key, and credit pool — this file is
updated as they ship. Users can request and vote on new data sources in the
roadmap section at https://trawl.dev.
