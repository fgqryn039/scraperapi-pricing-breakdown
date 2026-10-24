# scraper api: how ScraperAPI's credit billing really works, what every plan costs, and which one to pick

A scraper API is a hosted service that fetches web pages for your code and deals with the ugly parts — rotating proxies, CAPTCHAs, JavaScript rendering, rate-limit blocks — so you send one HTTP request and get HTML or JSON back. If that's what you're after, ScraperAPI is one of the most established options: it starts free with 1,000 credits a month (5,000 for your first 7 days, no credit card), and paid plans run from $49/month for 100,000 credits up to custom Enterprise pricing.

The catch, and the reason this article exists, is that "1 credit per request" is only true for easy pages. A rendered Amazon product page costs 15 credits. A page behind Cloudflare with rendering on can burn 75. Below is the full plan list, the billing rules that actually determine your cost, and an honest comparison with ZenRows and ScrapingBee.

> **Quick pick:** most individual developers fit the **Hobby plan — $49/month for 100,000 credits, 20 concurrent requests**. If you're just evaluating, start with the free trial instead. 👉 [Check ScraperAPI's current plans and prices](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## What a scraper API like ScraperAPI actually does

You replace your proxy-and-browser stack with a single endpoint. The simplest request looks like `https://api.scraperapi.com?api_key=YOUR_KEY&url=https://example.com` — it works in curl, Python requests, Postman, or straight in your browser. On the server side, ScraperAPI rotates through its proxy pool (residential and mobile IPs included), solves CAPTCHAs, renders JavaScript in headless browsers when asked, and retries failed fetches. Your scraping logic still runs on your own server; the API is the fetch layer.

Every paid plan includes the same feature set: JavaScript rendering, premium proxies, automatic JSON parsing, sticky sessions (the `session_number` parameter pins you to one IP for 15 minutes, useful for multi-page crawls), custom headers, desktop and mobile user agents, automatic retries, and unlimited bandwidth. You're billed per request, not per GB, so a 2 MB page costs the same as a 20 KB one.

Beyond raw fetching, there's a set of structured data endpoints that return parsed JSON for Amazon, Walmart, eBay, Home Depot, Google Search/News/Jobs/Shopping/Maps, and Redfin, plus AI answer engines. There's also DataPipeline, a low-code runner that takes up to 100,000 URLs, keywords, or ASINs per project and delivers CSV/JSON on a schedule, and integrations for LangChain, n8n, Zapier, and an MCP server you can plug into Claude Code. If you'd rather not write a fetcher at all, that tier of the product matters more than the proxy numbers.

## ScraperAPI plans and pricing: the full table

ScraperAPI currently shows eight tiers on its pricing page. All prices in USD; annual billing knocks 10% off every paid plan.

| Plan | Monthly price | Annual (10% off) | Credits / month | Concurrency | Get started |
| --- | --- | --- | --- | --- | --- |
| Free | $0 | — | 1,000 | 5 | [Start free](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Hobby | $49 | $44.10/mo | 100,000 | 20 | [Get Hobby](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Startup | $149 | $134.10/mo | 1,000,000 | 50 | [Get Startup](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Business | $299 | $269.10/mo | 3,000,000 | 100 | [Get Business](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Scaling (most popular) | $475 | $427.50/mo | 5,000,000 | 200 | [Get Scaling](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Professional | $975 | $877.50/mo | 10,500,000 | 300 | [Get Professional](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50/mo | 21,500,000 | 500 | [Get Advanced](https://www.scraperapi.com/pricing/?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [Talk to sales](https://www.scraperapi.com/pricing/?fp_ref=coupons) |

A few plan-level differences that don't fit in the table:

- **Geotargeting is gated.** Hobby and Startup can only target US and EU. Country-level targeting outside those regions starts at Business ($299).
- **Analytics retention**: Hobby and Startup keep 30 days of dashboard history; Business and up keep it indefinitely.
- **Support**: every paid plan includes priority support, a dedicated support team, and Slack support.
- The documentation also references a Student plan and an education program for academic researchers, both applied for directly rather than bought off the pricing page.

## The credit system: where the real cost hides

The base rate is 1 credit for a standard page. Everything else is a multiplier, and ScraperAPI prices known-difficult domains explicitly:

| Request type | Credits |
| --- | --- |
| Standard unprotected page | 1 |
| Amazon | 5 |
| Google / Bing (incl. subdomains) | 25 |
| LinkedIn | 30 |
| `render=true` (JS rendering) | +10 |
| `premium=true` (residential proxy) | +10 |
| `screenshot=true` | +10 |
| `ultra_premium=true` | +30 |
| Cloudflare / DataDome / PerimeterX bypass | +10, automatic |

The combos are where most people get surprised, because they don't add linearly. `premium` plus `render` bills as 25 credits total, and `ultra_premium` plus `render` bills as a flat 75. Anti-bot bypass is detected and applied automatically — you can't opt out of the surcharge once you hit a protected page.

Run the math on the Hobby plan and this matters a lot. Its 100,000 credits sounds like 100,000 pages, but scraping Amazon product pages with rendering on costs 15 credits per request, so you're really looking at roughly 6,600 fetches. Point the same plan at a Cloudflare-protected site with ultra-premium rendering and you're down to about 1,300. That gap between the headline number and the real one is the single most important thing to understand before picking a plan.

Two tools keep this under control. The dashboard's **Domain Cost Estimator** tells you the exact credit cost of any URL before you run a batch, and you can set a `max_cost` per scrape so a single request can't blow past your ceiling. Every API response also carries a `sa-credit-cost` header showing what that request consumed — worth wiring into your logging before you scale up.

If you want to see real prices against your own target sites, 👉 [open ScraperAPI's pricing page and start with the free credits](https://www.scraperapi.com/pricing/?fp_ref=coupons) — testing your hardest domain first is the only reliable way to size a plan.

## Billing rules most reviews skip

These come from ScraperAPI's own plans-and-billing documentation, and they change how the plans behave in practice:

1. **Only successful requests are billed.** Requests returning HTTP 200 and 404 count as results and cost credits; other failures don't. Compare that with ZenRows, whose pricing page states a 404 or 410 "is a real answer, so it counts as a result" — broadly similar, but worth knowing which failures you pay for.
2. **Credits don't roll over.** Unused credits expire at each billing cycle. There's no banking quota for a heavy month next month.
3. **Hitting the concurrency limit returns a 429.** No charge, no queueing — the request is just rejected. If your code doesn't handle 429s with backoff, you'll lose throughput on the higher-value plans without spending a credit.
4. **Pay-as-you-go overflow is gated to Scaling and above.** On Scaling ($475), Professional, Advanced, and Enterprise, running out of credits triggers a prompt to continue at a fixed per-credit rate up to a spending cap you set. On Hobby, Startup, and Business, hitting the limit pauses scraping until renewal — your options are opting into auto-upgrade (which bumps you to the next tier, usually at a better per-credit price) or contacting support for a custom plan.
5. **Annual plans can renew early** for a fresh batch of credits if you burn through a year's allowance mid-cycle.

That fourth point is the quiet argument for Scaling if your volume is unpredictable: it's the cheapest tier where running dry doesn't stop your pipeline.

## ScraperAPI vs ZenRows and ScrapingBee

All three sell credit-based scraping APIs with the same core features. Here's how the entry and mid tiers actually line up, using each company's current official pricing:

| Credits/month | ScraperAPI | ZenRows | ScrapingBee |
| --- | --- | --- | --- |
| ~50–100K | $49 · 100K · 20 concurrent | $16 · 45K · 20 concurrent | $19 · 75K · 25 concurrent |
| ~250K–1M | $149 · 1M · 50 | $58 · 250K / $166 · 1.2M · 100 | $99 · 1M · 100 |
| ~3M | $299 · 3M · 100 | — (jumps to $458 for 5M) | $249 · 3M · 200 |
| ~5M+ | $475 · 5M · 200 | $458 · 5M · 200–400 | $599 · 8M · 400 |

Reading that honestly: **at the low end, ScraperAPI is the most expensive of the three.** ScrapingBee's $19 Hobby plan undercuts it, and ZenRows' $16 Build is cheaper still, though with half the credits. Around 1M credits, ScrapingBee sells the same volume for $99 with double ScraperAPI's concurrency. ScraperAPI's pricing only becomes competitive in the multi-million-credit range, where its $475/5M sits between ZenRows' $458/5M and ScrapingBee's $599/8M.

Credit consumption differs too, and it changes the per-request math. ZenRows charges 5 credits for JS rendering and 10 for premium proxies (25 combined), against ScraperAPI's +10 and +10. ZenRows also includes 5,000 free credits monthly on its free plan, five times ScraperAPI's 1,000 — if your monthly volume genuinely fits in a few thousand requests, its free tier is the more generous one.

Where ScraperAPI pulls ahead is the surrounding product: structured data endpoints for Amazon and Google that return clean parsed JSON, the DataPipeline scheduler, unlimited bandwidth on every plan, and the domain-based cost transparency described above. If your project is mostly standard HTML at low volume, the cheaper competitors do the job. If it's e-commerce or SERP data at scale, the structured endpoints can save you from writing and maintaining parsers, which is often worth more than a $30 monthly difference. The choice is yours to make; the clickable path here is 👉 [ScraperAPI's full plan list](https://www.scraperapi.com/pricing/?fp_ref=coupons).

## Free trial and refund: what you actually get

You get two free tiers, and they stack. Signing up puts you on the permanent free plan — 1,000 credits per month, 5 concurrent connections, no credit card. For your first 7 days, that expands to 5,000 credits so you can test at realistic volume. Registration needs only an email; the dashboard shows your API key and credit counter immediately.

Paid plans carry a 7-day no-questions-asked refund policy, and you can cancel any time from the billing dashboard without a fee. Combined with the card-free trial, the risk of trying ScraperAPI on a real project is close to zero: test your three hardest target domains in the first two days, and if the numbers don't work, you're out nothing.

One practical warning from the docs: the dashboard shows credit consumption and success rates, but it won't actively warn you when you're about to run dry. Set a `max_cost` and check the estimator before big runs.

## How to pay less

- **Annual billing is an automatic 10% off** every paid plan — no code needed. On Hobby that's $44.10/month, saving $58.80 a year.
- Third-party coupon trackers report a code **START10** giving new users 10% off the first month of any paid plan; it circulates through affiliate channels rather than ScraperAPI's own site, so treat it as a maybe and check the checkout total before paying.
- Don't enable `render=true` by default. Plenty of sites serve full HTML without it, and rendering is the most common silent credit drain.
- Use the free 5,000 trial credits on your *hardest* domain, not the easiest one. If the hard case works at an acceptable cost per page, the plan sizing takes care of itself.

## FAQ

**Does ScraperAPI have a free plan?**
Yes. 1,000 credits per month forever, with 5 concurrent connections, plus a 7-day window with 5,000 total credits after signup. No credit card required for either.

**What happens when I run out of credits?**
Scraping pauses until your next billing cycle by default. On Hobby through Professional you can opt into auto-upgrade to the next tier; on Scaling and above you can continue on pay-as-you-go at a fixed per-credit rate, capped at a monthly spending limit you define.

**Is ScraperAPI worth it for scraping Google?**
Only for modest volumes. Google searches cost 25 credits each, so even the $149 Startup plan's 1M credits cover about 40,000 Google queries a month. If SERP data is your main use case, compare that math against the structured Google endpoints before committing.

**Do unused credits roll over?**
No. Credits reset at each billing period and leftovers expire.

## Which plan should you pick

Still deciding? Here's the decision rule based on everything above:

- **Just exploring or under ~1,000 requests/month** → stay on the free plan; it's permanent, not a demo.
- **Personal project, mostly standard pages** → Hobby at $49, or honestly, one of the cheaper competitors at the $16–19 tier if budget is the deciding factor.
- **Amazon/e-commerce product data with rendering** → Startup or Business, after running the Domain Cost Estimator on your real targets; at 15 credits per rendered Amazon page, volume decides.
- **Unpredictable or spiky volume above ~2M credits** → Scaling, because it's the first tier with pay-as-you-go overflow so your pipeline doesn't stop mid-month.
- **Needs US/EU-only geotargeting and steady volume** → Startup covers it; country-level targeting outside those regions requires Business or higher.

Before you commit to anything, do one thing: grab the free trial, run your three most important URLs through the estimator, and multiply by your real monthly volume. If the numbers work, 👉 [lock in ScraperAPI with annual billing for the 10% discount](https://www.scraperapi.com/pricing/?fp_ref=coupons) — and if they don't, you've spent seven days and zero dollars finding out.
