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

* **remove_outliers**
    * Description: If set to true, it will remove all outliers with prices way too high or low from the results.
    * Example: "false"
    * Required: No
<br/>

* **site_id**
    * Description: The ebay site ID that eBay will use. Different terretories use different sites. 
To find site ID values, navigate to: https://developer.ebay.com/devzone/finding/callref/Enums/GlobalIdList.html
    * Example: "0" - United States
    * Required: No - (defaults to "0")
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
    "average_price": 226.47,
    "median_price": 220.5,
    "min_price": 149.99,
    "max_price": 319,
    "results": 189,
    "response_url": "https://www.ebay.com/sch/9355/i.html?_nkw=iPhone+-locked+-cracked+-case+-box+-read+-LCD+-face&LH_Sold=1&LH_Complete=1&_ipg=240&Model=Apple%2520iPhone%2520X&LH_ItemCondition=3000&Network=Unlocked&Storage%2520Capacity=256%2520GB",
    "products": [
        {
            "title": "Apple iPhone X - 256GB - (Unlocked)  - Works Great - (#8076)",
            "sale_price": 211.99,
            "date_sold": "Jun 11, 2022",
            "link": "https://www.ebay.com/itm/185450226888?hash=item2b2db1e4c8%3Ag%3AEmwAAOSw8DJierSB&LH_ItemCondition=3000"
        },
        {
            "title": "Apple iPhone X 256GB Unlocked Silver White A1901 iOS 13.6",
            "sale_price": 269,
            "date_sold": "Jun 11, 2022",
            "link": "https://www.ebay.com/itm/115419688300?epid=239160803&hash=item1adf8cad6c%3Ag%3ASQkAAOSwUWRiiTwo&LH_ItemCondition=3000"
        },
        {
            "title": "Apple iPhone X GSM Smartphone - Space Gray/256GB/Unlocked",
            "sale_price": 207.5,
            "date_sold": "Jun 11, 2022",
            "link": "https://www.ebay.com/itm/394105212129?epid=28039771047&hash=item5bc2804ce1%3Ag%3AGusAAOSw-cpinkmi&LH_ItemCondition=3000"
        },
        ...
    ]
```


## Code Examples <a name = "code_examples"></a>

##### cURL
```curl
curl --location --request POST 'https://ebay-sold-items-api.herokuapp.com/findCompletedItems' \
--header 'Content-Type: application/json' \
--data-raw '{
    "keywords": "iPhone",
    "excluded_keywords": "locked cracked case box read LCD face",
    "max_search_results": "240",
    "category_id": "9355",
    "remove_outliers": true,
    "site_id": "0",
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
            "value": "256 GB"
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
  'url': 'https://ebay-sold-items-api.herokuapp.com/findCompletedItems',
  'headers': {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    "keywords": "iPhone",
    "excluded_keywords": "locked cracked case box read LCD face",
    "max_search_results": "240",
    "category_id": "9355",
    "remove_outliers": true,
    "site_id": "0",
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
        "value": "256 GB"
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

url = "https://ebay-sold-items-api.herokuapp.com/findCompletedItems"

payload = json.dumps({
  "keywords": "iPhone",
  "excluded_keywords": "locked cracked case box read LCD face",
  "max_search_results": "240",
  "category_id": "9355",
  "remove_outliers": True,
  "site_id": "0",
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
      "value": "256 GB"
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

#### Long request time
If you've noticed that every once in a while your requests will take well over 10 seconds, there is a reason for that. eBay requires a captcha to be solved every few hours when requesting data for sold items. When our server detects a captcha is required, it will automatically solve it and send you the results for the data once it is available, but this can take a few seconds to complete. A captcha is only required once every few hours, but should not affect the success rate or results of the data.

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
- [Puppeteer](https://www.npmjs.com/package/puppeteer) - Browser Automation

## ✍️ Authors <a name = "authors"></a>
- [@colindaniels](https://github.com/colindaniels) - Idea & All Work

