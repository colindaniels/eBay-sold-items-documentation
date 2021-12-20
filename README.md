<h3 align="center">eBay Completed Items API</h3>


---

<p align="center"> This API is used to receive data on recently sold eBay listings by scraping eBay.com
Refine the request by filtering by keywords, excluded phrases, category, and aspect values. Aspect values are category-specific parameters used to search for more precise products. For example: model, condition, carrier, storage.
You will receive the number of total results, the average price of all products searched for, the minimum price, the maximum price, and the price and more information on each individual product.
    <br> 
</p>

## 📝 Table of Contents

- [Getting Started](#getting_started)
- [Routes](#routes)
- [Code Examples](#code_examples)
- [Troubleshooting](#troubleshooting)
- [Built Using](#built_using)


## 🏁 Getting Started <a name = "getting_started"></a>
This API is a POST methods and requires body data.

### Request URL
The URL you will be getting data from

```
POST https://ebay-average-selling-price.p.rapidapi.com/findCompletedItems
```

## Routes <a name = "routes"></a>

Returns all data for completed sales
```
/findCompletedItems
```
### Request

#### Headers
* content-type: application/json
* x-rapidapi-host: ebay-average-selling-price.p.rapidapi.com
* x-rapidapi-key: */*YOUR KEY/**

#### Body Data
* **keywords**
    * Description: Keywords entered into the eBay searchbar to refine results. Seperated by spaces
    * Example: "iPhone"
    * Required: Yes
<br/>

* **max_search_results**
    * Description: Maximum amount of search results. **Only allowed values:** 25, 50, 100, 200
    * Example: 200,
    * Required: Yes
<br/>

* **excluded_keywords**
    * Description: Phrases you would like to exclude from the search. Seperated by spaces
    * Example: "locked cracked case box read"
    * Required: No
<br/>

* **category_id**
    * Description: A unique ID given to seperate categories of products on eBay. To find a category ID to use, navigate to: https://www.isoldwhat.com/
    * Example: "9355" - Cell Phones & Smartphones
    * Required: No (**HIGHLY RECOMMENDED**)
<br/>

* **aspects**
    * Description: A list of eBay aspects to help refile search results. For each aspect, the only allwoed parameters are: "name" and "value" and values for each are required. These aspects are the same as the filters you see on the eBay site when looking for spicific models or colors of products.
    * Example: 
    ```json
    [
        { 
            "name": "Model", 
            "value": "Apple iPhone X" 
        },
        {
            "name": "Storage Capacity",
            "value": "64 GB"
        }
    ]
    ```
    * Required: No
<br/>

### Response
#### Example
```json
{
    "success": true,
    "average_price": 271.13,
    "min_price": 120,
    "max_price": 369.26,
    "results": 173,
    "response_url": "https://www.ebay.com/sch/9335/i.html?_nkw=iPhone+-locked+-cracked+-case+-box+-read&LH_Sold=1&LH_Complete=1&_ipg=200&Model=Apple%2520iPhone%2520X&LH_ItemCondition=3000&Network=Unlocked&Storage%2520Capacity=64%2520GB",
    "products": 
    [
        {
            "title": "Apple iPhone X - 64GB - Space Gray (Unlocked) A1901 (GSM) (CA)",
            "sale_price": 237.5,
            "date_sold": "Dec 8, 2021",
            "link": "https://www.ebay.com/itm/353792150540?epid=239093115&hash=item525fa7cc0c%3Ag%3A%7EuMAAOSwmehhp4oM&LH_ItemCondition=3000"
        },
        {
            "title": "Apple iPhone X Space Gray 64GB Unlocked(GSM+CDMA) G208",
            "sale_price": 305,
            "date_sold": "Dec 8, 2021",
            "link": "https://www.ebay.com/itm/255262751845?epid=16032638976&hash=item3b6ed87c65%3Ag%3AAsgAAOSwFtVhrMCq&LH_ItemCondition=3000"
        },
        {
            "title": "Apple iPhone X Space Gray 64GB Unlocked(GSM+CDMA) G210",
            "sale_price": 276,
            "date_sold": "Dec 8, 2021",
            "link": "https://www.ebay.com/itm/255262748698?epid=16032638976&hash=item3b6ed8701a%3Ag%3AivAAAOSwcBphrL9u&LH_ItemCondition=3000"
        }
        ...
    ]
```


## Code Examples <a name = "code_examples"></a>

##### cURL
```curl
curl --location --request POST 'https://ebay-average-selling-price.p.rapidapi.com/findCompletedItems' \
--header 'Content-Type: application/json' \
--data-raw '{
    "keywords": "iPhone",
    "excluded_keywords": "locked cracked case box read",
    "max_search_results": "200",
    "category_id": "9355",
    "aspects": 
    [
        {
            "name": "Model",
            "value": "Apple iPhone X"
        },
        {
            "name": "LH_ItemCondition",
            "value": "3000"
        },
        {
            "name": "Network",
            "value": "Unlocked"
        },
        {
            "name": "Storage Capacity",
            "value": "64 GB"            
        }
    ]
}'
```
<br/>

##### NodeJS - Request
```javascript
var request = require('request');
var options = {
  'method': 'POST',
  'url': 'https://ebay-average-selling-price.p.rapidapi.com/findCompletedItems',
  'headers': {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    "keywords": "iPhone",
    "excluded_keywords": "locked cracked case box read",
    "max_search_results": "200",
    "category_id": "9355",
    "aspects": [
      {
        "name": "Model",
        "value": "Apple iPhone X"
      },
      {
        "name": "LH_ItemCondition",
        "value": "3000"
      },
      {
        "name": "Network",
        "value": "Unlocked"
      },
      {
        "name": "Storage Capacity",
        "value": "64 GB"
      }
    ]
  })

};
request(options, function (error, response) {
  if (error) throw new Error(error);
  console.log(response.body);
});

```
<br/>

##### Python - Requests
```python
import requests
import json

url = "https://ebay-average-selling-price.p.rapidapi.com/findCompletedItems"

payload = json.dumps({
  "keywords": "iPhone",
  "excluded_keywords": "locked cracked case box read LCD",
  "max_search_results": "200",
  "category_id": "9355",
  "aspects": [
    {
      "name": "Model",
      "value": "Apple iPhone X"
    },
    {
      "name": "LH_ItemCondition",
      "value": "3000"
    },
    {
      "name": "Network",
      "value": "Unlocked"
    },
    {
      "name": "Storage Capacity",
      "value": "64 GB"
    }
  ]
})
headers = {
  'Content-Type': 'application/json'
}

response = requests.request("POST", url, headers=headers, data=payload)

print(response.text)
```


## 🔧 Troubleshooting <a name = "troubleshooting"></a>
#### Aspects
Some aspects on eBay have a different url value then what is shown on the site. If the aspect is not working, make sure that aspect can be used with the category you have selected. Then visit the response_url and select the aspect you want shown. Navigate to the url bar and look for a substring where the new aspect was updated. For example eBay shows "condition" as one of the aspect names, but the actual value in the url is "LH_ItemCondition", and the valeus are condition id's and not condition names.

#### Conditions
Conditions on eBay have correlated ids. These ids can be found here: https://developer.ebay.com/devzone/finding/callref/enums/conditionIdList.html
Not all conditions work for every category. Make sure you visit the response_url to see what aspects you can use.

#### Anything else
Reach out to us through RapidAPI and we will update our API and help you with anything.


## ⛏️ Built Using <a name = "built_using"></a>
- [Express](https://expressjs.com/) - Server Framework
- [NodeJs](https://nodejs.org/en/) - Server Environment
- [Axios](https://axios-http.com) - Server Requests
- [Cheerio](https://cheerio.js.org/) - HTML Parsing

## ✍️ Authors <a name = "authors"></a>
- [@colindaniels](https://github.com/colindaniels) - Idea & All Work

