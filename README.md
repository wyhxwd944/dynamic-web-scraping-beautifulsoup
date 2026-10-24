# dynamic web scraping python beautifulsoup: Two Working Routes for JavaScript-Rendered Pages, Plus the Credit Math Most Guides Skip

You followed a BeautifulSoup tutorial, it worked on the practice site, then it returned nothing on your real target. That's the dynamic web scraping problem in one sentence: the data you want gets assembled by JavaScript *after* the HTML arrives, and BeautifulSoup only parses what the server sent.

The fix is smaller than most people expect. You keep your BeautifulSoup code and change where the HTML comes from: either render the page yourself with Selenium, or let a rendering API like ScraperAPI do it and parse the response as usual. This guide walks through both routes, then covers ScraperAPI's current plans in detail — including its credit pricing rules, which is where most comparison articles go quiet.

> **Quick take:** for a one-off job on a few hundred pages, run headless Selenium locally. For anything recurring, or any site with anti-bot protection, send URLs through ScraperAPI with `render=true` and parse the returned HTML with BeautifulSoup exactly as before. Plans start at **$49/month for 100,000 credits** ($44.10/month billed annually), and the 7-day trial comes with 5,000 credits and no credit card.
>
> 👉 [Start the 7-day ScraperAPI trial with 5,000 credits](https://www.scraperapi.com/signup?fp_ref=coupons)

## Why BeautifulSoup alone can't scrape dynamic websites

BeautifulSoup is a parser, not a browser. It turns an HTML string into Python objects you can search with `find()` and `select()`. It never executes JavaScript, never fires XHR requests, and never waits for anything to load — so if the data arrives via an AJAX call after page load, it simply isn't in the string BeautifulSoup received.

You can confirm a site is dynamic in under a minute, no code needed. Open the page in Chrome or Edge, press F12, then Ctrl+Shift+P to open the Command Menu, type "disable", pick **Disable JavaScript**, and reload. If large chunks of the page vanish, the content is generated client-side — and that's precisely the part your `requests` + BeautifulSoup combo will never see. A partial-disappearance case exists too: some sites fall back to basic functionality, for example infinite scroll degrading to normal pagination.

Every solution to this problem has the same shape: obtain the *fully rendered* HTML somehow, then feed it to BeautifulSoup. The two practical ways to get that HTML are a real automated browser, or an API that renders on your behalf.

## Route 1: Selenium + BeautifulSoup (the DIY route)

The standard pattern looks like this:

python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from bs4 import BeautifulSoup

options = Options()
options.add_argument("--headless=new")
driver = webdriver.Chrome(options=options)

try:
    driver.get("https://quotes.toscrape.com/js/")
    WebDriverWait(driver, 10).until(
        EC.presence_of_element_located((By.CLASS_NAME, "quote"))
    )
    soup = BeautifulSoup(driver.page_source, "lxml")
    for quote in soup.select("div.quote"):
        print(quote.select_one("span.text").text)
finally:
    driver.quit()


The line that matters is `driver.page_source`. By the time you read it, the browser has executed the JavaScript, so the HTML contains everything — your existing BeautifulSoup selectors work unchanged. Setup needs three pieces: the browser itself, a matching driver (ChromeDriver versions track Chrome releases), and `pip install selenium`. The explicit `WebDriverWait` matters more than beginners think; without it, `page_source` can be captured before the AJAX call finishes, which produces the classic "works sometimes, empty other times" bug.

For small jobs this is the right tool. At scale, three costs show up that tutorials rarely mention:

1. **Resources.** Each headless browser instance eats hundreds of MB of RAM, and rendering a full page per request is slow.
2. **Maintenance.** Driver updates, browser updates, flaky headless rendering on sites that fingerprint automation.
3. **Blocks.** Sites behind Cloudflare, DataDome, or PerimeterX detect and challenge automated browsers. Solving that yourself means building proxy rotation and fingerprint handling — a project of its own.

There's a lighter variant worth trying before any of this: open DevTools → Network tab, find the AJAX endpoint the page actually calls, and hit it directly with `requests`. When the endpoint is unprotected, this beats every rendering approach on speed and cost. It breaks when the request is signed, session-bound, or gated by anti-bot checks — which on high-value targets is usually the case.

## Route 2: a rendering API, with your BeautifulSoup code untouched

ScraperAPI sits between your script and the target site. You send the URL to its endpoint; it fetches through its proxy pool, renders JavaScript when asked, handles CAPTCHAs and retries, and returns plain HTML. Only the fetch step changes:

python
import requests
from bs4 import BeautifulSoup

payload = {
    "api_key": "YOUR_API_KEY",
    "url": "https://quotes.toscrape.com/js/",
    "render": "true",
}
r = requests.get("https://api.scraperapi.com/", params=payload)
soup = BeautifulSoup(r.text, "lxml")
for quote in soup.select("div.quote"):
    print(quote.select_one("span.text").text)


`render=true` is the only difference from a static-page request. Other parameters from the official Python docs: `premium=true` for residential and mobile IPs, `country_code` for geotargeting, `session_number` to reuse a proxy, `screenshot=true` for page screenshots (it enables rendering automatically), `autoparse=true` for JSON output on supported sites, and `device_type=desktop|mobile`. Notably, `country_code`, `session_number`, `keep_headers`, `device_type`, and `autoparse` add no credit cost.

All plans include JS rendering, premium proxies, CAPTCHA and anti-bot handling, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee — rendering isn't a paid add-on on top of any tier.

👉 [Check ScraperAPI's full plan lineup and current pricing](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## ScraperAPI plans and pricing

Annual billing takes 10% off every plan. Prices below show monthly / annual-equivalent per month:

| Plan | Credits/mo | Concurrent threads | Geotargeting | Price (mo / annual) | Subscribe |
| --- | --- | --- | --- | --- | --- |
| Hobby | 100,000 | 20 | US & EU only | $49 / $44.10 | [Start Hobby](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Startup | 1,000,000 | 50 | US & EU only | $149 / $134.10 | [Start Startup](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Business | 3,000,000 | 100 | Global (country-level) | $299 / $269.10 | [Start Business](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Scaling (most popular) | 5,000,000 | 200 | Global (country-level) | $475 / $427.50 | [Start Scaling](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Professional | 10,500,000 | 300 | Global | $975 / $877.50 | [Start Professional](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Advanced | 21,500,000 | 500 | Global | $1,975 / $1,777.50 | [Start Advanced](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Enterprise | 22,000,000+ | 500+ | Global | Custom | [Contact sales](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

There's also a free option: sign up and get 1,000 free API credits with a maximum of 5 concurrent connections, before you pick any plan.

## What rendering actually costs: the credit math most guides skip

Headline prices look simple until you notice credits aren't consumed uniformly. Per ScraperAPI's pricing page and documentation:

- A standard page: **1 credit**
- An Amazon product page: **5 credits**
- A Google or Bing search: **25 credits**
- LinkedIn: **30 credits**
- Sites behind Cloudflare, DataDome, or PerimeterX, when bypassed: **+10 credits** per request
- `render=true`: **10 credits** — or **75** if combined with `ultra_premium=true`
- `premium=true`: **10 credits** — or **25** when combined with JS rendering (can't stack with ultra-premium)
- `ultra_premium=true` alone: **30 credits**

Run the numbers for a dynamic-page project and the Hobby plan's 100,000 credits means roughly 20,000 Amazon pages, 4,000 Google queries — or only about **10,000 JavaScript-rendered pages**. If your targets are dynamic, budget at 10 credits per page, not 1. That single conversion changes which plan you need.

Two protections worth knowing before you subscribe. First, exceeding your concurrent-thread limit returns a 429 and the request is rejected rather than queued — the official page states there's no extra charge for going over, so add retry logic with backoff and you're fine. Second, credits for failed requests are refunded, which matters a lot when you're hammering a site that blocks a chunk of your attempts. And if you want certainty before spending anything, the dashboard's Domain Cost Estimator shows the exact credit cost for any URL, and a `max_cost` setting caps what a single request can burn.

## Hidden limits by plan (from the fine print)

Three restrictions sit in plan details rather than the price column, and they change the real decision:

- **Geotargeting on Hobby and Startup covers only the US and EU.** If your targets need proxies in Asia or anywhere else, Business at $299/month is the entry point — a real jump from $149, not a rounding error.
- **Pay-as-you-go overage only exists from Scaling up.** On Hobby through Business, running out of credits mid-month means waiting for the reset or upgrading. You can optionally enable auto-upgrade on Hobby through Professional, which moves your account to the next plan automatically when credits run out instead of pausing your pipeline.
- **Analytics history on Hobby and Startup is 30 days only**; Business and above keep unlimited history. Irrelevant for a weekend project, relevant if you monitor costs over time.

If you're sizing up alternatives: one independent pricing breakdown puts ScrapingBee's entry tier at around $49/month for roughly 17,000–20,000 results, and Bright Data's entry plans around $499/month, aimed at enterprise teams. How those numbers convert depends on each provider's own credit definition, so compare cost-per-target-page rather than the sticker price.

## FAQ

**Can BeautifulSoup render JavaScript?**
No. Rendering means interpreting HTML, JavaScript, CSS, and images into what you see in a browser; BeautifulSoup only parses HTML strings into Python objects. Render first (with Selenium or an API), then parse.

**Does `render=true` work on every plan?**
Yes — JS rendering is included on all plans, including the free 1,000-credit tier. The cost is 10 credits per rendered request regardless of tier.

**What happens when my credits run out?**
By default, scraping pauses until credits renew. Hobby through Professional can opt into auto-upgrade; Scaling and above can switch on pay-as-you-go to keep scraping at a fixed per-credit rate up to a monthly cap you set.

**Is Selenium still worth learning?**
Yes, for debugging and small jobs. Being able to watch a browser load your target page is the fastest way to understand what's dynamic about it, and DevTools network inspection is a skill you'll use with either route.

## Which route fits your project

- **One-off scrapes, a few hundred pages, no blocking:** local Selenium + BeautifulSoup. It's free and you already know the stack.
- **Recurring jobs, dynamic pages, any sign of anti-bot protection:** ScraperAPI Hobby or Startup. Rendering plus proxies plus retries at $49/month beats maintaining a browser farm and proxy rotation for a solo developer.
- **Production pipelines with traffic spikes:** Scaling or above, where pay-as-you-go absorbs the peaks. Need country-level targeting outside the US and EU? Business is your floor either way.
- **Amazon, Google, or Walmart as your main target:** look at Structured Data Endpoints first — they return parsed JSON directly, so for those sites you can skip the BeautifulSoup parsing step entirely.

One last check before you commit: run your actual target URLs through the trial with `render=true`, then look up the per-request credit cost in the dashboard's cost estimator. That one number decides whether your monthly volume fits 100,000 credits or needs the Startup tier. If it fits, 👉 [sign up for the ScraperAPI trial](https://www.scraperapi.com/signup?fp_ref=coupons) and keep your BeautifulSoup code exactly as it is.
