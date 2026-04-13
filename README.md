# eBay Completed Items API

Retrieve data on recently sold eBay listings. Refine results by keywords, excluded phrases, category, aspects, and eBay territory. Returns pricing statistics (average, median, min, max) and detailed information on each sold product including price, currency, condition, buying format, and more.

## Table of Contents

- [Getting Started](#getting-started)
- [Routes](#routes)
- [Parameters](#parameters)
- [Site IDs](#site-ids)
- [Response](#response)
- [Code Examples](#code-examples)
- [Troubleshooting](#troubleshooting)

## Getting Started

This API uses POST requests with JSON body data.

### Request URL

```
POST https://ebay-api.scrapechain.com/findCompletedItems
```

### Headers

```
Content-Type: application/json
```

## Routes

### `/findCompletedItems`

Returns completed/sold item data with pricing statistics.

## Parameters

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `keywords` | string | Yes | - | Search keywords separated by spaces |
| `max_search_results` | number | No | `60` | Maximum results. Allowed values: `60`, `120`, `240` |
| `excluded_keywords` | string | No | - | Words to exclude from search, separated by spaces |
| `category_id` | string | No | - | eBay category ID. Find IDs at [isoldwhat.com](https://www.isoldwhat.com/). **Highly recommended** for accurate results |
| `remove_outliers` | boolean | No | `false` | Remove products with prices that are statistical outliers |
| `site_id` | string | No | `"0"` | eBay territory site ID. See [Site IDs](#site-ids) below |
| `aspects` | array | No | - | Category-specific filters. See [Aspects](#aspects) below |

### Aspects

Aspects are category-specific filters matching eBay's sidebar filters (model, storage, condition, etc.). Each aspect requires a `name` and `value`.

```json
[
    { "name": "Model", "value": "Apple iPhone X" },
    { "name": "Storage Capacity", "value": "256 GB" }
]
```

## Site IDs

Different eBay territories use different site IDs. Each territory has its own domain, currency, and locale.

| Site ID | Country | Domain | Currency |
|---|---|---|---|
| `0` | United States | ebay.com | USD |
| `2` | Canada (English) | ebay.ca | C |
| `3` | United Kingdom | ebay.co.uk | £ |
| `15` | Australia | ebay.com.au | AU |
| `16` | Austria | ebay.at | EUR |
| `23` | Belgium (French) | befr.ebay.be | EUR |
| `71` | France | ebay.fr | EUR |
| `77` | Germany | ebay.de | EUR |
| `101` | Italy | ebay.it | EUR |
| `123` | Belgium (Dutch) | benl.ebay.be | EUR |
| `146` | Netherlands | ebay.nl | EUR |
| `186` | Spain | ebay.es | EUR |
| `193` | Switzerland | ebay.ch | CHF |
| `201` | Hong Kong | ebay.com.hk | HK$ |
| `203` | Ireland | ebay.ie | EUR |
| `205` | Ireland | ebay.ie | EUR |
| `207` | Malaysia | ebay.com.my | RM |
| `210` | Canada (French) | cafr.ebay.ca | $C |
| `211` | Philippines | ebay.ph | PHP |
| `212` | Poland | ebay.pl | zł |
| `216` | Singapore | ebay.com.sg | S$ |

## Response

### Fields

| Field | Type | Description |
|---|---|---|
| `success` | boolean | Whether the request was successful |
| `average_price` | number | Average sale price across all results |
| `median_price` | number | Median sale price |
| `min_price` | number | Lowest sale price |
| `max_price` | number | Highest sale price |
| `results` | number | Number of products returned |
| `total_results` | number | Total results found on eBay |
| `response_url` | string | The eBay URL that was scraped |
| `products` | array | Array of sold products |

### Product Fields

| Field | Type | Description |
|---|---|---|
| `title` | string | Listing title |
| `sale_price` | number | Price the item sold for |
| `currency` | string \| null | Currency symbol from the listing (e.g. `$`, `£`, `EUR`) |
| `condition` | string | Item condition |
| `buying_format` | string \| null | `"Auction"`, `"Buy It Now"`, or `"Accepts Offers"` |
| `date_sold` | string | Date the item was sold |
| `image_url` | string \| null | Product image URL |
| `shipping_price` | number \| null | Shipping cost (`0` for free shipping) |
| `link` | string | URL to the eBay listing |
| `item_id` | string \| null | eBay item ID |

### Example Response

```json
{
    "success": true,
    "average_price": 226.47,
    "median_price": 220.50,
    "min_price": 149.99,
    "max_price": 319.00,
    "results": 189,
    "total_results": 2400,
    "response_url": "https://www.ebay.com/sch/9355/i.html?_nkw=iPhone&LH_Sold=1&LH_Complete=1&_ipg=240",
    "products": [
        {
            "title": "Apple iPhone X - 256GB - (Unlocked) - Works Great",
            "sale_price": 211.99,
            "currency": "$",
            "condition": "Pre-Owned",
            "buying_format": "Buy It Now",
            "date_sold": "Jun 11, 2022",
            "image_url": "https://i.ebayimg.com/images/g/.../s-l500.jpg",
            "shipping_price": 0,
            "link": "https://www.ebay.com/itm/185450226888",
            "item_id": "185450226888"
        },
        {
            "title": "Apple iPhone X 256GB Unlocked Silver White",
            "sale_price": 269.00,
            "currency": "$",
            "condition": "Pre-Owned",
            "buying_format": "Auction",
            "date_sold": "Jun 11, 2022",
            "image_url": "https://i.ebayimg.com/images/g/.../s-l500.jpg",
            "shipping_price": 5.99,
            "link": "https://www.ebay.com/itm/115419688300",
            "item_id": "115419688300"
        }
    ]
}
```

## Code Examples

### cURL

```bash
curl -X POST 'https://ebay-api.scrapechain.com/findCompletedItems' \
  -H 'Content-Type: application/json' \
  -d '{
    "keywords": "iPhone",
    "excluded_keywords": "locked cracked case box read",
    "max_search_results": 240,
    "category_id": "9355",
    "remove_outliers": true,
    "site_id": "0",
    "aspects": [
        { "name": "Model", "value": "Apple iPhone X" },
        { "name": "LH_ItemCondition", "value": "3000" },
        { "name": "Network", "value": "Unlocked" },
        { "name": "Storage Capacity", "value": "256 GB" }
    ]
}'
```

### Python

```python
import requests

url = "https://ebay-api.scrapechain.com/findCompletedItems"

response = requests.post(url, json={
    "keywords": "iPhone",
    "excluded_keywords": "locked cracked case box read",
    "max_search_results": 240,
    "category_id": "9355",
    "remove_outliers": True,
    "site_id": "0",
    "aspects": [
        {"name": "Model", "value": "Apple iPhone X"},
        {"name": "LH_ItemCondition", "value": "3000"},
        {"name": "Network", "value": "Unlocked"},
        {"name": "Storage Capacity", "value": "256 GB"}
    ]
})

print(response.json())
```

### JavaScript (axios)

```javascript
const axios = require('axios');

const response = await axios.post('https://ebay-api.scrapechain.com/findCompletedItems', {
    keywords: 'iPhone',
    excluded_keywords: 'locked cracked case box read',
    max_search_results: 240,
    category_id: '9355',
    remove_outliers: true,
    site_id: '0',
    aspects: [
        { name: 'Model', value: 'Apple iPhone X' },
        { name: 'LH_ItemCondition', value: '3000' },
        { name: 'Network', value: 'Unlocked' },
        { name: 'Storage Capacity', value: '256 GB' }
    ]
});

console.log(response.data);
```

## Troubleshooting

### Aspects not working

Some eBay aspects have a different URL value than what is shown on the site. If an aspect is not working:

1. Make sure the aspect is available for the category you selected
2. Visit the `response_url` from your API response
3. Select the aspect filter you want on eBay's site
4. Check the URL bar for the actual parameter name (e.g. eBay shows "Condition" but the URL uses `LH_ItemCondition`)

### Conditions

Conditions on eBay use numeric IDs. Common values:

- `1000` - New
- `1500` - Open Box
- `2000` - Refurbished
- `2500` - Seller Refurbished
- `3000` - Used
- `7000` - For Parts

Use these as aspect values: `{ "name": "LH_ItemCondition", "value": "3000" }`
