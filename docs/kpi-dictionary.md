### 📖 KPI Dictionary — Full List

1. **Net Sales**
   - **Definition:** Revenue after deducting discounts, allowances and customer returns
   - **Formula:** Gross Sales − Returns − Discounts
   - **Grain:** Line item / order
   - **Decision it drives:** Top-line health, sales growth tracking, budget performance
   - **SQL field:** `net_sales_gbp`
   - **Verdict:** ✏️ **Rename** → `net_sales_before_returns_gbp` — it doesn't deduct refunds; compute true net sales (net sales before returns − refunds) at the reporting layer - which was done. 

2. **Gross Sales Before Discount**
   - **Definition:** Total revenue at standard listed selling price, before checkout promos or refunds
   - **Formula:** Listed Selling Price × Quantity
   - **Grain:** Line item
   - **Decision it drives:** E-commerce demand, list-price effectiveness
   - **SQL field:** `gross_sales_before_discount_gbp`
   - **Verdict:** ✅ **Keep** — correct, maps perfectly to Shopify's structure

3. **Gross Sales if at RRP**
   - **Definition:** Potential revenue if all units sold at full Recommended Retail Price
   - **Formula:** RRP × Quantity
   - **Grain:** Line item
   - **Decision it drives:** Brand equity value, measuring markdown depth
   - **SQL field:** `gross_sales_if_at_rrp_gbp`
   - **Verdict:** ✅ **Keep**

4. **Checkout Discount**
   - **Definition:** Cart-level promotional codes or discounts applied at checkout
   - **Formula:** Sum of discount amounts applied
   - **Grain:** Line item / order
   - **Decision it drives:** Promotion code effectiveness, checkout conversion
   - **SQL field:** `checkout_discount_gbp`
   - **Verdict:** ✅ **Keep**

5. **RRP Markdown**
   - **Definition:** Total value written off from RRP to the net selling price on sold units
   - **Formula:** (RRP × Qty) − Net Sales
   - **Grain:** Line item / SKU
   - **Decision it drives:** Margin erosion tracking, clearance planning
   - **SQL field:** `rrp_markdown_gbp`
   - **Verdict:** ✅ **Keep** — note this is markdown on sold units, not remaining inventory

6. **Sales Units**
   - **Definition:** Physical count of products sold
   - **Formula:** Sum of units sold
   - **Grain:** SKU / order
   - **Decision it drives:** Inventory replenishment, velocity tracking, space allocation
   - **SQL field:** `sales_units` (filtered to paid orders)
   - **Verdict:** ✅ **Keep**

7. **Returns / Return Rate**
   - **Definition:** Value or quantity of returned inventory
   - **Formula:** Refunds / Net Sales × 100
   - **Grain:** Line item / SKU
   - **Decision it drives:** Quality control, sizing issues, supplier audits
   - **SQL fields:** `is_returned`, `return_quantity`, `refund_amount_gbp`
   - **Verdict:** ✅ **Keep** — highly structured

8. **COGS (Cost of Goods Sold)**
   - **Definition:** The wholesale cost of goods sold
   - **Formula:** Sales Units × Unit Cost
   - **Grain:** Line item / SKU
   - **Decision it drives:** Profitability, gross margin, supplier renegotiations
   - **SQL field:** `sales_at_cost_gbp`
   - **Verdict:** ✏️ **Rename** → `cogs_gbp` to match standard retail accounting terms

9. **Gross Profit**
   - **Definition:** Raw profit after subtracting cost of goods sold
   - **Formula:** Net Sales − COGS
   - **Grain:** SKU / store / period
   - **Decision it drives:** Business viability, covering operating expenses
   - **SQL fields:** `gross_profit_gbp`, `lifetime_true_gross_profit` (also deducts sample execution costs)
   - **Verdict:** ✅ **Keep**

10. **Gross Margin %**
    - **Definition:** Percentage of net sales kept as gross profit
    - **Formula:** Gross Profit / Net Sales × 100
    - **Grain:** SKU / store (aggregation)
    - **Decision it drives:** Pricing strategy, discount ceilings, floor space productivity
    - **SQL field:** `lifetime_gross_margin_pct`
    - **Verdict:** ✅ **Keep** — BI tools must always calculate this dynamically; never sum or average the % column

11. **Full-Price Sales %**
    - **Definition:** Proportion of sales made at full list price
    - **Formula:** Full-Price Gross Sales / Gross Sales Before Discount × 100
    - **Grain:** SKU / category
    - **Decision it drives:** Brand strength, markdown avoidance
    - **SQL field:** `full_price_gross_sales_gbp` exists, but no percentage field
    - **Verdict:** ➕ **Create** → `full_price_sales_pct = SAFE_DIVIDE(full_price_gross_sales_gbp, gross_sales_before_discount_gbp)`. This was actually done at the report/looker level with a calculated field. 

12. **Discounted Gross Sales**
    - **Definition:** Gross revenue of items sold with a checkout discount
    - **Formula:** Gross sales value where discount > 0
    - **Grain:** Line item / SKU
    - **Decision it drives:** Promotional dependence evaluation
    - **SQL field:** `discounted_gross_sales_gbp`
    - **Verdict:** ✅ **Keep**

13. **Sell-Through Rate**
    - **Definition:** Percentage of received stock sold
    - **Formula:** Units Sold / Received Units × 100
    - **Grain:** SKU / brand / category
    - **Decision it drives:** Stock health, velocity, buy performance
    - **SQL field:** `sell_through_rate` (uses fallback lifetime inventory proxy)
    - **Verdict:** 🔄 **Rethink** — swap the proxy denominator for the imported `received_units` column. In the spirit of getting something useful as quickly as possible and that places performance into the context for the whole season, we agreed to use Ordered units for the demoninator. Ideally, we would have 2 measures, Total Sellthrough and Received Sellthrough

14. **Weeks of Cover**
    - **Definition:** Estimated weeks current stock will last at current sales pace
    - **Formula:** Current Stock / Average Weekly Sales
    - **Grain:** SKU / store / warehouse
    - **Decision it drives:** Re-ordering schedule, markdown clearance timing, out-of-stock risk
    - **SQL fields:** `stock_weeks_cover` (PW-based) vs `weeks_of_cover` (28-day average-based)
    - **Verdict:** 🔄 **Rethink** — standardise on the 28-day trailing average (`weeks_of_cover`); deprecate the PW version. Again, in the spirit of moving in an agile way and producing the MVP, we opted to go for a Spot cover dividing previous weeks sales units into current stock holding units. Not completely optimised, but still highly useful and actionable. 

---


