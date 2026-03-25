# Customer Segmentation using RFM Analysis — UK Online Retail

---

## Why I did this project

I kept seeing RFM analysis mentioned in data analytics job descriptions and YouTube videos about e-commerce analytics, so I wanted to actually try it myself rather than just know what the acronym stands for.

I found the UK Online Retail dataset on Kaggle — it's a real transactional dataset from a UK gift shop with over 500K rows. Felt like a good enough dataset to work with since it's messy in the right ways (missing customer IDs, cancelled orders, weird negative quantities) and actually requires some cleaning before you can do anything useful with it.

---

## What the project does

The idea is pretty simple — instead of treating all customers the same, RFM analysis scores every customer on three things:

- **Recency** — how recently did they buy?
- **Frequency** — how often do they buy?
- **Monetary** — how much have they spent in total?

Each customer gets a score from 1 to 5 on each metric, and based on those scores they get placed into a segment like "Champion", "At Risk", or "Lost". The business can then target each group differently instead of sending the same generic email to everyone.

---

## Tools I used

- SQL for all the analysis and scoring logic
- Excel for the dashboard and charts

---

## Dataset

UK Online Retail II dataset from Kaggle — 541,909 transactions between Dec 2010 and Dec 2011.

---

## What I actually did

### Cleaning the data first

The raw data had some issues I had to deal with before touching any analysis:
- Around 135,000 rows had no CustomerID — dropped those since you can't do customer-level analysis without an ID
- Cancelled orders had InvoiceNo starting with "C" and negative quantities — removed those too
- A few rows had UnitPrice as 0 which didn't make sense, filtered those out

After cleaning I was left with about 397,884 rows and 4,338 unique customers.

### Calculating RFM metrics in SQL

```sql
SELECT
    CustomerID,
    JULIANDAY('2011-12-10') - JULIANDAY(MAX(DATE(InvoiceDate))) AS recency,
    COUNT(DISTINCT InvoiceNo)           AS frequency,
    ROUND(SUM(Quantity * UnitPrice), 2) AS monetary
FROM online_retail
WHERE CustomerID IS NOT NULL
  AND Quantity > 0 AND UnitPrice > 0
GROUP BY CustomerID
```

### Scoring customers 1–5 using NTILE

This was actually the part that confused me the most. For Recency, a lower number of days is better (means they bought recently), so the ORDER BY has to be DESC — I got this backwards the first time and all my Champions were actually the oldest customers which made no sense.

```sql
SELECT *,
    NTILE(5) OVER (ORDER BY recency  DESC) AS r_score,
    NTILE(5) OVER (ORDER BY frequency ASC) AS f_score,
    NTILE(5) OVER (ORDER BY monetary  ASC) AS m_score
FROM rfm_raw
```

### Assigning segments with CASE WHEN

```sql
CASE
    WHEN r_score>=4 AND f_score>=4 AND m_score>=4  THEN 'Champions'
    WHEN r_score>=3 AND f_score>=3                 THEN 'Loyal Customers'
    WHEN r_score>=4 AND f_score<=2                 THEN 'Recent Customers'
    WHEN r_score>=3 AND f_score<=2 AND m_score>=3  THEN 'Potential Loyalists'
    WHEN r_score<=2 AND f_score>=3 AND m_score>=3  THEN 'At Risk'
    WHEN r_score<=2 AND f_score>=4                 THEN 'Cant Lose Them'
    WHEN r_score<=2 AND f_score<=2 AND m_score<=2  THEN 'Lost'
    ELSE 'Need Attention'
END AS segment
```

---

## Results

| Segment | Customers | Revenue (£) | Revenue Share |
|---|---|---|---|
| Champions | 1,010 | £58,65,418 | 65.8% |
| Loyal Customers | 945 | £14,72,718 | 16.5% |
| At Risk | 536 | £6,23,412 | 7.0% |
| Lost | 915 | £3,21,009 | 3.6% |
| Others | 938 | £6,09,827 | 7.1% |

The thing that stood out to me most was how concentrated the revenue was. I expected it to be more evenly spread but Champions — which is only about 1 in 4 customers — were responsible for nearly 66% of all revenue. Classic Pareto but still surprising to see it play out so clearly in real data.

The other thing I found interesting was the "At Risk" segment — 536 customers who used to buy frequently and spend a lot, but haven't come back recently. These are probably the most valuable people to target with a re-engagement campaign because they already know the brand.

---

## What's in the Excel file

The workbook has 4 sheets:
- Dashboard with KPI cards and charts
- Full table of all 4,338 customers with their RFM scores and segment
- Top 20 customers by revenue
- All the SQL queries documented step by step

---

## Author

Manya Pahwa
manyapahwa21@gmail.com

