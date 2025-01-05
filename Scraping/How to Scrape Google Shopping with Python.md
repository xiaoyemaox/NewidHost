
# How to Scrape Google Shopping with Python

## Introduction

Scraping Google Shopping data can be a highly effective way to gather product details such as prices, ratings, and store information. Using Python, you can leverage libraries like `parsel` and `requests` to extract this data without relying on JavaScript-rendering tools like Selenium. This article provides a complete guide, with examples, to help you get started.

Stop wasting time on proxies and CAPTCHAs! ScraperAPI's simple API handles millions of web scraping requests, so you can focus on the data. Get structured data from Amazon, Google, Walmart, and more. 👉 [Start your free trial today!](https://bit.ly/Scraperapi)

---

## Key Steps to Scrape Google Shopping Data

1. **Set Up Your Environment**:
   Use the `requests` library to fetch HTML content and `parsel` to parse and extract the required elements from the HTML.

2. **Avoid Detection**:
   - Set custom headers to mimic a real browser. By default, `requests` uses the `python-requests` user-agent, which Google can easily detect and block.
   - Rotate user-agents to further reduce the chance of getting blocked. Here's a [list of user-agents](https://developers.whatismybrowser.com/useragents/explore/) you can use.

3. **Simplify CSS Selector Extraction**:
   - Use tools like [SelectorGadget](https://selectorgadget.com/) to identify the CSS selectors for the elements you want to scrape.

4. **Extract Data**:
   - Extract product data such as images, titles, prices, store names, and links using CSS selectors and regular expressions.

---

## Python Code to Scrape Google Shopping

Here’s a Python script that uses `requests` and `parsel` to scrape data from Google Shopping:

```python
import requests
import json
import re
from parsel import Selector

# Set parameters for the Google Shopping search
params = {
    "q": "minecraft",  # Search query
    "hl": "en",        # Language
    "gl": "us",        # Country
    "tbm": "shop"      # Shopping tab
}

# Set custom headers
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36"
}

# Fetch the HTML content
response = requests.get("https://www.google.com/search", params=params, headers=headers, timeout=30)
selector = Selector(response.text)

# Extract images and other product data
def get_original_images():
    all_script_tags = "".join(
        [
            script.replace("</script>", "</script>\n")
            for script in selector.css("script").getall()
        ]
    )
    
    image_urls = []
    for result in selector.css(".Qlx7of .sh-dgr__grid-result"):
        # Extract image URLs using regex
        url_with_unicode = re.findall(rf"var\s?_u='(.*?)';var\s?_i='{result.attrib['data-pck']}';", all_script_tags)
        if url_with_unicode:
            url_decode = bytes(url_with_unicode[0], 'ascii').decode('unicode-escape')
            image_urls.append(url_decode)
    return image_urls

def get_suggested_search_data():
    google_shopping_data = []
    for result, thumbnail in zip(selector.css(".Qlx7of .i0X6df"), get_original_images()):
        title = result.css(".tAxDx::text").get()
        product_link = "https://www.google.com" + result.css(".Lq5OHe::attr(href)").get()
        price = result.css(".a8Pemb::text").get()
        store = result.css(".aULzUe::text").get()

        google_shopping_data.append({
            "title": title,
            "product_link": product_link,
            "price": price,
            "store": store,
            "thumbnail": thumbnail
        })

    print(json.dumps(google_shopping_data, indent=2, ensure_ascii=False))

# Run the function
get_suggested_search_data()
```

---

## Using ScraperAPI for Simplified Web Scraping

If you’re looking for an easier and more robust solution, consider using ScraperAPI. It automatically handles proxy rotation, CAPTCHAs, and bot detection, allowing you to focus on extracting data.

### Code Example with ScraperAPI

```python
from serpapi import GoogleSearch
import os
import json

# Set up parameters for ScraperAPI
params = {
    "q": "minecraft",
    "tbm": "shop",
    "location": "Dallas",
    "hl": "en",
    "gl": "us",
    "api_key": os.getenv("API_KEY")  # Replace with your ScraperAPI key
}

search = GoogleSearch(params)
results = search.get_dict()

# Extract shopping results
shopping_results = results.get("shopping_results", [])
print(json.dumps(shopping_results, indent=2, ensure_ascii=False))
```

---

## Sample Output

The script outputs JSON data for each product found on Google Shopping:

```json
[
  {
    "title": "Minecraft Mini Mob 4-Piece Figure Mood Light Set",
    "product_link": "https://www.google.com/shopping/product/15256303704867209410",
    "price": "$29.99",
    "store": "Oriental Trading Company",
    "thumbnail": "https://encrypted-tbn1.gstatic.com/shopping?q=tbn:ANd9GcS7Xddy..."
  },
  {
    "title": "Minecraft Explorer Kit - Build Minecraft in The Real World",
    "product_link": "https://www.google.com/shopping/product/10073223339448590299",
    "price": "$99.99",
    "store": "Make-A-Fort",
    "thumbnail": "https://encrypted-tbn2.gstatic.com/shopping?q=tbn:ANd9GcTJ..."
  }
]
```

---

## Conclusion

Scraping Google Shopping with Python is highly effective using tools like `requests`, `parsel`, and ScraperAPI. For basic scraping, the Python script provided above is a great start. However, for more complex and large-scale projects, a service like [ScraperAPI](https://bit.ly/Scraperapi) simplifies the process by handling proxies, CAPTCHAs, and more.

👉 [Start your free trial with ScraperAPI today!](https://bit.ly/Scraperapi)
