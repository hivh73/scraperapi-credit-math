# ai agent web scraping: what actually breaks, what it costs per page, and how to pick a plan before your credits vanish

An AI agent that scrapes the web usually fails in one of three ways: it fetches an empty HTML shell from a JavaScript-heavy site, it gets fingerprinted and blocked by Cloudflare or DataDome, or it hits a login wall with no way to hold a session. A web scraping API fixes all three at the infrastructure level. Your agent makes one HTTP call; someone else's residential proxies, headless browsers, and anti-bot logic handle the hard part.

This article walks through how to wire a scraping API into an agent workflow without burning your budget, using ScraperAPI as the worked example. The interesting part is the credit system: "100,000 credits per month" does not mean 100,000 pages, and the difference between 1 credit and 75 credits per request decides which plan you should actually buy. Most comparisons stop at the sticker price. We won't.

> **Quick take:** if your agent mostly scrapes normal pages and e-commerce product data, the Hobby plan at $49/month (100,000 credits, 20 concurrent threads) is the usual starting point. The 7-day trial gives you 5,000 credits with no credit card, which is enough to test your real target URLs before spending anything. 👉 [Start the 7-day free trial and test your target URLs](https://www.scraperapi.com/signup?fp_ref=coupons)

## Why a plain fetch() breaks for AI agents

Agents break on the web because the modern web is hostile to stateless HTTP requests. Three failure modes show up over and over:

- **SPAs return empty shells.** React and Next.js sites serve an HTML skeleton first; client-side JavaScript fills in the actual content. Sites like Stripe and Vercel docs work this way. A plain `fetch()` gets the shell — no product data, no pricing, nothing.
- **Anti-bot middleware blocks cloud IPs.** Cloudflare, Akamai, and DataDome score requests by IP reputation, TLS fingerprint, user-agent string, and browser behavior. Cloud function IP ranges are well-known and often pre-blocked, so your agent gets a 403 or a challenge page instead of content.
- **Session-gated content is out of reach.** Paywalls, login walls, and cart-context pricing need a persistent authenticated browser session, which a stateless API key can't provide on its own.

A scraping API solves all three with infrastructure you don't have to run: residential and mobile proxy pools, headless browsers that mimic real Chrome, and bypass logic updated as anti-bot vendors update. That's also why it fits agent tool loops so well — one tool, predictable output, no browser to babysit. For multi-step tasks like logging in or filling forms, you'd pair it with a cloud browser; for the dominant agent pattern of "fetch this URL, return clean data," a scraping API is the simpler and cheaper tool.

ScraperAPI covers the standard feature set on every paid plan: JS rendering, premium residential and mobile IPs, CAPTCHA and anti-bot detection, automatic retries, JSON auto-parsing, custom session support, plus structured-data endpoints for Amazon and Google, a DataPipeline crawler, and unlimited bandwidth.

## ScraperAPI plans: the full current lineup

Here is every plan currently shown on ScraperAPI's pricing page. Prices are USD; annual billing takes 10% off each tier.

| Plan | Price (monthly / annual) | Credits | Concurrent threads | Best for | Link |
| --- | --- | --- | --- | --- | --- |
| Free (signup credits) | $0 | 1,000 | 5 | Trying it out | [ Sign up for free credits](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Hobby | $49 / $44.10 | 100,000 | 20 | Small projects, personal agents | [ Get Hobby](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Startup | $149 / $134.10 | 1,000,000 | 50 | Low-volume production | [ Get Startup](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Business | $299 / $269.10 | 3,000,000 | 100 | Mid-scale production | [ Get Business](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Scaling | $475 / $427.50 | 5,000,000 | 200 | Growing pipelines (marked Most popular) | [ Get Scaling](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Professional | $975 / $877.50 | 10,500,000 | 300 | Regular high-volume scraping | [ Get Professional](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Advanced | $1,975 / $1,777.50 | 21,500,000 | 500 | Continuous multi-source pipelines | [ Get Advanced](https://www.scraperapi.com/signup?fp_ref=coupons) |
| Enterprise | Custom quote | 22,000,000+ | 500+ | Above 22M requests/month | [ Contact sales](https://www.scraperapi.com/contact-sales?fp_ref=coupons) |

Two plan differences that don't show up in the price column but do affect agent behavior: **Hobby and Startup only support US & EU geotargeting**, while Business and above get country-level targeting anywhere. Analytics history is limited to the last 30 days on the two cheaper plans and unlimited from Business up. All paid plans include the full feature set — JS rendering, premium proxies, CAPTCHA solving, the Amazon/Google structured-data endpoints — so you won't pay extra to unlock basics the way some competitors gate features behind higher tiers.

Credits reset monthly and do not roll over. If your agent's usage is spiky, you're either leaving credits on the table in quiet months or sizing up a tier you don't need. When credits do run out, scraping pauses by default; on Hobby through Professional you can opt into auto-upgrade (the account moves to the next plan automatically), and on Scaling and above you can use pay-as-you-go at a fixed per-credit rate up to a monthly cap you set.

## The credit math that decides your real cost

This is the part most write-ups skip, and it's where the money actually goes. One credit is the cost of a standard page request. Everything else costs more, sometimes much more:

| Request type | Credits |
| --- | --- |
| Standard page (normal domain) | 1 |
| E-commerce pages (Amazon, Walmart, eBay) | 5 |
| Google / Bing SERPs (all subdomains) | 25 |
| LinkedIn | 30 |
| Anti-bot bypass (Cloudflare, DataDome, PerimeterX, Akamai) | +10 per request |
| `render=true` (JS rendering) | +10 |
| `premium=true` (premium proxies) | +10 |
| `screenshot=true` | +10 |
| `ultra_premium=true` (paid plans only) | +30 |
| `ultra_premium + render` | 75 |

Run the combination: a JS-rendered Amazon product page behind Cloudflare protection costs 5 + 10 + 10 = **25 credits**. On Hobby, that's 4,000 pages a month, not 100,000. A Google SERP scrape with rendering runs 35 credits per request. This is exactly the trap that one independent review flagged — at 75 credits for a rendered protected page, a 1,000-credit allowance covers roughly 13 pages.

Three things soften the blow, and all of them are free:

1. Every response includes a `sa-credit-cost` header showing exactly what that request consumed, so your agent can log its own burn rate.
2. The dashboard's Domain Cost Estimator and a dedicated cost-check endpoint documented in the ScraperAPI docs let you query the exact credit cost of any URL with any parameters — before you run anything at scale.
3. There's a `max_cost` parameter that caps what a single request can spend. For autonomous agents, this is the safety rope that stops a misbehaving loop from eating a month's credits in an afternoon.

Billing itself is forgiving in one specific way: **only successful requests (200 and 404 status codes) are charged**, plus requests you actively cancel within 70 seconds. Blocks and timeouts don't burn credits.

The honest budgeting method: take your agent's ten most common URLs, run them through the cost estimator with the exact parameters you'll use, multiply by expected monthly volume, then pick the tier with about 30% headroom. The API Playground lets you verify results on real targets before committing a single paid credit. 👉 [Test your real target URLs with the free trial credits](https://www.scraperapi.com/signup?fp_ref=coupons)

## How it stacks up against the alternatives

The 2026 third-party data is messier than any vendor homepage admits. Here's what independent testing actually measured:

| Dimension | ScraperAPI | ScrapingBee | ZenRows |
| --- | --- | --- | --- |
| JS-rendered median speed | ~3.5s (fastest of the three) | ~4.2s | ~4.8s |
| Measured success on protected sites | ~62–64% (Proxyway 2026) | Moderate, struggles on aggressive Cloudflare | Vendor-claimed 99.93%; ~40% in one July 2026 third-party benchmark |
| Cost per successful page | Lowest on easy/e-commerce (~$0.0005) | Middle (~$0.001–0.002) | Highest on protected (~$0.007) |
| Dedicated e-commerce structured-data endpoints | Yes (Amazon, Google, more) | No | No |
| Free tier | 7-day trial, 5,000 credits | 1,000 credits, no card | $1 allowance, no card |

A few honest conclusions fall out of that table. ScraperAPI is the fastest of the three and cheapest per successful page on easy and e-commerce targets, and its dedicated Amazon/Google endpoints mean your agent doesn't need a separate extraction step — the structured data comes back ready to parse. On heavily protected sites, its measured success rate around 62–64% is weak in absolute terms; competitors tuned per anti-bot vendor do better there, and ZenRows is the most commonly cited option despite the gap between its advertised 99.93% and the ~40% one benchmark measured. That gap is the real lesson: treat every vendor's success-rate claim as a ceiling, not a floor, and never plan capacity around it.

For context on where the whole market sits: an independent 24-URL benchmark across six tools put the best overall success rate at 75% — on hard targets, no scraping API is anywhere near perfect, and Firecrawl scored zero on the entire e-commerce category in that run. ScrapingBee, meanwhile, has no MCP server, which matters if native tool exposure via Model Context Protocol is part of your agent stack. Static content on a tight budget also favors ScrapingBee's $0.083–$0.196 per 1,000 pages on simple targets.

None of this changes the core recommendation for the majority of agent workloads: e-commerce monitoring, price tracking, news aggregation, and anything hitting normal or moderately protected pages is ScraperAPI's sweet spot on cost. 👉 [Check current prices across all ScraperAPI plans](https://www.scraperapi.com/pricing/?fp_ref=coupons)

## Wiring ScraperAPI into your agent

Integration is a plain REST call, so it drops into virtually any agent framework in minutes:

1. Grab your API key from the dashboard after signing up.
2. Point your agent's fetch tool at the ScraperAPI endpoint with your target URL as a parameter.
3. Add parameters per request: `render=true` for SPAs, `autoparse=true` for structured data on supported domains, `country_code` and `session_number` for geo and session control.
4. Read the `sa-credit-cost` response header and log it, so cost per task is observable.
5. Handle 429 with backoff. When you exceed your plan's concurrent-thread cap, the API returns a 429 and turns the request away — no queue, no extra charge — so your retry logic just needs to wait and resend.

Parameters that shape agent behavior cost nothing extra: `wait_for_selector`, `country_code`, `session_number`, `device_type`, `output_format`, and `keep_headers` are all free add-ons. Set `max_cost` on requests from autonomous loops so a single weird page can't quietly burn 75 credits.

One caution for latency-sensitive agents: independent tests on hard targets have clocked responses at 15 seconds and up on defended pages, well above the ~3.5s median. If your agent's reasoning loop blocks on a scrape, design for the slow case — timeouts, parallel calls, or caching — not for the median.

## What ScraperAPI is not right for

Being clear about the limits saves you a refund request later:

- **Aggressive anti-bot is its weak spot.** If most of your agent's targets sit behind hard Cloudflare, Akamai, or DataDome setups, the measured 62–64% success rate means roughly a third of requests failing, and each bypass attempt costs 10 credits even when the bypass logic works. Per-vendor-tuned alternatives may serve you better here.
- **Tiny budgets scraping static pages.** ScraperAPI's free allowance is a one-time 1,000 signup credits plus the 7-day trial, while ScrapingBee offers a permanent 1,000-credit monthly free tier. For trivial static scraping, that difference matters.
- **Rich media.** Competitors offer PDF generation and custom JS injection; ScraperAPI does neither, per the same head-to-head comparison.
- **Pre-built site catalogs.** If you just need structured data from a handful of big sites at volume, services with pre-built per-site endpoints (like Anakin's Wire, covering 940+ sites) can undercut running a general scraper, since you're paying for clean JSON rather than proxy and rendering cycles.

## FAQ

**Is there a free trial?** Yes — 7 days with 5,000 API credits, no credit card required. There's also a signup allowance of 1,000 credits with up to 5 concurrent connections, and you can contact support for extra testing credits.

**Do failed scrapes cost credits?** No. Only 200 and 404 responses are billed, plus requests you cancel within 70 seconds. Blocked attempts and timeouts are free.

**What happens when I run out of credits?** Scraping pauses until credits renew. You can opt into auto-upgrade (Hobby through Professional) to move up a tier automatically, or pay-as-you-go (Scaling and above) to keep scraping at a fixed per-credit rate up to a cap you set. Annual plans can also renew early for a fresh batch of credits.

**Can my agent check a URL's cost before scraping it?** Yes — via the cost-check endpoint or the dashboard's Domain Cost Estimator. Pass the full parameter set you intend to use and you get the exact credit price back.

**Which plan for e-commerce monitoring?** Work backwards from the credit math: Amazon costs 5 credits per request before add-ons, so 10,000 rendered Amazon pages a month runs about 150,000 credits — comfortably within Hobby, with room to spare.

## Picking a plan, concretely

- Mostly normal pages, light JS, personal project → **Hobby** ($49/month, or $44.10 on annual billing) is plenty.
- Production agent scraping hundreds of thousands of pages monthly → **Startup** or **Business**, depending on how much rendering you do.
- Heavy Amazon or Google extraction, or daily multi-source pipelines → **Scaling** and up, where the effective per-credit cost drops below $0.10.
- Just checking whether it works on your targets → free signup credits, then the trial, in that order.

Before you commit to any tier, do two things: run your real target URLs through the cost estimator, and spend the trial's 5,000 credits on the exact sites your agent will hit. The plan that looks cheapest on the pricing page and the plan that's cheapest for your actual workload are often different — the credit multipliers are what separate them. 👉 [Claim your trial credits and test before you buy](https://www.scraperapi.com/signup?fp_ref=coupons)
