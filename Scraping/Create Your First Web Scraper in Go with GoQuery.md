
# Create Your First Web Scraper in Go with GoQuery

Web scraping has long been a popular topic in programming. While Python has been the go-to language for many web scraping projects—using libraries like `requests`, Selenium, and Scrapy—scraping is not limited to Python. Go, a statically typed, compiled language, also offers excellent tools for building efficient, scalable web scrapers. Thanks to features like **Goroutines**, Go can scrape hundreds of web pages concurrently, making it an ideal choice for data-intensive scraping tasks.

In this tutorial, we’ll introduce Go’s HTTP library and the **GoQuery** library, which functions like jQuery for HTML parsing. By the end, you’ll learn how to write a basic web scraper in Go and create a practical scraper for Yelp listings.

---

## Why Choose Go for Web Scraping?

Go's strengths for web scraping include:

- **Concurrency**: Built-in support for Goroutines allows efficient scraping of multiple web pages in parallel.
- **Compiled Code**: Distribute your scraper as a standalone executable without requiring a runtime environment.
- **Scalability**: Handle large-scale scraping tasks with high performance.

Stop wasting time on proxies and CAPTCHAs! ScraperAPI's simple API handles millions of web scraping requests, so you can focus on the data. Get structured data from Amazon, Google, Walmart, and more. 👉 [Start your free trial today!](https://bit.ly/Scraperapi)

---

## Building a Basic Web Scraper in Go

The first step is to familiarize yourself with Go's built-in `net/http` library. Let’s create a simple program that fetches the HTML content of a web page.

### Example: Fetching a Web Page with Go’s HTTP Library

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
)

func getListing(listingURL string) {
	response, err := http.Get(listingURL)
	if err != nil {
		fmt.Println(err)
		return
	}
	defer response.Body.Close()

	if response.StatusCode == 200 {
		bodyText, err := ioutil.ReadAll(response.Body)
		if err != nil {
			fmt.Println(err)
		}
		fmt.Printf("%s\n", bodyText)
	}
}

func main() {
	getListing("http://adnansiddiqi.me")
}
```

### How It Works:
1. The `getListing` function makes an HTTP GET request to the provided URL using `http.Get`.
2. It reads and prints the HTML content of the response using `ioutil.ReadAll`.
3. Default headers, timeouts, and user-agent settings are used, which may cause issues when scraping certain websites.

---

## Writing a Real-World Yelp Scraper

Let’s build a scraper to extract Yelp listings for **Mobile Phone Repair Shops in San Francisco**. This scraper will:
- Set custom headers to mimic a browser.
- Use the `goquery` library for HTML parsing.

### Yelp Scraper Code

```go
package main

import (
	"fmt"
	"net/http"
	"strings"
	"time"

	"github.com/PuerkitoBio/goquery"
)

func getListing(listingURL string) []string {
	var links []string

	// HTTP client with timeout
	client := &http.Client{
		Timeout: 30 * time.Second,
	}

	// Create a new HTTP request
	request, err := http.NewRequest("GET", listingURL, nil)
	if err != nil {
		fmt.Println(err)
		return links
	}

	// Set headers
	request.Header.Set("pragma", "no-cache")
	request.Header.Set("cache-control", "no-cache")
	request.Header.Set("dnt", "1")
	request.Header.Set("upgrade-insecure-requests", "1")
	request.Header.Set("referer", "https://www.yelp.com/")

	// Make the request
	resp, err := client.Do(request)
	if err != nil {
		fmt.Println(err)
		return links
	}
	defer resp.Body.Close()

	if resp.StatusCode == 200 {
		// Parse HTML with goquery
		doc, err := goquery.NewDocumentFromReader(resp.Body)
		if err != nil {
			fmt.Println(err)
			return links
		}

		// Extract links using a CSS selector
		doc.Find(".lemon--ul__373c0__1_cxs a").Each(func(i int, s *goquery.Selection) {
			link, _ := s.Attr("href")
			if strings.Contains(link, "biz/") {
				link = "https://yelp.com" + link
				links = append(links, link)
			}
		})
	}

	return links
}

func main() {
	links := getListing("https://www.yelp.com/search?find_desc=Mobile+Phone+Repair&find_loc=San+Francisco%2C+CA")
	for _, link := range links {
		fmt.Println(link)
	}
}
```

### Key Steps in the Yelp Scraper:
1. **HTTP Client**: A custom client is created with a 30-second timeout.
2. **Custom Headers**: Headers are set to avoid detection as a bot.
3. **HTML Parsing**: The `goquery` library extracts links from the HTML using CSS selectors.
4. **Filter Results**: Links are filtered to include only those relevant to Yelp business pages.

---

## Handling Anti-Scraping Mechanisms

Many websites, including Yelp, implement anti-scraping measures like CAPTCHAs, IP blocking, and rate limiting. To bypass these challenges:

- **Rotate Proxies**: Use a pool of proxies to avoid IP bans.
- **Set User-Agent**: Mimic a browser by setting a realistic User-Agent header.
- **Use ScraperAPI**: Simplify scraping with ScraperAPI’s built-in proxy rotation and CAPTCHA bypass.

Stop struggling with scraping challenges! Use ScraperAPI to scrape websites seamlessly without worrying about IP blocks or browser emulation. 👉 [Try it now!](https://bit.ly/Scraperapi)

---

## Advantages of Using Go for Web Scraping

1. **Performance**: Go’s compiled nature and lightweight Goroutines make it highly efficient for concurrent tasks.
2. **Ease of Distribution**: Distribute your scraper as a single executable without requiring dependencies.
3. **Scalability**: Handle large-scale scraping with minimal resources.

---

## Conclusion

This introductory tutorial demonstrates how to build web scrapers in Go using the `net/http` library and the `goquery` package. Go’s concurrency and performance make it a powerful choice for building scalable scrapers. Whether you’re scraping small datasets or managing large-scale operations, Go has you covered.

### Simplify Scraping with ScraperAPI

If you want to avoid the hassles of IP bans and CAPTCHAs, ScraperAPI is the perfect solution. It provides:

- Proxy management.
- Automatic CAPTCHA bypass.
- Seamless integration with Go, Python, and other languages.

👉 [Start your free trial today!](https://bit.ly/Scraperapi)

### Additional Resources
- [GoQuery Documentation](https://github.com/PuerkitoBio/goquery)
- [Yelp Scraper Code on GitHub](https://github.com/kadnan/yelp-scraper-go)
