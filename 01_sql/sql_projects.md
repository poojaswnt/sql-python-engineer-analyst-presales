# SQL Projects v2 — 15 Comprehensive Practice Projects

> **How to use this file**
> Each project has: a real business framing, Kaggle dataset link, schema setup SQL, 15 questions (Q1–Q8 medium, Q9–Q14 complex, Q15 hard capstone), and answers at the bottom.
> Run in [DB Browser for SQLite](https://sqlitebrowser.org/).
> **Chat #2 · June 2026 (v2)**

---

## Dataset Index

| ID | Dataset | Domain | Kaggle URL |
|----|---------|--------|-----------|
| D1 | Olist Brazilian E-Commerce | E-Commerce | https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce |
| D2 | UK Online Retail II | Retail | https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci |
| D3 | Healthcare Dataset | Healthcare | https://www.kaggle.com/datasets/prasad22/healthcare-dataset |
| D4 | Financial Transactions + Fraud | Banking | https://www.kaggle.com/datasets/computingvictor/transactions-fraud-datasets |
| D5 | DataCo Supply Chain | Manufacturing | https://www.kaggle.com/datasets/shashwatwork/dataco-smart-supply-chain-for-big-data-analysis |
| D6 | Airline Passenger Satisfaction | Travel | https://www.kaggle.com/datasets/teejmahal20/airline-passenger-satisfaction |
| D7 | IBM HR Analytics | Hi-Tech | https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset |

---

## Project 1 — Marketplace Health Report
**Dataset:** D1 — Olist E-Commerce | **Domain:** E-Commerce

### Business Context
You are a data analyst presenting to Olist's board. They want a complete health check of the marketplace: order volumes, revenue, delivery performance, seller quality, and customer behaviour — all in one session.

### Schema
```sql
CREATE TABLE customers (customer_id TEXT PRIMARY KEY, customer_unique_id TEXT, customer_city TEXT, customer_state TEXT);
CREATE TABLE orders (order_id TEXT PRIMARY KEY, customer_id TEXT, order_status TEXT, order_purchase_timestamp TEXT, order_delivered_timestamp TEXT, order_estimated_delivery TEXT);
CREATE TABLE order_items (order_id TEXT, seller_id TEXT, product_id TEXT, price REAL, freight_value REAL);
CREATE TABLE order_payments (order_id TEXT, payment_type TEXT, payment_value REAL);
CREATE TABLE order_reviews (order_id TEXT, review_score INTEGER, review_creation_date TEXT);
CREATE TABLE products (product_id TEXT PRIMARY KEY, product_category_name TEXT);
CREATE TABLE sellers (seller_id TEXT PRIMARY KEY, seller_city TEXT, seller_state TEXT);
```

### Questions

**Q1.** Monthly order volume and total revenue — show year-month, order count, and total payment value. Sort chronologically.

**Q2.** Order status breakdown: count and % for each status.

**Q3.** Top 10 product categories by total revenue (join order_items → products).

**Q4.** Top 10 sellers by revenue. Also show their average review score.

**Q5.** What % of orders were delivered on time (delivered before or on estimated delivery date)?

**Q6.** Average delivery time in days (purchase → delivery) by state.

**Q7.** Revenue breakdown by payment type.

**Q8.** How many customers made more than one order? What % are repeat buyers?

**Q9.** Calculate the average review score per product category. Which categories have the worst customer satisfaction (minimum 100 reviews to qualify)?

**Q10.** Build a seller scorecard: for each seller, compute total orders, total revenue, avg review score, on-time delivery rate (%), and rank sellers by revenue using DENSE_RANK(). Show top 20.

**Q11.** Identify "problem sellers": sellers whose on-time delivery rate is below 60% AND average review score is below 3.5 AND they have at least 50 orders. How many are there? What is their combined revenue? What categories do they sell?

**Q12.** Customer repeat-purchase funnel:
- How many customers made exactly 1 order?
- 2 orders? 3 orders? 4+?
- What is the average revenue per customer at each level?
- What is the average review score at each level?

**Q13.** Week-over-week order volume trend. Use LAG() to calculate WoW change (absolute and %). Flag weeks where volume dropped more than 20% from the previous week.

**Q14.** Revenue concentration (Pareto): using a cumulative SUM() window function, find what % of sellers generate 80% of total revenue. Is there a long tail problem?

**Q15.** Full monthly cohort table: for each acquisition month, show customer count, month-0 revenue, and retention rate at months 1, 2, and 3. Output as a pivot table (CASE WHEN for each month column). Flag cohorts with month-1 retention below 5%.

---

## Project 2 — Retail Revenue Intelligence
**Dataset:** D2 — UK Online Retail II | **Domain:** Retail

### Business Context
You've been hired by this UK retailer to build their first ever revenue intelligence dashboard. The data is messy (returns mixed with sales, NULLs in customer IDs, inconsistent descriptions). Your SQL must handle all of it.

### Schema
```sql
CREATE TABLE uk_retail (
    invoice_no TEXT, stock_code TEXT, description TEXT,
    quantity INTEGER, invoice_date TEXT, unit_price REAL,
    customer_id TEXT, country TEXT
);
```

### Questions

**Q1.** Separate gross sales from returns. Show: total invoices, total sales revenue, total return value, net revenue, and return rate (%) in one query.

**Q2.** Revenue by country — top 15, with % of total and cumulative % (to see Pareto distribution).

**Q3.** Monthly net revenue trend (sales minus returns) with 3-month rolling average.

**Q4.** Top 20 products by net revenue (gross - returns). Include: stock_code, description, units sold, units returned, return rate %, net revenue.

**Q5.** Find "ghost customers" — rows with NULL customer_id. What % of revenue do they represent? Which countries have the most ghost rows?

**Q6.** Average basket size (revenue per invoice) by country and by month. Has basket size trended up or down over the dataset period?

**Q7.** Price volatility: for each product (stock_code), find the min, max, and avg unit price across all invoices. Flag products where max price is more than 2× the min price (possible pricing errors or tiered pricing).

**Q8.** Day-of-week analysis: which day has highest revenue? Highest average basket? Most returns? Build a single query with a column per metric per day.

**Q9.** Customer cohort analysis: for customers with known IDs, group by first purchase month (cohort). For each cohort, calculate total customers, avg first-purchase value, and how many made a second purchase within 90 days.

**Q10.** Product lifecycle: for each product, find the first invoice date, last invoice date, total months active, and average monthly revenue. Classify as: Active (sold in last 60 days), Declining (no sale in 60-180 days), Dead (no sale in 180+ days).

**Q11.** Find "loyal big spenders": customers (known IDs only) who have placed 5+ invoices and whose total net revenue is in the top 20% of all customers. Show their purchase frequency, avg basket size, and favourite product (by revenue).

**Q12.** Return pattern investigation: are returns concentrated in specific time windows after purchase? For each return, calculate days between the original sale invoice and the return invoice. Show distribution by: same-day, 1-7 days, 8-30 days, 31-90 days, 90+ days.
*(Hint: JOIN uk_retail to itself on stock_code + customer_id, where one side has negative quantity)*

**Q13.** Seasonal decomposition proxy: calculate the average revenue for each (weekday, hour) combination. Which time slots are peak? Build a heatmap-friendly output with weekday as rows and hour buckets (morning/afternoon/evening) as columns.

**Q14.** Detect potential data errors: find any of these anomalies:
- Unit price = 0 (free items — intentional?)
- Negative unit price
- Quantity > 10,000 (bulk orders or errors?)
- Description contains 'TEST', 'ADJUST', 'POSTAGE', 'DOT', 'MANUAL'
Show count and revenue impact of each anomaly type.

**Q15.** Build a full customer segmentation using RFM — but also add a 4th dimension: **Product Diversity Score** (number of distinct stock_codes purchased). Score 1-5 using NTILE. Create a combined RFMD score and segment customers into: Platinum (all ≥4), Gold (avg ≥3.5), Silver (avg ≥2.5), Bronze (rest). Show segment sizes and avg metrics.

---

## Project 3 — Hospital Operations & Cost Analysis
**Dataset:** D3 — Healthcare Dataset | **Domain:** Healthcare / Insurance

### Business Context
You're a healthcare data analyst supporting the CFO and CMO. They need insight into patient flow, cost drivers, readmission risk, and insurance performance.

### Schema
```sql
CREATE TABLE patients (
    name TEXT, age INTEGER, gender TEXT, blood_type TEXT,
    medical_condition TEXT, date_of_admission TEXT, doctor TEXT, hospital TEXT,
    insurance_provider TEXT, billing_amount REAL, room_number INTEGER,
    admission_type TEXT, discharge_date TEXT, medication TEXT, test_results TEXT
);
```

### Questions

**Q1.** Admissions by month — trend over the full dataset period.

**Q2.** Average, min, max billing amount by medical condition.

**Q3.** Gender and age distribution across admission types (Emergency, Elective, Urgent).

**Q4.** Which doctors have the highest patient volume? Top 10. Also show their avg billing and avg length of stay.

**Q5.** Insurance provider comparison: total patients, total billed, avg billed, % of patients billed above $30k.

**Q6.** Length of stay distribution by admission type. Show: avg, median (use NTILE to approximate), min, max, and % of stays > 14 days.

**Q7.** Test result breakdown (Normal / Abnormal / Inconclusive) by medical condition. Which conditions have the highest rate of Abnormal results?

**Q8.** Which medication is most commonly prescribed per medical condition?

**Q9.** High-cost patient identification: find patients in the top 5% of billing (use NTILE(20), select bucket 20). For these patients, show their medical conditions, admission types, and insurance providers. Are high-cost patients concentrated in specific conditions?

**Q10.** Readmission proxy analysis: since we don't have a patient ID, use `name` as a proxy. Find patients admitted more than once. For repeat admissions:
- How many patients returned?
- What is the avg time between admissions (days)?
- What is the total billed across all their visits?
- Did their medical condition change between visits?

**Q11.** Doctor performance scorecard: for each doctor, calculate:
- Total patients
- Avg billing amount
- Avg length of stay
- % of cases with Abnormal test results
- % of cases that were Emergency admissions
- Rank by total patients using DENSE_RANK()
Flag doctors whose Abnormal result rate is > 1.5× the overall average.

**Q12.** Insurance billing analysis: for each insurance provider, calculate the billing distribution using NTILE(4) to create quartiles. Show the avg billing in each quartile per insurer. Are some insurers paying more on average than others for similar conditions?

**Q13.** Seasonal admission patterns: are certain conditions more prevalent in certain months? Build a pivot showing: condition as rows, months as columns, patient count as values. Use CASE WHEN for the pivot.

**Q14.** Hospital workload heatmap: for each (hospital, admission_type) combination, calculate avg daily admissions (total admissions / days in dataset). Which hospital-admission type combinations are the most intensive? Flag any with avg > 3 daily admissions as "high load."

**Q15.** Risk stratification model in pure SQL:
Assign a "clinical risk score" to each patient based on:
- Age 65+ → +2 points
- Emergency admission → +2
- Abnormal test result → +3
- Billing > $40,000 → +1
- Length of stay > 14 days → +2
- Blood type is O- (universal donor/high demand) → +1
Sum the points as `risk_score`. Show distribution. What is the actual profile (condition, avg billing, avg stay) of high-risk patients (score ≥ 6)?

---

## Project 4 — Fraud Detection & Risk Analysis
**Dataset:** D4 — Financial Transactions + Fraud | **Domain:** Banking

### Business Context
You're a fraud analyst. The fraud team needs actionable intelligence to build detection rules and prioritise investigations. Every insight you surface could become a real-time rule in the transaction monitoring system.

### Schema
```sql
CREATE TABLE transactions (
    transaction_id TEXT PRIMARY KEY, date TEXT, customer_id TEXT,
    card_type TEXT, transaction_type TEXT, merchant_name TEXT,
    merchant_category TEXT, amount REAL, is_fraud INTEGER
);
```

### Questions

**Q1.** Fraud overview: total transactions, fraud count, fraud rate %, total fraud value, avg fraudulent transaction amount vs avg legitimate amount.

**Q2.** Fraud rate by merchant category — rank categories by fraud rate (min 100 transactions to qualify).

**Q3.** Fraud by hour of day: which hours have fraud rate above overall average? Express as a "risk multiplier" (category_fraud_rate / overall_fraud_rate).

**Q4.** Fraud by day of week: same analysis. Which days are the riskiest?

**Q5.** Card type analysis: fraud rate and avg fraud amount per card type.

**Q6.** Transaction type breakdown: fraud rate per transaction type (online, in-store, ATM, etc.).

**Q7.** Fraud amount distribution: use NTILE(10) to decile all FRAUD transactions. Show: decile, amount range (min/max), count per decile. Where is fraud concentrated by amount?

**Q8.** Top 20 merchant names by fraud count. What % of all fraud do these 20 merchants represent?

**Q9.** Customer fraud velocity: for each customer with at least 1 fraud transaction, calculate:
- Total transactions
- Fraud count
- Fraud rate
- Days between first and last fraud transaction
- Avg days between consecutive fraud events (use LAG on fraud transactions only)
Classify customers as: Isolated (1 fraud), Repeat (2-4), Frequent (5+).

**Q10.** Fraud clustering by time: find 7-day rolling windows with the highest fraud density. For each day, calculate: fraud count in the preceding 7 days, fraud value in the preceding 7 days. Flag windows where fraud count exceeds 2× the rolling 30-day average.

**Q11.** Fraud immediately following legitimate transactions: for each customer, use LAG() to find cases where a fraud transaction occurred within 1 hour of a legitimate transaction. How many such "compromise events" exist? What is the avg time gap? Which merchant categories are most often associated with the legitimate transaction that precedes fraud?

**Q12.** Merchant risk scoring: for each merchant, calculate:
- Total transaction count
- Fraud count and fraud rate
- Avg legitimate transaction amount
- Avg fraudulent transaction amount
- Ratio of fraud amount to legitimate amount
- Risk tier: Low (fraud rate < 1%), Medium (1-5%), High (>5%)
List all High-risk merchants with their full profile.

**Q13.** Geographic fraud concentration (use merchant_category as a proxy for location if no geography): are certain merchant categories over-represented in fraud compared to their share of total transactions? Calculate "fraud lift" = (% of fraud transactions) / (% of all transactions) per category. Anything with lift > 2.0 is a high-risk category.

**Q14.** Temporal fraud patterns: is fraud increasing, stable, or decreasing over the dataset period? Calculate:
- Monthly fraud rate trend
- Month-over-month change in fraud rate
- 3-month rolling fraud rate
Classify the trend as Improving, Stable, or Worsening.

**Q15.** Fraud rule engine in SQL: build a scoring model. Assign risk points to each transaction based on your analysis findings:
- Merchant category in top-5 high-risk categories → +3
- Hour between 11pm-4am → +2
- Transaction type = online → +1
- Amount > 3× customer's avg transaction → +2
- Customer has prior fraud in last 30 days → +3
- Card type = prepaid (if applicable) → +1
Sum as `fraud_risk_score`. Validate: what is the actual fraud rate for score ≥ 7 vs score ≤ 2? Calculate the model's precision (% of high-score transactions that are actually fraud) and recall (% of fraud caught by high-score threshold).

---
---

## Project 5 — Monthly Revenue Trend: Same Question, Two Datasets
**Datasets:** D1 (Olist) + D2 (UK Retail) | **Domain:** Cross-domain

### Business Context
Same analytical question, two completely different data models. This project builds the skill of adapting a template query to unfamiliar schemas — essential for any analyst who joins a new company.

### Questions

**Q1.** Write the monthly revenue query for D1 (Olist) and D2 (UK Retail) separately. Show: year_month, revenue, order/invoice count.

**Q2.** MoM growth rate for each dataset — use LAG(). Which dataset shows more volatility?

**Q3.** Q4 seasonality: is Q4 (Oct-Dec) consistently the strongest quarter in both datasets? Calculate avg Q4 vs non-Q4 revenue across all years.

**Q4.** Best and worst single months for each dataset. What was happening around those dates?

**Q5.** Rolling 3-month average for each dataset.

**Q6.** For D1: revenue by Brazilian state per month. Which state grows fastest over the period?

**Q7.** For D2: UK vs international revenue split by month. Does UK's share change over time?

**Q8.** Year-over-year comparison: for each calendar month (Jan=1 through Dec=12), show avg revenue in year 1 vs year 2 for each dataset.

**Q9.** Find the top 5 single days by revenue for each dataset. Are any of these near holidays?

**Q10.** Anomaly detection: calculate mean and standard deviation of monthly revenue for each dataset. Flag months where revenue is more than 1.5 standard deviations from the mean. What's the business story behind each anomaly?
*(Hint: for stddev in SQLite, use: SQRT(AVG(revenue*revenue) - AVG(revenue)*AVG(revenue)))*

**Q11.** For D1: calculate revenue per active seller per month. How does this trend? Are more sellers joining but each making less, or is there concentration in fewer sellers?

**Q12.** For D2: revenue per active customer per month (known IDs only). Compare to the revenue per anonymous transaction. Are registered customers more valuable?

**Q13.** Build a "unified monthly report" using UNION ALL: combine D1 and D2 into one result set with columns: source, year_month, revenue, entity_count (orders or invoices), avg_transaction_value. Label each row 'Olist' or 'UKRetail'.

**Q14.** Within the unified report, RANK() each month's revenue within its own source. Then find months where both datasets were simultaneously in their top-10 — is there a global seasonality pattern?

**Q15.** Revenue forecasting in SQL: use the last 3 months of data for each dataset to calculate an "expected" next month revenue (simple average of last 3 months). Then compare this "forecast" to the actual last month. Express forecast error as %. Which dataset is more predictable?

---

## Project 6 — Supply Chain Risk Intelligence
**Dataset:** D5 — DataCo Supply Chain | **Domain:** Manufacturing

### Business Context
The VP of Operations has been told delivery problems are costing the company contracts. You need to build a comprehensive risk intelligence report that identifies root causes and prioritises fixes.

### Schema
```sql
CREATE TABLE supply_chain (
    order_id INTEGER, order_date TEXT, ship_date TEXT,
    delivery_status TEXT, shipping_mode TEXT,
    days_for_shipping_real INTEGER, days_for_shipment_scheduled INTEGER,
    customer_id INTEGER, customer_segment TEXT, customer_city TEXT, customer_country TEXT,
    product_name TEXT, category_name TEXT, department_name TEXT,
    order_region TEXT, order_country TEXT,
    sales REAL, quantity INTEGER, discount REAL, profit REAL,
    late_delivery_risk INTEGER
);
```

### Questions

**Q1.** Delivery status breakdown: on-time vs late vs cancelled. What is the overall late delivery rate?

**Q2.** Late delivery rate by shipping mode. Which mode has the worst performance?

**Q3.** Average actual vs scheduled shipping days per shipping mode. What is the average delay (actual - scheduled)?

**Q4.** Late delivery rate by product category — top 10 worst categories.

**Q5.** Late delivery rate by order region and country.

**Q6.** Discount impact: group orders into discount bands (0, 1-10%, 11-20%, >20%). Show late rate and avg profit per band. Is heavy discounting associated with worse delivery performance?

**Q7.** Profit vs delivery: avg profit for on-time vs late orders. What is the total profit lost to late deliveries (assume same avg profit counterfactual)?

**Q8.** Customer segment analysis: which segment (Consumer, Corporate, Home Office) has the worst delivery experience?

**Q9.** Monthly late delivery rate trend — is performance improving or deteriorating?

**Q10.** "High-risk order" definition: identify the combination of factors most associated with late delivery. Test each individually: shipping mode, category, region, discount band. Build a composite risk flag:
- `high_risk_mode` = 1 if shipping_mode in top-2 worst modes
- `high_risk_category` = 1 if category in top-5 worst categories
- `high_risk_region` = 1 if region in top-3 worst regions
- `risk_count` = sum of flags
Show actual late rate at each risk_count level (0, 1, 2, 3).

**Q11.** Severely late orders (actual > scheduled + 5 days): count, total revenue, avg profit. What % of all orders are severely late? Are they concentrated in specific products or regions?

**Q12.** Department-level P&L with delivery performance:
- Department name
- Total revenue
- Total profit
- Profit margin %
- Late delivery rate %
- On-time rate %
- Rank departments by profit margin (DENSE_RANK)
- Classify: Star (margin >15% AND late <20%), Problem (margin <5% OR late >40%), Normal (rest)

**Q13.** Shipping mode economics: for each shipping mode, calculate:
- Total orders
- Late rate
- Avg days delay
- Total revenue
- Total profit
- Revenue per order
- Profit per order
- Is the premium shipping mode actually worth its premium (revenue AND lower late rate)?

**Q14.** Customer-level risk: find customers who have received 3+ late deliveries. How many unique customers? What is their avg satisfaction likely to be (use late rate as a proxy)? What is their total revenue? These are churn risks.

**Q15.** Build a predictive late-delivery risk score in SQL. Using historical late rates calculated from subqueries:
Assign points to each order:
- shipping_mode = worst mode → +3
- category = top-5 worst → +2
- discount > 0.2 → +1
- region = top-3 worst → +2
Sum as `risk_score`. For each score level, show: total orders, actual late count, actual late rate. Does the score predict late delivery? Calculate precision at threshold ≥ 5.

---

## Project 7 — Customer Churn & Retention Analysis
**Dataset:** D1 + D7 (IBM HR as a proxy for B2B churn) | **Domain:** Cross-domain

### Business Context
"Churn" looks different in e-commerce (customers stop buying) vs enterprise (employees leave). The SQL patterns are the same. This project builds both.

### Part A — E-Commerce Churn (D1: Olist)

**Q1.** Define "churned customer" as: made at least 1 order, but last order was more than 180 days before the dataset's max order date. How many churned customers? What % of total?

**Q2.** What was the avg order value, avg review score, and avg number of orders for churned vs retained customers?

**Q3.** Which product categories are most associated with churned customers' last purchase? (The last thing they bought before they churned.)

**Q4.** Geographic churn: which Brazilian states have the highest churn rate? Show state, total customers, churned count, churn rate %.

**Q5.** Cohort churn: for each monthly acquisition cohort, what % of customers churned within 90 days of their first purchase?

**Q6.** "Win-back candidates": churned customers who had high lifetime value (top 25% by total spend) AND a good last review score (≥ 4). How many? What was their avg LTV?

**Q7.** Time-to-churn: for customers who eventually churned, calculate the time between their first and last order. Distribution: <30 days, 30-90, 90-180, 180-365, 365+.

### Part B — Employee Attrition (D7: IBM HR)

**Q8.** Overall attrition rate. Breakdown by department and job role.

**Q9.** Overtime and attrition: compare attrition rates for OverTime=Yes vs No. Is this the single strongest predictor?

**Q10.** Attrition by satisfaction level (1-4) across all four satisfaction dimensions: JobSatisfaction, EnvironmentSatisfaction, RelationshipSatisfaction, WorkLifeBalance. Which dimension is most strongly associated with attrition?

**Q11.** Salary and attrition: create 5 salary bands using NTILE(5) on MonthlyIncome. Show attrition rate per band. Is there a clear salary threshold below which attrition spikes?

**Q12.** Combined churn/attrition risk score:
For Olist customers: flag if churned (1=yes, 0=no).
For IBM employees: flag if attrition = 'Yes'.
Build a score system for each. Then compare: in both datasets, what are the top 3 predictors of churn/attrition you found? Write a SQL comment block summarising your findings.

**Q13.** Attrition survival curve proxy: group employees by YearsAtCompany (0, 1, 2, 3, ..., 15+). For each tenure group, calculate the attrition rate. At what tenure does attrition start to decline significantly?

**Q14.** Multi-factor attrition segments: create 4 employee segments using combinations of OverTime and JobSatisfaction:
- High Satisfaction + No Overtime (safe zone)
- High Satisfaction + Overtime (at risk despite satisfaction)
- Low Satisfaction + No Overtime (disengaged)
- Low Satisfaction + Overtime (flight risk)
Show segment sizes, attrition rates, avg monthly income, avg years at company.

**Q15.** Build the "Flight Risk Index" for every current employee (attrition = 'No'):
Assign points: OverTime=Yes (+3), JobSatisfaction ≤ 2 (+2), EnvironmentSatisfaction ≤ 2 (+2), YearsSinceLastPromotion > 4 (+2), NumCompaniesWorked > 5 (+1), DistanceFromHome > 20 (+1), PercentSalaryHike < 12 (+1).
Sum as `flight_risk_score`. How many employees score ≥ 7 (critical risk)? What departments are they in? What would it cost to lose all of them (calculate avg salary × 6 months as a rough turnover cost)?

---

## Project 8 — Airline Passenger Experience Analysis
**Dataset:** D6 — Airline Passenger Satisfaction | **Domain:** Travel & Transport

### Schema
```sql
CREATE TABLE flights (
    id INTEGER PRIMARY KEY, gender TEXT, customer_type TEXT, age INTEGER,
    type_of_travel TEXT, class TEXT, flight_distance INTEGER,
    inflight_wifi_service INTEGER, departure_arrival_time_convenient INTEGER,
    ease_of_online_booking INTEGER, gate_location INTEGER,
    food_and_drink INTEGER, online_boarding INTEGER, seat_comfort INTEGER,
    inflight_entertainment INTEGER, on_board_service INTEGER, leg_room_service INTEGER,
    baggage_handling INTEGER, checkin_service INTEGER, inflight_service INTEGER,
    cleanliness INTEGER, departure_delay_in_minutes INTEGER,
    arrival_delay_in_minutes INTEGER, satisfaction TEXT
);
```

### Questions

**Q1.** Overall satisfaction breakdown (Satisfied vs Neutral or Dissatisfied) by customer type (Loyal vs Disloyal) and travel type (Business vs Personal).

**Q2.** Average scores for all 14 service dimensions. Rank from best to worst.

**Q3.** Satisfaction rate by travel class (Business, Eco, Eco Plus). Which class has the highest satisfaction?

**Q4.** Delay analysis: avg departure and arrival delays. What % of flights had delays > 30 minutes? > 60 minutes?

**Q5.** Does delay length correlate with satisfaction? Group flights into delay buckets (0, 1-15, 16-45, 46-120, 120+ mins) and show satisfaction rate per bucket.

**Q6.** Flight distance vs satisfaction: group into short (<500 miles), medium (500-2000), long (>2000). Show satisfaction rate per group.

**Q7.** Age vs satisfaction: group passengers into age bands (Under 25, 25-40, 41-60, 60+). Show satisfaction rate per band and per class.

**Q8.** Service improvement priorities: for DISSATISFIED passengers only, calculate the avg score for each of the 14 service dimensions. The lowest-scoring dimensions for dissatisfied customers are the biggest improvement opportunities. Rank them.

**Q9.** Identify the "loyalty trap": loyal customers who are DISSATISFIED. How many? What are their top pain points (lowest service scores)? What class are they in?

**Q10.** Build a satisfaction driver analysis: for each service dimension, calculate the avg score for Satisfied vs Dissatisfied passengers. The dimensions with the biggest gap between satisfied and dissatisfied scores are the key satisfaction drivers. Rank by gap size.

**Q11.** Create a "Satisfaction Score" per passenger: average of all 14 service dimensions. Using this continuous score:
- What is the threshold (use NTILE analysis) below which passengers are mostly dissatisfied?
- Is there a bimodal distribution (two clusters) or a smooth gradient?
Show: score decile, avg service score, % satisfied.

**Q12.** Loyal vs disloyal customer differences: compare the avg scores on every dimension for loyal vs disloyal customers. Are loyal customers more tolerant of poor service? (Do they rate things higher despite similar experiences?)

**Q13.** Flight distance and delay interaction: does delay hurt satisfaction more on short vs long flights? Group by (distance_band × delay_band) and show satisfaction rate for each combination.

**Q14.** Gender and class interaction: is there a satisfaction difference by gender within each class? Build a pivot: gender as rows, class as columns, satisfaction rate as values. Are any gender-class combinations notably under-served?

**Q15.** Build an NPS-proxy model: classify each passenger as:
- Promoter: satisfaction = 'Satisfied' AND avg_service_score ≥ 4.0
- Passive: satisfaction = 'Satisfied' AND avg_service_score < 4.0
- Detractor: satisfaction = 'Neutral or Dissatisfied'
Calculate NPS = (% Promoters) - (% Detractors).
Then break NPS down by: class, customer_type, travel_type, age_band, delay_band.
Which segment has the best NPS? Which has the worst?

---
---

## Project 9 — RFM + Customer Lifetime Value Engine
**Dataset:** D1 — Olist | **Domain:** E-Commerce / CRM

### Business Context
The growth team needs a complete customer intelligence layer. Not just RFM scores — they want predicted CLV, win-back probability scores, and product affinity segments.

### Questions

**Q1.** Core RFM: for each customer, compute recency_days, frequency, monetary using the max order date as "today."

**Q2.** Score each on a 1-5 scale using NTILE(5). Remember: lower recency = higher R score.

**Q3.** Assign segments: Champions (R≥4, F≥4), Loyal (R≥3, F≥3), Potential (R≥3, F=2), New (R≥4, F=1), At Risk (R≤2, F≥3), Hibernating (R≤2, F≤2), Lost (R=1, F=1, M=1). Show segment counts and avg RFM per segment.

**Q4.** Segment by geographic state: which Brazilian states are richest in Champions? In Lost customers?

**Q5.** Segment vs review score: join to order_reviews. Do Champions leave better reviews? Show avg review score per RFM segment.

**Q6.** Segment vs product category: join to order_items + products. Which product categories drive Champions? Which are associated with At Risk customers?

**Q7.** Payment method preference by segment: do high-value customers prefer credit card? Show payment type distribution per segment.

**Q8.** Purchase interval analysis: for customers with 2+ orders, calculate the avg days between consecutive orders (use LAG). Do Champions buy more frequently (shorter intervals)?

**Q9.** Customer Lifetime Value calculation:
- Historical CLV = total revenue per customer
- For customers with 2+ orders: estimate future CLV = avg_order_value × avg_orders_per_month × 12 (annualised)
- Show both for each RFM segment's average

**Q10.** Win-back probability score: for At Risk and Lost customers, build a score:
- Had 3+ orders → +2
- Last avg review ≥ 4 → +2
- Historical CLV in top 50% → +2
- Last order was < 365 days ago → +1
- Product category = high-repeat category (one you identified in Q6) → +1
Sum as `winback_score`. How many have score ≥ 5?

**Q11.** Product affinity — first purchase predicts second: for customers with 2+ orders, what category was most often bought first? Among customers whose FIRST purchase was in category X, what % bought category X again, and what other categories did they buy next? Build a transition matrix (CASE WHEN pivot).

**Q12.** High-value customer journey: for Champions specifically, reconstruct their purchase sequence. Do they buy across many categories, or are they single-category loyalists?
- For each Champion: count distinct categories, total orders, time span (first to last order), avg days between orders.
- Classify: Deep Loyalist (1 category), Cross-Buyer (2-3 categories), Diversified (4+).

**Q13.** Customer health index (CHI): combine multiple signals into one score:
- Frequency score (F component of RFM): 0-5
- Review trend: avg of last 2 reviews vs first 2 reviews (improving = +2, flat = 0, declining = -1)
- Recency: 0 if R≥4, -2 if R≤2
- Spend trend: is last order value above or below customer's personal avg? (+1 / -1)
Show CHI distribution and avg CLV at each CHI level.

**Q14.** Seller-customer relationship quality: for each (seller, customer) pair with 2+ orders, calculate:
- Total orders together
- Total revenue
- Avg review score
- Avg days between reorders with same seller
Is there evidence of "relationship value" (customers who repeatedly buy from the same seller have higher LTV)?

**Q15.** Full CRM dashboard query: write a single multi-CTE query that outputs one row per customer with ALL of the following fields:
- customer_unique_id, customer_state
- total_orders, total_revenue, avg_order_value
- recency_days, r_score, f_score, m_score, rfm_segment
- predicted_annual_clv (from Q9)
- winback_score (for churned only, else NULL)
- last_review_score, avg_review_score
- favourite_category (highest revenue category for this customer)
- chi_score
- customer_tier: 'Platinum' (CLV top 10%), 'Gold' (top 10-30%), 'Silver' (top 30-60%), 'Standard' (rest)

---

## Project 10 — Revenue Time-Series Intelligence
**Dataset:** D2 — UK Retail II | **Domain:** Retail

### Business Context
Finance wants a complete time-series intelligence system: anomaly detection, seasonality quantification, trend isolation, and a forecasting baseline — all in SQL.

### Questions

**Q1.** Daily net revenue (sales minus returns). Fill in zero for missing days using a recursive date CTE.

**Q2.** 7-day and 30-day rolling averages. Add a rolling standard deviation:
- `rolling_7d_stddev = SQRT(AVG(rev²) - AVG(rev)² over 7 days)`

**Q3.** Anomaly detection: flag days where revenue deviates from rolling 7d average by more than 2 standard deviations. Show date, actual revenue, expected range (mean ± 2sd), and anomaly type (spike or trough).

**Q4.** Weekly aggregation: calculate weekly revenue and WoW % change. What is the coefficient of variation (stddev/mean) of weekly revenue? (Higher = more volatile)

**Q5.** Seasonality index: for each calendar month (Jan=1, Dec=12), calculate `avg_monthly_revenue / overall_monthly_avg`. This gives a seasonality multiplier. December = 1.4 means Dec is 40% above average.

**Q6.** Trend isolation: using a rolling 12-week average as the "trend," calculate the "de-trended" revenue for each week: `actual / rolling_12w_avg`. Values > 1 = above trend, < 1 = below. This separates seasonality from trend.

**Q7.** Revenue per customer per visit (known IDs only): calculate this for each month. Is the "value per visit" increasing (pricing up / bigger baskets) or decreasing (discounting / smaller orders)?

**Q8.** Product contribution stability: for the top 50 products by annual revenue, how stable is each product's monthly revenue? Calculate the coefficient of variation per product. Highly variable products are demand-volatile.

**Q9.** Revenue decomposition: group total revenue into:
- UK top-10 customers (by total spend)
- UK rest
- International
Show this split for each month. Is any group disproportionately driving volatility?

**Q10.** Detect "revenue cliffs": find months where UK revenue dropped more than 15% AND returned to growth within 2 months (V-shaped recovery). How many such events? What happened in the months before and after?

**Q11.** Hour-of-day revenue pattern: calculate avg revenue per hour bucket (0-3, 4-7, 8-11, 12-15, 16-19, 20-23). How much revenue comes from "off-hours" (before 9am and after 6pm)?

**Q12.** Return storm detection: find weeks where the return rate (returned revenue / gross revenue) exceeded 20%. List these weeks and describe what products drove the high return rate that week.

**Q13.** Long-range trend: calculate 3-month, 6-month, and 12-month rolling averages. When the 3-month average crosses above or below the 12-month average, it's a "signal" (like a moving average crossover in finance). Find all crossover dates.

**Q14.** Customer acquisition-revenue link: for each week, calculate both new customers (first purchase week) and total revenue. Is revenue growth driven by acquiring new customers or by existing customers spending more? Use LAG on both series to calculate correlation direction.

**Q15.** Full time-series summary report: write a single query with CTEs that produces a MONTHLY report with:
- Year, Month
- Gross sales, total returns, net revenue
- New customers (first purchase in this month), returning customers
- Revenue from new customers, revenue from returning customers
- MoM % change in net revenue
- Seasonality index for the month
- 3-month rolling avg
- Anomaly flag (1 if outside 1.5 stddev from 12-month mean)
- Running total revenue for the year

---

## Project 11 — Cohort Retention Deep Dive
**Dataset:** D1 — Olist | **Domain:** E-Commerce

### Business Context
Product leadership wants to understand retention at a granular level: not just are customers coming back, but when, why, and for how much.

### Questions

**Q1.** Basic cohort table: for each acquisition month, show retention rates at months 0-6.

**Q2.** Cohort size vs quality: does cohort size (customers acquired per month) predict retention? Plot acquisition volume against M1 retention.

**Q3.** Revenue retention: instead of counting customers, track revenue retention. For each cohort at each month number, show: active customers, revenue, revenue-per-active-customer.

**Q4.** Best and worst cohorts: rank cohorts by M1 retention. What distinguishes the best cohorts? (seasonal? higher first-order value? different product categories?)

**Q5.** Geographic retention: break cohort M1 retention by state. Which states have the best M1 retention?

**Q6.** Product-driven retention: customers whose first purchase was in category X — do they return at different rates? Compare M1 retention for customers whose first purchase was: electronics_computers vs beauty_health vs bed_bath_table.

**Q7.** Review-driven retention: customers who left a 5-star first review vs 1-star. What is the M1 retention rate for each review score?

**Q8.** Payment method and retention: do customers who pay with credit card (installments) return more often than boleto (cash) payers?

**Q9.** Seller quality and retention: customers who bought from high-rated sellers (avg review ≥ 4.5) vs low-rated sellers (avg review < 3.5) on their first purchase. How does this affect M1 and M3 retention?

**Q10.** Time-to-second-purchase: for customers who did make a second purchase, what was the distribution of days between purchase 1 and purchase 2? Use NTILE(4) to show quartiles. What is the "golden window" — the time period when most win-backs happen?

**Q11.** "Near-churn" detection: identify customers who are in their 4th-6th month post-acquisition and have NOT placed a second order yet. These are at risk of permanent churn. How many? What was their first-order value? What product did they buy?

**Q12.** Retention cliff analysis: find the month number where the steepest retention drop-off occurs. Use LAG to calculate the change in retention rate between consecutive months. Where does retention flatten out (stable loyal base)?

**Q13.** Seasonal cohort effects: customers acquired in December (holiday shoppers) vs acquired in other months — do holiday cohorts churn faster? Compare their M1-M6 retention curves.

**Q14.** Revenue impact of 1% retention improvement: for each cohort, calculate: if M1 retention had been 1% higher, how much additional revenue would have been generated? Sum this across all cohorts.

**Q15.** Full 12-month cohort pivot table with revenue retention: output a table with acquisition_month as rows and month_0 through month_11 as columns. Each cell shows BOTH customer retention % AND revenue retention %. Use CASE WHEN for the pivot structure.

---

## Project 12 — Fraud Deep Dive: Pattern Recognition
**Dataset:** D4 — Banking/Fraud | **Domain:** Banking

### Business Context
Fraud patterns are rarely random — they cluster in time, location, and behaviour. Your job is to surface the non-obvious patterns that rule-based systems miss.

### Questions

**Q1.** Fraud rate by every categorical dimension you have: merchant_category, transaction_type, card_type, day_of_week. Build all four in one query using UNION ALL with a `dimension_name` column.

**Q2.** Fraud amount percentiles: for fraudulent transactions, show P10, P25, P50, P75, P90, P95, P99 of transaction amounts using NTILE.

**Q3.** Fraud "rings": find groups of customers who all had fraud at the same merchant within the same 7-day window. These clusters could indicate a data breach.

**Q4.** Velocity anomalies: identify customers with 3+ transactions (any type) within any 1-hour window. What % of these "high-velocity" customers also have fraud in that window?

**Q5.** Sequential fraud patterns: for repeat-fraud customers, what is the typical time between their fraud events? Use LAG. Are second-fraud events more or less likely to be caught?

**Q6.** Merchant anomaly detection: for each merchant, calculate their avg transaction amount and standard deviation. Flag transactions where the amount exceeds merchant_avg + 3×merchant_stddev. What % of these outlier transactions are fraud?

**Q7.** The "first transaction" trap: is a customer's first-ever transaction more likely to be fraud? Calculate fraud rate for: first transaction (ROW_NUMBER = 1), second, third, and 4+. What pattern do you see?

**Q8.** Time-since-last-legitimate pattern: for fraudulent transactions, calculate the time since the customer's last legitimate transaction. Do fraudsters typically strike shortly after real activity (to blend in)?

**Q9.** Fraud escalation: among repeat-fraud customers, does the fraud amount tend to increase over time (escalate) or stay flat? Use LAG to compare each fraud event to the previous one for the same customer.

**Q10.** "Sleeper" accounts: find customers who had no transactions for 90+ days and then suddenly had a fraudulent transaction. These could indicate account takeover. How many such cases?

**Q11.** Merchant category switching: find customers who had fraud at a different merchant category than their usual spending. Build a profile of each customer's normal merchant category mix, then flag fraud transactions that are in a category accounting for less than 10% of their prior legitimate transactions.

**Q12.** Fraud by "account age": proxy account age as (date of first transaction to current fraud date). Do newer accounts have higher fraud rates?

**Q13.** Time-series fraud trend: monthly fraud rate with month-over-month change. Additionally: split by merchant_category — are some categories getting safer while others get worse?

**Q14.** Network detection: find pairs of customers who have BOTH had fraud at the SAME merchant on the SAME day. These pairs may share a compromised payment terminal. Use a self-join on (merchant_name, date, is_fraud=1).

**Q15.** Full fraud intelligence report: one query with multiple CTEs producing:
- Overall stats (rate, total loss)
- Top-5 riskiest merchant categories with lift score
- Top-10 riskiest customers with risk classification
- Monthly trend with anomaly flag
- Model validation: fraud_risk_score (from project 4 Q15) — show precision, recall, and F1-proxy at threshold ≥ 5

*(F1 = 2 × precision × recall / (precision + recall))*

---
---

## Project 13 — Supplier Performance & Profitability Scorecard
**Dataset:** D5 — Supply Chain | **Domain:** Manufacturing

### Business Context
The board wants a quarterly business review deck. Your SQL is the source of truth for every number in it.

### Questions

**Q1.** Department-level P&L: total revenue, total profit, profit margin %, total orders, avg order value.

**Q2.** QoQ (quarter-over-quarter) revenue growth per department.

**Q3.** Department × shipping mode performance matrix: late rate and avg profit for every combination.

**Q4.** Discount sensitivity: for each department, calculate: avg profit at 0 discount, 1-10%, 11-20%, >20%. At what discount level does profit go negative?

**Q5.** Product-level Pareto: rank products by revenue. What % of products generate 80% of revenue? Which products are in the "long tail" (bottom 50% of revenue, >80% of SKUs)?

**Q6.** Customer segment profitability: avg profit per order for Consumer vs Corporate vs Home Office. Which segment is most valuable (accounting for order volume)?

**Q7.** Regional profitability: avg profit margin by order_region. Are some regions systematically less profitable?

**Q8.** Shipping mode economics: for each shipping mode, calculate avg profit, avg late rate, and revenue contribution %. Build a simple 2×2 matrix: high/low profit × high/low late rate.

**Q9.** Profit trend: calculate monthly profit per department. Use LAG to find months where profit declined MoM. How many consecutive declining months is the worst case?

**Q10.** Loss leader analysis: products with negative avg profit but volume > 100 units. Total profit drain. Should the company stop stocking them?

**Q11.** Customer-level profitability: for customers with 10+ orders, rank by total profit contribution. What % of customers are responsible for 80% of total profit? (Customer Pareto)

**Q12.** Seasonality by department: for each department, calculate avg revenue per quarter (Q1-Q4 across all years). Which departments are most seasonal (highest quarterly variance)?

**Q13.** Supply chain efficiency score: for each department, create a composite score:
- Profit margin: 1-5 score (NTILE)
- On-time rate: 1-5 score (NTILE)
- Revenue growth (QoQ): 1-5 score (NTILE)
- Composite = (profit_score + ontime_score + growth_score) / 3
Rank departments. Which is your "star" department?

**Q14.** Detect profit-killing combinations: find the (product_category, shipping_mode, region, discount_band) combinations with the lowest avg profit. Are there specific combinations you'd advise eliminating?

**Q15.** Full QBR (Quarterly Business Review) report: one multi-CTE query producing one row per department per quarter with:
- Department, Quarter (e.g. '2017-Q4')
- Revenue, profit, margin %
- Orders, avg order value
- Late delivery rate %
- Best-selling category that quarter
- QoQ revenue change %
- QoQ profit change %
- Efficiency score (from Q13)
- Status: 🟢 Star / 🟡 Watch / 🔴 Alert

---

## Project 14 — Cross-Domain Intelligence Report
**Datasets:** D1 + D3 + D6 | **Domain:** Multi-domain

### Business Context
A consulting company serves clients across e-commerce, healthcare, and aviation. They want a unified analytics template. Your job is to prove the same SQL patterns apply across three completely different industries.

### Questions

**Q1.** "Primary KPI by entity" — one query per dataset:
- D1: total revenue per customer
- D3: total billing per patient (proxy by name)
- D6: avg satisfaction score per passenger
Show min, P25, median (NTILE), P75, max for each.

**Q2.** "Volume over time" — monthly counts for each dataset using the same CTE pattern:
- D1: orders per month
- D3: admissions per month
- D6: if no date, simulate monthly by ID range buckets
Show: source, month, count, MoM change %.

**Q3.** "Satisfaction/quality signal" — the key quality metric per entity:
- D1: avg review score per seller
- D3: % normal test results per doctor
- D6: % satisfied per (class, travel_type)
Who are the top and bottom 10 performers in each dataset?

**Q4.** "Segment by value" — quartile segmentation of primary KPI:
- D1: customer value quartiles (NTILE 4)
- D3: patient billing quartiles
- D6: satisfaction score quartiles
What characterises each quartile in each domain?

**Q5.** "Late/delayed/unsatisfied" — the failure metric:
- D1: late delivery rate by state
- D5: late delivery rate by region (use D5 data)
- D6: dissatisfied rate by class
For each, identify the worst performers and their magnitude.

**Q6.** "One-and-done" analysis (churn signal):
- D1: customers with exactly 1 order
- D3: patients with exactly 1 admission
- D7: employees who left within 1 year
What % of each population is "one-and-done"? What do they share?

**Q7.** UNION exercise: create a unified KPI summary:
```sql
SELECT 'Olist'       AS source, ... FROM olist_data
UNION ALL
SELECT 'Healthcare'  AS source, ... FROM healthcare_data
UNION ALL
SELECT 'Airline'     AS source, ... FROM airline_data
```
Output: source, total_records, primary_kpi_avg, primary_kpi_p75, top_entity_name, top_entity_value.

**Q8.** Anomaly detection across all three datasets: for each, calculate the mean and stddev of the primary KPI. Flag records more than 2 stddev above the mean. Show anomaly count and % per dataset.

**Q9.** Correlation-direction check (no math library needed): for each dataset, test whether there is a directional relationship between two variables:
- D1: does higher seller review → higher revenue? (Avg revenue per review bucket 1-5)
- D3: does higher billing → longer stay? (Avg billing per stay-length quartile)
- D6: does flight distance → higher satisfaction? (Avg satisfaction by distance band)

**Q10.** "Premium vs standard" comparison across datasets:
- D1: premium (credit_card installments > 1) vs standard (single payment) customers: revenue and retention
- D3: Emergency vs Elective admission: avg billing and avg stay
- D6: Business Class vs Economy: satisfaction rate and avg service score
Is premium/priority always more valuable/satisfied?

**Q11.** Predictive risk flag: using your domain-specific analysis, build a risk flag for each:
- D1: customer at churn risk (recency > 180d AND last review < 4)
- D3: patient high-cost risk (billing > 2× avg AND age > 65)
- D6: passenger NPS detractor (satisfaction = Dissatisfied AND service avg < 3)
Show count and % of population flagged in each domain.

**Q12.** Cross-domain timing analysis: for datasets with timestamps (D1, D3), find the busiest day of week and time of month (1st-7th, 8th-14th, etc.). Do e-commerce and healthcare have the same or different peak periods?

**Q13.** Value concentration: for each dataset, find what % of total primary KPI (revenue/billing/score) is driven by the top 10% of entities. Compare concentration across domains.

**Q14.** Build a "Business Report Card" per dataset using CASE WHEN to assign letter grades:
- D1: grade on avg review score, on-time delivery rate, revenue growth
- D3: grade on avg billing efficiency, test result accuracy (normal %), occupancy
- D6: grade on satisfaction rate, on-time rate, loyalty rate
Each metric: A (top 20%), B (20-40%), C (40-60%), D (60-80%), F (bottom 20%).

**Q15.** Master cross-domain capstone: one SQL script that produces a "portfolio summary" — one row per domain with 15 KPIs each. The final output should look like a consulting slide deck source table. Include: entity count, date range, primary KPI avg, top performer, bottom performer, satisfaction/quality rate, late/failure rate, churn/attrition rate, YoY growth estimate, and your SQL-generated 3-sentence business summary.

---

## Project 15 — Full Business Intelligence Capstone
**Datasets:** D1 + D4 + D5 | **Domain:** Multi-domain (Capstone)

### Business Context
You are presenting to the CEO of a holding company that owns: Olist marketplace (D1), a financial services arm (D4), and DataCo supply chain (D5). They have 45 minutes. Your SQL generates every slide.

### Deliverables

#### Section A — Headline KPIs (one query each)
**Q1.** Revenue: total from each business unit. Which is largest? What is the combined total?

**Q2.** Time coverage: date range (min to max date) for each unit. Are the periods comparable?

**Q3.** Customer/entity count: unique customers (D1), unique customers (D4), unique customers (D5).

**Q4.** YoY growth (if data spans 2 years): for each unit, YoY revenue change %.

#### Section B — Business Health
**Q5.** Olist: seller scorecard summary — avg on-time rate, avg review score, % of sellers with review < 3.

**Q6.** Banking: fraud rate, total fraud loss, top 3 fraud merchant categories.

**Q7.** Supply Chain: overall late delivery rate, worst department, worst shipping mode.

#### Section C — Risk Flags
**Q8.** Olist: high-risk sellers (on-time < 60% AND review < 3.5, min 50 orders). How many? Revenue impact?

**Q9.** Banking: high-risk customers (fraud rate > 10%, min 10 transactions). How many? Which merchant categories?

**Q10.** Supply Chain: departments with declining profit for 2+ consecutive quarters.

#### Section D — Growth Opportunities
**Q11.** Olist: which product categories have the best (revenue per order × avg review score) composite score? Top 3 categories to invest in.

**Q12.** Banking: which merchant categories have < 0.5% fraud rate AND high transaction volume? These are "safe growth" categories.

**Q13.** Supply Chain: which (department × shipping mode) combination has best profit AND best on-time rate? Where should ops invest?

#### Section E — Integrated View
**Q14.** Build the CEO one-pager: a UNION ALL query combining one row per business unit with columns:
- unit_name, date_range, total_revenue, entity_count, growth_pct, quality_score, risk_score, top_opportunity
*(quality and risk expressed as 1-10 based on the metrics above)*

**Q15.** Auto-narrative: use string concatenation to generate a 3-sentence SQL-generated summary for each business unit:
```sql
SELECT unit_name,
    'In ' || date_range || ', ' || unit_name || ' generated $' || ROUND(revenue/1000000.0,1) || 'M revenue across '
    || entity_count || ' customers. ' ||
    'Top risk: ' || top_risk_flag || '. ' ||
    'Best opportunity: invest in ' || top_opportunity || ' which shows ' || opportunity_metric || '.'
    AS executive_summary
FROM summary_cte;
```

---

## Answer Key

*Attempt all questions independently first. Selected key answers below.*

---

### Project 1 Key Answers

```sql
-- Q10: Seller scorecard with DENSE_RANK
WITH seller_metrics AS (
    SELECT
        oi.seller_id,
        COUNT(DISTINCT oi.order_id)                                   AS total_orders,
        ROUND(SUM(oi.price), 2)                                       AS total_revenue,
        ROUND(AVG(r.review_score), 2)                                 AS avg_review,
        ROUND(100.0 * SUM(CASE WHEN o.order_delivered_timestamp <= o.order_estimated_delivery
                                AND o.order_delivered_timestamp IS NOT NULL THEN 1 ELSE 0 END)
              / COUNT(DISTINCT oi.order_id), 1)                       AS ontime_pct
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.order_id
    LEFT JOIN order_reviews r ON oi.order_id = r.order_id
    GROUP BY oi.seller_id
)
SELECT *,
       DENSE_RANK() OVER (ORDER BY total_revenue DESC) AS revenue_rank
FROM seller_metrics
ORDER BY revenue_rank
LIMIT 20;

-- Q11: Problem sellers
WITH seller_metrics AS ( ... same CTE as Q10 ... )
SELECT sm.*, s.seller_city, s.seller_state,
       GROUP_CONCAT(DISTINCT p.product_category_name) AS categories_sold
FROM seller_metrics sm
JOIN sellers s ON sm.seller_id = s.seller_id
JOIN order_items oi ON sm.seller_id = oi.seller_id
JOIN products p ON oi.product_id = p.product_id
WHERE sm.ontime_pct < 60
  AND sm.avg_review < 3.5
  AND sm.total_orders >= 50
GROUP BY sm.seller_id;

-- Q15: Monthly cohort pivot
WITH cohorts AS (
    SELECT c.customer_unique_id,
           DATE(MIN(o.order_purchase_timestamp), 'start of month') AS cohort_month
    FROM customers c JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_unique_id
),
activity AS (
    SELECT c.customer_unique_id,
           DATE(o.order_purchase_timestamp, 'start of month') AS active_month
    FROM customers c JOIN orders o ON c.customer_id = o.customer_id
),
base AS (
    SELECT a.customer_unique_id, c.cohort_month, a.active_month,
           CAST(ROUND((JULIANDAY(a.active_month) - JULIANDAY(c.cohort_month)) / 30) AS INT) AS mnum
    FROM cohorts c JOIN activity a ON c.customer_unique_id = a.customer_unique_id
),
sizes AS (SELECT cohort_month, COUNT(DISTINCT customer_unique_id) AS sz FROM base WHERE mnum=0 GROUP BY cohort_month)
SELECT b.cohort_month,
       sz.sz AS cohort_size,
       ROUND(100.0 * COUNT(DISTINCT CASE WHEN mnum=0 THEN b.customer_unique_id END)/sz.sz,1) AS m0,
       ROUND(100.0 * COUNT(DISTINCT CASE WHEN mnum=1 THEN b.customer_unique_id END)/sz.sz,1) AS m1,
       ROUND(100.0 * COUNT(DISTINCT CASE WHEN mnum=2 THEN b.customer_unique_id END)/sz.sz,1) AS m2,
       ROUND(100.0 * COUNT(DISTINCT CASE WHEN mnum=3 THEN b.customer_unique_id END)/sz.sz,1) AS m3,
       CASE WHEN ROUND(100.0 * COUNT(DISTINCT CASE WHEN mnum=1 THEN b.customer_unique_id END)/sz.sz,1) < 5
            THEN '⚠️ Low retention' ELSE '✅ OK' END AS retention_flag
FROM base b JOIN sizes sz ON b.cohort_month = sz.cohort_month
GROUP BY b.cohort_month, sz.sz
ORDER BY b.cohort_month;
```

---

### Project 4 Key Answers — Fraud Rule Engine

```sql
-- Q15: Fraud scoring and validation
WITH customer_prior_fraud AS (
    SELECT customer_id,
           date,
           SUM(is_fraud) OVER (
               PARTITION BY customer_id
               ORDER BY date
               ROWS BETWEEN 30 PRECEDING AND 1 PRECEDING
           ) AS fraud_in_last_30d
    FROM transactions
),
customer_avg AS (
    SELECT customer_id, AVG(amount) AS avg_amount
    FROM transactions WHERE is_fraud = 0
    GROUP BY customer_id
),
high_risk_categories AS (
    SELECT merchant_category
    FROM (
        SELECT merchant_category,
               ROUND(100.0 * SUM(is_fraud) / COUNT(*), 2) AS fraud_rate
        FROM transactions GROUP BY merchant_category
        ORDER BY fraud_rate DESC LIMIT 5
    )
),
scored AS (
    SELECT t.*,
           (CASE WHEN t.merchant_category IN (SELECT merchant_category FROM high_risk_categories) THEN 3 ELSE 0 END
          + CASE WHEN CAST(STRFTIME('%H', t.date) AS INT) BETWEEN 23 AND 23
                   OR CAST(STRFTIME('%H', t.date) AS INT) BETWEEN 0 AND 4 THEN 2 ELSE 0 END
          + CASE WHEN t.transaction_type = 'online' THEN 1 ELSE 0 END
          + CASE WHEN t.amount > ca.avg_amount * 3 THEN 2 ELSE 0 END
          + CASE WHEN cpf.fraud_in_last_30d > 0 THEN 3 ELSE 0 END
           ) AS fraud_risk_score
    FROM transactions t
    LEFT JOIN customer_avg ca ON t.customer_id = ca.customer_id
    LEFT JOIN customer_prior_fraud cpf ON t.transaction_id = cpf.transaction_id  -- simplified
)
SELECT
    CASE WHEN fraud_risk_score >= 7 THEN 'HIGH (≥7)'
         WHEN fraud_risk_score >= 4 THEN 'MEDIUM (4-6)'
         ELSE 'LOW (≤3)' END AS risk_tier,
    COUNT(*) AS total_transactions,
    SUM(is_fraud) AS actual_fraud,
    ROUND(100.0 * SUM(is_fraud) / COUNT(*), 2) AS actual_fraud_rate,
    -- Precision for HIGH tier: % of flagged-high that are fraud
    ROUND(100.0 * SUM(CASE WHEN fraud_risk_score >= 7 THEN is_fraud END)
          / NULLIF(SUM(CASE WHEN fraud_risk_score >= 7 THEN 1 END), 0), 2) AS precision_at_7
FROM scored
GROUP BY risk_tier;
```

---

### Project 9 Key Answers — Full CRM Dashboard

```sql
-- Q15: Full CRM dashboard query
WITH orders_base AS (
    SELECT c.customer_unique_id, c.customer_state,
           COUNT(DISTINCT o.order_id)      AS total_orders,
           SUM(p.payment_value)            AS total_revenue,
           AVG(p.payment_value)            AS avg_order_value,
           MAX(o.order_purchase_timestamp) AS last_order
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_payments p ON o.order_id = p.order_id
    GROUP BY c.customer_unique_id, c.customer_state
),
max_date AS (SELECT MAX(order_purchase_timestamp) AS md FROM orders),
rfm AS (
    SELECT customer_unique_id,
           CAST(JULIANDAY((SELECT md FROM max_date)) - JULIANDAY(last_order) AS INT) AS recency_days,
           total_orders AS frequency, total_revenue AS monetary,
           NTILE(5) OVER (ORDER BY JULIANDAY((SELECT md FROM max_date)) - JULIANDAY(last_order)) AS r_score,
           NTILE(5) OVER (ORDER BY total_orders)  AS f_score,
           NTILE(5) OVER (ORDER BY total_revenue) AS m_score
    FROM orders_base
),
reviews AS (
    SELECT c.customer_unique_id,
           ROUND(AVG(r.review_score), 2)  AS avg_review,
           (SELECT review_score FROM order_reviews r2
            JOIN orders o2 ON r2.order_id = o2.order_id
            JOIN customers c2 ON o2.customer_id = c2.customer_id
            WHERE c2.customer_unique_id = c.customer_unique_id
            ORDER BY r2.review_creation_date DESC LIMIT 1) AS last_review
    FROM customers c
    JOIN orders o ON c.customer_id = o.customer_id
    JOIN order_reviews r ON o.order_id = r.order_id
    GROUP BY c.customer_unique_id
),
fav_category AS (
    SELECT c.customer_unique_id,
           p.product_category_name AS fav_cat
    FROM (
        SELECT c.customer_unique_id, p.product_category_name,
               SUM(oi.price) AS cat_rev,
               ROW_NUMBER() OVER (PARTITION BY c.customer_unique_id ORDER BY SUM(oi.price) DESC) AS rn
        FROM customers c
        JOIN orders o ON c.customer_id = o.customer_id
        JOIN order_items oi ON o.order_id = oi.order_id
        JOIN products p ON oi.product_id = p.product_id
        GROUP BY c.customer_unique_id, p.product_category_name
    ) x JOIN customers c ON x.customer_unique_id = c.customer_unique_id
    WHERE rn = 1
)
SELECT
    ob.customer_unique_id, ob.customer_state,
    ob.total_orders, ROUND(ob.total_revenue, 2) AS total_revenue,
    ROUND(ob.avg_order_value, 2),
    rfm.recency_days, rfm.r_score, rfm.f_score, rfm.m_score,
    CASE
        WHEN rfm.r_score >= 4 AND rfm.f_score >= 4 THEN 'Champions'
        WHEN rfm.r_score >= 3 AND rfm.f_score >= 3 THEN 'Loyal'
        WHEN rfm.r_score >= 4 AND rfm.f_score = 1  THEN 'New'
        WHEN rfm.r_score <= 2 AND rfm.f_score >= 3 THEN 'At Risk'
        ELSE 'Standard'
    END AS rfm_segment,
    rv.avg_review, rv.last_review,
    fc.fav_cat AS favourite_category,
    NTILE(10) OVER (ORDER BY ob.total_revenue DESC) AS clv_decile,
    CASE
        WHEN NTILE(10) OVER (ORDER BY ob.total_revenue DESC) = 1  THEN 'Platinum'
        WHEN NTILE(10) OVER (ORDER BY ob.total_revenue DESC) <= 3 THEN 'Gold'
        WHEN NTILE(10) OVER (ORDER BY ob.total_revenue DESC) <= 6 THEN 'Silver'
        ELSE 'Standard'
    END AS customer_tier
FROM orders_base ob
JOIN rfm ON ob.customer_unique_id = rfm.customer_unique_id
LEFT JOIN reviews rv ON ob.customer_unique_id = rv.customer_unique_id
LEFT JOIN fav_category fc ON ob.customer_unique_id = fc.customer_unique_id;
```

---

*End of SQL Projects v2*
*Chat #2 · June 2026*
