# Olist E-Commerce — Revenue, Delivery & Retention Dashboard

**Analyst:** Duncan Chicho (duncanalyst) | Open to Remote  
**Level:** Intermediate-Advanced Power BI Analyst  
**Dataset:** Olist Brazilian E-Commerce Dataset — Kaggle.com  
**Records:** 99,441 orders | 4 relational tables | 415,418 total rows  

---

## Tools & Skills Used

| Tool | Purpose |
|---|---|
| Microsoft SQL Server (SSMS) | Data storage, cleaning, modelling |
| Power BI Desktop | Dashboard design and visualisation |
| Power Query (M Language) | Data transformation and calculated columns |
| DAX | Custom measures — delivery days, repeat rate, avg order value |

---

## Pipeline

- Kaggle Dataset
  ↓
- Microsoft SQL Server — 4 tables, PKs/FKs enforced, data quality audited
  ↓
- Power BI Desktop — SQL Server connector (Import mode)
  ↓
- Power Query — cleaning, YearMonth column, Year column
  ↓
 -Model View — star schema, 3 relationships
↓
- 2-Page Interactive Dashboard
  ↓
- GitHub Portfolio
---

## Business Problem

Olist is Brazil's largest e-commerce marketplace. This dashboard answers  
three core business questions:

1. **Growth** — How did order volume trend over 24 months?
2. **Revenue** — Which states and payment types drive the most value?
3. **Retention** — Are customers coming back?

---

## Dashboard Pages

**Page 1 — Executive Summary**
- Total Orders KPI | Total Revenue KPI
- Revenue by State bar chart
- Order Volume Growth line chart (2016—2018)
- Year slicer — filter all visuals by year
- Reset bookmark button

**Page 2 — Delivery Performance** *(Hidden — accessible via drill through only)*
- Avg Delivery Days KPI
- Delivery Performance by State bar chart
- Order Status Breakdown donut
- Right-click any state on Page 1 Revenue by State chart → Drill through → Delivery Performance

**Page 3 — Customer & Payment Analysis**
- Revenue by Payment Type donut
- Avg Order Value by Payment Method bar chart
- Customer Purchase Frequency table
- Repeat Customer Rate KPI — 0.00%

---

## Data Model

| Table | Type | Rows |
|---|---|---|
| Orders | Fact — central | 99,441 |
| OrderItems | Fact | 112,650 |
| Payments | Fact | 103,886 |
| Customers | Dimension | 99,441 |

> Orders → Customers is 1-to-1 in this dataset — reflecting the 0% repeat  
> purchase rate. In a normal e-commerce dataset this relationship would be 1-to-many.

---

## Data Quality & Cleaning

| Table | Issue | Action Taken |
|---|---|---|
| Payments | 3 rows — `not_defined` payment type | Removed in Power Query |
| Orders | 2,965 NULL `order_delivered_customer_date` | Retained — excluded in DAX delivery measures only |
| Orders | Needed YearMonth column for line chart | Added in Power Query — format `2016-09` |
| Orders | Needed Year column for slicer | Added in Power Query — integer format |
| Customers | Clean | No action required |
| OrderItems | Clean | No action required |

---

## DAX Measures

```dax
-- Average delivery days — delivered orders only, NULLs excluded
Avg Delivery Days = 
CALCULATE(
    AVERAGEX(
        FILTER(
            Orders,
            NOT(ISBLANK(Orders[order_delivered_customer_date]))
            && Orders[order_status] = "delivered"
        ),
        DATEDIFF(
            Orders[order_purchase_timestamp],
            Orders[order_delivered_customer_date],
            DAY
        )
    )
)

-- Average order value
Avg Order Value = AVERAGE(Payments[payment_value])

-- Repeat customer rate
Repeat Customer Rate = 
VAR RepeatCustomers = 
    CALCULATE(
        COUNTROWS(Customers),
        FILTER(Customers, CALCULATE(COUNTROWS(Orders)) > 1)
    )
VAR Result = DIVIDE(RepeatCustomers, COUNTROWS(Customers), 0) * 100
RETURN IF(ISBLANK(Result), 0, Result)
```

---

## Key Findings

### Finding 1 — Explosive Growth in 14 Months
- September 2016: 4 orders — platform launch
- October 2016: 324 orders — first marketing push
- November 2017: 7,289 orders — Black Friday Brazil peak
- August 2018: 6,512 orders — sustained mature volume

> **Data note:** November 2016 recorded zero orders — confirmed in raw SQL.  
> September and October 2018 show 16 and 4 orders — dataset truncation at  
> export, not a business decline.

### Finding 2 — Credit Card Dominates Revenue
- Credit card: 78.34% of total revenue | ~$163 avg order value
- Boleto: 17.92% | ~$145 avg order value
- Voucher: 2.37% | ~$63 avg order value
- Credit card customers spend ~$18 more per order than boleto customers

### Finding 3 — Delivery Speed Is a Competitive Problem
- Brazil average: 12 days
- SP average: 8.7 days — fastest, highest revenue state
- Remote northern states: 25—30 days — slowest
- 97.02% of orders reach delivered status

> **Methodology note:** Initial 1-day discrepancy between SQL (8 days) and  
> Power BI (9 days) resolved by aligning DAX filter to `order_status = 'delivered'`  
> only — matching the original SQL WHERE clause exactly. True average: 8.7 days.

### Finding 4 — São Paulo Dominates
- SP: $5.77M revenue | 40,501 orders | 8.7 days avg delivery — Rank 1
- RJ: $2.06M revenue | 12,350 orders — Rank 2
- Remote states: lowest order volume but highest avg order values ($240+)
- Reconciliation note: Power BI reports SP at 40,501 orders and $5,770,266 revenue. Original SQL CTE reported 40,493 orders and $5,769,081 — difference caused by INNER JOIN to Payments excluding 1 unmatched order. Direct count query confirms Power BI figure is correct.

### Finding 5 — Zero Repeat Customers
- 96,478 unique customers — not a single repeat buyer
- Repeat Customer Rate: **0.00%**
- Every customer acquired is lost after one purchase
- Amazon repeat purchase rate: 90%+ — Olist has a fundamental retention gap

---

## Recommendations

**1. Fix delivery speed urgently**  
12-day average is 6x slower than global leaders. Regional fulfilment centres  
in RJ, MG and BA would directly improve the 68% of orders taking over 1 week.

**2. Launch a customer retention programme**  
0% repeat rate is an existential risk. Email re-engagement, loyalty discounts  
and personalised recommendations should be immediate priorities.

**3. Target remote high-value customers**  
States like AP, AC and AL have the highest avg order values ($240+) but lowest  
volumes. A targeted campaign in these states yields high revenue per customer acquired.

**4. Protect and grow credit card customers**  
At 78.34% of revenue and $18 higher avg order value, credit card users are the  
most valuable segment. Instalment offers and exclusive promotions would deepen  
this relationship.

---

## Project Files

| File | Description |
|---|---|
| `OlistEcommerce_Dashboard.pbix` | Power BI dashboard — 3 pages, all visuals and DAX measures |
| `README.md` | This file |
| `screenshots/` | Dashboard page screenshots |

---

## Related Projects

- 🔗 [Olist E-Commerce — SQL Analysis](https://github.com/NC-Dan/olist-ecommerce-sql-analysis) — SQL foundation this dashboard is built on
- 🔗 [IBM HR Attrition — SQL](https://github.com/NC-Dan/ibm-hr-attrition-sql-analysis)
- 🔗 [Credit Card Fraud Detection — SQL](https://github.com/NC-Dan/credit-card-fraud-detection-sql)
- 🔗 [Global Superstore Sales Dashboard — Excel](https://github.com/NC-Dan/global-superstore-sales-dashboard)
- 🔗 [Healthcare Analytics Dashboard — Excel](https://github.com/NC-Dan/healthcare-analytics-dashboard)

---

Connect on LinkedIn 🔗 [linkedin.com/in/duncanalyst](https://www.linkedin.com/in/duncanalyst)  
GitHub Portfolio 🔗 [https://github.com/duncanalyst](https://github.com/duncanalyst)
