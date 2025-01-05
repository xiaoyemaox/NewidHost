
# How to Scrape Instagram in 2025

![How to Scrape Instagram in 2024](https://scrapfly.io/blog/content/images/how-to-scrape-instagram_banner_light.svg)

## Introduction

In this Python web scraping tutorial, we’ll explore how to scrape [Instagram](https://www.instagram.com/)—one of the most popular social media platforms. We'll walk you through creating an Instagram scraper to extract data from profiles and posts, focusing on using the unofficial Instagram API while avoiding login requirements and handling Instagram’s web scraping blocks. Let’s dive in!

> **Legal Disclaimer**: This tutorial is for educational purposes only. Always respect the website’s terms of service and applicable legal frameworks. Avoid scraping private or non-public data, adhere to local privacy laws like GDPR, and consult a legal expert if necessary.

---

## Why Scrape Instagram?

Instagram is a goldmine of public data, which can be used for:

- **Lead Generation**: Identifying influencers or profiles with similar audiences to expand your business reach.
- **Sentiment Analysis**: Analyzing public posts and comments for trends, opinions, and reactions to current events.

Stop wasting time on proxies and CAPTCHAs! ScraperAPI’s simple API handles millions of web scraping requests, so you can focus on the data. Get structured data from Instagram, Amazon, Google, Walmart, and more. 👉 [Start your free trial today!](https://bit.ly/Scraperapi)

For more examples of Instagram scraping use cases, check out our [dedicated guide](https://bit.ly/Scraperapi).

---

## Setting Up Your Instagram Scraper

To create an Instagram scraper, you’ll need Python and the following libraries:

- [`httpx`](https://pypi.org/project/httpx/): To send HTTP requests and retrieve data.
- [`JMESPath`](https://pypi.org/project/jmespath/): To parse and clean JSON data.

For advanced scraping at scale, we’ll also demonstrate how to integrate the [ScraperAPI Python SDK](https://bit.ly/Scraperapi).

---

## Scraping Instagram Profile Pages

### Step 1: Extracting Profile Data
When visiting a profile page, Instagram triggers a backend API call to retrieve the profile’s JSON data. We can use this API to scrape publicly accessible profile information without logging in.

Here’s how to set up a scraper for Instagram profile pages:

```python
import httpx

def scrape_instagram_profile(username):
    url = f"https://www.instagram.com/{username}/?__a=1"
    headers = {"User-Agent": "Mozilla/5.0"}
    response = httpx.get(url, headers=headers)
    return response.json()

data = scrape_instagram_profile("google")
print(data)
```

### Example Output
```json
{
  "biography": "Google unfiltered—sometimes with filters.",
  "external_url": "https://linkin.bio/google",
  "edge_followed_by": {"count": 13015078},
  "edge_follow": {"count": 33},
  "is_verified": true,
  "profile_pic_url": "https://example.com/profile_pic.jpg",
  "username": "google",
  "full_name": "Google"
}
```

This script retrieves details like bio, follower count, profile picture URL, and verification status.

---

## Parsing Instagram Profile Data

To clean up the raw JSON data and extract only the relevant fields, we can use JMESPath:

```python
import jmespath

def parse_profile_data(data):
    return jmespath.search("""
        {
            username: username,
            full_name: full_name,
            bio: biography,
            followers: edge_followed_by.count,
            following: edge_follow.count,
            profile_pic: profile_pic_url,
            is_verified: is_verified
        }
    """, data)

parsed_data = parse_profile_data(data)
print(parsed_data)
```

### Parsed Output
```json
{
  "username": "google",
  "full_name": "Google",
  "bio": "Google unfiltered—sometimes with filters.",
  "followers": 13015078,
  "following": 33,
  "profile_pic": "https://example.com/profile_pic.jpg",
  "is_verified": true
}
```

---

## Scraping Instagram Posts

### Step 1: Extracting Post Data
Instagram uses GraphQL to load post data dynamically. We can send POST requests to its GraphQL endpoint to retrieve details for each post.

Here’s an example GraphQL query for post data:

```python
import httpx

def scrape_instagram_post(shortcode):
    url = "https://www.instagram.com/graphql/query/"
    params = {
        "query_hash": "2c5d4d8b70cad329c4a6ebe3abb6eedd",
        "variables": {"shortcode": shortcode}
    }
    headers = {"User-Agent": "Mozilla/5.0"}
    response = httpx.get(url, params=params, headers=headers)
    return response.json()

post_data = scrape_instagram_post("CJ9KxZ2l8jT")
print(post_data)
```

### Step 2: Parsing Post Data
To simplify post data, we can parse it with JMESPath:

```python
def parse_post_data(data):
    return jmespath.search("""
        {
            id: id,
            shortcode: shortcode,
            caption: edge_media_to_caption.edges[0].node.text,
            likes: edge_media_preview_like.count,
            comments: edge_media_to_comment.count,
            is_video: is_video,
            video_url: video_url,
            display_url: display_url
        }
    """, data)

parsed_post = parse_post_data(post_data)
print(parsed_post)
```

---

## Avoiding Instagram Blocking with ScraperAPI

Instagram imposes strict rate limits and blocks scraping attempts that exceed the limit. To bypass these restrictions, we can use [ScraperAPI](https://bit.ly/Scraperapi) to manage proxies and CAPTCHA challenges.

Here’s how to enhance your scraper with ScraperAPI:

```python
from scrapfly import ScrapflyClient, ScrapeConfig

scrapfly = ScrapflyClient(key="YOUR_API_KEY")

response = scrapfly.scrape(ScrapeConfig(
    url="https://www.instagram.com/google/?__a=1",
    asp=True,  # Anti-scraping protection
    country="US",
    render_js=False  # Disable JS rendering unless needed
))

print(response.scrape_result['content'])
```

---

## Frequently Asked Questions

### Is web scraping Instagram legal?
Yes, scraping publicly available data at respectful rates is generally legal. However, you must adhere to Instagram’s terms of service and local privacy laws, such as GDPR.

### Can I scrape Instagram without logging in?
Yes, this guide demonstrates how to scrape public Instagram data without requiring login credentials, avoiding potential risks like account suspension.

### How do I get an Instagram user ID from their username?
You can scrape the user’s profile and extract the `id` field:

```python
data = scrape_instagram_profile("google")
user_id = data["graphql"]["user"]["id"]
print(user_id)
```

---

## Final Thoughts

Web scraping Instagram opens up numerous possibilities, from gathering valuable insights to automating data analysis. In this guide, we covered how to scrape Instagram profiles, posts, and hashtags while avoiding blocks using [ScraperAPI](https://bit.ly/Scraperapi).

Happy scraping, and remember to scrape responsibly!

👉 [Start using ScraperAPI today!](https://bit.ly/Scraperapi)
