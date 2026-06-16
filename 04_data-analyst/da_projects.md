# Data Analytics Projects — 15 Hands-On Exercises

> **How to use this file**
> Projects are graded 🟢 (beginner) → 🟡 (intermediate) → 🔴 (advanced).
> Each project has 15 questions. Attempt each question before checking answers.
> All code should run with: `pandas`, `numpy`, `scipy`, `matplotlib`, `seaborn`, `plotly`, `duckdb`.
> Answers are at the bottom of this file — scroll past the last project.
> The guiding question for every project: **"What decision does this analysis enable?"**

---

## Datasets

| ID | Name | File(s) | What it contains |
|----|------|---------|-----------------|
| D1 | Olist Brazilian E-Commerce | `olist_orders_dataset.csv`, `olist_order_items_dataset.csv`, `olist_order_payments_dataset.csv`, `olist_customers_dataset.csv`, `olist_order_reviews_dataset.csv`, `olist_sellers_dataset.csv`, `olist_products_dataset.csv`, `product_category_name_translation.csv` | Orders, payments, reviews, customers, sellers on a Brazilian marketplace |
| D2 | UK Online Retail II | `online_retail_II.csv` | Transactional retail data, UK-based, 2009–2011 |
| D3 | Healthcare Dataset | `healthcare_dataset.csv` | Synthetic patient admissions, diagnoses, billing, outcomes |
| D4 | Financial Transactions / Fraud | `transactions.csv` | Labelled fraud/non-fraud transaction records |
| D5 | DataCo Supply Chain | `DataCo_Supply_Chain.csv` | Order fulfillment, shipping, product profitability |
| D6 | Airline Passenger Satisfaction | `airline_passenger_satisfaction.csv` | Passenger survey — service ratings, delays, satisfaction |
| D7 | IBM HR Analytics | `WA_Fn-UseC_-HR-Employee-Attrition.csv` | Employee attributes, attrition labels |

---

## Project 1 🟢 — Exploratory Data Analysis: Olist E-Commerce
**Dataset:** D1 — Olist | **Focus:** EDA, distributions, data quality, first impressions

### Business Context
You've just joined Olist's analytics team. Your manager asks you to "get familiar with the data" before your first real project. This is your onboarding EDA.

### Setup
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from pathlib import Path

orders    = pd.read_csv("data/olist/olist_orders_dataset.csv",
                        parse_dates=["order_purchase_timestamp",
                                     "order_delivered_customer_date",
                                     "order_estimated_delivery_date"])
payments  = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
customers = pd.read_csv("data/olist/olist_customers_dataset.csv")
reviews   = pd.read_csv("data/olist/olist_order_reviews_dataset.csv")
items     = pd.read_csv("data/olist/olist_order_items_dataset.csv")
sellers   = pd.read_csv("data/olist/olist_sellers_dataset.csv")
products  = pd.read_csv("data/olist/olist_products_dataset.csv")
category  = pd.read_csv("data/olist/product_category_name_translation.csv")
```

### Questions

**Q1.** Run the EDA checklist on the `orders` table. Report: total rows, number of columns, columns with > 5% nulls, and the date range of `order_purchase_timestamp`. What do you notice about the data coverage?

**Q2.** How many unique customers (`customer_id`) are in the orders table? How many unique customers (`customer_unique_id`) are in the customers table? Why are these numbers different? What does that tell you about how Olist structures its customer identity?

**Q3.** What is the distribution of `order_status`? Create a bar chart showing count and percentage for each status. Which status is dominant, and which ones are unusual enough to warrant investigation?

**Q4.** Compute the total payment per order (aggregate `olist_order_payments_dataset`). Join to orders. Plot the distribution of order values. Report: mean, median, P25, P75, P90, P99. What is the mean-to-median ratio, and what does it tell you?

**Q5.** Plot monthly order volume as a line chart over the full date range. Identify: the peak month, the lowest month (after the dataset gets going — ignore months with very few orders), and any obvious seasonal patterns. What year did Olist grow fastest?

**Q6.** Calculate the on-time delivery rate: orders where `order_delivered_customer_date <= order_estimated_delivery_date`. Report the overall rate. Is it above or below 90%?

**Q7.** What is the distribution of `review_score` (1–5)? Is it normal, skewed, or bimodal? Calculate the mean and median. What percentage of customers gave 5 stars? What percentage gave 1 or 2 stars?

**Q8.** How many sellers are on the platform? What is the distribution of orders per seller (histogram)? What does the top 10% of sellers look like vs the median seller? Is this Pareto-distributed?

**Q9.** Which are the top 10 product categories by number of orders? Join products → category translation → order items → orders. Plot as a horizontal bar chart.

**Q10.** How many orders have more than one item? What percentage of orders are multi-item? What is the average number of items per order? Does multi-item ordering correlate with higher or lower review scores?

**Q11.** Compute `delivery_days` = `order_delivered_customer_date - order_purchase_timestamp` (in days). What is the distribution? Plot a histogram capped at 60 days. What is the median delivery time? What percentage of orders arrive in under 10 days?

**Q12.** Which Brazilian states have the most customers? Which have the highest average order value? Are these the same states, or do they differ? Create a dual-metric table sorted by customer count.

**Q13.** Look for data quality issues: are there orders with `order_delivered_customer_date` before `order_purchase_timestamp`? Are there any duplicate `order_id` values? Are there negative payment amounts? Report what you find.

**Q14.** What payment methods do customers use? (`payment_type` in the payments table.) What share of orders uses credit card vs boleto vs voucher? Do different payment methods correlate with different order values?

**Q15.** Write a 5-bullet "EDA Summary" as if handing off to a colleague who hasn't seen this data. Each bullet should be one insight (not a description of what you did). Format: "Finding: [X]. Implication: [Y]."

---

## Project 2 🟢 — Revenue Analysis: UK Online Retail
**Dataset:** D2 — UK Online Retail II | **Focus:** Revenue metrics, GMV, AOV, seasonality, top/bottom performers

### Business Context
You're an analyst at a UK online retailer. The commercial director wants a revenue health report covering the full 2009–2011 period before a board meeting.

### Setup
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv("data/uk_retail/online_retail_II.csv",
                 encoding="latin-1", low_memory=False)
df.columns = df.columns.str.lower().str.strip().str.replace(" ", "_")
df["invoicedate"] = pd.to_datetime(df["invoicedate"], errors="coerce")
df["revenue"]     = df["quantity"] * df["price"]
```

### Questions

**Q1.** What is the total GMV for the full dataset period? Exclude returns (negative quantity). What percentage of invoice lines are returns? What is the NMV (GMV of non-return lines only)?

**Q2.** Compute monthly GMV. Plot as a line chart. Identify: peak month, any obvious seasonality. Is there a clear year-end peak? What happened in December each year?

**Q3.** What is the average order value (AOV) per invoice? (An invoice = one customer's order.) Note: `invoice` column, not `stockcode`. Plot the distribution of invoice values. Report mean, median, and P90.

**Q4.** Who are the top 10 customers by total spend? What share of total GMV do they represent? What is the bottom 10%'s share? How concentrated is revenue across the customer base?

**Q5.** What are the top 20 products by total revenue? (`stockcode` and `description`.) What percentage of total GMV do the top 20 products represent? Plot as a horizontal bar chart.

**Q6.** Which customers have the highest repeat purchase rate? Define a "purchase" as a unique invoice date (not invoice number — a customer might have multiple invoices same day). Find customers with the most distinct purchase months.

**Q7.** Compute week-of-year average GMV. Which weeks of the year are highest and lowest? Is there a week-level seasonal pattern beyond just December?

**Q8.** The UK is the primary market, but international customers also appear. How many distinct countries are there? What is the GMV by country (top 10)? What percentage of GMV is non-UK?

**Q9.** Are there products that are primarily returned? Find products where returns (negative quantity lines) exceed 30% of forward orders. List the top 10 by return rate. What do they have in common?

**Q10.** Compute the month-over-month GMV growth rate. What was the highest single-month growth? The biggest single-month decline? When did these occur?

**Q11.** Build a customer segmentation by spend tier: bottom 25% / middle 50% / top 25%. For each tier: total customers, total GMV, average GMV per customer, average number of invoices. What does the top 25% look like compared to the bottom 25%?

**Q12.** Is there a day-of-week pattern in orders? Plot average daily GMV by day of week. Are weekends lower than weekdays? By how much?

**Q13.** Find products with unusually high unit prices (P95+). Are these products also high volume? Or high price but low volume? What does the price-volume relationship look like (scatter plot)?

**Q14.** Compute the customer cohort acquisition month: the earliest invoice month per customer. For each acquisition cohort, what is the total 12-month cumulative GMV? Which cohort has generated the most revenue in their first year?

**Q15.** Write a one-page revenue health summary for the board: GMV trend, top 3 findings, and 2 recommendations. Use the Pyramid Principle structure: governing thought → 3 supporting arguments → data bullets.

---

## Project 3 🟢 — Operational Metrics: Healthcare
**Dataset:** D3 — Healthcare | **Focus:** LOS, billing analysis, readmission patterns, operational KPIs

### Business Context
You're an analyst at a hospital group. The COO wants an operational review covering length of stay, billing patterns, and patient outcomes across admission types and medical conditions.

### Setup
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv("data/healthcare/healthcare_dataset.csv")
df["date_of_admission"] = pd.to_datetime(df["Date of Admission"])
df["discharge_date"]    = pd.to_datetime(df["Discharge Date"])
df["los"]               = (df["discharge_date"] - df["date_of_admission"]).dt.days
df.columns              = df.columns.str.lower().str.replace(" ", "_")
```

### Questions

**Q1.** What is the overall average and median Length of Stay (LOS)? Plot the distribution. Is it normal, skewed, or something else? What does the shape tell you operationally?

**Q2.** How does LOS differ by `admission_type` (Emergency / Elective / Urgent)? Create a box plot. Which admission type has the widest variation? What does high LOS variance in one type mean operationally?

**Q3.** Which medical conditions have the highest average LOS? Rank the top 10. Are there conditions with both high LOS AND high billing amounts — the most resource-intensive cases?

**Q4.** What is the distribution of `billing_amount`? Report mean, median, and P90. Is it right-skewed? What is the IQR? Are there billing amounts that look like outliers?

**Q5.** Does LOS correlate with billing amount? Compute the Pearson correlation and plot a scatter with a trend line. Is the relationship linear? What does a weak correlation here tell you?

**Q6.** What is the distribution of patients by `blood_type`? Is it roughly proportional to population blood type distribution (O > A > B > AB)? Does blood type affect LOS or billing?

**Q7.** Which doctors (by `doctor` field) have the highest average billing per patient? The lowest? Is billing variation across doctors significant — or likely within normal range? Run a one-way ANOVA.

**Q8.** What share of patients are admitted on each day of the week? Is there a weekend effect — fewer admissions on Saturday/Sunday? Does LOS differ by admission day of week?

**Q9.** How does billing amount vary by `insurance_provider`? Is there a payer mix pattern — are certain providers associated with longer stays or higher bills? (In real healthcare analytics, this drives revenue cycle strategy.)

**Q10.** What percentage of patients are readmitted within 30 days? (Approximate: filter for patients admitted within 30 days of a prior discharge — requires joining on patient name or ID if available.) If the dataset doesn't support this, compute what you can and explain what additional data you'd need.

**Q11.** Create a hospital operational scorecard: for each `hospital` (if available) or each medical condition: avg LOS, avg billing, patient count, % emergency admissions. Sort by avg billing descending.

**Q12.** Is there a trend in average billing amount over time (by admission year/month)? Plot monthly average billing. Is billing inflation visible in the data?

**Q13.** What test result category (`test_results`: Normal / Abnormal / Inconclusive) is most common? Does test result correlate with LOS or billing? Run a chi-square test: is the distribution of test results independent of admission type?

**Q14.** Build a simple risk stratification: patients are "high risk" if LOS > 10 days OR billing > 75th percentile. What percentage are high risk? What are the top 3 medical conditions in the high-risk group?

**Q15.** Write a 5-point operational summary for the COO. For each point: the metric, the finding, and one operational implication. Use plain language — no statistical jargon.

---

## Project 4 🟢 — Anomaly Analysis: Financial Fraud
**Dataset:** D4 — Financial Transactions / Fraud | **Focus:** Fraud patterns, statistical profiling, anomaly detection

### Business Context
You're joining a fraud analytics team. Before building any model, you need to understand the data: what does fraud look like statistically, and where does it concentrate?

### Setup
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import scipy.stats as stats

df = pd.read_csv("data/fraud/transactions.csv")
```

### Questions

**Q1.** What is the overall fraud rate (% of transactions labelled fraud)? How many total transactions are there? Is this an imbalanced dataset? Why does class imbalance matter for fraud analysis?

**Q2.** What is the distribution of transaction amounts for fraud vs non-fraud? Plot both distributions on the same chart. What are the mean, median, and P90 amounts for each group? Does fraud concentrate in high-value or low-value transactions?

**Q3.** Run a t-test: is the mean transaction amount significantly different between fraud and non-fraud? Report the t-statistic, p-value, and your plain-English interpretation.

**Q4.** Does fraud rate vary by `category` (merchant category)? Create a table: category, fraud count, total count, fraud rate, sorted by fraud rate descending. Which 3 categories have the highest fraud rate?

**Q5.** Run a chi-square test: is fraud independent of merchant category? What does the result tell you about whether category is a useful fraud signal?

**Q6.** Does fraud rate vary by time of day? Create an `hour` column from the transaction timestamp. Plot fraud rate by hour. Is there a time-of-day pattern?

**Q7.** Does fraud rate vary by day of week? Plot fraud rate Monday–Sunday. Is weekend fraud higher or lower?

**Q8.** What is the distribution of transaction amounts for fraud cases specifically? Are there "clusters" — do fraudsters prefer certain amount ranges? Plot a histogram of fraud-only amounts.

**Q9.** Compute z-scores for transaction amount (standardise across all transactions). What percentage of fraud transactions have z-scores > 2? > 3? Compare to non-fraud. Is extreme amount a useful fraud signal?

**Q10.** Are there customers with unusually high fraud rates? (Compute fraud rate per `customer` — if available — or per account.) What does the distribution of per-customer fraud rates look like?

**Q11.** Create a simple rule-based fraud flag: `predicted_fraud = (amount > P95) | (category in top_3_fraud_categories) | (hour in highest_fraud_hours)`. What is the precision and recall of this simple rule on the labelled data?

**Q12.** Plot a confusion matrix for your rule-based flag: true positives, false positives, true negatives, false negatives. What is the cost of a false positive (legitimate transaction flagged) vs a false negative (fraud missed)? Which matters more?

**Q13.** Is there a seasonal pattern in fraud? Plot monthly fraud rate over time. Is fraud concentrated in any month(s)?

**Q14.** Compute the fraud "loss amount": sum of `amount` for all transactions where `is_fraud = 1`. What percentage of total transaction value is fraud loss? If the company processes $10M/month, what does this imply in absolute dollar losses?

**Q15.** Write a fraud risk briefing: 5 bullet points summarising where fraud concentrates (by category, amount, time). For each bullet: the signal, its strength (how much higher is fraud rate vs baseline?), and how it could be used operationally.

---

## Project 5 🟢 — Delivery Performance: Supply Chain
**Dataset:** D5 — DataCo Supply Chain | **Focus:** OTIF, cycle time, SLA analysis, delivery drivers

### Business Context
You're an analyst at a logistics company. The VP of Operations wants to understand delivery performance: where are we meeting SLAs, where are we failing, and what drives late deliveries?

### Setup
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

df = pd.read_csv("data/supply_chain/DataCo_Supply_Chain.csv",
                 encoding="latin-1")
df.columns = df.columns.str.lower().str.strip().str.replace(" ","_")
# Identify and parse date columns
date_cols = [c for c in df.columns if "date" in c.lower()]
for col in date_cols:
    df[col] = pd.to_datetime(df[col], errors="coerce")
```

### Questions

**Q1.** What is the overall OTIF (on-time in-full) rate? Identify the relevant columns for scheduled vs actual delivery. Define late = actual delivery date > scheduled. Report the on-time rate. Is it above or below the 95% industry benchmark?

**Q2.** Compute order cycle time: from order date to actual delivery date. What is the distribution? Mean, median, and P90? Plot a histogram.

**Q3.** Does late delivery rate vary by `shipping_mode`? Compute late rate for each shipping mode. Which mode is most reliable? Which is worst? Create a bar chart.

**Q4.** Does late delivery vary by `customer_region` or `customer_country`? Show the top 5 regions/countries by late delivery rate. Is geography a strong signal?

**Q5.** What is the late delivery rate by `category_name` or `product_category_name`? Are certain product categories systematically late?

**Q6.** Does `order_item_quantity` affect delivery performance? Do larger orders (more items) arrive late more often? Compute late rate by quantity bucket (1 item, 2–5 items, 6+ items).

**Q7.** Plot late delivery rate over time (by month). Is performance improving, declining, or stable? Identify any months with unusually high or low late rates.

**Q8.** What is the distribution of `order_profit_per_order`? Is profit right-skewed? What percentage of orders are unprofitable (negative profit)?

**Q9.** Does late delivery affect profit? Compare average profit per order for on-time vs late orders. Is the difference statistically significant? (Run a t-test.)

**Q10.** What is the overall gross margin (profit/sales)? Compute gross margin by product category. Which categories are most profitable? Which lose money?

**Q11.** Which customers (by `customer_id` or `customer_name`) generate the most revenue? Do high-revenue customers have better or worse delivery SLA rates?

**Q12.** Build an operational scorecard: by `shipping_mode`, compute: order count, total revenue, avg cycle time, on-time rate, avg profit margin. Sort by total revenue.

**Q13.** Is there a day-of-week or time-of-month pattern in order placement? Do orders placed late in the month have higher late delivery rates (end-of-period rush)?

**Q14.** What is the `late_delivery_risk` column distribution (if present)? Does it actually predict late delivery in the actual data? Compute the precision/recall of this risk label.

**Q15.** Write an operations performance brief for the VP: 3 key findings about delivery performance, 2 findings about profitability, and 2 specific recommended actions with expected impact.

---

## Project 6 🟡 — Cohort Analysis: Olist Customer Retention
**Dataset:** D1 — Olist | **Focus:** Cohort retention curves, churn identification, retention trend

### Business Context
The growth team wants to understand customer retention: are recent customer cohorts more loyal than early ones? Is Olist improving at keeping customers, or not?

### Questions

**Q1.** Build the cohort table: assign each `customer_unique_id` their acquisition cohort (month of first order). Count unique customers per cohort.

**Q2.** For each customer, compute all their order months. Join to their cohort. Create a `months_since_acquisition` column.

**Q3.** Build the cohort retention matrix: rows = cohort month, columns = months_since_acquisition (0, 1, 2, ...), values = number of customers active that month.

**Q4.** Normalise the retention matrix by cohort size (Month 0 = 100%). Display as a heatmap. What does the overall retention curve look like?

**Q5.** What is the average Month-1 retention rate across all cohorts? Month-3? Month-6? What do these numbers tell you about customer loyalty?

**Q6.** Compare the first 6 cohorts (earliest) vs the last 6 cohorts (most recent). Is Month-1 retention improving or declining over time?

**Q7.** Plot the average retention curve (across all cohorts) as a line chart. Does it show a "fast drop then plateau" pattern (common in e-commerce) or a "steady decline"?

**Q8.** Identify the cohort with the highest Month-3 retention. What was happening in that period (month, year)? Look at the monthly order volume — was it a high-growth period?

**Q9.** Compute the "at-risk" customers: customers who made their first (and only) purchase more than 90 days ago. How many are there? What % of total customers?

**Q10.** Does repeat purchase rate differ by customer state? Join to the customers table. Which states have the highest and lowest repeat rates?

**Q11.** Does the first-order value predict whether a customer will return? Segment first orders by value tier (bottom 25% / middle 50% / top 25%). Compare return rates across tiers.

**Q12.** Compute revenue cohort analysis: for each cohort, what is the total cumulative revenue generated (summing across all months)? Which cohort is the most valuable?

**Q13.** What is the average time between a customer's first and second order (for customers who do return)? Plot the distribution of "time to second order." What is the median?

**Q14.** Run a statistical test: is the repeat purchase rate significantly different between customers who gave 5-star reviews vs 1–3-star reviews on their first order? What are the implications for customer experience strategy?

**Q15.** Write a cohort analysis summary: 3 key findings about retention patterns, your assessment of whether Olist has a retention problem or not, and one recommendation with a specific metric target (e.g., "improve Month-1 retention from X% to Y% by implementing Z").

---

## Project 7 🟡 — RFM Segmentation: UK Retail
**Dataset:** D2 — UK Online Retail II | **Focus:** Full RFM build, segment business interpretation, action mapping

### Business Context
The marketing team wants to run targeted campaigns but has no customer segmentation. You'll build the full RFM model and translate it into actionable marketing segments.

### Questions

**Q1.** Prepare the data: exclude returns (negative quantity), exclude rows with no `customer_id`. Set a reference date = the last invoice date + 1 day. Compute per-customer: Recency (days since last purchase), Frequency (number of distinct invoices), Monetary (total spend).

**Q2.** Plot the distribution of each RFM dimension (3 separate histograms). Are they skewed? Apply log-transform to Monetary and check if the distribution improves.

**Q3.** Score each dimension 1–5 using quintile bucketing. Remember: for Recency, lower days = higher score. Verify: is each quintile roughly equal in size?

**Q4.** Validate MECE: does every customer have all three scores? Are there any nulls? Print a validation summary.

**Q5.** Define segments using the following rules and assign every customer to exactly one:
- Champions: R≥4, F≥4, M≥4
- Loyal: R≥3, F≥3
- At Risk: R≤2, F≥3
- New Customers: R≥4, F≤2
- Lost: R≤2, F≤2, M≤2
- Potential Loyalist: R≥3, F≤2
- Needs Attention: everything else

**Q6.** How many customers are in each segment? What % of total customers? Plot a bar chart.

**Q7.** For each segment: compute average Recency, Frequency, Monetary, and total GMV contribution. Which segment generates the most total revenue? Which has the highest average spend?

**Q8.** Champions are your best customers. Profile them: average invoice value, most purchased product categories, geographic distribution. What do they buy?

**Q9.** At Risk customers used to be active but haven't bought recently. What was their average time between purchases when active? How long ago did they last buy? What would a win-back campaign look like for them?

**Q10.** The "Needs Attention" segment is often poorly defined. Re-examine yours: what do customers in this segment look like? Should any be reclassified into a more actionable segment?

**Q11.** Is there a seasonal pattern in when customers become "At Risk"? Plot the month of last purchase for At Risk customers. Are they clustering in any period?

**Q12.** Does the RFM segment predict future behaviour? Compute the % of customers in each segment who made a purchase in the final month of the dataset. Champions should have the highest rate; Lost should have the lowest.

**Q13.** Build a one-page segment playbook: for each segment, write: (a) who they are in plain language, (b) recommended marketing action, (c) success metric. This is your deliverable to the marketing team.

**Q14.** What is the total addressable revenue if you could bring all "At Risk" customers back to "Loyal" status? Compute: At Risk count × (Loyal avg spend − At Risk avg spend).

**Q15.** A marketing campaign has a budget of £50,000. Using your RFM analysis, recommend how to allocate that budget across segments. Justify with: expected reach, expected revenue uplift per £ spent, and risk.

---

## Project 8 🟡 — Customer Satisfaction Analysis: Airline
**Dataset:** D6 — Airline Passenger Satisfaction | **Focus:** Driver analysis, NPS-style scoring, segmentation

### Business Context
The airline's customer experience team wants to know: what drives passenger satisfaction, and which segments should be prioritised for improvement?

### Setup
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import scipy.stats as stats

df = pd.read_csv("data/airline/airline_passenger_satisfaction.csv")
```

### Questions

**Q1.** What is the overall satisfaction rate (% satisfied vs neutral/dissatisfied)? Is the dataset balanced between the two classes? Plot the distribution.

**Q2.** How does satisfaction vary by `customer_type` (Loyal vs Disloyal) and `type_of_travel` (Business vs Personal)? Create a cross-tab showing satisfaction rate for each combination. Which combination is most satisfied? Least?

**Q3.** Satisfaction ratings are given for 14 service dimensions (seat comfort, food, etc.). Compute the mean rating for each dimension. Which 3 dimensions score highest? Which 3 score lowest?

**Q4.** Which service dimensions have the strongest correlation with overall satisfaction? Compute the correlation of each service rating with the binary satisfaction label. Rank by correlation strength. This is a simple driver analysis.

**Q5.** Is satisfaction significantly different between short-haul (<1000km) and long-haul flights? Define flight length from `flight_distance`. Run a chi-square test. What does the result suggest about how to segment improvement efforts?

**Q6.** How does satisfaction vary by flight delay? Create buckets: no delay (0 min), short delay (1–30 min), medium delay (31–120 min), long delay (120+ min). Plot satisfaction rate by bucket. At what delay length does satisfaction drop most sharply?

**Q7.** The airline has Economy and Business class passengers. Does the satisfaction driver pattern differ by class? For each class separately, find the top 3 satisfaction drivers. Do Economy passengers care about different things than Business passengers?

**Q8.** Among dissatisfied passengers, which service dimension is most frequently rated 1 or 2 (very poor)? Is this the same dimension as the lowest average score from Q3, or different?

**Q9.** Build a simple NPS-style proxy: passengers who rated satisfaction as "satisfied" are Promoters; "neutral or dissatisfied" are Detractors. Compute NPS by `customer_type`. How different is Loyal NPS vs Disloyal NPS?

**Q10.** Is there an "age effect"? Compute satisfaction rate by age group (bins: 18–25, 26–35, 36–45, 46–55, 55+). Which age group is hardest to satisfy?

**Q11.** Run an ANOVA: does the mean satisfaction score (treat the binary as 0/1) differ significantly across customer age groups? Report F-statistic and p-value.

**Q12.** Compute a "low-effort improvement" list: service dimensions with below-average scores AND high correlation with satisfaction. These are the dimensions where improvement is most likely to move the overall satisfaction needle.

**Q13.** Create a 2×2 prioritisation matrix: x-axis = correlation with satisfaction (importance), y-axis = mean service rating (current performance). Dimensions in the top-left quadrant (high importance, low performance) are your top priorities.

**Q14.** Is there a geographic pattern? If `destination_city` or origin data is available, does satisfaction vary by route? If not, analyse by `class` and `type_of_travel` combinations as a proxy.

**Q15.** Write a customer experience improvement brief for the airline: 3 headline findings, your prioritised list of 3 service improvements (from Q12/Q13), the segment to focus on first, and why. Use the Pyramid Principle.

---

## Project 9 🟡 — Attrition Analysis: HR
**Dataset:** D7 — IBM HR Analytics | **Focus:** Statistical testing, attrition drivers, risk segmentation

### Business Context
The CHRO wants to understand what drives employee attrition. This analysis will inform the retention strategy and determine where to focus HR interventions.

### Setup
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import scipy.stats as stats

df = pd.read_csv("data/hr/WA_Fn-UseC_-HR-Employee-Attrition.csv")
df["Attrition_binary"] = (df["Attrition"] == "Yes").astype(int)
```

### Questions

**Q1.** What is the overall attrition rate? How many employees left vs stayed? Is this dataset imbalanced? What does a high voluntary attrition rate typically cost a company (frame it in terms of salary multiple)?

**Q2.** How does attrition rate vary by `department`? By `job_role`? Create two bar charts. Which roles have the highest attrition? Is there a pattern in types of roles (individual contributor vs manager)?

**Q3.** Compare the age distribution of employees who left vs stayed. Use overlapping histograms or a box plot. Is there a significant age difference? Run a t-test.

**Q4.** Does `monthly_income` differ between employees who left vs stayed? Run a t-test. Report the mean income for each group and the p-value. Is income a significant predictor of attrition?

**Q5.** Does `job_satisfaction` differ between leavers and stayers? Run a t-test. What about `environment_satisfaction`? Which satisfaction dimension is a stronger predictor of attrition?

**Q6.** Does `years_at_company` differ between leavers and stayers? Are leavers more likely to be newer employees or more tenured? Plot the distribution of tenure for both groups.

**Q7.** Does `work_life_balance` affect attrition? Compute attrition rate by work-life balance score (1–4). Is the relationship monotonic (worse balance → higher attrition)?

**Q8.** Does `overtime` affect attrition? Compute attrition rate for employees who work overtime vs those who don't. Run a chi-square test. How large is the difference?

**Q9.** Does `distance_from_home` affect attrition? Bin into: <5km, 5–15km, 15–30km, >30km. Plot attrition rate by distance bucket.

**Q10.** Run a correlation analysis: compute the point-biserial correlation between `Attrition_binary` and each numeric column. What are the top 5 variables most correlated with attrition?

**Q11.** Build a risk segmentation: define "high flight risk" as employees who meet at least 3 of these criteria: job satisfaction ≤ 2, overtime = Yes, years_at_company ≤ 2, income below median for their role. What % of the workforce is high risk? What is their actual attrition rate vs the rest?

**Q12.** Is attrition significantly different between employees who have had recent promotions (`years_since_last_promotion = 0`) vs those who haven't been promoted in 3+ years? Run a chi-square test.

**Q13.** Is there an interaction effect between overtime and job satisfaction? Compute the attrition rate for each combination of (overtime: Yes/No) × (job satisfaction: Low/High, split at median). Does overtime matter more when satisfaction is low?

**Q14.** What is the "regrettable attrition" rate? Define regrettable attrition as employees with `performance_rating >= 3` who left. What % of total attrition is regrettable? What is their average income, tenure, and job level?

**Q15.** Write a retention strategy brief for the CHRO: top 3 drivers of attrition (with effect sizes), the profile of the highest-risk employee, 3 specific retention interventions with the metric each would move, and how to measure success in 6 months.

---

## Project 10 🟡 — A/B Testing: Fraud Intervention
**Dataset:** D4 — Financial Fraud | **Focus:** Experimental design, hypothesis testing, business decision

### Business Context
The fraud team ran a 4-week experiment: half of transactions went through a new fraud screening model (Treatment), half went through the old rules engine (Control). You need to evaluate whether the new model should be deployed.

### Setup
```python
import pandas as pd
import numpy as np
import scipy.stats as stats
from statsmodels.stats.proportion import proportion_effectsize, zt_ind_solve_power

df = pd.read_csv("data/fraud/transactions.csv")

# Simulate the A/B assignment (since we don't have one in the raw data)
# In a real test, this would come from your experiment tracking system
np.random.seed(42)
df["test_group"] = np.where(np.random.rand(len(df)) < 0.5, "treatment", "control")

# The new model (treatment) is slightly better at catching fraud
# but also flags more legitimate transactions
# Simulate treatment effect: treatment catches 15% more fraud but has 10% more false positives
```

### Questions

**Q1.** What is the fraud rate in the Control group? In the Treatment group? What is the absolute difference? What is the relative lift?

**Q2.** Run a chi-square test to determine if the difference in fraud detection rate is statistically significant. Report the chi-square statistic, p-value, and your interpretation in plain English.

**Q3.** Compute the 95% confidence interval for the difference in fraud rates. Does the CI include zero? What does this mean for the decision?

**Q4.** The new model also has a "false positive rate" — it flags legitimate transactions as fraud. Compute this rate for control vs treatment. Is the false positive rate significantly different between groups?

**Q5.** Define the business trade-off: a false negative (missed fraud) costs on average the transaction amount. A false positive (blocking a legitimate transaction) costs $5 in customer service + 15% probability of the customer churning (avg customer LTV = $200). Compute the expected cost per 1,000 transactions for each group.

**Q6.** Based on Q5, what is the net economic benefit of deploying the treatment model? Show the math.

**Q7.** Before the experiment, the team should have calculated a minimum detectable effect (MDE). What MDE would the experiment have been powered to detect, given: the sample sizes in each group, alpha = 0.05, and power = 0.80?

**Q8.** Is the test result practically significant, not just statistically significant? What is Cohen's h (effect size for proportions)? Interpret: small/medium/large.

**Q9.** Does the treatment effect vary by merchant category? Compute fraud detection rate improvement (treatment − control) for each category. Is the new model uniformly better, or better in some categories and worse in others?

**Q10.** Is there a time trend in the experiment? Plot fraud rate by week for control vs treatment. Does the treatment effect change over the 4-week period?

**Q11.** Check for selection bias: is the distribution of transaction amounts the same in control and treatment? Run a t-test on transaction amounts between groups. If they're different, what does that mean for your results?

**Q12.** Compute the "novelty effect" check: does the treatment perform better in weeks 1–2 vs weeks 3–4? If performance degrades over time, the model may only work for new transaction patterns.

**Q13.** What is the decision you'd make based on this experiment? Write a formal recommendation: deploy, don't deploy, or run longer. Include: the statistical evidence, the business case, the risks, and what you'd monitor post-deployment.

**Q14.** What would have happened if you had NOT run this as a proper A/B test, and instead just compared fraud rates before and after the new model was deployed? What confounding factors could have misled you?

**Q15.** Write the experiment results summary that you'd share with the fraud team and the VP of Risk: methodology (2 sentences), headline result (1 sentence), statistical confidence (1 sentence), business impact (2 sentences), recommendation (2 sentences), caveats (2 sentences). Total: < 1 page.

---

## Project 11 🔴 — Problem Framing + Executive Analysis: Olist
**Dataset:** D1 — Olist | **Focus:** Translating a vague brief, structured problem framing, executive-ready output

### Business Context
You receive this message from the VP of Marketplace:
> *"Hey, I've been hearing from the seller success team that our sellers are struggling. Can you look into it and tell me what you find? We're thinking about doing something to support them but I'm not sure what exactly. Would be great to have something by Friday."*

This project is about the full analytical process: framing → scoping → analysis → communication.

### Questions

**Q1.** Write the clarifying questions you'd send back before starting any analysis. Use the 5-question diagnostic from the tutorial. What are you trying to learn from each question?

**Q2.** Without waiting for answers (the VP is in meetings), use the Framing Document template to frame the most useful version of this question. Define: the decision it informs, the precise analytical question, the metrics and dimensions, the success criteria, and what's out of scope.

**Q3.** Build a seller performance table: for each `seller_id` with at least 10 orders, compute: total GMV, order count, average review score, late delivery rate, and number of distinct product categories.

**Q4.** Define "struggling" quantitatively. Try at least two definitions: (a) review score < 3.5, (b) late delivery rate > 15%. How many sellers meet each definition? Do the two definitions overlap?

**Q5.** Does "struggling" correlate with low GMV? Compare: average GMV for struggling sellers vs non-struggling sellers. Run a t-test. Is the difference significant?

**Q6.** What is the distribution of seller GMV? Is it Pareto-distributed? What % of total GMV comes from the bottom 25% of sellers? From the top 25%?

**Q7.** Build a seller issue tree: are struggling sellers concentrated in specific regions? Specific product categories? Specific time periods (recent vs established sellers)?

**Q8.** Among sellers with declining review scores (compare first 6 months of activity vs most recent 6 months), what do they have in common? Is it a specific category, region, or order volume level?

**Q9.** What is the relationship between seller tenure (months active) and performance? Do newer sellers struggle more? Plot average review score by seller tenure month.

**Q10.** Compute the ROI of a hypothetical "seller coaching program": if coaching could bring the bottom 25% of sellers to the median performance level, what would be the GMV uplift? What would be the review score improvement?

**Q11.** Using the Pyramid Principle, structure your finding into: governing thought, 3 supporting arguments, data bullets. Write this as if you were presenting to the VP verbally in 3 minutes.

**Q12.** Create a one-page executive brief (in markdown) using the template from the tutorial: finding, why it matters, evidence, recommendation, confidence level.

**Q13.** The VP asks: "Should we coach all struggling sellers or focus on a subset?" Which subset would you recommend targeting first, and why? Define the selection criteria precisely.

**Q14.** Design the success metrics for the seller coaching program: what would you measure at 30, 60, and 90 days? What threshold would indicate the program is working? What would indicate it should be changed?

**Q15.** The VP responds: "This is great, but my colleague in Finance is asking whether we're even sure seller quality drives customer satisfaction, or if they just think that." Write the analysis that addresses this challenge. Is there a causal claim here? What evidence supports or undermines it? What would you need to prove causality?

---

## Project 12 🔴 — Root Cause Analysis: Supply Chain Decline
**Dataset:** D5 — DataCo Supply Chain | **Focus:** Structured RCA, 5 Whys in data, issue tree decomposition

### Business Context
The VP of Logistics sends you this note:
> *"Our delivery performance scores in Q4 have been significantly worse than Q3. I need to know exactly why before our board review next week."*

### Questions

**Q1.** Quantify the symptom: compute on-time delivery rate by quarter. Is Q4 actually worse than Q3? By how much? When exactly (which week/month) did the decline begin?

**Q2.** Draw the issue tree for this problem (write it in text/markdown structure). Branch 1: carrier/logistics issue. Branch 2: product/warehouse issue. Branch 3: external/demand issue. Is this MECE? Add sub-branches.

**Q3.** Test Branch 1 (shipping mode): does the on-time rate differ by shipping mode in Q4 vs Q3? Did one shipping mode get significantly worse while others stayed stable?

**Q4.** Test Branch 2 (product type): does on-time rate differ by product category in Q4 vs Q3? Did a specific category deteriorate?

**Q5.** Test by geography: does on-time rate differ by destination region in Q4 vs Q3? Is the decline concentrated in specific regions?

**Q6.** Is the decline in on-time rate driven by longer cycle times, or by worse estimation (delivery promise changing)? Compare: actual delivery days Q4 vs Q3. And: scheduled delivery days Q4 vs Q3.

**Q7.** Run the 5 Whys on the most likely branch from Q3–Q5. Document each "why" with the data evidence. Where does the chain end?

**Q8.** Is there a volume/capacity explanation? Did order volume increase in Q4? If capacity was stressed (more orders per shipping mode or region), does that correlate with the on-time decline?

**Q9.** Look for a "change point" — the specific week when performance deteriorated. Plot weekly on-time rate. Does it look like a sudden drop (pointing to a specific event) or a gradual decline (pointing to a systemic issue)?

**Q10.** Cross-validate your finding two ways: once by filtering for the specific segment (mode/category/region) you identified, and once by checking whether the on-time rate outside that segment stayed stable.

**Q11.** Quantify the impact: how many orders were late in Q4 that would have been on-time under Q3 performance levels? What is the GMV at risk from SLA penalties or customer churn?

**Q12.** Using the SCR framework (Situation → Complication → Resolution), write the narrative for the root cause you've identified. Keep it to 3 paragraphs.

**Q13.** What additional data would help you either confirm or rule out your root cause hypothesis? List 3 specific data sources or questions that would strengthen the analysis.

**Q14.** Design a monitoring dashboard for delivery performance: what 5 metrics would you track? At what threshold would you trigger an alert? How frequently would the dashboard refresh?

**Q15.** Write the board-ready root cause summary: 1 sentence on the symptom, 1 sentence on the root cause, 1 sentence on the scope (which segment is affected), 1 sentence on the recommended fix, 1 sentence on expected timeline to recover. Maximum 5 sentences total.

---

## Project 13 🔴 — Regression and Storytelling: Healthcare Billing
**Dataset:** D3 — Healthcare | **Focus:** What drives billing amounts, regression analysis, presenting to a CFO

### Business Context
The CFO wants to understand the drivers of billing amount variation across patients. She needs this for budget planning and contract negotiation with insurance providers.

### Questions

**Q1.** What is the range of billing amounts? Compute mean, median, P10, P90, standard deviation. Is the distribution normal? Does log-transform improve normality? (Use a Q-Q plot or skewness test.)

**Q2.** Build a simple univariate analysis: for each categorical variable (admission type, medical condition, insurance provider, blood type), compute mean billing amount and run an ANOVA. Which variable shows the strongest billing variation?

**Q3.** Does LOS predict billing amount? Compute the Pearson correlation. Plot a scatter with trend line. Is the relationship linear, or is there a threshold effect (e.g., billing jumps after 7+ days)?

**Q4.** Build a multiple linear regression model predicting `billing_amount` from: LOS, age, admission type (encoded), medical condition (encoded). Report: R², coefficients, and p-values.

**Q5.** Interpret the regression coefficients in plain language (not statistical jargon): "Each additional day of stay is associated with an increase of $X in billing, holding other factors constant."

**Q6.** Which variable has the largest absolute effect on billing? Compute standardised coefficients (scale inputs to mean=0, std=1 before fitting) to make coefficients comparable.

**Q7.** Check regression assumptions: plot residuals vs fitted values (should show no pattern). Plot a Q-Q plot of residuals. Are there obvious violations?

**Q8.** Are there high-leverage outlier patients (very high billing relative to their predicted value)? Identify the top 10 residuals. What do these patients have in common?

**Q9.** Does the billing model perform differently across insurance providers? Fit the regression separately for each provider. Do the coefficients differ? What does that imply for contract negotiations?

**Q10.** Build a simple billing risk score: score each patient 1–5 based on predicted billing. Validate: do patients scoring 5 actually have the highest billing? What is the average billing per risk tier?

**Q11.** Translate the regression findings into a business narrative: "Patients admitted as emergencies with [X condition] who stay [Y] days generate $Z more in billing than the average patient."

**Q12.** Using the Pyramid Principle: governing thought → 3 arguments → data. Write the structure for presenting the billing driver findings to the CFO.

**Q13.** The CFO asks: "Can we use this to predict next year's billing budget?" Assess the model for forecasting: what are the key assumptions? What would need to hold for the prediction to be reliable? What is the model's uncertainty range?

**Q14.** Build a one-page CFO brief: what drives billing variation, the 3 biggest controllable factors, a billing estimate for a hypothetical "typical" patient, and confidence interval.

**Q15.** What analysis would you NOT do with this data, and why? (Think about: patient privacy, HIPAA implications, ethical use of billing prediction, limitations of a synthetic dataset.) What would you tell the CFO about the limits of this analysis?

---

## Project 14 🔴 — Cross-Domain Analysis: Supply Chain Impact on Satisfaction
**Dataset:** D1 (Olist) + D5 (Supply Chain) | **Focus:** Cross-dataset analysis, causal reasoning, joint insights

### Business Context
The Olist leadership team has a hypothesis: delivery performance drives customer satisfaction. They want a rigorous analysis that either confirms this or challenges it.

### Questions

**Q1.** State the analytical hypothesis formally: "On-time delivery is positively associated with review score." What would you need to see to confirm or refute this?

**Q2.** Build the Olist delivery performance table: for each delivered order, compute: actual delivery days, promised delivery days, is_late flag, days early/late (negative = early, positive = late), and review score.

**Q3.** Compute the correlation between `is_late` and `review_score`. Is it statistically significant? Is it practically significant (effect size)?

**Q4.** Compute correlation between `delivery_days` (actual) and `review_score`. Is actual delivery speed a stronger predictor than lateness per se?

**Q5.** Is there a threshold effect? Compute average review score for each delivery day bucket (1–5, 6–10, 11–15, ...). Does satisfaction drop gradually or is there a cliff edge at a certain number of days?

**Q6.** Do the supply chain principles from D5 (OTIF rate by shipping mode, category) generalise to the Olist context? Compare the pattern of late deliveries by product category in Olist to DataCo's late delivery pattern by category.

**Q7.** Is the delivery-satisfaction relationship the same across product categories? Compute the correlation between delivery days and review score separately for each category. Does it vary? What does that mean?

**Q8.** Is the delivery-satisfaction relationship the same across customer regions? Some customers may be more tolerant of delays (e.g., remote areas where long delivery is expected). Compute correlation by customer state.

**Q9.** Build a "delivery experience" composite score: weight recency of delivery + days early/late + is_late. Correlate this score with review_score. Is a composite score more predictive than individual metrics?

**Q10.** Causal challenge: delivery may affect reviews, but product quality also affects reviews. How would you control for product quality? Compute review score residual after removing category-level average. Does the delivery-satisfaction relationship hold in residuals?

**Q11.** Simpson's Paradox check: does the aggregate delivery-satisfaction relationship hold within each product category, or does it reverse for any category?

**Q12.** Quantify the business impact: if Olist improved its on-time rate by 5 percentage points, by how much would average review score increase? Show the calculation and its assumptions explicitly.

**Q13.** Challenge the causal direction: could low review scores (unhappy customers) cause late delivery complaints (reporting bias in delivery dates)? What evidence would help distinguish causation direction?

**Q14.** What is the strongest possible case for the hypothesis "delivery drives satisfaction"? What is the strongest possible case against it? Present both sides fairly.

**Q15.** Write a final brief: does delivery performance drive customer satisfaction? State your conclusion, confidence level, key evidence, the main alternative explanation, and what additional analysis would increase confidence. This is a memo to the CEO.

---

## Project 15 🔴 — Full Analyst Capstone
**Datasets:** All D1–D7 | **Focus:** Ambiguous brief → framing → analysis → recommendation document

### Business Context
You receive this brief from a consulting client:

> *"We're considering acquiring a data analytics capability — either building it internally or acquiring a company with existing data assets. We want a report that helps us understand: what kinds of data-driven insights are possible across different industries, what the key metrics are, and what analytical patterns a strong analytics function would employ. We have data samples from several industries. Please prepare a comprehensive analytical report."*

This is a capstone that tests everything: framing, EDA, metrics, statistics, storytelling, and BI thinking.

### Questions

**Q1.** Reframe this brief. The original brief is too vague to answer. Write the 3 most useful, specific analytical questions that would address the client's actual need (building/acquiring analytics capability).

**Q2.** Run a comparative EDA across all 7 datasets. For each dataset, report: row count, time coverage (if applicable), key metric column, and the single most interesting finding from a 5-minute EDA.

**Q3.** For each of the 5 industry datasets (E-Commerce/D1, Healthcare/D3, Fraud/D4, Supply Chain/D5, HR/D7): identify the North Star metric and compute it. Present a one-line result for each.

**Q4.** Across industries, which industry shows the clearest evidence of a "performance problem" in its data? Justify with a specific metric and its deviation from benchmark.

**Q5.** Build a cross-industry KPI dashboard (in code): for each industry, compute 3 KPIs and present them in a formatted summary table. This simulates what an analytics function would track.

**Q6.** Apply the 5 Whys to the most significant "problem" you found in Q4. What is the root cause, and what data would you need to confirm it?

**Q7.** Demonstrate A/B test analysis on the fraud dataset (from Project 10 if done, otherwise simulate). Explain in non-technical terms what A/B testing enables and why it's critical for business decision-making.

**Q8.** Build a customer segmentation on the Olist dataset using RFM. Present the segment size and revenue contribution. This demonstrates a "productionisable" analytical pattern.

**Q9.** Run a regression on the healthcare dataset (from Project 13 if done). Present the 3 key drivers of billing amount in plain language. This demonstrates statistical analysis capability.

**Q10.** Build one interactive Plotly dashboard covering the Olist dataset: monthly revenue, top 10 categories, customer state distribution. Save as HTML. This is a deliverable demonstrating BI capability.

**Q11.** Write the "Analytics Maturity Assessment" section: rate each of the 5 industry datasets on a maturity scale (1–4): how rich is the data, how much analytical potential does it have, and what's missing that would make it more valuable?

**Q12.** Write the "Key Analytical Patterns" section of the report: describe 5 reusable analytical patterns you've applied across datasets (e.g., cohort analysis, RFM, regression, A/B testing, funnel analysis). For each: the pattern name, what business question it answers, and which dataset demonstrates it.

**Q13.** Write the "Recommendations" section: based on your analysis, what 3 capabilities would be most valuable for the client to build first? Rank them by: (a) speed to value, (b) breadth of applicability across industries, (c) technical complexity.

**Q14.** Write the Executive Summary (1 page): problem statement, methodology, 5 key findings, 3 recommendations. Use the Pyramid Principle structure.

**Q15.** Self-assessment: write a brief reflection on this capstone. What was the hardest question? What would you do differently? What analysis would you add if you had 2 more weeks? This develops the metacognitive skill of knowing what you don't know.

---

## Answer Key

---

### Project 1 — Key Answers

```python
# Q2: Why two different customer ID counts
customers = pd.read_csv("data/olist/olist_customers_dataset.csv")
orders    = pd.read_csv("data/olist/olist_orders_dataset.csv")

unique_customer_id     = orders["customer_id"].nunique()
unique_customer_unique = customers["customer_unique_id"].nunique()

print(f"customer_id (order-level):     {unique_customer_id:,}")
print(f"customer_unique_id (person):   {unique_customer_unique:,}")
print(f"\nRatio: {unique_customer_id/unique_customer_unique:.2f}x")
# Olist assigns a new customer_id per order for privacy
# customer_unique_id is the actual person
# Ratio > 1 means some people placed multiple orders (rare — ~3%)


# Q4: Payment distribution and mean/median insight
payments = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
pay_agg  = payments.groupby("order_id")["payment_value"].sum()

stats_report = pd.Series({
    "mean":   pay_agg.mean(),
    "median": pay_agg.median(),
    "P25":    pay_agg.quantile(0.25),
    "P75":    pay_agg.quantile(0.75),
    "P90":    pay_agg.quantile(0.90),
    "P99":    pay_agg.quantile(0.99),
    "skew":   pay_agg.skew(),
})
print(stats_report.round(2))
print(f"\nMean/Median ratio: {pay_agg.mean()/pay_agg.median():.2f}x")
# ~1.4x — moderate right skew
# Interpretation: the mean is pulled up by a few large B2B-style orders
# The median (~R$100) is the better measure of "typical" order value


# Q6: On-time delivery rate
orders["order_purchase_timestamp"]    = pd.to_datetime(orders["order_purchase_timestamp"])
orders["order_delivered_customer_date"] = pd.to_datetime(orders["order_delivered_customer_date"])
orders["order_estimated_delivery_date"] = pd.to_datetime(orders["order_estimated_delivery_date"])

delivered = orders[orders["order_status"] == "delivered"].copy()
delivered["is_on_time"] = (delivered["order_delivered_customer_date"]
                           <= delivered["order_estimated_delivery_date"])
otd_rate = delivered["is_on_time"].mean()
print(f"On-time delivery rate: {otd_rate:.1%}")
# Olist typical: ~92% — slightly above 90% benchmark
# Note: only 'delivered' orders have delivery dates


# Q13: Data quality checks
print("Data quality checks:")
# Future delivery dates
future = orders[orders["order_delivered_customer_date"]
                > pd.Timestamp.now()]
print(f"  Orders with future delivery date: {len(future)}")

# Delivered before ordered
bad_dates = delivered[delivered["order_delivered_customer_date"]
                      < delivered["order_purchase_timestamp"]]
print(f"  Delivered before ordered: {len(bad_dates)}")

# Duplicate order_ids
dup_orders = orders["order_id"].duplicated().sum()
print(f"  Duplicate order_ids: {dup_orders}")

# Negative payments
neg_payments = payments[payments["payment_value"] < 0]
print(f"  Negative payment values: {len(neg_payments)}")
```

---

### Project 6 — Key Answers

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

orders    = pd.read_csv("data/olist/olist_orders_dataset.csv",
                        parse_dates=["order_purchase_timestamp"])
customers = pd.read_csv("data/olist/olist_customers_dataset.csv")

delivered = orders[orders["order_status"] == "delivered"].copy()
delivered["order_month"] = delivered["order_purchase_timestamp"].dt.to_period("M")
delivered = delivered.merge(
    customers[["customer_id","customer_unique_id"]], on="customer_id", how="left"
)

# Q1: Cohort assignment
first_order = (delivered.groupby("customer_unique_id")["order_month"]
               .min().reset_index(name="cohort"))
print("Cohort sizes (first 10):")
print(first_order["cohort"].value_counts().sort_index().head(10).to_string())

# Q3-Q4: Retention matrix and heatmap
delivered = delivered.merge(first_order, on="customer_unique_id", how="left")
delivered["months_since_acq"] = (delivered["order_month"] - delivered["cohort"]).apply(lambda x: x.n)

cohort_data = (delivered.groupby(["cohort","months_since_acq"])
               ["customer_unique_id"].nunique().reset_index(name="customers"))

cohort_pivot = cohort_data.pivot_table(
    index="cohort", columns="months_since_acq", values="customers"
)
cohort_sizes   = cohort_pivot[0]
retention_table = cohort_pivot.divide(cohort_sizes, axis=0)

fig, ax = plt.subplots(figsize=(16, 10))
sns.heatmap(
    retention_table.iloc[:18, :12],
    annot=True, fmt=".0%", cmap="Blues",
    vmin=0, vmax=0.15, ax=ax, linewidths=0.5,
)
ax.set_title("Olist Customer Retention by Cohort\n"
             "(% of cohort making a purchase in each subsequent month)",
             fontweight="bold")
ax.set_xlabel("Months Since First Purchase")
ax.set_ylabel("Acquisition Cohort")
plt.tight_layout()
plt.savefig("outputs/cohort_retention.png", dpi=150, bbox_inches="tight")

# Q5: Average retention by month
print("\nAverage retention by months since acquisition:")
for m in [1, 3, 6, 12]:
    if m in retention_table.columns:
        avg = retention_table[m].dropna().mean()
        print(f"  Month {m:>2}: {avg:.1%}")
# Typical Olist values:
# Month 1: ~3-4% (very low repeat rate — single-purchase marketplace)
# Month 3: ~2%
# Month 6: ~1.5%
# This is characteristic of a pure marketplace, not a subscription business
```

---

### Project 9 — Key Answers

```python
import pandas as pd
import numpy as np
import scipy.stats as stats

df = pd.read_csv("data/hr/WA_Fn-UseC_-HR-Employee-Attrition.csv")
df["Attrition_binary"] = (df["Attrition"] == "Yes").astype(int)

# Q1: Overall attrition rate
attrition_rate = df["Attrition_binary"].mean()
print(f"Overall attrition rate: {attrition_rate:.1%}")
print(f"Employees left: {df['Attrition_binary'].sum()}")
print(f"Employees stayed: {(df['Attrition_binary'] == 0).sum()}")
print(f"\nCost framing:")
avg_salary = df["MonthlyIncome"].mean() * 12
print(f"  Average annual salary: ${avg_salary:,.0f}")
print(f"  Replacement cost (1.5× salary): ${avg_salary * 1.5:,.0f}")
total_leavers = df["Attrition_binary"].sum()
print(f"  Total replacement cost this cohort: ${avg_salary * 1.5 * total_leavers:,.0f}")


# Q4: Income difference — t-test
left   = df[df["Attrition"] == "Yes"]["MonthlyIncome"]
stayed = df[df["Attrition"] == "No"]["MonthlyIncome"]
t_stat, p_val = stats.ttest_ind(left, stayed)

print(f"\nMonthly Income — attrition vs retention:")
print(f"  Left:   mean=${left.mean():,.0f}  median=${left.median():,.0f}")
print(f"  Stayed: mean=${stayed.mean():,.0f}  median=${stayed.median():,.0f}")
print(f"  t-statistic: {t_stat:.3f}")
print(f"  p-value: {p_val:.6f}")
print(f"  Significant: {'Yes ✅' if p_val < 0.05 else 'No ❌'}")
print(f"  Interpretation: Employees who left earned "
      f"${stayed.mean() - left.mean():,.0f}/month less on average.")


# Q8: Overtime effect
overtime_table = pd.crosstab(df["OverTime"], df["Attrition"])
chi2, p_val, dof, expected = stats.chi2_contingency(overtime_table)
overtime_rates = df.groupby("OverTime")["Attrition_binary"].mean()

print(f"\nOvertime vs Attrition:")
print(f"  Attrition rate — Yes overtime: {overtime_rates.get('Yes', 0):.1%}")
print(f"  Attrition rate — No overtime:  {overtime_rates.get('No', 0):.1%}")
print(f"  Chi-square: {chi2:.2f}, p={p_val:.6f}")
print(f"  Employees who work overtime are "
      f"{overtime_rates.get('Yes',0)/overtime_rates.get('No',1):.1f}x more likely to leave")


# Q10: Point-biserial correlations
numeric_cols = df.select_dtypes(include=np.number).columns
correlations = {}
for col in numeric_cols:
    if col == "Attrition_binary":
        continue
    r, p = stats.pointbiserialr(df["Attrition_binary"], df[col])
    correlations[col] = {"r": r, "p": p, "abs_r": abs(r)}

corr_df = pd.DataFrame(correlations).T.sort_values("abs_r", ascending=False)
print(f"\nTop 10 predictors of attrition (point-biserial r):")
print(corr_df.head(10)[["r","p"]].round(4).to_string())
# Typical top predictors: OverTime (encoded), MonthlyIncome (negative), Age (negative)
# StockOptionLevel (negative), JobLevel (negative), TotalWorkingYears (negative)
```

---

### Project 11 — Key Answers

```python
# Q3: Seller performance table
import pandas as pd
import scipy.stats as stats

orders   = pd.read_csv("data/olist/olist_orders_dataset.csv",
                       parse_dates=["order_purchase_timestamp",
                                    "order_delivered_customer_date",
                                    "order_estimated_delivery_date"])
items    = pd.read_csv("data/olist/olist_order_items_dataset.csv")
reviews  = pd.read_csv("data/olist/olist_order_reviews_dataset.csv")

delivered = orders[orders["order_status"] == "delivered"].copy()
delivered["is_late"] = (delivered["order_delivered_customer_date"]
                        > delivered["order_estimated_delivery_date"]).astype(int)

seller_reviews = (reviews
    .merge(items[["order_id","seller_id"]], on="order_id", how="left")
    .groupby("seller_id")["review_score"]
    .agg(avg_review="mean", review_count="count")
    .reset_index())

seller_perf = (items
    .merge(delivered[["order_id","is_late"]], on="order_id", how="inner")
    .groupby("seller_id")
    .agg(
        gmv       = ("price",    "sum"),
        orders    = ("order_id", "count"),
        late_rate = ("is_late",  "mean"),
    )
    .reset_index()
    .merge(seller_reviews, on="seller_id", how="left")
    .query("orders >= 10")
    .query("review_count >= 5"))

print(f"Sellers in analysis: {len(seller_perf):,}")
print(f"\nPerformance summary:")
print(seller_perf[["gmv","avg_review","late_rate"]].describe().round(3))

# Q4: Define "struggling"
struggling_review = seller_perf[seller_perf["avg_review"] < 3.5]
struggling_late   = seller_perf[seller_perf["late_rate"] > 0.15]
struggling_both   = seller_perf[(seller_perf["avg_review"] < 3.5)
                                 & (seller_perf["late_rate"] > 0.15)]

print(f"\n'Struggling' sellers:")
print(f"  By review < 3.5: {len(struggling_review):,} ({len(struggling_review)/len(seller_perf):.1%})")
print(f"  By late rate > 15%: {len(struggling_late):,} ({len(struggling_late)/len(seller_perf):.1%})")
print(f"  By both criteria: {len(struggling_both):,} ({len(struggling_both)/len(seller_perf):.1%})")

# Q5: Does struggling correlate with low GMV?
seller_perf["is_struggling"] = (seller_perf["avg_review"] < 3.5)
struggling_gmv    = seller_perf[seller_perf["is_struggling"]]["gmv"]
nonstruggling_gmv = seller_perf[~seller_perf["is_struggling"]]["gmv"]

t_stat, p_val = stats.ttest_ind(struggling_gmv, nonstruggling_gmv)
print(f"\nGMV: struggling vs non-struggling sellers:")
print(f"  Struggling avg GMV: R${struggling_gmv.mean():,.0f}")
print(f"  Non-struggling avg GMV: R${nonstruggling_gmv.mean():,.0f}")
print(f"  t-test p-value: {p_val:.4f}")
print(f"  {'Significant ✅' if p_val < 0.05 else 'Not significant ❌'}")
```

---

### Project 15 — Capstone Key Structure

```python
# Q3: North Star metrics across industries
import pandas as pd
import numpy as np

# E-Commerce (Olist)
orders = pd.read_csv("data/olist/olist_orders_dataset.csv",
                     parse_dates=["order_purchase_timestamp"])
payments = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
gmv = payments["payment_value"].sum()
delivered_ids = orders[orders["order_status"]=="delivered"]["order_id"]
nmv = payments[payments["order_id"].isin(delivered_ids)]["payment_value"].sum()
print(f"E-Commerce NSM (GMV): R${gmv:,.0f}")
print(f"  NMV (delivered): R${nmv:,.0f} ({nmv/gmv:.1%} of GMV)")

# Healthcare
hc = pd.read_csv("data/healthcare/healthcare_dataset.csv")
hc["los"] = (pd.to_datetime(hc["Discharge Date"]) -
             pd.to_datetime(hc["Date of Admission"])).dt.days
print(f"\nHealthcare NSM (avg LOS): {hc['los'].mean():.1f} days")
print(f"  Avg billing: ${hc['Billing Amount'].mean():,.0f}")

# HR
hr = pd.read_csv("data/hr/WA_Fn-UseC_-HR-Employee-Attrition.csv")
attrition_rate = (hr["Attrition"] == "Yes").mean()
print(f"\nHR NSM (voluntary attrition rate): {attrition_rate:.1%}")

# Fraud
fraud = pd.read_csv("data/fraud/transactions.csv")
fraud_rate = fraud["is_fraud"].mean() if "is_fraud" in fraud.columns else None
if fraud_rate:
    print(f"\nFraud NSM (fraud rate): {fraud_rate:.3%}")

# Supply Chain
sc = pd.read_csv("data/supply_chain/DataCo_Supply_Chain.csv", encoding="latin-1")
sc.columns = sc.columns.str.lower().str.strip().str.replace(" ","_")
late_col = [c for c in sc.columns if "late" in c.lower() and "risk" not in c.lower()]
if late_col:
    otif = 1 - sc[late_col[0]].mean()
    print(f"\nSupply Chain NSM (OTIF): {otif:.1%}")
```

---

*End of Data Analytics Projects*
*Chat #4 · June 2026*
