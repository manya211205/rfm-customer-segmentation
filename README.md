# 🛒 Customer RFM Segmentation — UK Online Retail

![Excel](https://img.shields.io/badge/Tool-Excel-217346?style=flat&logo=microsoftexcel&logoColor=white)
![SQL](https://img.shields.io/badge/Tool-SQL-4479A1?style=flat&logo=mysql&logoColor=white)
![Power BI](https://img.shields.io/badge/Tool-Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)

---

## 📌 Project Overview

This project segments **4,338 customers** from a UK-based online retail store into behavioral groups using **RFM (Recency, Frequency, Monetary) Analysis** — a proven customer analytics framework widely used in D2C and e-commerce companies.

The goal is to help the business understand **who their best customers are**, **who is at risk of churning**, and **where to focus retention vs. acquisition efforts**.

---

## 📊 Dataset

| Property | Details |
|---|---|
| Source | [UCI Machine Learning Repository — Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii) |
| Transactions | 541,909 rows |
| Time Period | December 2010 – December 2011 |
| Geography | Primarily United Kingdom |
| Columns | InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country |

---

## 🛠️ Tools Used

- **SQL (SQLite)** — RFM metric calculation, scoring, and segmentation logic
- **Excel** — Dashboard, charts, and segment visualization
- **Power BI / Tableau** — (optional extension for interactive reporting)

---

## 🔢 Methodology

### Step 1 — Data Cleaning
- Removed rows with missing `CustomerID` (~135K rows)
- Removed cancelled orders (InvoiceNo starting with 'C')
- Removed zero or negative `Quantity` and `UnitPrice` values
- Final clean dataset: **397,884 rows | 4,338 customers**

### Step 2 — RFM Metric Calculation (SQL)
Calculated three core metrics per customer using SQL aggregation:

| Metric | Definition |
|---|---|
| **Recency** | Days since last purchase (vs. snapshot date: 2011-12-10) |
| **Frequency** | Total number of distinct orders placed |
| **Monetary** | Total revenue generated (Quantity × UnitPrice) |

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

### Step 3 — RFM Scoring (NTILE)
Each customer was scored 1–5 on each metric using SQL `NTILE(5)` window functions:
- **R Score 5** = most recent buyers
- **F Score 5** = most frequent buyers
- **M Score 5** = highest spenders

```sql
SELECT *,
    NTILE(5) OVER (ORDER BY recency  DESC) AS r_score,
    NTILE(5) OVER (ORDER BY frequency ASC) AS f_score,
    NTILE(5) OVER (ORDER BY monetary  ASC) AS m_score
FROM rfm_raw
```

### Step 4 — Segment Assignment (CASE WHEN)
8 segments assigned using business logic in SQL:

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

## 📈 Key Findings

| Segment | Customers | Revenue (£) | Revenue Share |
|---|---|---|---|
| 🏆 Champions | 1,010 | £58,65,418 | **65.8%** |
| 💚 Loyal Customers | 945 | £14,72,718 | 16.5% |
| ⚠️ At Risk | 536 | £6,23,412 | 7.0% |
| 🔴 Lost | 915 | £3,21,009 | 3.6% |
| Others | 938 | £6,09,827 | 7.1% |

### 💡 Business Insights

1. **Champions drive the business** — Just 23% of customers (Champions) generate 66% of total revenue. Retaining these customers is the #1 priority.

2. **At-Risk segment is a big opportunity** — 536 high-value customers haven't purchased recently. A targeted re-engagement campaign (discount, loyalty reward) could recover significant revenue.

3. **915 Lost customers** — These customers haven't purchased in a long time. A win-back email campaign with a strong offer could reactivate even 10–15% of them.

4. **Recent Customers need nurturing** — 307 customers made their first or recent purchase but haven't become loyal yet. Onboarding flows and follow-up offers can convert them to Loyal Customers.

---

## 📁 Files in This Repository

| File | Description |
|---|---|
| `RFM_Customer_Segmentation.xlsx` | Full Excel workbook with dashboard, RFM data, top customers, and SQL documentation |
| `README.md` | Project documentation (this file) |

---

## 🗂️ Excel Workbook Structure

The Excel file contains **4 sheets**:

| Sheet | Contents |
|---|---|
| 📊 RFM Dashboard | KPI cards, segment summary table, revenue bar chart, customer pie chart |
| 📋 RFM Data | All 4,338 customers with RFM scores and segment labels |
| 🏆 Top 20 Customers | Highest-value customers with gold/silver/bronze highlights |
| 📝 SQL Queries | Full SQL code used — documented step by step |

---

## 🚀 How to Reproduce

1. Download the dataset from [Kaggle — UK Online Retail](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci)
2. Load into SQLite / any SQL environment
3. Run the SQL queries in the **📝 SQL Queries** sheet in order (Steps 1–4)
4. Export results to Excel and replicate the dashboard

---

## 👤 Author

Manya Pahwa
- 📧 manaypahwa21@gmail.com
---


