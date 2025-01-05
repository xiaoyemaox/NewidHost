
# Building a Web Scraping API with Java, Spring Boot, and Jsoup

## Introduction

In this tutorial, we will build an API to extract vehicle advertisements from two popular websites. The API will scrape data based on the vehicle model provided and return a consolidated list of ads, enabling easy integration with a UI. This approach simplifies browsing ads across multiple platforms by displaying all relevant results in one place.

Stop wasting time on proxies and CAPTCHAs! ScraperAPI's simple API handles millions of web scraping requests, so you can focus on the data. Get structured data from Amazon, Google, Walmart, and more. 👉 [Start your free trial today!](https://bit.ly/Scraperapi)

---

## Tools and Frameworks Used

To build this API, we will use the following tools and libraries:

- **Spring Boot**: A Java-based framework for building production-ready web applications.
- **Lombok**: A Java library that reduces boilerplate code by providing annotations for commonly used methods like getters and setters.
- **Jsoup**: A Java library for parsing HTML and extracting data from web pages.
- **Apache Commons**: A utility library for handling strings and other Java utilities.

---

## Getting Started

### Step 1: Initialize a Spring Boot Project

To set up the project:

1. Visit [Spring Initializr](https://start.spring.io/).
2. Include the following dependencies:
   - **Spring Web**: For building RESTful APIs.
   - **Lombok**: To simplify the codebase.

Once the project is initialized, add the following dependencies to the `pom.xml` file:

```xml
<dependencies>
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   <dependency>
      <groupId>org.jsoup</groupId>
      <artifactId>jsoup</artifactId>
      <version>1.13.1</version>
   </dependency>
   <dependency>
      <groupId>org.apache.commons</groupId>
      <artifactId>commons-lang3</artifactId>
      <version>3.11</version>
   </dependency>
   <dependency>
      <groupId>org.projectlombok</groupId>
      <artifactId>lombok</artifactId>
      <optional>true</optional>
   </dependency>
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
   </dependency>
</dependencies>
```

---

### Step 2: Analyze the Target HTML Structure

Before scraping data, we must inspect the HTML structure of the target websites:

1. **[Ikman.lk](https://ikman.lk/):** Ads are located under the class name `list--3NxGO`.
2. **[Riyasewana.com](https://riyasewana.com/):** Ads are located under the `div` tag with the ID `content`.

To inspect HTML in Chrome:
1. Open the website in a browser.
2. Right-click the page and select **Inspect**.
3. Identify the DOM elements containing the required data.

---

## Implementation

### Step 1: Configure Application Properties

Define the website URLs in the `application.yml` or `application.properties` file:

```yaml
website:
  urls: https://ikman.lk/en/ads/sri-lanka/vehicles?sort=relevance&buy_now=0&urgent=0&query=,https://riyasewana.com/search/
```

### Step 2: Create a Model Class

The model class `ResponseDTO` will map the scraped data:

```java
package com.scraper.api.model;

import lombok.Data;

@Data
public class ResponseDTO {
    private String title;
    private String url;
}
```

### Step 3: Implement the Service Layer

The service layer contains the logic to scrape data from the websites:

```java
package com.scraper.api.service;

import com.scraper.api.model.ResponseDTO;
import org.apache.commons.lang3.StringUtils;
import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Service;

import java.io.IOException;
import java.util.HashSet;
import java.util.List;
import java.util.Set;

@Service
public class ScraperServiceImpl implements ScraperService {

    @Value("#{'${website.urls}'.split(',')}")
    private List<String> urls;

    @Override
    public Set<ResponseDTO> getVehicleByModel(String vehicleModel) {
        Set<ResponseDTO> responseDTOS = new HashSet<>();
        for (String url : urls) {
            if (url.contains("ikman")) {
                extractDataFromIkman(responseDTOS, url + vehicleModel);
            } else if (url.contains("riyasewana")) {
                extractDataFromRiyasewana(responseDTOS, url + vehicleModel);
            }
        }
        return responseDTOS;
    }

    private void extractDataFromRiyasewana(Set<ResponseDTO> responseDTOS, String url) {
        try {
            Document document = Jsoup.connect(url).get();
            Element content = document.getElementById("content");
            Elements ads = content.getElementsByTag("a");

            for (Element ad : ads) {
                ResponseDTO responseDTO = new ResponseDTO();
                if (!StringUtils.isEmpty(ad.attr("title"))) {
                    responseDTO.setTitle(ad.attr("title"));
                    responseDTO.setUrl(ad.attr("href"));
                    responseDTOS.add(responseDTO);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void extractDataFromIkman(Set<ResponseDTO> responseDTOS, String url) {
        try {
            Document document = Jsoup.connect(url).get();
            Element list = document.getElementsByClass("list--3NxGO").first();
            Elements ads = list.getElementsByTag("a");

            for (Element ad : ads) {
                ResponseDTO responseDTO = new ResponseDTO();
                if (StringUtils.isNotEmpty(ad.attr("href"))) {
                    responseDTO.setTitle(ad.attr("title"));
                    responseDTO.setUrl("https://ikman.lk" + ad.attr("href"));
                    responseDTOS.add(responseDTO);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

### Step 4: Create the Controller

The controller exposes the API endpoint to fetch vehicle ads:

```java
package com.scraper.api.controller;

import com.scraper.api.model.ResponseDTO;
import com.scraper.api.service.ScraperService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Set;

@RestController
@RequestMapping("/")
public class ScraperController {

    @Autowired
    private ScraperService scraperService;

    @GetMapping("/{vehicleModel}")
    public Set<ResponseDTO> getVehicleByModel(@PathVariable String vehicleModel) {
        return scraperService.getVehicleByModel(vehicleModel);
    }
}
```

---

## Testing the API

Run the project and test the API using a REST client by providing a vehicle model. The API will return the titles and URLs of ads from both websites.

Example Response:

```json
[
    {
        "title": "Toyota Corolla for Sale",
        "url": "https://ikman.lk/en/ads/toyota-corolla"
    },
    {
        "title": "Nissan Sunny",
        "url": "https://riyasewana.com/view/nissan-sunny"
    }
]
```

---

## Source Code

The complete source code is available in the project repository. Start building your own web scraping API today!

Stop wasting time on proxies and CAPTCHAs! ScraperAPI's simple API handles millions of web scraping requests, so you can focus on the data. Get structured data from Amazon, Google, Walmart, and more. 👉 [Start your free trial today!](https://bit.ly/Scraperapi)
