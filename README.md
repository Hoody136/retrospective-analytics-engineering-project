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







