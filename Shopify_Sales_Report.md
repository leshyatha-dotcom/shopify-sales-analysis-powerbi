# Shopify Sales Analysis — Power BI Project Report

## Stage 0: Problem Statement

**Goal:** This dashboard analyzes Shopify sales data to help store stakeholders (owners, marketing, and operations teams) understand revenue trends, customer purchasing behavior, and retention. It supports decisions like which products to promote, when to run campaigns, and how to reduce customer churn.

**Business questions answered:**
1. What is our total revenue, order volume, and average order value?
2. How does revenue trend over time — are there peaks or dips?
3. Which products generate the most and least revenue?
4. How many unique customers do we have, and what share are repeat buyers?
5. Who are our top-spending customers, and what share of revenue do they contribute?

**Key KPIs tracked:** Total Revenue, Total Orders, Average Order Value, Repeat Purchase Rate, Unique Customers.

**Main objective:** Build an interactive Power BI dashboard that surfaces these KPIs and trends in one view. Three steps: (1) clean and model the data with a star schema, (2) build DAX measures for each KPI, (3) design an interactive dashboard with slicers for exploration.

## Stage 1: Data Preparation & Modeling

- Imported the Shopify sales dataset (7,431 rows, 19 columns) including order ID, customer ID, invoice date, product type, quantity, price, and billing address fields.
- Cleaned the data in Power Query: removed duplicate rows (based on the unique line-item ID — no duplicates were found), replaced 11 missing Product IDs and 4 missing Variant IDs with placeholder values instead of deleting rows (to preserve revenue data), trimmed and standardized text casing on City, Province, Country, and Product Type columns, and converted Product Id/Variant Id to Text type since they are identifiers, not numeric values.
- Built a dedicated **Date table** using `CALENDAR()`, with Year, Month Name, Month Number, and Quarter helper columns, and marked it as an official Date table. This is essential for accurate time-intelligence calculations (month-over-month growth, trend analysis) since it guarantees a continuous, gap-free calendar independent of which dates actually have transactions.
- Created a **star schema**: `Date` (dimension table) connected to `shopify_sales` (fact table) via a Many-to-one relationship on Invoice Date → Date.

## Stage 2: Transaction Performance Analysis

**DAX measures created:**

```
Total Revenue = SUM(shopify_sales[Total Price Usd])
Total Orders = DISTINCTCOUNT(shopify_sales[Order Number])
Average Order Value = DIVIDE([Total Revenue], [Total Orders])
Previous Month Revenue = CALCULATE([Total Revenue], DATEADD('Date'[Date], -1, MONTH))
MoM Growth % = DIVIDE([Total Revenue] - [Previous Month Revenue], [Previous Month Revenue])
```

- **Total Revenue:** 4.60M | **Total Orders:** 7,431 | **Average Order Value:** 618.89
- **Revenue trend:** The dataset spans a short window (18–24 March 2025). Revenue rises steadily from 18 March, peaks around 22 March, then declines toward 24 March — suggesting a mid-week buying peak.
- **Revenue by product:** Running Shoes generate the most revenue, followed by Tennis, Walking, Cycling, and Climbing Shoes. Sandals, Gift Cards, Flip-Flops, and Boots underperform significantly and may benefit from bundling or promotion.
- **Orders by time of day:** A bar chart of Total Orders by Order Hour shows order volume is concentrated in the morning-to-afternoon hours, with far fewer orders overnight. This suggests customer-facing promotions and flash sales would perform best if timed for daytime hours.

## Stage 3: Customer Purchasing Behaviour

- **Unique customers:** 4,431 out of 7,431 total orders — meaning the average customer places about 1.68 orders.
- **Top customers:** A table sorted by Total Revenue shows the highest-spending customers (e.g., Customer 1556 at 7,627.69, Customer 6056 at 4,272.08). These top customers contribute a disproportionate share of revenue and should be prioritized for retention efforts (loyalty offers, early access to new products).
- **Customer segments by spend tier:** Customers were grouped into High, Medium, and Low value tiers based on total spend (a simplified RFM approach). Medium-value customers make up the largest group by count, while a smaller High-value group contributes revenue disproportionate to its size.
- **Returning vs first-time buyers by product:** A stacked chart of Total Revenue by Product Type and Customer Type shows Running Shoes and Tennis Shoes draw strong revenue from both first-time and returning customers, suggesting these categories are effective at both acquisition and retention.

## Stage 4: Retention & Long-Term Customer Value

**DAX measures created:**

```
Repeat Customers = CALCULATE(DISTINCTCOUNT(shopify_sales[Customer Id]), FILTER(VALUES(shopify_sales[Customer Id]), CALCULATE(COUNTROWS(shopify_sales)) > 1))
Repeat Purchase Rate = DIVIDE([Repeat Customers], DISTINCTCOUNT(shopify_sales[Customer Id]))
```

- **Repeat customers:** ~2,000 | **Repeat purchase rate:** 46.02%
- **Purchase frequency:** 1.68 orders per customer on average.
- **CLV (estimated):** 1.04K per customer (Average Order Value × Purchase Frequency).
- **Average gap between first and second purchase:** 2.73 days — customers who return tend to do so quickly, within a few days of their first order.
- **Cohort analysis:** Not built as a full month-by-month cohort table. Since every transaction in this dataset falls within a single week (18–24 March 2025), all customers belong to the same first-purchase cohort, so a cohort table would only show one meaningful row. A longer date range would be needed to track how retention decays across multiple monthly cohorts.
- **Recommendation:** Customers who order more than once are worth the most in lifetime value. Since repeat customers return within ~3 days on average, a follow-up email or discount code sent within 1-2 days of a first purchase could capture even more of this fast-returning segment, pushing the 46% repeat rate higher.

## Stage 5: Dashboard Design & Insights

**Layout:** KPI cards (Revenue, AOV, Orders, Repeat Customers, Repeat Rate) sit at the top for an at-a-glance summary. Below them are trend and breakdown visuals (Revenue by Date, Revenue by Product Type, Unique Customers, Top Customers table). Interactive slicers for **Product Type** and **Date range** let stakeholders filter the entire page.

**Key insights:**
1. Running Shoes are the clear revenue leader — worth featuring prominently in marketing.
2. Revenue peaks mid-week (around 22 March) — a good window for time-limited promotions.
3. Nearly half of customers are repeat buyers (46%), showing decent early loyalty.
4. A small group of top-spending customers contributes a large share of revenue — a VIP/loyalty program could protect this segment.
5. Low-performing categories (Boots, Flip-Flops, Gift Cards) may need bundling, discounting, or reduced inventory investment.

**Recommendations:**
- Increase marketing spend behind Running and Tennis Shoes.
- Launch a loyalty/VIP tier for top-spending customers to protect high-value relationships.
- Test a second-purchase incentive (discount code after first order) to push the repeat rate above 46%.
- Investigate bundling underperforming categories with best-sellers to clear inventory.
