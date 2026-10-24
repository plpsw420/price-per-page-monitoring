# ecommerce price monitoring: what it actually costs per page, which plan fits your catalog size, and the credit math most reviews skip

Competitor prices in fast-moving categories change hourly, so ecommerce price monitoring means watching every product URL continuously and catching changes before you lose the sale. There are two very different ways to do it: buy a ready-made SaaS dashboard (upload your SKUs, get daily email alerts), or run your own scraper through an API that handles proxies, CAPTCHAs, and rendering for you. This guide covers the second route in depth, using ScraperAPI as the working example, because it's the option most "best price monitoring tools" listicles either skip or explain badly — especially when it comes to what a page actually costs.

> **Quick take:** If you can write basic code, ScraperAPI's Hobby plan at **$49/month** (100,000 credits, 20 concurrent threads) is enough for monitoring a few thousand standard product pages. Add JS rendering or hit Amazon, and credits burn 5–15x faster — the math below shows exactly how to budget for that. Sign up with the free trial first, not with your wallet: **👉 Start the 7-day free trial with 5,000 credits, no credit card needed**

## Why "price per page" is the number that matters, not the plan price

The pricing page shows a monthly fee. Your bill is decided by something else: how many credits each of your requests consumes.

For most standard product pages, a request costs 1 credit. But e-commerce domains get special pricing because they're harder to scrape:

| Target type | Credits per request |
| --- | --- |
| Normal (flat) pages | 1 |
| Amazon, Walmart, eBay | 5 |
| Google, Bing (all subdomains) | 25 |
| LinkedIn | 30 |

On top of that, parameters stack costs:

- `render=true` (headless browser rendering): +10 credits
- `premium=true` (premium residential/mobile pool): +10 credits
- Both combined: 25 credits
- `ultra_premium=true` (paid plans only): +30, or 75 combined with rendering
- Bypassing Cloudflare, Datadome, PerimeterX, or Akamai: +10 credits per scrape

Here's what that means in practice for a price monitoring job. Say you're watching Amazon product pages with JS rendering on. That's 5 + 10 = **15 credits per page**. A Startup plan's 1,000,000 credits then covers roughly 40,000–100,000 pages depending on the exact parameter mix — a fraction of what the headline number suggests. Independent reviewers who ran this math reached the same conclusion: on protected targets needing rendering plus a premium proxy, one successful scrape can cost 20–30x the plan's per-credit price on ordinary sites.

The flip side: if your targets are ordinary store sites without heavy bot protection, the default pool clears most requests at 1 credit each, no rendering needed. A 100,000-credit Hobby plan then covers 100,000 checks a month — which is a lot of product URLs checked daily.

ScraperAPI gives you two tools to keep this under control before you commit:

1. The **Domain Cost Estimator** in the dashboard, which tells you the exact credit cost of any URL before you run it. There's also an API endpoint (`/account/urlcost`) that does the same thing with any parameters applied.
2. A `max_cost` parameter per scrape, so a single request can't blow past a limit you set. Every response also includes a `sa-credit-cost` header showing what that request actually consumed.

Do this on the free trial and you'll know your real per-page cost before paying for anything.

## ScraperAPI plans and prices (all current tiers)

Annual billing takes 10% off every tier. Prices below are monthly rate / annual-billed rate:

| Plan | Monthly price | Annual (per month) | Credits | Concurrent threads | Get started |
| --- | --- | --- | --- | --- | --- |
| Hobby | $49 | $44.10 | 100,000 | 20 | [ Start with Hobby](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Startup | $149 | $134.10 | 1,000,000 | 50 | [ View Startup pricing](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Business | $299 | $269.10 | 3,000,000 | 100 | [ See Business plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Scaling (most popular) | $475 | $427.50 | 5,000,000 | 200 | [ Check Scaling plan](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Professional | $975 | $877.50 | 10,500,000 | 300 | [ View Professional](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Advanced | $1,975 | $1,777.50 | 21,500,000 | 500 | [ View Advanced](https://www.scraperapi.com/pricing?fp_ref=coupons) |
| Enterprise | Custom | Custom | 22,000,000+ | 500+ | [ Contact sales](https://www.scraperapi.com/contact-sales/?fp_ref=coupons) |

Every plan — including Hobby — ships with the full feature set: JS rendering, premium residential and mobile IPs, JSON auto-parsing, structured data endpoints, CAPTCHA and anti-bot handling, custom sessions and headers, automatic retries, unlimited bandwidth, and a 99.9% uptime guarantee.

A few differences don't fit in the table but affect which tier you need:

- **Geotargeting**: Hobby and Startup are limited to US and EU; Business and up get country-level targeting worldwide.
- **Analytics dashboard**: lower tiers keep only the last 30 days of stats; Business and up keep everything.
- **Support**: Hobby and Startup get email and ticket support only. If your monitoring job matters — say, you wake up and a target site has changed its bot protection overnight — priority support and Slack channels start at the higher tiers.
- **Overage behavior**: by default, scraping pauses when credits run out until renewal. On Hobby through Professional you can opt into auto-upgrade (the account bumps to the next plan automatically). Scaling and above offer pay-as-you-go at a fixed per-credit rate up to a monthly cap you set. Annual plans can also renew early for a fresh batch of credits, which is worth remembering if you burn through a batch mid-cycle.
- **Concurrency limits**: exceed your thread count and the API returns a 429 — the request is refused, not queued, and you're not charged for it.

There's also a permanent free plan: sign up and get 1,000 free credits with up to 5 concurrent connections, no credit card. Between that and the 7-day/5,000-credit trial, you can validate your per-page cost on real targets without spending anything.

## Building the monitoring loop: what ScraperAPI does and doesn't do

ScraperAPI is a scraping API, so it handles the hard part — getting reliable HTML or structured data back from any product URL — and you handle the rest. That "rest" is genuinely small for a developer:

1. Fetch each product URL on your schedule (a cron job works fine; fast-moving categories every 1–6 hours, slower ones daily).
2. Parse the price. For Amazon, Walmart product/search pages, Google Shopping, and Google Search, ScraperAPI returns parsed JSON directly, so you don't maintain your own parser when the site layout changes.
3. Compare against the previous value and store the history.

That last mile — alerts, dashboards, repricing rules — is yours to build. If you want those out of the box, that points you toward SaaS tools instead (see below).

Two more operational details worth knowing before you scale up. Sticky sessions via `session_number` last about an hour, which is fine for a scraping run but not for workflows needing the same IP for days. And Hobby's 20 concurrent threads are plenty for text-heavy price checks across thousands of pages, but if you're checking tens of thousands of URLs several times a day, you'll hit the ceiling and need a higher tier.

## The billing fine print that surprises people

These rules are in the docs, not on the pricing page, and they directly affect your monitoring costs:

- **404 responses are billed.** Only 200 and 404 status codes count as "successful" and consume credits. A deleted product page costs you a check anyway — which is arguably what you'd want to know, but budget for it.
- **Cancelled requests are billed too.** If you abort a request within the 70-second window before it finishes, it still consumes credits.
- **Structured endpoints save money overall.** Getting parsed JSON for Amazon costs the same 5 credits as raw HTML, and `autoparse=true` adds nothing — but it removes an entire maintenance burden.

If a target suddenly starts burning credits (the site added Cloudflare, for instance), the +10 bypass cost kicks in per request. The `max_cost` cap plus the cost-check endpoint let you catch this on day one instead of on day 30 of a billing cycle.

## When a ready-made SaaS tool is the better call

If nobody on your team writes code, a scraping API is the wrong layer to buy. The SaaS price monitoring market has mature options where you upload a product list, approve automated competitor matches, and get daily email alerts — no code at all:

| Tool | Starting price | Notable details |
| --- | --- | --- |
| Price2Spy | $39.95/mo | URL-based tracking, up to 8 checks/day, MAP monitoring with screenshots |
| Pricefy | Free (50 SKUs); paid from $49/mo | Repricing simulator included |
| Priceva | Free (20 products); paid from $99/mo | Formula-based repricing, stock monitoring |
| Prisync | $99/mo | Shopify/Magento/BigCommerce sync, dynamic repricing built in; API access costs +20% |
| PricingHunter | $99/mo | Unlimited competitors, daily alerts, unlimited price history, 14-day trial |

The trade-off is straightforward: these tools start charging per tracked product or SKU, so costs climb with catalog size, and you get their fixed feature set. PricingHunter, for instance, has no built-in dynamic repricing; Prisync's API costs extra. If you're tracking 50 SKUs on Shopify, a $99 SaaS subscription is the sensible choice. If you're checking 20,000 URLs hourly across several marketplaces, per-SKU pricing collapses fast and an API priced per request wins on both cost and flexibility. It's a two-tier market: developer-run APIs and no-code SaaS dashboards barely overlap, so the right question is which layer you actually need — not which product is "best" overall.

## Who should (and shouldn't) pick this route

Good fit: a solo developer or small team running price monitoring, SERP tracking, or lead-gen scrapers against standard e-commerce and search targets, at roughly 20,000–200,000 pages a month, without wanting to build and maintain proxy rotation and retry logic. Registration takes minutes, one GET request gets you a page, and the structured Amazon/Walmart endpoints remove the parser-maintenance headache entirely.

Poor fit, based on documented limits: anyone scraping hard targets like LinkedIn or heavily protected sites at serious volume (the ultra-premium multiplier makes it pricier than bandwidth-billed residential proxies), anyone needing IP persistence beyond an hour for ad verification or account workflows, and anyone who wants to pick specific IPs or ASNs — the API doesn't expose that control.

## FAQ

**Does the free trial require a credit card?**
No. The 7-day trial includes 5,000 credits, and the permanent free plan adds another 1,000 credits on signup with up to 5 concurrent connections.

**How many product pages can I monitor on the $49 Hobby plan?**
It depends entirely on your targets. 100,000 credits means about 20,000 checks monthly on Amazon without rendering (5 credits each), or about 6,600 with rendering (15 credits each) — versus 100,000 checks on standard sites. Run the Domain Cost Estimator on your actual URLs before assuming anything.

**What happens when I run out of credits mid-month?**
Scraping pauses by default until renewal. Hobby through Professional can turn on auto-upgrade to move to the next tier automatically; Scaling and above can enable pay-as-you-go up to a monthly cap you set.

**Can I set how often prices get checked?**
You control the schedule entirely since it's your script hitting the API. Fast-moving electronics typically need 1–6 hour intervals on high-competition listings; slower categories do fine with daily checks. Remember that higher frequency costs proportionally more credits.

## Picking your plan before you click buy

Match your situation to the tier:

- **Testing the waters or a small catalog of standard store pages**: use the free trial and 1,000-credit free plan first, then Hobby at $49 if the data comes back clean. **👉 Check the current Hobby pricing and start your trial**
- **Amazon-heavy monitoring with rendering**: do the per-page math (likely 15+ credits each), then pick a tier where real pages × credits leaves 20% headroom. For most mid-size catalogs that lands on Startup or Business. **👉 View Startup and Business annual pricing**
- **Tens of thousands of URLs at sub-hourly frequency**: Business or Scaling, where country-level geotargeting and higher concurrency stop being bottlenecks.
- **Enterprise volume (22M+ requests/month)**: custom pricing with a dedicated account manager — worth negotiating rather than stacking Advanced plans.

One last check before you subscribe: run your five most important product URLs through the API Playground, note the exact credit cost each one returns, and multiply by your check frequency. That single calculation settles the Hobby-vs-Startup-vs-Business question more reliably than any review — including this one. When you're ready, **👉 grab the free 5,000-credit trial and run that test today**.
