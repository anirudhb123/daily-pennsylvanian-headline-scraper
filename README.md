## CIS 3500 HW2 Part IV

This site uses GitHub Actions to create a web scraper to keep track of the top headline from the most recent guide published by the Daily Pennsylvanian each day. 

## Changes Made to Scraper

Original ```scrape_data_point()``` code

```python
def scrape_data_point():
    """
    Scrapes the main headline from The Daily Pennsylvanian home page.

    Returns:
        str: The headline text if found, otherwise an empty string.
    """
    req = requests.get("https://www.thedp.com")
    loguru.logger.info(f"Request URL: {req.url}")
    loguru.logger.info(f"Request status code: {req.status_code}")

    if req.ok:
        soup = bs4.BeautifulSoup(req.text, "html.parser")
        target_element = soup.find("a", class_="frontpage-link")
        data_point = "" if target_element is None else target_element.text
        loguru.logger.info(f"Data point: {data_point}")
        return data_point
```

Modified ```scrape_data_point()``` code

```python
def scrape_data_point():
    """
    Scrapes the top headline from the latest guide published by The Daily Pennsylvanian. 

    Returns:
        str: The headline text if found, otherwise an empty string.
    """
    req = requests.get("https://www.thedp.com/page/guides")
    loguru.logger.info(f"Request URL: {req.url}")
    loguru.logger.info(f"Request status code: {req.status_code}")

    if req.ok:
        soup = bs4.BeautifulSoup(req.text, "html.parser")
        target_chunk = soup.find("h2")
        data_point = "" if target_chunk is None else target_chunk

        stage1 = data_point.find("a")['href']
        
        req2 = requests.get(stage1)

        if req2.ok: 
            soup = bs4.BeautifulSoup(req2.text, "html.parser")
            target_chunk = soup.find("article").find("h1")
            data_point = "" if target_chunk is None else target_chunk

            stage2 = data_point.find("a").text
            loguru.logger.info(f"Data point: {stage2}")

            return stage2
```


## Reasoning behind my Approach
In order to obtain the top headline from the most recent guide published by the DP, I needed to make two requests, one to the page containing the DP's guides, and one to the page that the most recent guide on this page with all of the guides led to. Thus, the first change I made was to make my initial request to ```https://www.thedp.com/page/guides``` as opposed to ```https://www.thedp.com"```. Then, to find the hyperlink corresponding to the first headline in this page, which I saw from looking at the HTML for this page was contained within an h2 tag, I used ```soup.find(h2)```. The actual hyperlink was within an a tag, so I used ```.find("a")['href']``` on the chunk I got from within the h2 tag (as the format of an a tag is something like ```<a href="ACTUAL WEBSITE URL">text displayed on page</a>```) to actually obtain the hyperlink. 

I then used this obtained URL to make a second request, and, after checking that it was ok (as the sample code did for the first request),
I again used BeautifulSoup to parse the HTML on this page (represented by ```req2.text```). I again checked the HTML code on this page and found that the top headline was within an h1 tag that was itself within an article tag. Thus, I used ```soup.find("article").find("h1")``` to obtain the chunk of HTML code that this headline was in. Since this headline was also within an a tag, I then used ```.find("a").text`` (as this time I was looking to obtain the text displayed on the page and not the URL it led to) and logged and returned this headline. 

Thus, my modified scraping function satisfies my initial aim of obtaining the top headline from the most recent guide published by the DP. 
