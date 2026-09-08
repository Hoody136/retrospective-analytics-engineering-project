# retrospective-analytics-engineering-project

# Headline
### In summer of 2026, I built end-to-end commercial reporting for a Luxury Fashion Retailer on Shopify (+1 physical London store). It reduced time-to-insights from days to minutes. This in turn drove a +15% uplift in weekly Revenue. 

I also saved my client 100 euros per month in costs by opting for Google's Dataform over dbt (for data transformation & logic). 

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
- | **Source**                          | **Grain**        | **Key fields**  
- | Shopify orders export               | Order line item  | Qty/price/discounts/refunds/shipping etc     
- | Shopify products export             | Product variant  | Variant SKU, live selling price
- | Shopify inventory export            | SKU (snapshot)   | SKU, current stock on hand
- | Range plan (buyer's sheet)          | SKU              | Brand/season/category/cost/RRP/qty received etc
- | Sample sale file                    | Order line item  | Same structure as Shopify orders (formatted before upload)
- | Date dimension (built, not sourced) | Date             | ISO year/week, 4-4-5 month, season       

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



