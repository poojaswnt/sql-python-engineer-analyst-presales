# Python Projects — 15 Practice Projects for Data Analysts

> **How to use this file**
> Same 7 datasets as `sql_projects.md`. Same difficulty ladder (🟢→🟡→🔴).
> Every project has 15 questions. Q1–Q8 build skills progressively; Q9–Q14 are complex multi-step; Q15 is a hard capstone that ties everything together.
> SQL equivalents are called out inline — build dual-language fluency deliberately.
> Setup code is provided at the top of each project. Answers are at the bottom — attempt independently first.
>
> **Chat #2 · June 2026 (v2 — Expanded)**

---

## Dataset Reference

| ID | Dataset | Domain | Kaggle URL |
|----|---------|--------|-----------|
| D1 | Olist Brazilian E-Commerce | E-Commerce | https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce |
| D2 | UK Online Retail II | Retail | https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci |
| D3 | Healthcare Dataset | Healthcare | https://www.kaggle.com/datasets/prasad22/healthcare-dataset |
| D4 | Financial Transactions + Fraud | Banking | https://www.kaggle.com/datasets/computingvictor/transactions-fraud-datasets |
| D5 | DataCo Supply Chain | Manufacturing | https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis |
| D6 | Airline Passenger Satisfaction | Travel | https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction |
| D7 | IBM HR Analytics | Hi-Tech/HR | https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset |

---

## Standard Setup (run once at the top of every notebook)

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import sqlite3
from pathlib import Path
from scipy import stats

sns.set_theme(style="whitegrid")
pd.set_option("display.max_columns", 50)
pd.set_option("display.float_format", "{:.2f}".format)

DATA_DIR = Path("data")       # adjust to your local path
OUT_DIR  = Path("outputs")
OUT_DIR.mkdir(exist_ok=True)
```

---

## Project 1 🟢 — First Full EDA: Olist Orders
**Dataset:** D1 — Olist | **Files:** `olist_orders_dataset.csv`, `olist_order_payments_dataset.csv`, `olist_customers_dataset.csv`, `olist_order_reviews_dataset.csv`

### Business Context
You just joined a Brazilian e-commerce analytics team. Your manager hands you the Olist dataset and asks for a complete first-pass report. No cleaning shortcuts — go through every step.

### Setup
```python
orders   = pd.read_csv(DATA_DIR / "olist_orders_dataset.csv")
payments = pd.read_csv(DATA_DIR / "olist_order_payments_dataset.csv")
customers= pd.read_csv(DATA_DIR / "olist_customers_dataset.csv")
reviews  = pd.read_csv(DATA_DIR / "olist_order_reviews_dataset.csv")
```

**Q1.** Print the shape, dtypes, and first 5 rows of `orders`. How many columns are date columns stored as strings? List them.
> *Expected: 8 columns, 6 are date strings. Shape ~99441 × 8.*

**Q2.** Parse all date columns as datetime using `pd.to_datetime(..., errors="coerce")`. After parsing, how many NaT values exist in each date column? Which nulls are *expected* (structural) vs unexpected?
> *Hint: Cross-check with `order_status`. Nulls in `order_delivered_customer_date` for non-delivered orders = expected.*

**Q3.** Null audit: build a table showing `column`, `null_count`, `null_pct`, and `sample_non_null_value` for every column with > 0 nulls. Sort by `null_pct` descending.

**Q4.** Duplicate check: how many exact duplicate rows? How many duplicate `order_id` values? If `unique(order_id)` ≠ `len(orders)`, investigate why.

**Q5.** How many orders are in each `order_status`? Show as a DataFrame with count and % of total. Plot as a horizontal bar chart sorted by count.
> *SQL equivalent: `SELECT order_status, COUNT(*), COUNT(*)*100.0/SUM(COUNT(*)) OVER() FROM orders GROUP BY order_status`*

**Q6.** Merge `payments` onto `orders` using `order_id`. Aggregate payments first: total `payment_value` per `order_id`. After merging, how many orders have no payment record? What `order_status` are they?

**Q7.** Distribution of order values: plot a histogram of `payment_value` with 50 bins. Add vertical lines for mean and median. Is the data right-skewed? Calculate skewness numerically. What does a skewness > 1 mean for how you should report "average order value"?

**Q8.** Add `order_month` = year-month string (format `"YYYY-MM"`) from `order_purchase_timestamp`. Group by `order_month`, count orders. Plot as a line chart. What growth trend do you see? Which month was the peak?

**Q9.** Merge `customers` on `customer_id`. Which 10 states (`customer_state`) have the highest order volume? Calculate for each: order count, total revenue, average order value. Plot top 10 by revenue as a horizontal bar chart.

**Q10.** Geographic pricing: do some states have significantly higher average order values than others? Plot a scatter: x = order count (volume), y = avg order value. Annotate the top 5 states by avg order value. Is high volume correlated with high value?

**Q11.** Merge `reviews` onto your DataFrame. Take the first review per `order_id` (`drop_duplicates("order_id", keep="first")`). What is the distribution of `review_score` (1–5)? Plot as a bar chart. Calculate the weighted average score.

**Q12.** Delivery speed calculation: add `delivery_days` = `order_delivered_customer_date` - `order_purchase_timestamp` in days. Drop rows with NaT in either column. Plot a histogram. Calculate: mean, median, P75, P90, P99. What does P90 tell you that mean doesn't?

**Q13.** Review score vs delivery speed: group orders into delivery bands: `≤5d`, `6–10d`, `11–20d`, `21–30d`, `>30d`. For each band, calculate avg review score, order count, and % of total. Plot avg review score per band as a bar chart. Write 2 sentences summarising the relationship.

**Q14.** Late delivery analysis: create `is_late` = 1 if `order_delivered_customer_date > order_estimated_delivery_date`, else 0. What is the overall late rate? Group by `order_month` — plot the late rate over time as a line chart. Is Olist getting better or worse at on-time delivery?

**Q15.** Full EDA dashboard: create a 2×3 subplot figure with:
- [0,0] Monthly order volume (line)
- [0,1] Order status distribution (horizontal bar)
- [0,2] Payment value distribution (histogram with mean/median lines)
- [1,0] Delivery time distribution (histogram)
- [1,1] Top 10 states by revenue (horizontal bar)
- [1,2] Review score vs delivery band (bar chart)

Add a `fig.suptitle("Olist E-Commerce — EDA Dashboard", fontsize=16)`. Save as `olist_eda_dashboard.png`.

---

## Project 2 🟢 — Data Cleaning: UK Online Retail
**Dataset:** D2 — UK Online Retail II | **File:** `online_retail_II.xlsx` or `.csv`

### Business Context
This is a real-world retail transaction export. It's messy. Your job is to clean it and document every decision.

### Setup
```python
# Note: file may be xlsx
try:
    df = pd.read_excel(DATA_DIR / "online_retail_II.xlsx", dtype={"Customer ID": str})
except:
    df = pd.read_csv(DATA_DIR / "online_retail_II.csv",  dtype={"Customer ID": str})

print(f"Loaded: {df.shape}")
print(df.dtypes)
```

**Q1.** Run a full structural inspection: shape, dtypes, null counts with %. Which column has the most nulls? What is the implication of that many null `Customer ID` values for customer-level analysis?

**Q2.** Convert `InvoiceDate` to datetime. Extract: `year`, `month`, `day`, `hour`, `dayofweek`, `day_name`. What range of dates does the data cover?

**Q3.** Add `line_total = Quantity * Price`. How many rows have negative `line_total`? What are negative values? (Returns? Adjustments?) What is the total value of negative rows?

**Q4.** Find rows where `Price = 0` or `Price < 0`. How many? Sample the `Description` values. Are these real products or system entries?

**Q5.** Find rows where `Description` contains any of: `'TEST'`, `'ADJUST'`, `'POSTAGE'`, `'MANUAL'`, `'DOT'`, `'BANK CHARGES'`. Use `str.contains` with `|` (OR) and `case=False`. How many rows? Create `is_system_entry` flag.

**Q6.** Define `is_return = Quantity < 0`. Separately define `is_system_entry` from Q5. Now define `is_valid`: a row is valid if `not is_return AND not is_system_entry AND Price > 0 AND Quantity > 0`. What % of original rows are valid?

**Q7.** Create `df_clean = df[df["is_valid"]].copy()`. Print a cleaning report:
```
Cleaning Report
---------------
Original rows:        ______
Returns removed:      ______
System entries removed: ______
Zero/negative price:  ______
Final clean rows:     ______
Rows removed (%):     ______
```

**Q8.** In `df_clean`, standardise `Description`: strip whitespace, convert to title case. How many unique descriptions before vs after standardisation? What caused duplicates?

**Q9.** Check for duplicate transactions in `df_clean`: same `Invoice` + `StockCode` + `Quantity` + `Price`. How many? Drop them with `keep="first"`.

**Q10.** Product performance table: group `df_clean` by `StockCode`. Calculate: `description` (most common), `total_qty`, `total_revenue`, `n_invoices`, `avg_price`, `price_cv` (coefficient of variation = std/mean of Price). Sort by `total_revenue` descending. Show top 20. Export to `products_clean.csv`.

**Q11.** Price volatility investigation: flag products with `price_cv > 0.3` (high price variation). How many? Sample 5 and look at their price history over time. Is this tiered pricing (different bundle sizes) or data quality issues?

**Q12.** Customer-level summary (known Customer IDs only): for each customer, calculate: `total_spend`, `n_invoices`, `avg_basket_size`, `first_purchase_date`, `last_purchase_date`, `active_days` (last - first). Sort by `total_spend` descending. How concentrated is revenue — what % of customers generate 80% of revenue?

**Q13.** Country analysis: `df_clean.groupby("Country")`. Calculate total revenue and order count. UK dominates — what % of revenue comes from UK? Among non-UK countries, which are top 5?

**Q14.** "Ghost revenue" estimate: for rows where `Customer ID` is null (but `is_valid = True`), what is the total `line_total`? As a % of total clean revenue, how significant is the unattributed revenue problem?

**Q15.** Write a `clean_uk_retail(path: str) -> pd.DataFrame` function that:
1. Loads the file (handles both `.xlsx` and `.csv`)
2. Converts `InvoiceDate` to datetime
3. Adds `line_total`, `is_return`, `is_system_entry`, `is_valid`
4. Filters to valid rows only
5. Standardises `Description`
6. Drops duplicates
7. Prints a cleaning report showing rows removed at each step
8. Returns the cleaned DataFrame

Test it: call the function, then verify the output has no nulls in `Price`, `Quantity`, or `line_total`.

---

## Project 3 🟢 — GroupBy and Aggregation: Healthcare Costs
**Dataset:** D3 — Healthcare | **File:** `healthcare_dataset.csv`

### Setup
```python
df = pd.read_csv(DATA_DIR / "healthcare_dataset.csv")
df["Date of Admission"] = pd.to_datetime(df["Date of Admission"])
df["Discharge Date"]    = pd.to_datetime(df["Discharge Date"])
df["length_of_stay"]    = (df["Discharge Date"] - df["Date of Admission"]).dt.days
print(df.head())
print(df.dtypes)
```

**Q1.** Full structural inspection. What does one row represent? What are the unique values of `Admission Type` and `Medical Condition`?

**Q2.** What is the overall average, median, and P90 `Billing Amount`? Is it right-skewed? Calculate skewness. Plot a histogram with mean and median lines.
> *SQL: `SELECT AVG(billing_amount), PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY billing_amount) FROM healthcare`*

**Q3.** Billing by medical condition: `groupby("Medical Condition")["Billing Amount"].agg(["mean","median","std","count"])`. Sort by mean descending. Plot as a horizontal bar chart. Which condition is most expensive?

**Q4.** Billing by admission type (Emergency / Elective / Urgent): same analysis. Is emergency care significantly more expensive than elective? Plot box plots side by side to show the full distribution, not just averages.

**Q5.** Gender analysis: value counts for `Gender`. Average billing by gender. Is the difference meaningful? Run a t-test (`scipy.stats.ttest_ind`) to check statistical significance.

**Q6.** Age distribution: plot a histogram of `Age`. Create `age_band` with `pd.cut`: `[0,25)`, `[25,40)`, `[40,60)`, `[60,100)`. Patient count and avg billing per band. Which age band has the most patients? The highest billing?

**Q7.** Insurance provider analysis: which provider covers the most patients? Calculate avg billing per provider. Is there evidence that providers negotiate different rates? Plot as a bar chart sorted by avg billing.

**Q8.** Top doctors by patient count (top 15). For each, calculate: patient count, avg billing, avg length of stay, % emergency admissions. Which doctor handles the most emergencies?

**Q9.** Pivot table: `Medical Condition` as rows, `Admission Type` as columns, values = average `Billing Amount`. Fill NaN with 0. Which (condition, type) combination is most expensive?
> *SQL: `SELECT medical_condition, admission_type, AVG(billing_amount) FROM h GROUP BY 1,2` then pivot*

**Q10.** Box plot of billing by medical condition: use `sns.boxplot`. Sort conditions by median billing. What does the width of the boxes tell you? Which condition has the most variable billing?

**Q11.** Outlier detection: use IQR method on `Billing Amount`. How many rows are below `Q1 - 1.5×IQR` or above `Q3 + 1.5×IQR`? What conditions and admission types dominate the high-end outliers?

**Q12.** Correlation analysis: scatter plots of Age vs Billing Amount, and Length of Stay vs Billing Amount. Add regression lines with `sns.regplot`. What do the correlation coefficients say? Run `scipy.stats.pearsonr` and print r and p-value for each.

**Q13.** Hospital comparison: group by `Hospital`. Calculate patient count, avg billing, avg length of stay, % normal test results. Plot: x = patient count, y = avg billing, bubble size = avg LOS, colour = % normal tests. Which hospitals have high billing AND low normal test rates?

**Q14.** Monthly admissions trend: extract month and year. Plot monthly admissions as a line chart. Is there seasonality? Use `df.groupby(df["Date of Admission"].dt.to_period("M")).size()`.

**Q15.** Write `billing_report(df: pd.DataFrame) -> dict` that returns:
```python
{
    "total_patients":    ...,
    "avg_billing":       ...,
    "median_billing":    ...,
    "most_expensive_condition": ...,
    "cheapest_condition":       ...,
    "most_common_admission":    ...,
}
```
And saves a 2×2 dashboard figure: [billing histogram, billing by condition bar, billing by admission box plot, age vs billing scatter]. Save as `healthcare_billing_report.png`.

---

## Project 4 🟢 — Fraud Analytics: Detection and Profiling
**Dataset:** D4 — Banking/Fraud | **Files:** transaction CSV + fraud labels

### Setup
```python
df = pd.read_csv(DATA_DIR / "transactions.csv")
# If fraud flag is in a separate file:
# fraud = pd.read_csv(DATA_DIR / "fraud_labels.csv")
# df = df.merge(fraud, on="transaction_id", how="left")
df["date"] = pd.to_datetime(df["date"])
df["hour"]      = df["date"].dt.hour
df["dayofweek"] = df["date"].dt.dayofweek
df["day_name"]  = df["date"].dt.day_name()
print(f"Fraud rate: {df['is_fraud'].mean():.2%}")
```

**Q1.** What is the fraud rate overall? How many fraud vs non-fraud transactions? What is the total financial loss from fraud (sum of fraudulent amounts)?

**Q2.** Amount distribution: plot overlapping histograms of fraud vs non-fraud amounts (use `alpha=0.5`). What is the median and mean amount for each group? Run a Mann-Whitney U test (`scipy.stats.mannwhitneyu`) to test if amounts are significantly different.

**Q3.** Fraud by `merchant_category`: calculate count, fraud count, fraud rate, avg fraud amount. Sort by fraud rate. Top 5 riskiest categories. Plot fraud rate as a bar chart.
> *SQL: `SELECT merchant_category, SUM(is_fraud)*1.0/COUNT(*) AS fraud_rate FROM t GROUP BY 1 ORDER BY 2 DESC`*

**Q4.** Fraud by hour of day: `df.groupby("hour")["is_fraud"].mean()`. Plot as a bar chart. Which 3 hours are riskiest? Is there a pattern (late night? early morning?)?

**Q5.** Fraud by day of week: same analysis. Are weekends riskier than weekdays? Plot and state your inference.

**Q6.** Transaction velocity: count transactions per `(customer_id, date)`. Flag `high_velocity = True` if ≥ 3 transactions in one day. What % of high-velocity days contain at least one fraud transaction? Compare to the baseline fraud rate.

**Q7.** Create `risk_flag = 1` if the transaction meets ANY of:
- `merchant_category` in top-5 fraud categories (from Q3)
- `hour` in top-3 risky hours (from Q4)
- `amount > 500`
What % of transactions are flagged? Of those flagged, what % are actually fraud? (This is your flag precision.)

**Q8.** False positives: of transactions flagged in Q7 that are NOT fraud — what is the most common merchant_category? Knowing this, how would you refine the rule?

**Q9.** Customer-level fraud profile: group by `customer_id`. Calculate: total transactions, fraud count, fraud rate. How many customers have a fraud rate > 20%? What is the avg transaction count for these "high-risk" customers vs the rest?

**Q10.** First-transaction risk: add `transaction_number` = `groupby("customer_id").cumcount() + 1` (ordered by date within each customer). Plot fraud rate for transaction number 1, 2, 3, 4, 5, 6–10, 11+. Is the first transaction riskier?

**Q11.** Fraud heatmap: `pd.crosstab(df["hour"], df["dayofweek"], values=df["is_fraud"], aggfunc="mean")`. Plot with `sns.heatmap`. Columns: Mon–Sun. Which time × day combination is the hotspot?

**Q12.** Merchant anomaly detection: for each merchant, calculate avg and std of legitimate transaction amounts. Then for each transaction, `amount_zscore = (amount - merchant_avg) / merchant_std`. Flag `amount_anomaly = True` if `|zscore| > 3`. What % of anomalies are fraud?

**Q13.** Cumulative fraud by customer: sort each customer's transactions by date. Add `prior_fraud_count = groupby().shift(1).fillna(0).cumsum()`. Plot fraud rate by prior fraud count (0, 1, 2, 3+). Does having prior fraud predict future fraud?

**Q14.** Build a composite `fraud_score` (0–10 scale):
```python
df["fraud_score"] = (
    df["merchant_category"].isin(top5_categories).astype(int) * 3  +
    df["hour"].isin(top3_hours).astype(int)                  * 2  +
    (df["amount_zscore"].abs() > 2).astype(int)              * 2  +
    (df["prior_fraud_count"] > 0).astype(int)                * 3
)
```
For each score level (0–10), calculate: transaction count, fraud count, fraud rate. Plot fraud rate by score level as a bar chart. Does the score discriminate well?

**Q15.** Precision-recall curve: for thresholds 0 through 10 on your `fraud_score`, calculate:
- `precision = fraud_flagged / total_flagged`
- `recall    = fraud_flagged / total_fraud`
- `f1        = 2 * (precision * recall) / (precision + recall)`

Plot precision and recall as lines on the same chart against threshold. At what threshold is F1 maximised? Write 3 sentences interpreting what precision and recall trade-offs mean for a fraud team.

---

## Project 5 🟢 — Time Series Basics: Revenue Trends
**Datasets:** D1 + D2 | **Domain:** Cross-domain time-series

### Setup
```python
# Olist
orders   = pd.read_csv(DATA_DIR / "olist_orders_dataset.csv",
                        parse_dates=["order_purchase_timestamp"])
payments = pd.read_csv(DATA_DIR / "olist_order_payments_dataset.csv")
olist = orders.merge(
    payments.groupby("order_id")["payment_value"].sum().reset_index(),
    on="order_id", how="left"
).rename(columns={"payment_value": "revenue"})

# UK Retail (use cleaned version from Project 2, or load raw)
uk = pd.read_csv(DATA_DIR / "online_retail_II.csv", dtype={"Customer ID": str})
uk["InvoiceDate"] = pd.to_datetime(uk["InvoiceDate"], errors="coerce")
uk["line_total"]  = uk["Quantity"] * uk["Price"]
uk_clean = uk[(uk["Quantity"] > 0) & (uk["Price"] > 0)].copy()
```

**Q1.** For Olist: extract `order_month = order_purchase_timestamp.dt.to_period("M")`. Group by month, sum revenue. How many months of data are there? What is the monthly revenue range (min/max)?

**Q2.** Plot Olist monthly revenue as a line chart. Add a 3-month rolling average. Do you see growth? Are there anomalous months? Mark the highest month with an annotation.

**Q3.** Month-over-month % change for Olist: `pct_change(1)`. Plot as a bar chart (green for positive, red for negative). Which month had the biggest growth? The biggest drop?

**Q4.** Same analysis for UK Retail: monthly net revenue (`line_total`). Plot on a separate figure. What trend do you see in UK vs Olist?

**Q5.** Seasonality index for Olist: for each calendar month (1–12), calculate `avg_revenue_that_month / overall_monthly_avg`. Which calendar month over-indexes? Which under-indexes? Is there a holiday effect?

**Q6.** Same seasonality index for UK Retail. Does December spike more strongly in UK vs Olist? Plot both seasonality indices as a grouped bar chart (x = calendar month, two bars per month).

**Q7.** Anomaly detection for Olist: for each month, calculate rolling mean and rolling std (window = 3). Flag months where actual revenue is outside `mean ± 2 × std`. How many anomalies? Mark them on the line chart with red dots and annotations.

**Q8.** Year-over-year comparison for Olist: for each calendar month (1–12), compare revenue in the first year of data vs the second. What is the average YoY growth rate? Plot as grouped bars.

**Q9.** UK Retail: split revenue by country each month — UK vs non-UK. Create a stacked area chart. Has the non-UK share been growing?

**Q10.** Combined plot: put Olist and UK Retail monthly revenue on the same figure. Normalise both to index = 100 at their first month (so you can compare growth rate, not absolute levels). Which grew faster?

**Q11.** Rolling volatility for Olist: calculate 3-month rolling standard deviation of monthly revenue. Is the business getting more or less predictable over time? Plot rolling std over time.

**Q12.** Moving average crossover for Olist: plot both a 2-month and a 6-month rolling average. Highlight months where the 2M MA crosses above the 6M MA (bullish signal) and below (bearish signal). How many crossovers?

**Q13.** New vs returning customers (Olist): for each month, classify orders as "new customer" (first-ever order) vs "returning customer". Plot as a stacked bar chart. Is the repeat customer share growing?

**Q14.** Naive forecast: predict next month's Olist revenue = average of last 3 months. Calculate absolute % error vs the actual last month. What would a 5% / 10% / 20% error look like in absolute R$ terms? Is your forecast better than "last month's value" (random walk)?

**Q15.** Write `time_series_report(df, date_col, value_col, title="")` that:
1. Resamples to monthly
2. Calculates MoM % change, 3M rolling average, seasonality index
3. Flags anomalies (outside 2 std of 3M rolling mean)
4. Plots a 2-row figure: top = line + rolling avg + anomaly dots, bottom = MoM % bar chart
5. Returns a summary DataFrame with all calculated columns

Apply to both Olist and UK Retail.

---

## Project 6 🟡 — Advanced GroupBy: Supply Chain Performance
**Dataset:** D5 — DataCo Supply Chain | **File:** `DataCoSupplyChainDataset.csv`

### Setup
```python
df = pd.read_csv(DATA_DIR / "DataCoSupplyChainDataset.csv", encoding="latin-1")
df.columns = df.columns.str.lower().str.strip().str.replace(" ", "_")

# Parse dates
df["order_date_(dateorders)"] = pd.to_datetime(df["order_date_(dateorders)"], errors="coerce")
df.rename(columns={"order_date_(dateorders)": "order_date"}, inplace=True)

df["profit_margin"] = df["order_profit_per_order"] / df["sales"].replace(0, np.nan)
print(df.shape)
print(df["late_delivery_risk"].value_counts())
```

**Q1.** What is the overall late delivery rate? What is the average delivery delay in days (actual - scheduled)? Plot a histogram of delay days.

**Q2.** Late delivery rate by `shipping_mode`. Which mode is worst? Which is best? Calculate both rate AND average delay. Plot as a side-by-side bar chart (rate and delay as two bars per mode).

**Q3.** Late delivery rate by `category_name` (top 15). Plot as a horizontal bar chart sorted by late rate. Which product category has the worst logistics performance?

**Q4.** Profit analysis by `department_name`: total sales, total profit, profit margin %, order count, avg discount. Sort by profit margin. Which departments are most and least profitable?

**Q5.** Discount analysis: use `pd.cut` to create discount bands: `None (0%)`, `Low (0-10%)`, `Medium (10-20%)`, `High (20%+)`. Late rate and avg profit per band. Do higher discounts correlate with higher late delivery rates?

**Q6.** `Department × Shipping Mode` profit heatmap: `pd.pivot_table(aggfunc="mean", values="order_profit_per_order")`. Plot with `sns.heatmap`. Which (department, mode) combination generates the highest average profit per order?

**Q7.** Monthly profit trend: group by month, sum `order_profit_per_order`. Plot as a line. Use `.shift(1)` to add a `mom_change` column. Which month had the biggest profit decline?

**Q8.** Loss leaders: products with `avg_profit < 0` AND `order_count > 100`. List: product name, category, avg profit per order, total orders, total profit drain. These products cost money at scale.

**Q9.** Customer profitability: group by `customer_id`. Calculate total profit, total orders, avg discount. Find the top decile (top 10%) of customers by profit contribution. What share of total profit do they represent?

**Q10.** Product volatility: for each product, calculate monthly revenue. Compute coefficient of variation (std/mean) across months. Which 10 products are most volatile? These are hard to forecast.

**Q11.** Regional analysis: group by `order_region`. Calculate total revenue, total profit, margin %, late rate. Which region has the best margins? Which has the worst logistics?

**Q12.** Shipping mode economics: for each mode, calculate: total revenue, total profit, profit margin %, on-time rate, avg days late. Create a 2×2 subplot: revenue by mode, margin by mode, late rate by mode, scatter (margin vs late rate per mode with mode labels).

**Q13.** Declining departments: for each department, compute monthly profit. Use `.diff()` to find month-over-month changes. Count consecutive negative months for each department. Which department has had the longest losing streak?

**Q14.** Composite performance score per department:
```python
for metric in ["profit_margin", "on_time_rate", "revenue_growth"]:
    df_dept[metric + "_score"] = pd.qcut(df_dept[metric].rank(method="first"),
                                          5, labels=[1,2,3,4,5])
df_dept["composite_score"] = (df_dept[["profit_margin_score",
                                         "on_time_rate_score",
                                         "revenue_growth_score"]]
                                .astype(float).mean(axis=1).round(1))
```
Rank departments. Which is overall best? Overall worst?

**Q15.** Build a QBR (Quarterly Business Review) table: one row per `(department, quarter)`. Columns: `revenue`, `profit`, `margin_pct`, `orders`, `late_rate`, `best_category` (highest revenue category that quarter), `qoq_revenue_change`. Add `status`: 🟢 if margin > 15% AND late_rate < 20%, 🟡 if either threshold missed, 🔴 if both missed. Export to `qbr_table.csv`.

---

## Project 7 🟡 — Multi-Table Joins: Olist Full Dataset
**Dataset:** D1 — All 8 Olist tables

### Setup
```python
tables = {
    "orders":      "olist_orders_dataset.csv",
    "customers":   "olist_customers_dataset.csv",
    "order_items": "olist_order_items_dataset.csv",
    "products":    "olist_products_dataset.csv",
    "sellers":     "olist_sellers_dataset.csv",
    "payments":    "olist_order_payments_dataset.csv",
    "reviews":     "olist_order_reviews_dataset.csv",
    "category_translation": "product_category_name_translation.csv",
}

dfs = {}
for name, file in tables.items():
    dfs[name] = pd.read_csv(DATA_DIR / file)
    print(f"{name:30s} {dfs[name].shape}")
```

**Q1.** Print the shape and key columns of each table. Identify the join keys. Draw (in comments) the entity-relationship structure. What is the grain of each table?

**Q2.** Join `orders` ← `customers`. Verify: row count should not change. Add a `verify_merge` function:
```python
def verify_merge(before, after, step):
    print(f"{step}: {before} → {after} rows ({after-before:+d})")
```

**Q3.** Join `order_items` onto the result. This is 1:N (one order, many items) — row count WILL increase. How many rows now? What is the average items per order?

**Q4.** Join `products` on `product_id`. What % of order_items have a matching product? Which product_ids have no match (orphan items)?

**Q5.** Join `category_translation` to get English category names. How many products lack an English name? What are the most common Portuguese category names that have no translation?

**Q6.** Join `sellers` on `seller_id`. Add `seller_state`. Are orders being sold cross-state (customer_state ≠ seller_state)? What % are cross-state?

**Q7.** Aggregate `payments` to one row per order (sum `payment_value`). Join onto your base table. Verify: row count should not change after this join.

**Q8.** Join `reviews`: keep only the most recent review per order. Join and verify row count.

**Q9.** Final base table validation: print shape, null % per column, and confirm no duplicate `order_id` in the base table. Save to `olist_base_table.csv`.

**Q10.** Seller scorecard from base table: group by `seller_id`. Calculate: total orders, total revenue, avg price, avg review score, on-time rate, top category. Export `seller_scorecard.csv`.

**Q11.** Category performance: group by English category name. Calculate: total revenue, order count, avg review score, avg delivery days, seller count. Sort by revenue. Which category has the highest review score? The lowest?

**Q12.** Cross-sell analysis: for orders with 2+ items, find all pairs of categories purchased together. Use a self-merge on `order_id`:
```python
items = base[base["order_has_multiple_items"]].copy()
pairs = items.merge(items, on="order_id", suffixes=("_a", "_b"))
pairs = pairs[pairs["category_a"] < pairs["category_b"]]  # avoid duplicates
pairs.groupby(["category_a","category_b"]).size().nlargest(10)
```

**Q13.** Geographic revenue matrix: `pd.pivot_table` with `seller_state` as rows, `customer_state` as columns, values = revenue sum. Which seller-state → customer-state corridor has the highest revenue? Plot as a heatmap (top 8 seller states × top 8 customer states).

**Q14.** Anti-join: find sellers who have fulfilled orders but have NEVER received a 5-star review. Use `merge(..., how="left")` + indicator + null filter.

**Q15.** Write `build_olist_base_table(data_dir: str, validate: bool = True) -> pd.DataFrame` that performs all 8 merges, optionally prints a validation report at each step, and returns the final base table. The function should handle missing files gracefully (warn and skip).

---

## Project 8 🟡 — Window Functions: Airline Satisfaction
**Dataset:** D6 — Airline Passenger Satisfaction

### Setup
```python
df = pd.read_csv(DATA_DIR / "train.csv")   # or airline_passenger_satisfaction.csv
# Combine train + test if both available
try:
    test = pd.read_csv(DATA_DIR / "test.csv")
    df   = pd.concat([df, test], ignore_index=True)
    print(f"Combined: {df.shape}")
except FileNotFoundError:
    pass

# Service dimension columns
service_cols = [
    "Inflight wifi service", "Departure/Arrival time convenient",
    "Ease of Online booking", "Gate location", "Food and drink",
    "Online boarding", "Seat comfort", "Inflight entertainment",
    "On-board service", "Leg room service", "Baggage handling",
    "Checkin service", "Inflight service", "Cleanliness"
]
df["service_score"] = df[service_cols].mean(axis=1)
print(f"Overall satisfaction rate: {(df['satisfaction']=='satisfied').mean():.1%}")
```

**Q1.** What is the satisfaction rate overall? By `Class` (Business/Eco/Eco Plus)? By `Customer Type` (Loyal/Disloyal)? By `Type of Travel` (Business/Personal)? Show as a grouped bar chart.

**Q2.** Average score for each of the 14 service dimensions. Plot as a horizontal bar chart sorted from worst to best. Which 3 dimensions score lowest? These are the biggest pain points.

**Q3.** Distribution of `Flight Distance`: plot a histogram. Create `distance_band` with `pd.qcut(q=5)`. Satisfaction rate per band. Do long-haul passengers rate better or worse?

**Q4.** Delay impact: create `total_delay = Departure Delay in Minutes + Arrival Delay in Minutes`. Create delay bands: `None (0)`, `Short (1-30)`, `Medium (31-120)`, `Long (120+)`. Satisfaction rate per band. Plot as a bar chart.

**Q5.** Pain point analysis: for DISSATISFIED passengers only, calculate avg score per service dimension. Compare to avg score for SATISFIED passengers. Create a `gap` column = satisfied_avg - dissatisfied_avg. Plot gaps as a bar chart. The largest gaps = most differentiated dimensions.

**Q6.** `transform()` — add group means as new columns:
```python
df["class_avg_service"]   = df.groupby("Class")["service_score"].transform("mean")
df["travel_avg_service"]  = df.groupby("Type of Travel")["service_score"].transform("mean")
df["relative_score"]      = df["service_score"] - df["class_avg_service"]
```
Plot a histogram of `relative_score`. What does it mean when `relative_score > 0`?

**Q7.** Ranking: within each `Class`, rank passengers by `service_score` using `groupby().rank(method="dense", ascending=False)`. Add as `rank_within_class`. Show the top 5 and bottom 5 across all classes.

**Q8.** NPS proxy segmentation: `pd.qcut(service_score, q=4)` → label as `Promoter (Q4)`, `Passive (Q3)`, `Neutral (Q2)`, `Detractor (Q1)`. NPS = `%Promoter - %Detractor`. Calculate overall NPS. Calculate NPS by class.

**Q9.** Age × Class satisfaction heatmap: create `age_band` with `pd.cut`. `pd.crosstab(df["age_band"], df["Class"], values=df["satisfaction"].map({"satisfied":1,"neutral or dissatisfied":0}), aggfunc="mean")`. Plot as heatmap.

**Q10.** Delay tolerance by class: for each `Class`, calculate avg satisfaction at each delay band. Do Business class passengers tolerate delays better than Economy? Plot line chart with 3 lines (one per class).

**Q11.** Loyal vs disloyal — value of loyalty: for each `Customer Type`, show satisfaction rate, avg service_score, avg flight distance, % Business class. Are loyal customers higher-value in terms of class of travel?

**Q12.** Two-variable interaction: `pd.crosstab(df["Class"], df["Type of Travel"], values=df["service_score"], aggfunc="mean")`. Which (class, travel type) combination rates highest? Lowest?

**Q13.** Wifi vs overall satisfaction: group passengers by `Inflight wifi service` score (1–5). Calculate satisfaction rate and avg service_score at each wifi level. Plot both as lines on the same chart. Is wifi score the strongest single predictor of overall satisfaction?

**Q14.** Top driver analysis: for each service dimension, calculate the Pearson correlation with overall satisfaction (map satisfaction to 1/0 first). Sort by correlation. Create a horizontal bar chart of "satisfaction drivers". Compare to Q5 gap analysis — do both methods agree on the top drivers?

**Q15.** Write `analyse_satisfaction(df: pd.DataFrame) -> dict` that returns:
```python
{
    "overall_nps":           ...,
    "satisfaction_by_class": ...,   # Series
    "top3_pain_points":      ...,   # list of column names
    "top3_drivers":          ...,   # list of column names
    "segment_sizes":         ...,   # dict {Promoter: n, ...}
}
```
And saves a 2×3 dashboard figure.

---

## Project 9 🟡 — RFM Segmentation
**Dataset:** D1 — Olist

### Setup
```python
orders   = pd.read_csv(DATA_DIR / "olist_orders_dataset.csv",
                        parse_dates=["order_purchase_timestamp"])
payments = pd.read_csv(DATA_DIR / "olist_order_payments_dataset.csv")
customers= pd.read_csv(DATA_DIR / "olist_customers_dataset.csv")

# Build transaction table
pay_agg = payments.groupby("order_id")["payment_value"].sum().reset_index()
txn = (orders
       .merge(pay_agg, on="order_id", how="left")
       .merge(customers[["customer_id","customer_unique_id","customer_state"]], on="customer_id"))
txn = txn[txn["order_status"] == "delivered"].copy()
print(f"Delivered orders: {len(txn):,}")
```

**Q1.** What is the distribution of order count per customer? How many customers have only 1 order? What % is this? (This is the "one-and-done" problem — important for CRM strategy.)

**Q2.** Calculate RFM components per `customer_unique_id`:
```python
ref = txn["order_purchase_timestamp"].max()
rfm = txn.groupby("customer_unique_id").agg(
    last_order = ("order_purchase_timestamp", "max"),
    frequency  = ("order_id", "count"),
    monetary   = ("payment_value", "sum")
).reset_index()
rfm["recency_days"] = (ref - rfm["last_order"]).dt.days
```
Print summary stats for all three components. What is the median recency? The max frequency?

**Q3.** Score each component 1–5 with `pd.qcut`. Remember: for recency, **lower days = better** → invert with `labels=[5,4,3,2,1]`. Handle duplicate bin edges with `duplicates="drop"`.

**Q4.** Plot distribution of each R/F/M score (1–5) as bar charts. Is the scoring roughly uniform? If one score skews heavily to 1, investigate why (often caused by many customers with the same value — e.g. frequency=1 for everyone).

**Q5.** Segment using `np.select`:

| Segment | Condition |
|---------|-----------|
| Champions | r≥4 AND f≥4 |
| Loyal | r≥3 AND f≥3 |
| Promising | r≥4 AND f≤2 |
| At Risk | r≤2 AND f≥3 |
| Lost | r=1 AND f=1 |
| Others | everything else |

**Q6.** Segment summary table: count, avg recency, avg frequency, avg monetary per segment. Sort by avg monetary descending. Which segment is most valuable? Which is largest?

**Q7.** Merge `customer_state`. Which states have the highest % of Champions? The highest % of Lost customers?

**Q8.** Scatter plot: x = recency_days, y = monetary, colour = segment, size = frequency. Keep a random sample of 3000 rows for readability. Which segment cluster is clearly identifiable?

**Q9.** Review score by segment: merge in `order_reviews`. Calculate avg review score per segment. Do Champions leave better reviews? Plot as bar chart.

**Q10.** Customer Lifetime Value estimate: for customers with 2+ orders, calculate avg days between orders. Estimate `predicted_annual_orders = 365 / avg_gap_days`. Estimate `predicted_clv = predicted_annual_orders × avg_order_value`. Show top 10 customers by predicted CLV.

**Q11.** RFM heatmap: `pd.crosstab(rfm["r_score"], rfm["f_score"], values=rfm["monetary"], aggfunc="mean")`. Plot as heatmap. Where are high-monetary customers clustered? Does high F always mean high M?

**Q12.** Category affinity by segment: merge in `order_items` + `products`. For each segment, show the top 3 product categories by revenue share. Do Champions buy different categories than Lost customers?

**Q13.** Revenue recovery calculation: of "At Risk" customers, if you converted 10% of them to "Loyal" (i.e. they started spending at the Loyal segment's avg monetary), what is the additional annual revenue? Show your calculation clearly.

**Q14.** Payment type by segment: `pd.crosstab(rfm["segment"], payment_type, normalize="index")` as %. Do Champions prefer credit card installments vs boleto? What does this imply for targeted promotions?

**Q15.** Build `rfm_pipeline(orders_df, payments_df, customers_df) -> pd.DataFrame` that does all steps (Q2–Q6), plots segment distribution + avg monetary bar, prints segment summary, and returns the full RFM DataFrame.

---

## Project 10 🟡 — Time-Series Intelligence: UK Retail Deep Dive
**Dataset:** D2 — UK Online Retail II

### Setup
```python
df = pd.read_csv(DATA_DIR / "online_retail_II.csv", dtype={"Customer ID": str})
df["InvoiceDate"] = pd.to_datetime(df["InvoiceDate"], errors="coerce")
df["line_total"]  = df["Quantity"] * df["Price"]
df_clean = df[(df["Quantity"] > 0) & (df["Price"] > 0) & df["InvoiceDate"].notna()].copy()
df_clean.set_index("InvoiceDate", inplace=True)
print(f"Date range: {df_clean.index.min()} → {df_clean.index.max()}")
```

**Q1.** Daily net revenue (sum `line_total`). Resample to daily. Fill missing days with 0. Plot 30 days of daily data. How spiky is daily revenue? What is the coefficient of variation (std/mean)?

**Q2.** Monthly revenue: resample to `"ME"`. Plot line chart. Add 3-month rolling average. What overall trend do you see?

**Q3.** Day-of-week pattern: average revenue per day of week (0=Mon, 6=Sun). Plot as bar chart. Which day is highest? Is Sunday consistently low (is the shop closed)?

**Q4.** Hour-of-day pattern: average revenue per hour. Plot as bar chart. When does the shop peak? Are there clear opening/closing hours?

**Q5.** Anomaly detection: 7-day rolling mean and std. Flag days where actual revenue is outside `mean ± 2×std`. How many anomalies? Plot the full time series with anomaly dots in red.

**Q6.** Month-over-month % change. Most volatile month? Calculate the rolling 3-month standard deviation of MoM changes — is volatility increasing or decreasing?

**Q7.** Seasonality index by calendar month (1–12): `avg_monthly / overall_monthly_avg`. Which month over-indexes the most? Does December spike strongly (Christmas effect)?

**Q8.** Revenue by country, monthly: UK vs non-UK. Calculate non-UK share each month. Plot as stacked bar. Is international business growing or shrinking?

**Q9.** Return rate trend: reload original `df` (with returns). Monthly `|return_revenue| / gross_revenue`. Flag months with return rate > 15%. Plot with a 15% threshold line.

**Q10.** Product revenue stability: for top 100 products by total revenue, calculate monthly revenue and then coefficient of variation. Scatter: x = total revenue, y = CV. Label the 5 most volatile top products. Are the most stable products also the highest revenue?

**Q11.** 4-week vs 12-week MA crossover: plot both MAs. Annotate months where 4W crosses above 12W (possible trend reversal). How many crossovers are there? Does each crossover predict a sustained trend change?

**Q12.** Customer cohort revenue: define cohort by first purchase month. For each cohort, track total revenue generated each subsequent month. Limit to 6-month follow-up. Which cohort generated the most total 6-month revenue?

**Q13.** Big spender tracking: identify top 10 customers by total spend. For each, plot their monthly spend as a line chart. Are top spenders consistent or bursty (big orders once, then nothing)?

**Q14.** Forecast: last 3 months average as naive forecast for the following month. Calculate absolute % error. Compare: is the 3M average better or worse than using last month's value as the forecast?

**Q15.** Write `time_series_dashboard(df, date_col, value_col, title="")` → returns (summary_df, figure). The figure must include: actual + rolling avg line, MoM% bar chart, seasonality index bar, day-of-week bar, anomalies highlighted, return rate line. All in one 2×3 subplot grid.

---

## Project 11 🔴 — Cohort Retention Analysis
**Dataset:** D1 — Olist

### Setup
```python
orders   = pd.read_csv(DATA_DIR / "olist_orders_dataset.csv",
                        parse_dates=["order_purchase_timestamp"])
payments = pd.read_csv(DATA_DIR / "olist_order_payments_dataset.csv")
customers= pd.read_csv(DATA_DIR / "olist_customers_dataset.csv")

pay_agg = payments.groupby("order_id")["payment_value"].sum().reset_index()
base = (orders
        .merge(pay_agg, on="order_id", how="left")
        .merge(customers[["customer_id","customer_unique_id","customer_state"]], on="customer_id"))
base = base[base["order_status"] == "delivered"].copy()
base["order_month"] = base["order_purchase_timestamp"].dt.to_period("M")
```

**Q1.** For each `customer_unique_id`, find `cohort_month` = the month of their first-ever order.
```python
first_order = base.groupby("customer_unique_id")["order_month"].min().rename("cohort_month")
base = base.join(first_order, on="customer_unique_id")
```
How many distinct cohorts are there? How many customers are in the largest cohort?

**Q2.** Add `months_since_first`: how many months after the cohort month each order occurred.
```python
base["months_since_first"] = (base["order_month"] - base["cohort_month"]).apply(lambda x: x.n)
```

**Q3.** Build the cohort customer count table:
```python
cohort_customers = (base
    .groupby(["cohort_month","months_since_first"])["customer_unique_id"]
    .nunique()
    .reset_index(name="customers"))
cohort_table = cohort_customers.pivot(index="cohort_month", columns="months_since_first", values="customers")
```
Print the first 5 cohort months and first 6 columns (months 0–5).

**Q4.** Retention rates: divide every cell by the month-0 value (the cohort size).
```python
cohort_size   = cohort_table[0].rename("cohort_size")
retention     = cohort_table.divide(cohort_size, axis=0)
```
Format all values as percentages rounded to 1 decimal place.

**Q5.** Plot the cohort heatmap:
```python
fig, ax = plt.subplots(figsize=(14, 8))
sns.heatmap(retention.iloc[:, :12],  # first 12 months
            annot=True, fmt=".0%",
            cmap="YlOrRd_r",    # dark = high retention
            vmin=0, vmax=0.15,  # most retention rates will be <15%
            ax=ax)
ax.set_title("Customer Retention Cohort Analysis — Olist")
```
What is the typical month-1 retention rate? Month-3? Month-6?

**Q6.** Which cohort has the best month-1 retention? Which has the worst? Investigate: what was happening at Olist during those months (seasonal? promotional?)?

**Q7.** Average retention curve: across all cohorts, what is the average retention at month 1, 2, 3, ... 12?
```python
avg_retention = retention.mean(axis=0)
```
Plot as a line chart. At what month does the curve "flatten" (retention stabilises)? This is where your loyal core is revealed.

**Q8.** Revenue cohort: instead of customer counts, build the same pivot with `payment_value.sum()`. Build a revenue retention heatmap. Do revenue retention rates differ significantly from customer retention rates? What does a higher revenue retention imply?

**Q9.** Geographic cohorts: split by `customer_state`. For the top 5 states (by cohort size), compute their average month-1 and month-3 retention. Which states retain customers best? Plot as a grouped bar chart.

**Q10.** First-category cohorts: merge in `order_items` + `products`. Group by the category of the customer's FIRST purchase. Compute month-1 retention for each first-category. Which category as a first purchase leads to the best repeat behaviour?

**Q11.** "Near-churner" identification: customers who had their last order 3–6 months ago and have NOT returned since. Build this list with their cohort, first order value, and product category. This is a winback candidate list.

**Q12.** Revenue impact modelling: if month-1 retention improved by 2 percentage points across all cohorts, estimate the additional revenue generated. Show:
```
Additional customers retained per cohort: cohort_size × 0.02
Additional revenue per retained customer: avg_order_value_in_month_1
Total impact: sum across all cohorts
```

**Q13.** Seasonality in cohort sizes: plot cohort sizes (month-0 customers) over time. Are certain months larger cohorts? Does the company acquire more customers in certain months? Does cohort SIZE correlate with cohort RETENTION quality?

**Q14.** Visualise the "retention cliff": overlay all cohort retention curves on one chart (one line per cohort, x = months since first order, y = retention rate). Limit to cohorts with at least 6 months of follow-up. Do curves cluster, or is there high variability between cohorts?

**Q15.** Write `cohort_analysis(df, customer_col, date_col, value_col) -> tuple` that:
- Accepts any transaction DataFrame
- Returns `(customer_retention_table, revenue_retention_table, summary_stats_dict)`
- `summary_stats` includes: avg_month1_retention, avg_month3_retention, total_cohorts, best_cohort, worst_cohort
- Saves a 2×1 figure: left = customer retention heatmap, right = revenue retention heatmap

---

## Project 12 🔴 — Fraud Pattern Recognition: Advanced
**Dataset:** D4 — Banking/Fraud

### Setup
```python
df = pd.read_csv(DATA_DIR / "transactions.csv")
df["date"]      = pd.to_datetime(df["date"])
df              = df.sort_values(["customer_id", "date"]).reset_index(drop=True)
df["hour"]      = df["date"].dt.hour
df["dayofweek"] = df["date"].dt.dayofweek
df["day"]       = df["date"].dt.date.astype(str)
print(f"Shape: {df.shape} | Fraud rate: {df['is_fraud'].mean():.2%}")
```

**Q1.** Transaction numbering: add `txn_number` = chronological rank within each customer.
```python
df["txn_number"] = df.groupby("customer_id").cumcount() + 1
```
What % of customers have only 1 transaction? 2–5? 6–20? 20+?

**Q2.** Customer baseline stats: calculate `customer_avg_amount` and `customer_std_amount` using ONLY non-fraud transactions (to avoid contaminating the baseline with fraud values).
```python
legit_stats = (df[df["is_fraud"]==0]
               .groupby("customer_id")["amount"]
               .agg(cust_avg="mean", cust_std="std")
               .reset_index())
df = df.merge(legit_stats, on="customer_id", how="left")
```

**Q3.** Amount z-score: `amount_zscore = (amount - cust_avg) / cust_std.fillna(1)`. Flag `amount_anomaly = |zscore| > 3`. What % of fraud transactions are flagged as amount anomalies? What % of amount anomalies are fraud?

**Q4.** Cumulative prior fraud: for each transaction, how many fraud events has this customer had BEFORE this transaction?
```python
df["prev_fraud"] = df.groupby("customer_id")["is_fraud"].shift(1).fillna(0)
df["cum_prior_fraud"] = df.groupby("customer_id")["prev_fraud"].cumsum()
```
Show fraud rate at each level of `cum_prior_fraud` (0, 1, 2, 3+). Does having prior fraud predict future fraud?

**Q5.** Repeat fraud customer analysis: flag `is_repeat_fraud_cust` = True if the customer has 2+ fraud events total. What % of all fraud comes from repeat-fraud customers? What % of customers are repeat-fraud?

**Q6.** Velocity detection: count transactions per `(customer_id, day)`. Flag days with 3+ transactions as `high_velocity_day`. Join back to `df`. Fraud rate on high-velocity days vs non-high-velocity days. What is the fraud rate ratio?

**Q7.** Merchant risk tiers: for each `merchant_name` (or `merchant_category`), calculate fraud rate, total transactions. Assign `merchant_risk_tier`: High (fraud rate > 5%), Medium (1–5%), Low (<1%). How many merchants in each tier? What % of fraud comes from High-tier merchants?

**Q8.** Time gap between fraud events: for customers with 2+ fraud transactions, calculate `days_since_last_fraud` using `.diff()` within each customer group. What is the median gap? What is the P25? Do fraudsters tend to repeat quickly?

**Q9.** Fraud heatmap: `pd.crosstab(df["hour"], df["day_name"], values=df["is_fraud"], aggfunc="mean")`. Re-order columns Mon–Sun. Plot with `sns.heatmap`. Annotate with 2-decimal fraud rates. What is the single riskiest hour × day combination?

**Q10.** Fraud scoring model:
```python
top5_cats   = df.groupby("merchant_category")["is_fraud"].mean().nlargest(5).index
top3_hours  = df.groupby("hour")["is_fraud"].mean().nlargest(3).index

df["fraud_score"] = (
    df["merchant_category"].isin(top5_cats).astype(int) * 3 +
    df["hour"].isin(top3_hours).astype(int)              * 2 +
    (df["amount_zscore"].abs() > 2).astype(int)          * 2 +
    (df["cum_prior_fraud"] > 0).astype(int)              * 3
)
```
Show: score distribution (value_counts), fraud rate at each score, cumulative % of fraud captured at each score threshold.

**Q11.** Precision-Recall trade-off: for each threshold t ∈ [0, 10]:
```python
precision = df[df["fraud_score"] >= t]["is_fraud"].mean()
recall    = df[df["fraud_score"] >= t]["is_fraud"].sum() / df["is_fraud"].sum()
f1        = 2 * precision * recall / (precision + recall + 1e-9)
```
Build this as a DataFrame indexed by threshold. Find the threshold that maximises F1. Plot precision, recall, and F1 on the same chart.

**Q12.** Fraud ring detection: find groups of customers who all experienced fraud at the same merchant on the same day. These may indicate coordinated attacks.
```python
fraud_events = df[df["is_fraud"] == 1].copy()
fraud_groups = fraud_events.groupby(["merchant_name", "day"])["customer_id"].agg(["count", list])
fraud_groups = fraud_groups[fraud_groups["count"] >= 3]
```
How many such merchant-day combinations exist? What is the total number of customers affected?

**Q13.** Dollar impact analysis: calculate `fraud_loss_by_category` = total fraudulent amount per merchant category. Which category has the highest total dollar loss? Which has the highest per-transaction loss? These may differ from the highest fraud-rate category.

**Q14.** False positive cost: for your optimal threshold from Q11, how many legitimate transactions are incorrectly flagged? Assume the cost of incorrectly blocking a legitimate transaction is $5 (customer friction) and the cost of missing a fraud is the fraudulent amount. Calculate:
```
total_cost_of_model = false_positives × 5 + missed_fraud_loss
total_cost_no_model = all_fraud_loss
savings = total_cost_no_model - total_cost_of_model
```
Is your scoring model cost-effective?

**Q15.** Write `fraud_report(df: pd.DataFrame) -> dict` that returns:
```python
{
    "fraud_rate": ...,
    "total_fraud_loss": ...,
    "top5_risk_categories": [...],
    "optimal_threshold": ...,
    "f1_at_optimal": ...,
    "precision_at_optimal": ...,
    "recall_at_optimal": ...,
    "model_savings": ...,
}
```
And saves a 2×2 dashboard: fraud rate over time, score distribution, precision-recall chart, hour×day heatmap.

---

## Project 13 🔴 — Supplier & Profitability Scorecard
**Dataset:** D5 — DataCo Supply Chain

### Setup
```python
df = pd.read_csv(DATA_DIR / "DataCoSupplyChainDataset.csv", encoding="latin-1")
df.columns = df.columns.str.lower().str.strip().str.replace(" ", "_")
df["order_date"] = pd.to_datetime(df["order_date_(dateorders)"], errors="coerce")
df["profit_margin"] = df["order_profit_per_order"] / df["sales"].replace(0, np.nan)
df["month"]    = df["order_date"].dt.to_period("M")
df["quarter"]  = df["order_date"].dt.to_period("Q").astype(str)
df["year"]     = df["order_date"].dt.year
df["late_flag"]= (df["delivery_status"].str.contains("Late", case=False, na=False)).astype(int)
```

**Q1.** Department P&L table: `total_sales`, `total_profit`, `margin_pct`, `order_count`, `avg_discount`. Sort by `total_profit` descending. Which department is most profitable? Which is loss-making?

**Q2.** QoQ revenue per department: `groupby(["department_name","quarter"])["sales"].sum()`. Unstack to wide format. Add a `qoq_change_pct` column = last Q vs prior Q. Which department had the biggest QoQ decline most recently?

**Q3.** Pareto analysis: sort ALL products by total revenue descending. Calculate cumulative revenue and cumulative %. At what product rank does cumulative revenue cross 80%? How many products account for 80% of revenue (the Pareto 80/20 principle)?

**Q4.** Profit heatmap: `pd.pivot_table(df, values="order_profit_per_order", index="department_name", columns="shipping_mode", aggfunc="mean")`. Plot as heatmap. Which (department, shipping mode) combination is most profitable?

**Q5.** Loss leaders: products where `avg_profit_per_order < 0` AND `order_count > 50`. Calculate: product name, category, total orders, total profit drain. Sort by total profit drain ascending. What is the total cost of carrying loss leaders?

**Q6.** Discount elasticity: for each discount band (0%, 0-5%, 5-10%, 10-15%, 15-20%, 20%+), calculate: avg order value, avg profit, margin %. Plot margin % against discount band. Is there a threshold above which discounts destroy margin?

**Q7.** Monthly profit trend per department: pivot by (department, month). For each department, calculate the rolling 3-month profit mean and std. Flag departments whose most recent month's profit is more than 2 std below their rolling mean. These are "at risk" departments.

**Q8.** Customer Pareto: rank customers by total profit contribution (descending). Calculate cumulative profit %. What % of customers generate 80% of profit? What % of customers are loss-making (negative total profit)?

**Q9.** Volatility vs revenue scatter: for each product, compute monthly revenue and then `cv = std/mean`. Scatter: x = total revenue, y = cv. Size = order count. Label top-5 most volatile high-revenue products. These are your biggest forecasting challenges.

**Q10.** Shipping mode ROI: for each mode, calculate revenue, profit, margin %, avg order count, on-time rate, avg delay days. Plot as a 2×2 grid: revenue/mode bar, margin/mode bar, late_rate/mode bar, scatter (margin vs late_rate per mode with labels).

**Q11.** Seasonal revenue index per department: for each (department, calendar month), compute `avg_monthly_revenue / overall_avg`. Which department is most seasonal (highest std of monthly index)? Which is most stable?

**Q12.** Profit cliff detection: for each department's monthly profit series, find the single largest month-over-month profit drop (most negative `.diff()`). On what date did each department's worst month occur? Were they clustered (industry-wide event) or scattered?

**Q13.** Re-export QBR table: one row per `(department_name, quarter)`. Columns: `revenue`, `profit`, `margin_pct`, `orders`, `late_rate`, `qoq_revenue_change`, `best_product_category`, `status` (🟢/🟡/🔴). Use `pd.ExcelWriter` to save — one sheet per quarter.

**Q14.** Department efficiency frontier: scatter plot of `margin_pct` (x) vs `on_time_rate` (y) for each department. Add quadrant lines at median margin and median on-time rate. Label each point with department name. The top-right quadrant = "stars"; bottom-left = "laggards". Which departments fall in each quadrant?

**Q15.** Write `generate_qbr(df: pd.DataFrame, output_path: str)`:
1. Computes the full QBR table (Q13)
2. Saves to Excel with one sheet per quarter
3. Saves `qbr_efficiency_frontier.png` (Q14 scatter)
4. Saves `qbr_pareto.png` (Q3 cumulative revenue)
5. Prints a QBR narrative:
```
"Q[X] [YEAR] Summary: Best department: [dept] (margin: X%). 
Worst: [dept] (margin: Y%). [N] departments at 🔴 status. 
Late delivery rate improved/worsened by X% vs prior quarter."
```

---

## Project 14 🔴 — Cross-Domain Portfolio Analysis
**Datasets:** D1 + D3 + D6

### Setup
```python
# Olist KPI
olist_df = (pd.read_csv(DATA_DIR / "olist_orders_dataset.csv",
                         parse_dates=["order_purchase_timestamp"])
            .merge(pd.read_csv(DATA_DIR / "olist_order_payments_dataset.csv")
                     .groupby("order_id")["payment_value"].sum().reset_index(),
                   on="order_id", how="left"))
olist_df["kpi"] = olist_df["payment_value"]

# Healthcare KPI
health_df = pd.read_csv(DATA_DIR / "healthcare_dataset.csv")
health_df["kpi"] = health_df["Billing Amount"]

# Airline KPI
airline_df = pd.read_csv(DATA_DIR / "train.csv")
svc_cols = [c for c in airline_df.columns if any(k in c for k in ["service","comfort","boarding","wifi","food","seat","leg","baggage","checkin","cleanliness","entertainment"])]
airline_df["kpi"] = airline_df[svc_cols].mean(axis=1)
```

**Q1.** For each dataset, compute: `n`, `mean_kpi`, `median_kpi`, `std_kpi`, `p25`, `p75`, `p90`, `skewness`. Print as a unified comparison table (one row per domain). Where does each domain's KPI distribution differ most?

**Q2.** Distribution comparison: one figure with 3 subplots — histplot + KDE for each dataset's `kpi`. Use different colours. What distribution shape does each follow? Right-skewed? Bimodal? Uniform?

**Q3.** "One and done" analysis — customers/patients/passengers who used the service only once:
- D1: customers with exactly 1 delivered order
- D3: patients with exactly 1 admission (use `Name` as proxy, noting duplicates may exist)
- D6: N/A (no repeat metric) — instead: % "neutral or dissatisfied" passengers

What % of each population is single-event? What does this imply for retention strategy in each industry?

**Q4.** Segmentation: `pd.qcut(kpi, q=4)` → Q1 (lowest), Q2, Q3, Q4 (highest). For each dataset, show count and mean KPI per quartile. Build a unified DataFrame: `(domain, quartile, count, mean_kpi)`.

**Q5.** Anomaly detection: for each domain, flag records more than 2 std above the mean KPI. How many anomalies in each? As % of total? Do anomalies represent genuine high-value events or data errors?

**Q6.** UNION ALL equivalent: build a unified summary DataFrame:
```python
rows = []
for domain, df, kpi_col in [("Olist", olist_df, "kpi"),
                              ("Healthcare", health_df, "kpi"),
                              ("Airline", airline_df, "kpi")]:
    rows.append({
        "domain":      domain,
        "n":           len(df),
        "mean_kpi":    df[kpi_col].mean(),
        "median_kpi":  df[kpi_col].median(),
        "p90_kpi":     df[kpi_col].quantile(0.9),
        "skewness":    df[kpi_col].skew(),
    })
master = pd.DataFrame(rows)
```

**Q7.** Satisfaction/quality correlation (within each domain):
- D1: group by avg review score. Does higher review → higher payment_value?
- D3: group by billing quartile. Does higher billing → longer length of stay?
- D6: group by delay band. Does more delay → lower service_score?
Plot bar charts. State a one-sentence inference per domain.

**Q8.** Top performer / bottom performer per domain:
- D1: top/bottom seller by avg review score (min 50 orders)
- D3: top/bottom doctor by % "Normal" test results (min 20 patients)
- D6: top/bottom by satisfaction rate per `Class`
Build a table: `domain, performer_type, name, metric, value`.

**Q9.** Value concentration (Pareto) for each domain:
- D1: % of customers generating 80% of revenue
- D3: % of billing categories generating 80% of total billing
- D6: % of passengers with `service_score` above overall mean
Plot as horizontal bar chart showing Pareto concentration per domain.

**Q10.** Build a `domain_eda(df, kpi_col, group_col, label)` function that:
- Computes summary stats
- Plots: distribution, top-10 group_col by mean kpi, anomaly flags
- Returns a dict of stats

Call it for each domain and print all three summaries.

**Q11.** Time trend (D1 and D3 have time data): monthly KPI volume for each. Plot on the same chart (normalised to index=100 at start). Which domain shows more growth? More volatility?

**Q12.** Create a 3-column normalised heatmap comparing domains on standardised metrics. Normalise each metric 0–1 so scales are comparable. Use `sns.heatmap`. Which domain scores highest on each dimension?

**Q13.** Write `generate_portfolio_narrative(master_df: pd.DataFrame) -> str` that creates an automated text summary:
```python
for _, row in master_df.iterrows():
    print(f"{row['domain']}: {row['n']:,} records. "
          f"Avg KPI: {row['mean_kpi']:.1f} (P90: {row['p90_kpi']:.1f}). "
          f"Distribution: {'right-skewed' if row['skewness'] > 1 else 'approximately normal'}.")
```

**Q14.** Cross-domain scatter: for any two domains that share a common dimension (e.g. both have a "satisfaction" proxy), scatter them against each other. Does high satisfaction in e-commerce correlate with high satisfaction in airlines (across customer demographics)?

**Q15.** Full portfolio report: `portfolio_report(data_dir: str, output_dir: str)`:
1. Loads all three datasets
2. Runs `domain_eda` on each
3. Builds `master` summary DataFrame
4. Generates narrative
5. Saves master to `portfolio_kpis.csv`
6. Saves a 3×2 subplot figure: one row per domain, columns = distribution + top-10 group chart
7. Returns master DataFrame

---

## Project 15 🔴 — End-to-End Pipeline Capstone
**Datasets:** D1 + D4 + D5

### Build a reusable, production-style analysis pipeline.

### Setup
```python
from dataclasses import dataclass, field
from typing import Optional, Dict, List
import logging
logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
logger = logging.getLogger(__name__)
```

**Q1.** Write `DataLoader` class:
```python
class DataLoader:
    def __init__(self, data_dir: str):
        self.data_dir = Path(data_dir)
        self._cache: Dict[str, pd.DataFrame] = {}

    def _load(self, filename, **kwargs) -> pd.DataFrame:
        if filename not in self._cache:
            path = self.data_dir / filename
            if not path.exists():
                logger.warning(f"File not found: {path}")
                return pd.DataFrame()
            self._cache[filename] = pd.read_csv(path, **kwargs)
        return self._cache[filename].copy()

    def load_olist(self) -> pd.DataFrame: ...   # implement
    def load_fraud(self) -> pd.DataFrame: ...   # implement
    def load_supply_chain(self) -> pd.DataFrame: ...   # implement
```
Each method should return a cleaned, merged base table. Test each one.

**Q2.** Write `DataValidator` class:
```python
class DataValidator:
    def __init__(self, warn_threshold: float = 0.2):
        self.warn_threshold = warn_threshold
        self.issues: List[str] = []

    def check_nulls(self, df: pd.DataFrame, name: str) -> bool: ...
    def check_duplicates(self, df: pd.DataFrame, key_col: str, name: str) -> bool: ...
    def check_row_count(self, df: pd.DataFrame, min_rows: int, name: str) -> bool: ...
    def check_date_range(self, df, date_col, expected_start, expected_end, name) -> bool: ...
    def report(self) -> None:
        if not self.issues:
            print("✓ All validation checks passed")
        else:
            for issue in self.issues:
                print(f"⚠️  {issue}")
```

**Q3.** Write KPI functions:
```python
def olist_kpis(df: pd.DataFrame) -> dict:
    return {
        "total_orders":         len(df["order_id"].unique()),
        "total_revenue":        df["payment_value"].sum(),
        "avg_order_value":      df["payment_value"].mean(),
        "median_order_value":   df["payment_value"].median(),
        "late_delivery_rate":   (df["is_late"]).mean() if "is_late" in df.columns else None,
        "avg_review_score":     df["review_score"].mean() if "review_score" in df.columns else None,
        "repeat_customer_rate": (df.groupby("customer_unique_id").size() > 1).mean(),
    }

def fraud_kpis(df: pd.DataFrame) -> dict: ...

def supply_kpis(df: pd.DataFrame) -> dict: ...
```

**Q4.** Write `AnomalyDetector` class:
```python
class AnomalyDetector:
    def __init__(self, method: str = "rolling", window: int = 3, threshold: float = 2.0):
        self.method    = method     # "rolling" or "zscore" or "iqr"
        self.window    = window
        self.threshold = threshold
        self.mean_: Optional[float] = None
        self.std_:  Optional[float] = None

    def fit(self, series: pd.Series) -> "AnomalyDetector":
        if self.method == "zscore":
            self.mean_ = series.mean()
            self.std_  = series.std()
        return self

    def predict(self, series: pd.Series) -> pd.Series:
        """Returns boolean mask — True where anomaly."""
        if self.method == "rolling":
            roll_mean = series.rolling(self.window, min_periods=1).mean()
            roll_std  = series.rolling(self.window, min_periods=1).std().fillna(0)
            return (series - roll_mean).abs() > self.threshold * roll_std
        elif self.method == "zscore":
            z = (series - self.mean_) / (self.std_ + 1e-9)
            return z.abs() > self.threshold
        elif self.method == "iqr":
            Q1, Q3 = series.quantile(0.25), series.quantile(0.75)
            IQR = Q3 - Q1
            return (series < Q1 - 1.5*IQR) | (series > Q3 + 1.5*IQR)

    def plot(self, series: pd.Series, title: str = "") -> None:
        mask = self.predict(series)
        fig, ax = plt.subplots(figsize=(12, 4))
        ax.plot(series.index, series.values, color="steelblue", label="Value")
        ax.scatter(series.index[mask], series.values[mask],
                   color="red", s=60, zorder=5, label="Anomaly")
        ax.set_title(title or "Anomaly Detection")
        ax.legend(frameon=False)
        plt.tight_layout()
        plt.show()
```
Test: apply to Olist monthly revenue, fraud monthly rate, and supply chain monthly profit.

**Q5.** Write `CohortAnalyzer` class (reuse Project 11 logic):
```python
class CohortAnalyzer:
    def __init__(self):
        self.retention_: Optional[pd.DataFrame] = None
        self.revenue_:   Optional[pd.DataFrame] = None
        self.cohort_sizes_: Optional[pd.Series] = None

    def fit(self, df: pd.DataFrame, customer_col: str,
            date_col: str, value_col: str) -> "CohortAnalyzer": ...

    def plot_retention(self, title: str = "Cohort Retention") -> None: ...

    def get_summary(self) -> dict:
        return {
            "total_cohorts":       len(self.retention_),
            "avg_m1_retention":    self.retention_.get(1, pd.Series()).mean(),
            "avg_m3_retention":    self.retention_.get(3, pd.Series()).mean(),
            "best_cohort":         self.retention_.get(1, pd.Series()).idxmax(),
            "worst_cohort":        self.retention_.get(1, pd.Series()).idxmin(),
        }
```

**Q6.** Apply RFM pipeline (from Project 9) to Olist data. Apply fraud scoring (from Project 12) to fraud data. Apply efficiency scoring (from Project 13) to supply chain. Verify all three run without errors and return expected shapes.

**Q7.** Write `generate_executive_report(olist_kpis, fraud_kpis, supply_kpis, output_dir)`:
- Builds a `master_kpis` DataFrame (one row per business unit)
- Saves to `master_kpis.csv`
- Plots a 3-column KPI summary: one bar chart per business unit (top 5 KPIs)
- Saves to `executive_dashboard.png`

**Q8.** Write `run_pipeline(data_dir: str, output_dir: str) -> dict`:
```python
def run_pipeline(data_dir: str, output_dir: str) -> dict:
    output_dir = Path(output_dir)
    output_dir.mkdir(exist_ok=True)
    results = {}

    loader    = DataLoader(data_dir)
    validator = DataValidator()

    for name, loader_fn in [("olist", loader.load_olist),
                              ("fraud", loader.load_fraud),
                              ("supply", loader.load_supply_chain)]:
        logger.info(f"Loading {name}...")
        df = loader_fn()
        if df.empty:
            logger.warning(f"Skipping {name} — empty DataFrame")
            continue
        validator.check_nulls(df, name)
        validator.check_row_count(df, min_rows=100, name=name)
        results[name] = df

    validator.report()
    return results
```

**Q9.** Add anomaly detection to the pipeline: after computing monthly KPIs for each business unit, run `AnomalyDetector` on each. Any anomalous months should be logged as warnings and included in the executive report.

**Q10.** Edge case handling: update `run_pipeline` so that:
- If a dataset has > 50% null in any column → log warning, continue with available columns
- If a dataset has fewer than 100 rows → log warning, skip that dataset
- All errors are caught and logged (never crash the pipeline)
- A `pipeline_log.txt` is written with timestamps and outcomes for each step

**Q11.** Unit test: write `test_pipeline()` that:
- Calls `run_pipeline` with the real data directory
- Asserts `results` is not empty
- Asserts each DataFrame has > 0 rows
- Asserts each KPI function returns a dict with all expected keys
- Prints PASS or FAIL for each assertion

**Q12.** Write `generate_readme(output_dir: str, results: dict, kpis: dict)` that creates `README.md` in the output directory with:
- Date and time the pipeline ran
- Datasets processed and row counts
- Key KPIs from master table (formatted as a markdown table)
- Files generated (list)
- Any warnings from the validator

**Q13.** Performance profiling: add timing to `run_pipeline` using `time.perf_counter`. Print how long each step took. Which step is the bottleneck? Suggest (in comments) how to optimise it.

**Q14.** Configuration: move all hardcoded thresholds into a `PipelineConfig` dataclass:
```python
@dataclass
class PipelineConfig:
    data_dir:            str   = "data"
    output_dir:          str   = "outputs"
    null_warn_threshold: float = 0.2
    min_rows:            int   = 100
    anomaly_threshold:   float = 2.0
    anomaly_window:      int   = 3
    rfm_segments:        int   = 5
    fraud_score_threshold: int = 4
```
Update `run_pipeline` to accept a `config: PipelineConfig` argument.

**Q15.** Final integration: write a `main.py` script that:
1. Parses command-line arguments for `data_dir` and `output_dir` using `argparse`
2. Loads `PipelineConfig` from a JSON config file if provided (`--config config.json`)
3. Calls `run_pipeline(config)`
4. Calls `generate_executive_report(...)`
5. Calls `generate_readme(...)`
6. Prints "Pipeline complete. Outputs written to {output_dir}." with a summary of files generated

---

## Answer Key

---

### Project 1 — Selected Answers

```python
# Q7: Skewness check
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

payments = pd.read_csv("data/olist_order_payments_dataset.csv")
order_val = payments.groupby("order_id")["payment_value"].sum()

skewness = order_val.skew()
print(f"Mean:     R${order_val.mean():.2f}")
print(f"Median:   R${order_val.median():.2f}")
print(f"Skewness: {skewness:.2f}")
# Mean > Median → right-skewed. Skewness > 1 → highly skewed.
# Implication: report MEDIAN as "typical order value", not mean.
# Mean is inflated by large outlier orders.

fig, ax = plt.subplots(figsize=(12, 5))
ax.hist(order_val.clip(upper=order_val.quantile(0.99)), bins=50,
        color="steelblue", edgecolor="white")
ax.axvline(order_val.mean(),   color="red",    linestyle="--", label=f"Mean R${order_val.mean():.0f}")
ax.axvline(order_val.median(), color="orange", linestyle=":",  label=f"Median R${order_val.median():.0f}")
ax.set_title("Order Value Distribution (capped at P99)")
ax.set_xlabel("Payment Value (R$)")
ax.legend(frameon=False)
plt.tight_layout()
plt.show()

# Q14: Late delivery trend
orders = pd.read_csv("data/olist_orders_dataset.csv",
                      parse_dates=["order_purchase_timestamp",
                                   "order_delivered_customer_date",
                                   "order_estimated_delivery_date"])
delivered = orders.dropna(subset=["order_delivered_customer_date",
                                   "order_estimated_delivery_date"]).copy()
delivered["is_late"] = (delivered["order_delivered_customer_date"] >
                         delivered["order_estimated_delivery_date"]).astype(int)
print(f"Overall late rate: {delivered['is_late'].mean():.1%}")

delivered["order_month"] = delivered["order_purchase_timestamp"].dt.to_period("M").astype(str)
late_trend = delivered.groupby("order_month")["is_late"].mean()

fig, ax = plt.subplots(figsize=(14, 5))
ax.plot(late_trend.index, late_trend.values * 100, marker="o", color="coral")
ax.axhline(delivered["is_late"].mean() * 100, color="grey", linestyle="--",
           label=f"Overall avg {delivered['is_late'].mean():.1%}")
ax.set_title("Late Delivery Rate by Month", fontweight="bold")
ax.set_ylabel("Late Rate (%)")
ax.tick_params(axis="x", rotation=45)
ax.legend(frameon=False)
plt.tight_layout()
plt.show()
```

---

### Project 9 — RFM Pipeline Answer

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from pathlib import Path

def rfm_pipeline(
    orders_df: pd.DataFrame,
    payments_df: pd.DataFrame,
    customers_df: pd.DataFrame,
    n_segments: int = 5
) -> pd.DataFrame:
    """
    Full RFM pipeline for Olist data.
    Returns one row per customer_unique_id with RFM scores and segment.
    """
    # Aggregate payments to one row per order
    pay_agg = payments_df.groupby("order_id")["payment_value"].sum().reset_index()

    # Build transaction table — delivered orders only
    txn = (orders_df
           .merge(pay_agg, on="order_id", how="left")
           .merge(customers_df[["customer_id","customer_unique_id"]], on="customer_id"))
    txn = txn[txn["order_status"] == "delivered"].copy()
    txn["order_date"] = pd.to_datetime(txn["order_purchase_timestamp"])

    # Reference date
    reference_date = txn["order_date"].max()

    # RFM components
    rfm = txn.groupby("customer_unique_id").agg(
        last_order = ("order_date",     "max"),
        frequency  = ("order_id",       "count"),
        monetary   = ("payment_value",  "sum"),
    ).reset_index()
    rfm["recency_days"] = (reference_date - rfm["last_order"]).dt.days

    # Scoring — handle duplicates in qcut edges
    labels = list(range(1, n_segments + 1))
    rfm["r_score"] = pd.qcut(rfm["recency_days"],
                              n_segments, labels=labels[::-1], duplicates="drop")
    rfm["f_score"] = pd.qcut(rfm["frequency"].rank(method="first"),
                              n_segments, labels=labels, duplicates="drop")
    rfm["m_score"] = pd.qcut(rfm["monetary"],
                              n_segments, labels=labels, duplicates="drop")

    for col in ["r_score", "f_score", "m_score"]:
        rfm[col] = rfm[col].astype(float).astype(int)

    # Segmentation
    cond = [
        (rfm["r_score"] >= 4) & (rfm["f_score"] >= 4),
        (rfm["r_score"] >= 3) & (rfm["f_score"] >= 3),
        (rfm["r_score"] >= 4) & (rfm["f_score"] <= 2),
        (rfm["r_score"] <= 2) & (rfm["f_score"] >= 3),
        (rfm["r_score"] == 1) & (rfm["f_score"] == 1),
    ]
    segs = ["Champions", "Loyal", "Promising", "At Risk", "Lost"]
    rfm["segment"] = np.select(cond, segs, default="Others")

    # Visualise
    fig, axes = plt.subplots(1, 2, figsize=(14, 5))

    seg_order = rfm["segment"].value_counts().index
    rfm["segment"].value_counts().plot.bar(ax=axes[0], color="steelblue", edgecolor="white")
    axes[0].set_title("Customer Count per Segment")
    axes[0].tick_params(axis="x", rotation=30)

    rfm.groupby("segment")["monetary"].mean().sort_values().plot.barh(
        ax=axes[1], color="coral", edgecolor="white")
    axes[1].set_title("Avg Monetary Value per Segment")

    plt.tight_layout()
    plt.savefig("outputs/rfm_segments.png", dpi=120, bbox_inches="tight")
    plt.show()

    # Summary table
    print("\n=== RFM Segment Summary ===")
    summary = rfm.groupby("segment").agg(
        customers    = ("customer_unique_id", "count"),
        avg_recency  = ("recency_days",       "mean"),
        avg_freq     = ("frequency",          "mean"),
        avg_monetary = ("monetary",           "mean"),
    ).round(1)
    print(summary.sort_values("avg_monetary", ascending=False).to_string())

    return rfm


# Usage
DATA = Path("data")
orders    = pd.read_csv(DATA / "olist_orders_dataset.csv")
payments  = pd.read_csv(DATA / "olist_order_payments_dataset.csv")
customers = pd.read_csv(DATA / "olist_customers_dataset.csv")
rfm_df    = rfm_pipeline(orders, payments, customers)
```

---

### Project 12 — Fraud Scoring Answer

```python
def build_fraud_features(df: pd.DataFrame) -> pd.DataFrame:
    """Add all fraud detection features to the transaction DataFrame."""
    df = df.copy()
    df["date"] = pd.to_datetime(df["date"])
    df = df.sort_values(["customer_id", "date"]).reset_index(drop=True)

    df["hour"]      = df["date"].dt.hour
    df["dayofweek"] = df["date"].dt.dayofweek
    df["txn_number"]= df.groupby("customer_id").cumcount() + 1

    # Customer baselines from legitimate transactions only
    legit = df[df["is_fraud"] == 0]
    stats = legit.groupby("customer_id")["amount"].agg(
        cust_avg="mean", cust_std="std"
    ).reset_index()
    df = df.merge(stats, on="customer_id", how="left")
    df["amount_zscore"] = ((df["amount"] - df["cust_avg"])
                           / df["cust_std"].fillna(1))

    # Cumulative prior fraud
    df["prev_fraud"]       = df.groupby("customer_id")["is_fraud"].shift(1).fillna(0)
    df["cum_prior_fraud"]  = df.groupby("customer_id")["prev_fraud"].cumsum()

    # High-velocity flag
    daily_counts = df.groupby(["customer_id", df["date"].dt.date])["amount"].count().reset_index()
    daily_counts.columns = ["customer_id", "day", "daily_txn_count"]
    df["day"] = df["date"].dt.date
    df = df.merge(daily_counts, on=["customer_id", "day"], how="left")
    df["high_velocity"] = (df["daily_txn_count"] >= 3).astype(int)

    # Risk features
    top5_cats  = df.groupby("merchant_category")["is_fraud"].mean().nlargest(5).index.tolist()
    top3_hours = df.groupby("hour")["is_fraud"].mean().nlargest(3).index.tolist()

    df["fraud_score"] = (
        df["merchant_category"].isin(top5_cats).astype(int)    * 3 +
        df["hour"].isin(top3_hours).astype(int)                 * 2 +
        (df["amount_zscore"].abs() > 2).astype(int)             * 2 +
        (df["cum_prior_fraud"] > 0).astype(int)                 * 3
    )
    return df


def evaluate_fraud_score(df: pd.DataFrame) -> pd.DataFrame:
    """Build precision-recall table for each fraud_score threshold."""
    total_fraud = df["is_fraud"].sum()
    rows = []
    for t in range(11):
        flagged     = df[df["fraud_score"] >= t]
        tp          = flagged["is_fraud"].sum()
        precision   = tp / len(flagged)      if len(flagged) > 0 else 0
        recall      = tp / total_fraud       if total_fraud > 0  else 0
        f1          = (2 * precision * recall / (precision + recall)
                       if (precision + recall) > 0 else 0)
        rows.append({"threshold": t, "flagged": len(flagged),
                     "precision": precision, "recall": recall, "f1": f1})
    return pd.DataFrame(rows)
```

---

### Project 15 — Pipeline Skeleton Answer

```python
# Minimal working run_pipeline
from pathlib import Path
import pandas as pd
import logging
import time

logger = logging.getLogger(__name__)

def run_pipeline(data_dir: str, output_dir: str) -> dict:
    t0 = time.perf_counter()
    output_dir = Path(output_dir)
    output_dir.mkdir(exist_ok=True)

    loader    = DataLoader(data_dir)
    validator = DataValidator()
    results   = {}
    log_lines = [f"Pipeline started: {pd.Timestamp.now()}"]

    datasets = [("olist", loader.load_olist),
                ("fraud", loader.load_fraud),
                ("supply", loader.load_supply_chain)]

    for name, fn in datasets:
        t1 = time.perf_counter()
        try:
            df = fn()
            if df.empty:
                msg = f"SKIP {name}: empty DataFrame"
                logger.warning(msg); log_lines.append(msg); continue

            ok_null = validator.check_nulls(df, name)
            ok_rows = validator.check_row_count(df, 100, name)

            results[name] = df
            elapsed = time.perf_counter() - t1
            msg = f"OK   {name}: {len(df):,} rows in {elapsed:.2f}s"
            logger.info(msg); log_lines.append(msg)

        except Exception as e:
            msg = f"ERROR {name}: {type(e).__name__}: {e}"
            logger.error(msg); log_lines.append(msg)

    validator.report()
    total = time.perf_counter() - t0
    log_lines.append(f"Pipeline completed in {total:.2f}s")

    (output_dir / "pipeline_log.txt").write_text("\n".join(log_lines))
    return results
```

---

*End of Python Projects — v2 Expanded Edition*
*Chat #2 · June 2026*
