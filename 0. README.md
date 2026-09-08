
# retrospective-analytics-engineering-project

# Headline
### In summer of 2026, I built end-to-end commercial reporting for a Luxury Fashion Retailer on Shopify (+1 physical London store). It reduced time-to-insights from days to minutes. This in turn drove a +15% uplift in weekly Revenue. 

I also saved my client 100 euros per month in costs by opting for Google's Dataform over dbt (for data transformation & logic). 

# "What previously took hours across multiple spreadsheets can now be understood in minutes." — Robena, Commercial Director, Voyeur Voyeur

# Why this repo
The solution delivered works and the client is gaining value from it. But as I am a career Merchandiser, this is a retrospective to see what steps were missed and what could be improved from an Analytics Engineering perspective. 

# Problem
The operational reality - Robena, the commercial director, was spending days pulling together reports to gain insights. The process took so long, she had resulted to only doing so once per month, meaning she was always looking at severely lagging data. Robena is an excellent commercial director. A slick operator, but she was time-poor. 

## Problems in the data
If there is one thing I love, it's getting my grubby hands on a bit of raw data. Shopify has a few weird quirks. After looking at the source data, drilling down, isolating cases, I noticed a couple of things: 

### Weird quirk 1 - transaction discount vs line item discount
when a discount code is used, discount is only applied at the transaction, and not aportioned to the line leven or lineitem. That's a big problem. That means, rolling up and drilling down to the lowest level is not possible. 

### Weird quirk 2 - partial returns
You only know that, for a partial return, something was returned. The transaction gets attributed with a "partial_return" and there is a return value. However, you do not know which of the products in a multi-product transaction was returned. Again, big problem. No available solution with the data provided and available. This impacted about <2% of revenue so explained to Robena we would find a workardound but it would not be fixed. 

## Why was the existing reporting solution insufficient 
Because of the above data quirks. There was no enrichment too, meaning Robena could not simply pull off a report and roll up and drill down on her performance. No cost prices meant no visibility on profit. 

## Cost of not doing anything
Decisions made on gut feel rather than being data-driven. 

# Solution

## Goal was to work with the client in an agile way and deliver something useful as quickly as possible. This project took about 6 weeks, with the majority of time spent having free flowing conversations about the business and key metrics. I would work on something, we would discuss, then I would iterate. I think this worked very well. 

## What Mondays look like now
Robena, the commercial director, exports CSVs from Shopify and uploads them to Big Query. This takes about 10 minutes each week. Transformation & business logic is applied via dbt/Dataform and a Looker dashboard instantly updates, with multiple pages for increasing levels of granularity in performance. Answers: what sold last week and at what margin; which SKUs, brands and categories are driving profitable growth; which SKUs are selling at poor margins or are markdown-exposed; and what needs action this week. A template view also lets the team share sell-through performance with brand partners.

## What I built - one sentence per layer

- **Sources**: Shopify orders, products and inventory exports, the buyer's range plan, and a sample sale file.
- **Ingestion**: Weekly CSV uploads into untouched BigQuery raw tables, with personal data deleted before anything leaves Shopify.
- **Warehouse**: A raw layer preserved as-delivered, plus a purpose-built date dimension carrying ISO weeks, 4-4-5 retail months and the client's season calendar.
- **Transformation**: Dataform models that clean and standardise the sources, allocate transaction-level discounts and order costs down to order lines, enrich sales with range-plan attributes, and compute retail KPIs on a single declared grain.
- **Activation**: Looker Studio pages that drill from business summary → brand/category → SKU.

## What proof do I have it worked?
Weekly trade reporting went from days of manual assembly to a refresh measured in minutes. Evidence status: client-reported / directional. Client feedback: "Out of all freelancers we've worked with, you've added the most value."

# High-Level Architecture & DAG
<img width="896" height="815" alt="ink" src="https://github.com/user-attachments/assets/eea7a2a2-6f54-44ba-a6cc-fdb3b22ae9a5" />

<img width="955" height="365" alt="Screenshot 2026-09-08 at 11 50 43" src="https://github.com/user-attachments/assets/25e98402-c829-49a9-aad4-c9dc9140b1a3" />

<img width="952" height="344" alt="Screenshot 2026-09-08 at 11 50 50" src="https://github.com/user-attachments/assets/83ebc5ef-bb2a-4ac8-99d7-400dc95c0547" />

## Tech Stack
- **BigQuery** — serverless warehouse, no infrastructure for the client to manage, generous free tier at this data volume.
- **Dataform** (over dbt Cloud) — dbt was initially used to build, but then I took the SQL code and migrated it to Dataform (Google’s alternative) to save the client the 100 Euro-per-month cost. Same modular-SQL workflow (dbt patterns) but native to Google Cloud Platform with no additional seat cost for the client. (ADR-001)
- **Looker Studio** — free, familiar to the team, and stakeholder-friendly without BI licences.

# Sources
| Source | Grain | Key fields |
|---|---|---|
| Shopify orders export | Order line item | Qty, price, discounts, refunds, shipping, tax |
| Shopify products export | Product variant | Variant SKU, live selling price |
| Shopify inventory export | SKU (snapshot) | SKU, current stock on hand |
| Range plan (buyer's sheet) | SKU | Brand, season, category, cost, RRP, qty received |
| Sample sale file | Order line item | Same structure as Shopify orders (formatted before upload) |
| Date dimension (built, not sourced) | Date | ISO year/week, 4-4-5 month, season | 

## What was wrong with each source on day 1

- **Shopify orders**: discounts recorded at transaction level, not line level; contained PII.
- **Shopify products**: no commercial attributes (brand, season, cost, RRP) — drill-down impossible from Shopify alone.
- **Shopify inventory**: snapshot only — no received-units history.
- **Range plan**: manual formatting (currency symbols, merged cells) that broke ingestion.
- **Sample sales**: no order numbers — IDs had to be generated in SQL.
- **Trading Calendar**: The retail season calendar existed nowhere in any system — only in the Commercial Director's head (SS26 = Nov 2025–Oct 2026; W26 = May 2026–Apr 2027). Extracting that business rule from a human and encoding it as data was itself a sourcing task.

## What was not pulled / out of scope
Customer level analysis/channel data. Order or Receipt tracking (full picture of commitment was obtained from the original order). 

# Ingestion
To refresh the data, the team exports the weekly Shopify CSVs, confirms the range plan is clean and current, uploads to BigQuery raw tables, and runs the Dataform workflow. Approximately 10-15 minutes work per week.

## Why manual CSVs and not an API
Deliberate trade-off, optimised for adoption over automation. The client team already worked with these exports daily — the data stayed visible and tactile, and they could verify with their own eyes that the right file was uploaded. An API would remove a manual step but introduce an invisible failure surface (batching, pagination, auth) that a non-technical team couldn't inspect or trust. 

## Handling of sensitive customer data
Deleted from Shopify CSVs before upload and process written into the handover documents. Roughly 8 columns of data, situated next to eachother. Client simply highlights and deletes. 

# Warehouse

## Why BigQuery?
Serverless, cost-effective at this scale, and native to the client's Google ecosystem.

## Raw layer organisation and additional table builds
Raw tables land exactly as exported — the "photocopier, not editor" rule. Nothing is cleaned or renamed in the raw layer, so any downstream bug can be reconciled against an untouched copy of what the source actually said. dim_date: a purpose-built calendar dimension mapping every date to ISO year, ISO week, 4-4-5 retail month, and the client's commercial season. This is what makes like-for-like weekly and seasonal comparison possible at all.

# Transformation & Semantic Layer

## Grain Model
One row per order line item. Profiling sample transactions (a multi-line order, a partial return, a discount-code order) revealed that Shopify records discounts at transaction level — so I built allocation logic to apportion discounts, shipping, tax and refunds down to lines pro-rata by each line's share of the order.

## Code organization

| Layer | Model | What it does |
|---|---|---|
| Staging | `stg_order` | Cleans and unions Shopify + sample sale line items; one row per order line |
| Staging | `stg_products` | Variant SKU → live selling price |
| Staging | `stg_inventory` | Current stock snapshot per SKU |
| Staging | `stg_raw_range_sheet` | Brand, season, category, cost, RRP per SKU |
| Staging | `stg_dim_date` | ISO week, 4-4-5 month, season calendar |
| Intermediate | `int_orders_filled` | Allocates transaction-level discounts, shipping, tax, refunds to lines |
| Intermediate | `int_sku_sales_rolling` | PW / PWLY / 4-week / YTD rolling sales per SKU |
| Facts | `fct_order_lines` | KPI fact table: net sales, COGS, margin, markdown flags |
| Facts | `fct_inventory` | Stock fact per SKU |
| Dimensions | `dim_products` | Range plan + live price, one row per SKU |
| Marts | `mart_sku_performance` | Sell-through, weeks of cover, margin by SKU |
| Marts | `mart_inventory_enriched` | 28-day velocity and stock-risk view |                        


## KPI Definitions
All KPI definitions live in docs/kpi-dictionary.md, validated against standard retail math. Of 14 KPIs: 9 kept, 2 renamed (net_sales_gbp → net-sales-before-returns; sales_at_cost_gbp → COGS), 2 rethought (sell-through moved to received units; weeks of cover standardised on a 28-day trailing average), 1 created (full-price sales %). Full decisions and formulas in the dictionary.

# Quality & Trust
Raw-to-model totals compared for gross sales and units; allocated line-level discounts verified to sum back to the original order-level amounts; SKUs missing costs surfaced (they'd silently corrupt margin); duplicate order lines from overlapping weekly exports checked; all sales KPIs gated on paid orders; all ratios use SAFE_DIVIDE so zero-stock or zero-sales SKUs return NULL instead of breaking the pipeline.

# Activation & BI - Looker
<img width="1266" height="582" alt="Screenshot 2026-09-08 at 13 13 11" src="https://github.com/user-attachments/assets/8634a56c-5c91-4108-ab95-6b829478a718" />

<img width="675" height="286" alt="Screenshot 2026-09-08 at 13 17 38" src="https://github.com/user-attachments/assets/b2f7dca9-85bf-46d3-8a08-182e2d2e2a83" />

# Simplicity over frills - getting the reader to the answers fast
The looker dashboard was a series of sheets that contained the same KPIs from left to right (in my experience owning 200M Euro businesses, I want my eyes to get used to seeing the same important KPIs in context). Then, different dimensions can be instantly accessed by going to the relevant page, with increasingly granular detail, all the way to Product. 

## KPIs in the Looker Sheet

### 💰 Sales performance (how much, and is it moving?)

- **Lifetime Net Sales (GBP)** — total net revenue per season/category/brand/SKU since launch. *Why:* the headline measure of what each part of the range has actually contributed.
- **P-4W Net Sales (GBP)** — net sales over the past 4 weeks. *Why:* the "current trading" number — smooths single-week noise so Monday decisions aren't driven by one freak week.
- **PW Net Sales (GBP)** — previous week's net sales. *Why:* the freshest read — is it selling *right now*?
- **P-4W vs P-8W (%)** — this 4 weeks vs the prior 4 weeks. *Why:* momentum — is demand accelerating or decaying? (The wall of negative numbers tells the end-of-season markdown story at a glance.)
- **PW vs PW-1 (%)** — this week vs last week. *Why:* immediate reaction check — did last week's action (markdown, push, restock) work?

### 🏷️ Sell-through (is the buy working?)

- **Season Sell-Through (%)** — share of the season's stock sold to date. *Why:* the core buying KPI — did we buy the right things in the right depth?
- **Previous Week Sell-Through (%)** — last week's sell-through gain. *Why:* the weekly pulse of the season — how fast is stock converting *now*?
- **Weekly Average 4-Week Sell-Through (%)** — trailing 4-week average weekly sell-through. *Why:* smooths the weekly pulse so a promo spike doesn't trigger a false reorder.
- **Lifetime Sales Units / PW Sales Units** — units sold, lifetime and last week. *Why:* units, not pounds, drive replenishment and size-curve decisions (a £3,000 coat and a £150 top can both be "1 unit").

### 📦 Stock position (what's the risk?)

- **Stock Weeks Cover** — weeks current stock lasts at current sales pace. *Why:* the reorder/markdown trigger — too low = stockout risk, too high = cash tied up, markdown incoming.
- **Stock on Hand** — units in the warehouse. *Why:* the raw physical position behind every cover calculation.
- **Stock Value at Cost (GBP)** — cash tied up in stock. *Why:* this is the balance-sheet view — what the business has actually spent.
- **Stock Value at Live Price (GBP)** — potential revenue if everything sells at current price. *Why:* paired with at-cost, it frames the margin opportunity still sitting in the warehouse (and the gap between the two columns is the markdown exposure).

### ✂️ Discounting & margin (are we keeping the money?)

- **Lifetime Discount % / PW Discount %** — depth of discounting, lifetime and last week. *Why:* the brand-equity guardrail — the PW column catches creeping promo dependence before it becomes the strategy.
- **Lifetime Contribution Margin % / PW Contribution Margin %** — profit kept per pound of net sales, after costs. *Why:* the "is this worth selling?" KPI.
- **Lifetime Return Rate % / PW Return Rate** — share of sales coming back. *Why:* quality/sizing surveillance — a rising PW return rate on a strong seller is the margin-eraser early warning.

### 🔍 The dimensions (the drill path)

Season → pre/main → brand → category → sub-category → product name → size → colour → gender.

# AI Layer
AI was out of scope for this project, but it could have been layered in. From a Data Product perspective, I could have built an n8n pipeline that serves organised data to an LLM (frontier or Local) and then the output is a list of the most pressing actions to take, listed by the monetary impact (discount these SKUs, remove discount on these SKUs, buy more of these SKUs, reduce commitment on these SKUs). 

## Human vs AI in the workflow
What must remain Human in the Analytics Engineering workflow? Grain declarations, KPI definitions, QA judgment, and the commercial truth of what a number means. AI can draft SQL and documentation; the decision of what should be measured stays with the person who understands the business. **This is how I approached this project**. 

# Delivery & Handover
The following papers were created to handover the project, and I also delivered a 1.5 hour handover meeting with the commercial director. Papers produced: 
- master SQL (dbt/Dataform) file
- data architecture diagram + DAG
- KPI dictionary
- Looker dashboard guide
- updating and maintenance guide
- handover - who needs to do what (setting up roles, taking ownership of environments)
- Google cloud and why it won't cost you anything (yet)
- Data security and governance (handling sensitve data)
- I also produced a NotebookLM podcast handover to the client for rapid onboarding for a busy operator

# Conclusion
1. What was the outcome, restated in one sentence?
A luxury retailer moved from days of manual monthly reporting to a governed, self-serve weekly trade view. One set of trusted numbers, refreshed in minutes, that drives Monday-morning action.

2. What are the three things I'd do differently?
- **Decide KPI definitions before writing SQL**. In large part, this was done. Weeks of cover existed in two conflicting versions because definitions were discovered in code rather than agreed in a dictionary first.
Automate the manual controls.
- **PII deletion** and range-plan formatting checks are manual steps; they should be scripted so a busy week can't break them.
- **Tests from day one**. Reconciliation was manual spot-checking; next time, automated tests run on every refresh so trust is continuous, not remembered.

3. What would Phase 2 of this project be?
Scheduled API-based ingestion replacing manual CSV upload and automated test suite with alerting.


