# Data Analytics 101 — A Complete Tutorial

> **How to use this document**
> Read end to end. Every section builds on the previous one.
> All code runs with Python 3.8+ and standard analytics libraries.
> No API keys, no cloud accounts, no paid tools needed for any core example.
> Concept examples use 🎬 **Movies** throughout — same as SQL 101, Python 101, and DE 101.
> Practice examples use the 7 Kaggle datasets from `da_projects.md`.
> Every chapter asks the same question: **"What decision does this analysis enable?"**
>
> **Chat #4 · June 2026**

---

## Table of Contents

### Part I — What Is Data Analytics?
1. [Analytics vs Data Engineering vs Data Science — What Each Role Does](#1-analytics-vs-de-vs-ds)
2. [The Analytics Workflow — From Question to Recommendation](#2-the-analytics-workflow)
3. [Types of Analytics — Descriptive, Diagnostic, Predictive, Prescriptive](#3-types-of-analytics)
4. [What Good Analysis Actually Looks Like](#4-what-good-analysis-looks-like)

### Part II — Problem Framing
5. [Translating a Business Request into an Analytical Question](#5-translating-business-requests)
6. [The 5 Whys and Issue Trees — Structuring the Problem](#6-5-whys-and-issue-trees)
7. [MECE Thinking — Exhaustive and Non-Overlapping](#7-mece-thinking)
8. [Scoping an Analysis — What to Include and What to Leave Out](#8-scoping-an-analysis)

### Part III — Business Context and Frameworks
9. [How Businesses Think About Data — The Analyst's Mental Model](#9-business-mental-model)
10. [Common Business Problems by Domain](#10-business-problems-by-domain)
11. [North Star Metrics, OKRs, and AARRR](#11-north-star-okrs-aarrr)
12. [Industry KPI Reference — E-Commerce, Healthcare, Supply Chain, HR, Finance](#12-industry-kpis)

### Part IV — Exploratory Data Analysis
13. [The EDA Framework — A Repeatable Process](#13-eda-framework)
14. [Univariate Analysis — Understanding One Variable at a Time](#14-univariate-analysis)
15. [Bivariate Analysis — Relationships Between Variables](#15-bivariate-analysis)
16. [Multivariate Analysis — The Full Picture](#16-multivariate-analysis)
17. [Outlier Detection and Missing Data Patterns](#17-outliers-and-missing-data)

### Part V — Business Metrics in Depth
18. [Revenue Metrics — GMV, ARR, MRR, NRR, AOV](#18-revenue-metrics)
19. [Customer Metrics — CAC, LTV, Churn, NPS, DAU/MAU](#19-customer-metrics)
20. [Operational Metrics — SLAs, Cycle Time, Throughput](#20-operational-metrics)
21. [Cohort Analysis — Measuring Behaviour Over Time](#21-cohort-analysis)
22. [RFM Analysis — Recency, Frequency, Monetary](#22-rfm-analysis)

### Part VI — SQL for Analytics
23. [Window Functions for Analytics — The Full Reference](#23-window-functions)
24. [Cohort Queries and Retention Analysis in SQL](#24-cohort-sql)
25. [Funnel Analysis and Conversion in SQL](#25-funnel-sql)
26. [Period-over-Period and Running Totals in SQL](#26-period-over-period-sql)

### Part VII — Python for Analytics
27. [pandas for Analysis — Beyond Cleaning](#27-pandas-for-analysis)
28. [Matplotlib and Seaborn — Exploratory Visualisation](#28-matplotlib-seaborn)
29. [Plotly — Interactive Charts](#29-plotly)
30. [The Python Analytics Workflow — Putting It Together](#30-python-analytics-workflow)

### Part VIII — Statistical Thinking
31. [Distributions That Matter — A Practical Guide](#31-distributions)
32. [Hypothesis Testing in Plain English](#32-hypothesis-testing)
33. [A/B Testing — From Question to Decision](#33-ab-testing)
34. [Correlation, Causation, and When to Be Careful](#34-correlation-causation)

### Part IX — Data Storytelling
35. [Chart Selection — Choosing the Right Visual for Your Message](#35-chart-selection)
36. [The Pyramid Principle — Structure Before Slides](#36-pyramid-principle)
37. [SCR Framework — Situation, Complication, Resolution](#37-scr-framework)
38. [Communicating to Non-Technical Stakeholders](#38-communicating-findings)

### Part X — BI Tools
39. [Tableau — How It Works, Key Concepts, When to Use](#39-tableau)
40. [Power BI — Data Model, DAX Basics, vs Tableau](#40-power-bi)
41. [Looker and LookML — The Semantic Layer](#41-looker)
42. [BI Tool Decision Framework — Which to Use When](#42-bi-decision-framework)

### Part XI — Analytics in Practice
43. [End-to-End Analysis Walkthrough — Olist](#43-end-to-end-walkthrough)
44. [Root Cause Analysis — A Structured Workflow](#44-root-cause-analysis)
45. [From Analysis to Recommendation — The Last Mile](#45-analysis-to-recommendation)

### Appendix
- [Analytics Tool Stack Reference](#appendix-a-tool-stack)
- [Python Libraries Quick Reference](#appendix-b-python-libraries)
- [Metrics Glossary — 100+ Terms](#appendix-c-metrics-glossary)
- [SQL Analytics Patterns Cheatsheet](#appendix-d-sql-cheatsheet)

---
---

## 1. Analytics vs Data Engineering vs Data Science

### The confusion — and why it matters

These three roles are frequently confused, misused in job postings, and blurred in practice. The confusion costs organisations money (wrong hire) and costs practitioners career direction (wrong path). Here is a clear distinction.

🎬 **Movies analogy:** Making a film requires three distinct roles. The **production crew** (data engineers) builds the set, installs lighting, manages equipment — the infrastructure that makes everything else possible. The **director** (data analyst) works with what the crew has built, interprets the script, and produces the film — making meaning from available material. The **screenwriter** (data scientist) creates original narratives, often inventing new approaches — building models and frameworks that didn't exist before. All three are essential. None replaces the others.

---

### Role definitions — precise and practical

**Data Engineer:**
Builds and maintains the infrastructure that makes data usable. Responsible for pipelines, data warehouses, data quality, and the reliability of data systems. Output: clean, reliable, timely data that others can use. Primary question: *"Is the data correct, fresh, and accessible?"*

**Data Analyst:**
Interprets data to answer business questions and support decisions. Works with data that engineers have prepared. Translates between business language and data language. Output: insights, reports, dashboards, recommendations. Primary question: *"What happened, why did it happen, and what should we do about it?"*

**Data Scientist:**
Builds statistical models and machine learning systems to make predictions and automate decisions. Requires statistical and programming depth beyond most analysts. Output: models, predictions, experiments. Primary question: *"What will happen, and how can we automate a decision based on data?"*

---

### The overlap — and where it gets blurry

In practice, roles overlap especially in smaller organisations. An analyst at a startup may write their own pipelines. A data engineer at a mature company may build analytical models. A data scientist spends most of their time doing what analysts do.

| Task | DE | DA | DS |
|------|----|----|-----|
| Build ETL pipeline | ✅ Primary | ❌ | Occasionally |
| Write SQL reports | Occasionally | ✅ Primary | ✅ |
| Create dashboards | ❌ | ✅ Primary | Occasionally |
| Statistical testing | ❌ | ✅ | ✅ Primary |
| Build ML models | ❌ | Rarely | ✅ Primary |
| Define business metrics | Occasionally | ✅ Primary | ✅ |
| Data quality monitoring | ✅ Primary | Occasionally | ❌ |
| Interpret results for stakeholders | ❌ | ✅ Primary | Occasionally |

**The analytics engineer** is a newer hybrid role sitting between DE and DA: deeply SQL-proficient, builds dbt models, defines metrics in code, doesn't write business reports but makes it easy for analysts to do so.

---

### What analysts actually do — day to day

A common misconception is that analytics is primarily about charts and dashboards. The reality:

```
Typical analyst week breakdown (mature analytics team):

30% — Understanding the question
       ("What are we actually trying to learn?")
       Meetings, stakeholder conversations, scoping

25% — Data wrangling
       SQL queries, pandas cleaning, joining sources
       ("Why does this column have 40% nulls?")

20% — Analysis
       The actual computation, summarisation, visualisation

15% — Sanity checking
       ("Does this number make sense? Let me verify three ways")

10% — Communicating results
       Writing up findings, building the chart that tells the story
```

The 30% on understanding the question is the most important and most underestimated. An analyst who spends 5 minutes on the question and 3 days on code has inverted the priorities.

---

## 2. The Analytics Workflow

### The end-to-end process — from question to recommendation

Every good analysis follows the same basic arc. The steps are not always linear — you often loop back — but the sequence matters.

```
┌─────────────────────────────────────────────────────────────────────┐
│                    THE ANALYTICS WORKFLOW                            │
│                                                                      │
│  1. RECEIVE REQUEST         "Revenue is down. Find out why."        │
│          │                                                           │
│          ▼                                                           │
│  2. FRAME THE QUESTION      "Is it volume or price? Which segment?  │
│          │                   Which time period? vs what baseline?"  │
│          ▼                                                           │
│  3. IDENTIFY DATA           "I need orders, products, customers.    │
│          │                   Do I have that? Is it trustworthy?"    │
│          ▼                                                           │
│  4. EXPLORE (EDA)           "What does this data actually look like?│
│          │                   Distributions, nulls, anomalies."      │
│          ▼                                                           │
│  5. ANALYSE                 "Apply metrics, aggregations, stats,    │
│          │                   segmentation, time series."            │
│          ▼                                                           │
│  6. VALIDATE                "Does this make sense? Can I explain    │
│          │                   every number? Does it reconcile?"      │
│          ▼                                                           │
│  7. INTERPRET               "What does this mean for the business?" │
│          │                   Not "revenue fell 12%" but             │
│          │                   "Weekend orders dropped 40% in São     │
│          │                    Paulo after the UI change on Feb 3"   │
│          ▼                                                           │
│  8. RECOMMEND               "Based on this, here is what I suggest  │
│                              we do, with explicit trade-offs."      │
└─────────────────────────────────────────────────────────────────────┘
```

**The most common mistake:** jumping from Step 1 directly to Step 5. A business says "we want to understand our customers better" and the analyst opens a Jupyter notebook and starts groupby-ing. Two days later they have 40 charts that don't answer any specific question and can't be acted on.

**The most undervalued step:** Step 6 (validate). Always try to verify a key finding a second way. If SQL says revenue is R$1.2M last month, check: does the finance team's number match? Does the row count seem right? Does the sum of a different cut add to the same total?

---

### What a good analytical question looks like

Vague requests are the norm. Your job is to make them precise.

| Vague request | Precise analytical question |
|---------------|---------------------------|
| "How are our customers doing?" | "What is the 90-day retention rate by acquisition cohort for customers acquired in 2018, and how has it changed quarter over quarter?" |
| "Is the new feature working?" | "Did users who used Feature X in the first 7 days after launch have a higher 30-day retention rate than a matched control group who didn't?" |
| "Revenue is down" | "In which product categories, customer segments, and geographies did revenue decline relative to the same period last year, and when did the decline begin?" |
| "Tell me about our sellers" | "What is the distribution of sellers by revenue tier, and what attributes (location, product category, review score) predict which tier a new seller will end up in?" |

The precision comes from specifying: **metric** (what are we measuring), **dimension** (by what grouping), **time period** (over what window), **comparison** (vs what baseline), and **decision** (what will we do with this).

---

## 3. Types of Analytics

### The four types — with real examples

Analytics is often described as a hierarchy. Each type builds on the previous and adds more value — but also requires more sophistication.

```
                                        VALUE
                              ▲
PRESCRIPTIVE    "What should   │  ██████████████████████████  Highest
                we do?"        │  (optimisation, simulation,
                               │   automated decision-making)
PREDICTIVE      "What will     │  █████████████████
                happen?"       │  (forecasting, classification,
                               │   propensity scores)
DIAGNOSTIC      "Why did it    │  ████████████
                happen?"       │  (drill-down, root cause,
                               │   segmentation)
DESCRIPTIVE     "What          │  ████████
                happened?"     │  (reports, dashboards,
                               │   summaries)
                               └────────────────────────────▶
                                                          COMPLEXITY
```

---

**Descriptive analytics** — what happened

The foundation. Counts, sums, averages, distributions, trends over time. Most dashboards are descriptive.

🎬 *Movie box office report: "Inception grossed $836M globally. North America: $292M. International: $544M. Opening weekend: $62M."* These are facts. No interpretation yet.

```python
import pandas as pd

orders = pd.read_csv("data/olist/olist_orders_dataset.csv",
                     parse_dates=["order_purchase_timestamp"])

# Descriptive: what happened
monthly_orders = (orders
    .set_index("order_purchase_timestamp")
    .resample("M")["order_id"]
    .count()
    .reset_index(name="orders"))

print(monthly_orders.tail(6))
# This tells you what happened. It doesn't tell you why or what to do.
```

---

**Diagnostic analytics** — why did it happen

Drill-down, segmentation, root cause analysis. You've seen that revenue dropped — now you find out which product, which region, which customer segment, when exactly it started.

🎬 *"Inception underperformed in China. Why? Subtitles were delayed 2 weeks after opening weekend. Competing local release the same weekend. Fewer IMAX screens than projected."* Now you know why.

```python
# Diagnostic: why did orders drop in August 2018?
aug_drop = orders[
    orders["order_purchase_timestamp"].dt.to_period("M") == "2018-08"
]

# Break it down by state
by_state = (aug_drop
    .merge(pd.read_csv("data/olist/olist_customers_dataset.csv"),
           on="customer_id")
    .groupby("customer_state")["order_id"]
    .count()
    .sort_values(ascending=False))

# Compare to prior month to find the biggest drops
# → diagnostic: São Paulo dropped 34%, Rio dropped 28%, but Minas was flat
# → next question: what happened in SP and RJ specifically?
```

---

**Predictive analytics** — what will happen

Models that forecast future values or classify future events. Requires statistical or ML techniques.

🎬 *"Based on pre-sales, social media sentiment, and director track record, we predict Dune Part Two will gross $280M opening weekend globally — ±$40M."*

```python
from sklearn.linear_model import LinearRegression
import numpy as np

# Predictive: forecast next month's orders
# (simplified — real forecasting uses proper time series models)
monthly_orders["month_num"] = range(len(monthly_orders))
X = monthly_orders[["month_num"]]
y = monthly_orders["orders"]

model = LinearRegression().fit(X, y)
next_month_pred = model.predict([[len(monthly_orders)]])[0]
print(f"Predicted orders next month: {next_month_pred:,.0f}")
# This is a prediction. It will be wrong by some amount.
# The analyst's job is to communicate the uncertainty honestly.
```

---

**Prescriptive analytics** — what should we do

Takes prediction further: given the forecast and business constraints, what is the optimal action? Simulation, optimisation, automated decision systems.

🎬 *"To maximise global box office, release in North America 2 weeks before international markets, open on a Thursday, allocate 40% of marketing budget to digital, and avoid the same weekend as any superhero film."*

In practice, prescriptive analytics for business analysts means: given the analysis, making a clear recommendation with explicit trade-offs. Not just "here is what happened" but "here is what I recommend and why."

---

### Which type are you doing? — how to check

Before starting any analysis, ask yourself:

| Question | Type you're doing |
|----------|------------------|
| "How many / how much / what is the trend?" | Descriptive |
| "Why did this happen? What drove the change?" | Diagnostic |
| "What will happen next quarter?" | Predictive |
| "What should we do? What is the optimal action?" | Prescriptive |

Most analyst work is descriptive + diagnostic. That's not a limitation — done well, it creates enormous value.

---

## 4. What Good Analysis Actually Looks Like

### The difference between data and insight

Data is a number. Insight is what that number means and what you should do about it.

| Data (weak) | Insight (strong) |
|-------------|-----------------|
| "Revenue was R$1.2M last month" | "Revenue grew 18% MoM, driven entirely by São Paulo sellers — all other states were flat or declining" |
| "Late delivery rate is 8%" | "Late delivery rate increased from 4% to 8% between Q3 and Q4, concentrated in the electronics category and correlated with the new logistics partner onboarded in October" |
| "Average review score is 4.1" | "Review scores are bimodal — customers who received on time give 4.8 average; customers who received late give 2.9 average. The overall 4.1 masks two completely different experiences" |

🎬 **Movies analogy:** Data is raw footage. Insight is the edited scene that makes the audience feel something. The same footage can be cut into a comedy or a tragedy depending on how it's assembled and what context surrounds it. The analyst's job is the edit — creating meaning from footage.

---

### The five properties of good analysis

**1. Answers a specific question**
Every analysis should start with a question that could, in principle, have a wrong answer. "Tell me about revenue" is not a question. "Did revenue grow faster in new customer acquisition or existing customer retention last quarter?" is a question.

**2. Is reproducible**
Anyone with the same data and your code should get the same result. This means: no manual Excel adjustments, no undocumented filters, no "I just know this column has bad data."

**3. Has been validated**
Every key number has been checked at least twice — once with a second method, once with a sense check. "Does this number feel right? Is it consistent with what the business knows?"

**4. Communicates uncertainty honestly**
"Revenue will be R$1.5M next month" is a bad analysis. "Revenue is likely between R$1.3M and R$1.7M next month, based on the last 6 months of trend" is honest. All forecasts are wrong. Hiding the uncertainty doesn't make the forecast more accurate — it just makes the analyst less trusted when it misses.

**5. Leads to a decision**
The strongest test of any analysis: *what decision will be made differently because of this?* If the answer is "I don't know" or "none", the analysis may have been the wrong one to do.

---

### The analyst's mindset — five habits

**Be sceptical of your own results.** When you find something surprising, your first instinct should be "I've made a mistake" not "I've found something interesting." Most surprising findings are data quality issues or analysis errors. Verify before sharing.

**Think in comparisons, not absolutes.** "Revenue is R$1.2M" means nothing without context. R$1.2M vs what? Last month? Last year? Budget? Industry benchmark? An absolute number is always incomplete.

**Segment before concluding.** Averages hide distributions. A 4.1 average review score could mean everyone is mildly satisfied, or it could mean 80% love the product and 20% hate it. Always look at the distribution before using the mean.

**Name the decision.** Before writing a single line of code, write one sentence: "This analysis will help the team decide whether to [X]." If you can't finish that sentence, you don't have enough clarity to begin.

**Communicate early and often.** Don't disappear for a week and return with 40 slides. Share your approach after framing the question. Share a rough draft after the first pass. Surprises at presentation = trust lost.

---
---

# Part II — Problem Framing

## 5. Translating a Business Request into an Analytical Question

### Why framing is the hardest part of the job

Most analysts are trained to analyse. Very few are trained to frame. But the quality of your framing determines the quality of your analysis before you touch any data.

A poorly framed question leads to:
- Analysis that answers the wrong thing
- Weeks of work that doesn't get used
- Stakeholders who nod politely and change nothing
- You getting blamed when the business doesn't improve

A well-framed question leads to:
- Analysis that directly informs a decision
- Stakeholders who say "this is exactly what I needed"
- You becoming the person everyone wants in the room

🎬 **Movies analogy:** A director who starts filming before the script is finished ends up with beautiful shots that don't cut together into a coherent film. The framing (the script) is the work. The analysis (the filming) is just execution of the frame.

---

### The framing template — five questions to answer before starting

Before writing any SQL or opening any dataset, answer these five questions in writing:

```
ANALYTICAL QUESTION FRAMING TEMPLATE

1. BUSINESS CONTEXT
   What is happening in the business right now that prompted this request?
   What decision is being considered?

2. THE PRECISE QUESTION
   What specific question, if answered, would directly inform that decision?
   State it as a question that could have a wrong answer.

3. METRICS AND DIMENSIONS
   What will I measure? (the metric)
   By what groupings will I cut it? (dimensions)
   Over what time period?
   Compared to what baseline?

4. SUCCESS CRITERIA
   What result would cause us to take Action A?
   What result would cause us to take Action B?
   (If there's only one possible action regardless of the answer,
    don't do the analysis — the decision is already made)

5. DATA AVAILABLE
   What data do I need?
   Do I have it?
   Is it trustworthy for this purpose?
   What are the known limitations?
```

---

### Worked example — Olist revenue decline

**The vague request:**
> *"Revenue has been declining. We need to understand what's happening."*

A junior analyst opens a Jupyter notebook. A senior analyst fills in the template.

**Applying the template:**

```
1. BUSINESS CONTEXT
   Olist's monthly revenue has been declining for the past 3 months.
   The operations team is considering whether to invest in seller acquisition
   or double down on improving existing seller quality.
   The decision needs to be made by end of quarter.

2. THE PRECISE QUESTION
   Is the revenue decline driven by:
   (a) fewer orders (volume problem), or
   (b) lower revenue per order (value problem)?
   And within whichever it is: which seller categories, customer regions,
   or product types are driving the decline?

3. METRICS AND DIMENSIONS
   Metric:     Total revenue (R$), order count, average order value
   Dimensions: Month, seller state, product category, customer region
   Period:     Last 6 months vs same 6 months prior year
   Baseline:   Same month prior year (seasonal adjustment)

4. SUCCESS CRITERIA
   If it's a volume problem (fewer orders):
     → Invest in seller acquisition / marketing
   If it's a value problem (lower AOV):
     → Investigate pricing, product mix, discount behaviour
   If it's concentrated in a specific region or category:
     → Targeted intervention, not a general program

5. DATA AVAILABLE
   Need: olist_orders, olist_order_payments, olist_order_items,
         olist_products, olist_customers
   Known limitation: we don't have cancelled order reasons
   Known limitation: product category translations may be incomplete
```

Now we have a real question. The analysis almost writes itself.

```python
import pandas as pd
import numpy as np

# Load data
orders   = pd.read_csv("data/olist/olist_orders_dataset.csv",
                       parse_dates=["order_purchase_timestamp"])
payments = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
items    = pd.read_csv("data/olist/olist_order_items_dataset.csv")
products = pd.read_csv("data/olist/olist_products_dataset.csv")
category = pd.read_csv("data/olist/product_category_name_translation.csv")
customers= pd.read_csv("data/olist/olist_customers_dataset.csv")

# Aggregate payments to order level
payments_agg = (payments
    .groupby("order_id")["payment_value"]
    .sum()
    .reset_index(name="revenue"))

# Build analysis base
base = (orders
    .query("order_status == 'delivered'")
    .merge(payments_agg, on="order_id", how="left")
    .merge(customers[["customer_id","customer_state"]], on="customer_id", how="left"))

base["year_month"] = base["order_purchase_timestamp"].dt.to_period("M").astype(str)
base["year"]       = base["order_purchase_timestamp"].dt.year
base["month"]      = base["order_purchase_timestamp"].dt.month

# Step 1: is it volume or value?
monthly = base.groupby("year_month").agg(
    orders=("order_id", "count"),
    revenue=("revenue", "sum"),
    aov=("revenue", "mean"),
).reset_index()
monthly["aov"] = monthly["aov"].round(2)

print("Monthly trend (last 8 months):")
print(monthly.tail(8).to_string(index=False))

# Now look at YoY comparison by month
yoy = base.groupby(["year", "month"]).agg(
    orders=("order_id", "count"),
    revenue=("revenue", "sum"),
).reset_index()

yoy_wide = yoy.pivot(index="month", columns="year", values=["orders","revenue"])
yoy_wide.columns = [f"{m}_{y}" for m, y in yoy_wide.columns]
yoy_wide["orders_yoy_pct"] = (
    (yoy_wide["orders_2018"] - yoy_wide["orders_2017"])
    / yoy_wide["orders_2017"] * 100
).round(1)

print("\nYear-over-year order growth by month:")
print(yoy_wide[["orders_2017","orders_2018","orders_yoy_pct"]].to_string())
```

**What the framing gave us:** instead of 40 charts exploring "revenue", we now have 3 specific analyses that directly answer the framing question. The stakeholder gets a crisp answer: *"It's a volume problem, concentrated in São Paulo, and it started in September — which happens to be when the new logistics partner was onboarded."*

---

### The five-question diagnostic — when you get a vague request

When a stakeholder sends a vague request, resist the urge to start working. Instead, ask five clarifying questions:

**1. "What decision will this inform?"**
This is the single most important question. If they can't answer it, the analysis probably shouldn't happen yet.

**2. "What would change your mind?"**
Forces them to think about what result they'd need to see. If nothing would change their mind, they've already decided and are looking for validation, not analysis.

**3. "What time period matters?"**
"Revenue is down" — down vs what? Last week? Last year? Since a product launch? The comparison determines everything.

**4. "What level of precision do you need?"**
A back-of-envelope estimate in 2 hours vs a rigorous analysis in 2 weeks are both valid. Knowing which is needed changes how you work.

**5. "What do you already believe the answer is?"**
This reveals their hypothesis. You can then either confirm or challenge it — both are valuable. And it prevents you from spending a week proving what they already knew.

---

### Principles: the craft of framing

> These are the underlying principles. The worked examples above show them in action.

**Principle 1: The question comes before the data.**
Resist the pull to "just look at the data first." Looking at data without a question produces observations, not insights. Observations don't drive decisions.

**Principle 2: A good question has a falsifiable answer.**
"How are customers doing?" cannot be answered wrong. "Did customers acquired through paid search have higher 60-day LTV than customers acquired through organic?" can be answered wrong. If the question can't be wrong, it's not specific enough.

**Principle 3: Name the decision, not the analysis.**
Don't say "I'm going to analyse churn." Say "I'm going to determine whether the retention program should be expanded to all customer segments or just the high-value ones." The decision is the destination. The analysis is the route.

**Principle 4: Agree on success criteria before starting.**
Before analysis begins, write down: "If we see X, we'll do A. If we see Y, we'll do B." If this is impossible because you don't know what actions are available, that's a signal to have a different conversation before doing any analysis.

**Principle 5: Scope ruthlessly.**
Every analysis could be extended infinitely. The analyst who tries to answer everything answers nothing well. Define explicitly: what is in scope for this analysis, and what is deliberately out of scope (and why).

---

### Templates

**Template 1: The Framing Document (1 page)**
```
Analysis: [title]
Requested by: [stakeholder]
Decision it informs: [one sentence]
Precise question: [one to three questions]
Metrics: [list]
Dimensions: [list]
Time period: [dates]
Comparison baseline: [vs what]
Success criteria: if [result A] → [action A]; if [result B] → [action B]
Data needed: [list of tables/fields]
Known limitations: [what might be wrong or incomplete]
Out of scope: [what we're explicitly not answering]
Estimated effort: [hours/days]
Delivery format: [email / dashboard / slide deck / verbal]
```

**Template 2: The Clarifying Email (when request is too vague)**
```
Subject: Re: [their request] — quick clarifying questions

Hi [name],

Happy to dig into this. Before I start, a few quick questions so I can make
sure I'm answering the right thing:

1. What decision are you trying to make with this analysis?
2. Is there a specific time period that matters, or should I look at all time?
3. What would a "good" vs "concerning" result look like to you?
4. How much time do you have — do you need a quick overview this week,
   or a thorough analysis by end of next week?

This will take me [X hours] once I have the context above.

[your name]
```

**Template 3: The Hypothesis Statement**
```
I believe that [specific outcome] because [rationale based on business context].
The analysis will test this by [specific methodology].
I will be wrong if [the data shows this instead].
```

---

## 6. The 5 Whys and Issue Trees

### The 5 Whys — drilling to root cause

The **5 Whys** is a root cause analysis technique. You ask "why" repeatedly until you reach the underlying cause, not just a symptom.

🎬 **Movies analogy:** A film gets poor reviews. Why? The editing was rushed. Why? Post-production ran over schedule. Why? Principal photography went 3 weeks over. Why? The lead actor was sick for two weeks. Why? The shoot was scheduled in monsoon season in a location known for health issues. *Root cause: poor location scouting.* Not "the film was bad" — that's the symptom.

**The rule:** keep asking "why" until you reach something actionable — something you can actually change. If your answer to "why" is "just because" or "it's always been that way," you haven't found the root cause yet.

---

### Worked example — Olist late delivery rate increasing

**Symptom:** Late delivery rate increased from 6% to 11% in Q4 2018.

```
Why #1: Why did late delivery rate increase?
Answer: More orders are being delivered after the estimated date.

Why #2: Why are more orders being delivered late?
Answer: The gap between actual delivery and estimated delivery increased.
        But the actual delivery times didn't change — the estimates got worse.

Why #3: Why did delivery time estimates get worse?
Answer: A new algorithm was deployed in October that changed how estimated
        delivery dates are calculated.

Why #4: Why does the new algorithm produce worse estimates?
Answer: The new algorithm uses average carrier transit times, but doesn't
        account for product weight. Heavy items (electronics, furniture)
        take significantly longer than the average.

Why #5: Why wasn't this caught before deployment?
Answer: The algorithm was tested on a dataset that was 80% small items.
        The test set wasn't representative of the full product catalogue.

ROOT CAUSE: Algorithm validation used an unrepresentative test dataset.
ACTIONABLE FIX: Re-test the algorithm with a stratified sample by product
               weight and category before redeploying.
```

```python
# Finding the root cause in data — confirming the hypothesis above
import pandas as pd

orders  = pd.read_csv("data/olist/olist_orders_dataset.csv",
                      parse_dates=["order_purchase_timestamp",
                                   "order_delivered_customer_date",
                                   "order_estimated_delivery_date"])
items   = pd.read_csv("data/olist/olist_order_items_dataset.csv")
products= pd.read_csv("data/olist/olist_products_dataset.csv")

# Compute actual vs estimated delivery gap
delivered = orders[orders["order_status"] == "delivered"].copy()
delivered["actual_days"]    = (delivered["order_delivered_customer_date"]
                               - delivered["order_purchase_timestamp"]).dt.days
delivered["estimated_days"] = (delivered["order_estimated_delivery_date"]
                               - delivered["order_purchase_timestamp"]).dt.days
delivered["is_late"]        = (delivered["order_delivered_customer_date"]
                               > delivered["order_estimated_delivery_date"]).astype(int)
delivered["month"]          = delivered["order_purchase_timestamp"].dt.to_period("M").astype(str)

# Late rate by month
late_trend = delivered.groupby("month")["is_late"].agg(["mean","count"]).reset_index()
late_trend.columns = ["month","late_rate","orders"]
late_trend["late_rate"] = (late_trend["late_rate"] * 100).round(1)
print("Late delivery trend:")
print(late_trend[late_trend["month"] >= "2018-01"].to_string(index=False))

# Check: is it actual delays or estimate changes?
delivered["estimate_gap"] = delivered["estimated_days"] - delivered["actual_days"]
# If estimate_gap is increasing (estimates got longer), problem is in estimating
monthly_gap = delivered.groupby("month")["estimate_gap"].mean().reset_index()
print("\nEstimate gap trend (positive = estimates are conservative):")
print(monthly_gap[monthly_gap["month"] >= "2018-01"].to_string(index=False))

# Check by product weight — does heavier = more late?
order_weight = (items
    .merge(products[["product_id","product_weight_g"]], on="product_id", how="left")
    .groupby("order_id")["product_weight_g"]
    .sum()
    .reset_index(name="total_weight_g"))

delivered_with_weight = delivered.merge(order_weight, on="order_id", how="left")
delivered_with_weight["weight_bucket"] = pd.cut(
    delivered_with_weight["total_weight_g"],
    bins=[0, 500, 2000, 10000, float("inf")],
    labels=["<500g", "500g-2kg", "2kg-10kg", ">10kg"]
)

by_weight = delivered_with_weight.groupby("weight_bucket")["is_late"].agg(
    ["mean","count"]
).reset_index()
by_weight.columns = ["weight_bucket","late_rate","orders"]
by_weight["late_rate"] = (by_weight["late_rate"] * 100).round(1)
print("\nLate rate by product weight:")
print(by_weight.to_string(index=False))
# If >10kg items have 25% late rate vs <500g items at 5%, weight hypothesis confirmed
```

---

### Issue trees — structuring a complex problem

An **issue tree** (also called a logic tree) breaks a complex question into a hierarchy of sub-questions, each of which is independently answerable. The power: you can divide work across a team, each person answering one branch, and the answers combine into a complete picture.

**The MECE principle** governs issue trees: branches must be **Mutually Exclusive** (no overlap) and **Collectively Exhaustive** (no gaps). If your branches overlap or leave gaps, you'll either double-count or miss something.

**Issue tree — "Why is revenue declining?"**

```
WHY IS REVENUE DECLINING?
│
├── VOLUME DECLINE (fewer orders)
│   ├── New customer acquisition declining?
│   │   ├── Traffic to platform declining?
│   │   └── Conversion rate declining?
│   └── Existing customer retention declining?
│       ├── Purchase frequency declining?
│       └── Customer churn increasing?
│
└── VALUE DECLINE (lower revenue per order)
    ├── Average order value declining?
    │   ├── Product mix shifting to cheaper items?
    │   └── Discount / promotion rate increasing?
    └── Revenue recognition issue?
        ├── Cancellation rate increasing?
        └── Return/refund rate increasing?
```

This tree is MECE: revenue change is either volume or value (mutually exclusive, collectively exhaustive). Within volume, it's either new or existing customers (MECE). Within value, it's either AOV or revenue leakage (MECE).

```python
# Answering each branch of the issue tree
base = (orders
    .query("order_status in ['delivered', 'shipped', 'processing']")
    .merge(payments_agg, on="order_id", how="left"))

base["month"] = base["order_purchase_timestamp"].dt.to_period("M").astype(str)

# Branch 1: volume
monthly_volume = base.groupby("month")["order_id"].count().reset_index(name="orders")

# Branch 2: value
monthly_value = base.groupby("month")["revenue"].agg(
    total_revenue="sum",
    aov="mean",
).reset_index()

# Branch 3: cancellations
cancel_rate = (orders
    .groupby(orders["order_purchase_timestamp"].dt.to_period("M").astype(str))
    .apply(lambda g: (g["order_status"] == "canceled").mean())
    .reset_index(name="cancel_rate"))

print("Volume trend:")
print(monthly_volume.tail(6).to_string(index=False))
print("\nValue trend:")
print(monthly_value.tail(6).round(2).to_string(index=False))
print("\nCancellation rate trend:")
print(cancel_rate.tail(6).round(3).to_string(index=False))
```

---

### Principles: issue trees and 5 Whys

**Principle 1: MECE is the test for a good issue tree.**
After drawing your tree, ask: do any branches overlap? Could something fall into two branches? Are there scenarios not covered by any branch? If yes to any, restructure.

**Principle 2: The 5 Whys stops at the actionable.**
Keep asking why until you reach something you can change. "The market is competitive" is not actionable. "Our pricing algorithm doesn't adjust for competitor prices" is actionable.

**Principle 3: Use the issue tree to prioritise, not just structure.**
Once you have the tree, estimate which branch is most likely to contain the root cause. Analyse the highest-probability branch first. Don't boil the ocean.

**Principle 4: Validate each branch independently.**
Each branch of the issue tree should be answerable with data. If a branch can't be answered with available data, note it explicitly — "we can't rule this out" is still useful.

---

### Templates

**Template: Issue Tree Structure**
```
ROOT QUESTION: [The main question]
│
├── BRANCH A: [First dimension of the answer — mutually exclusive from B]
│   ├── Sub-question A1
│   └── Sub-question A2
│
└── BRANCH B: [Second dimension — collectively exhaustive with A]
    ├── Sub-question B1
    └── Sub-question B2

MECE CHECK:
- Are A and B mutually exclusive? [yes/no — if no, explain overlap]
- Are A and B collectively exhaustive? [yes/no — if no, what's missing?]
```

**Template: 5 Whys Documentation**
```
SYMPTOM: [observable problem with data]

Why #1: [immediate cause] — Evidence: [data point]
Why #2: [cause of cause 1] — Evidence: [data point]
Why #3: [cause of cause 2] — Evidence: [data point]
Why #4: [cause of cause 3] — Evidence: [data point]
Why #5: [root cause] — Evidence: [data point]

ROOT CAUSE: [one sentence]
RECOMMENDED ACTION: [specific, actionable next step]
WHO OWNS IT: [team or person]
```

---

## 7. MECE Thinking

### What MECE means and why analysts need it

**MECE** (Mutually Exclusive, Collectively Exhaustive) is a principle from McKinsey that has become standard in analytical and consulting work. It ensures that when you segment or categorise anything, you don't double-count and don't miss anything.

- **Mutually Exclusive:** no customer falls into two segments
- **Collectively Exhaustive:** every customer falls into some segment

🎬 **Movies analogy:** Categorising films as "action" and "has explosions" is NOT MECE — they overlap (most action films have explosions). Categorising as "fiction" and "non-fiction" IS MECE — every film is one or the other, and none is both.

---

### Where MECE breaks in analytics — common mistakes

**Mistake 1: Overlapping segments**
```
# NOT MECE — segments overlap
segments = {
    "High value":    revenue > 1000,   # a customer with R$1500 is both
    "Recent buyer":  days_since_purchase < 30,
    "Loyal":         orders > 10,
}
# One customer can be in multiple segments simultaneously
# Your counts won't add up to 100%

# MECE version:
segments = pd.cut(
    df["revenue"],
    bins=[0, 200, 500, 1000, float("inf")],
    labels=["Low", "Medium", "High", "Premium"]
)
# Every customer is in exactly one segment
```

**Mistake 2: Gaps in the segmentation**
```python
# NOT MECE — customers with 0 revenue are uncategorised
df["segment"] = np.where(df["revenue"] > 500, "High", 
                np.where(df["revenue"] > 100, "Medium", "Low"))
# What about customers with revenue = 0? They fall through.

# MECE version:
df["segment"] = np.where(df["revenue"] == 0, "Inactive",
                np.where(df["revenue"] > 500, "High",
                np.where(df["revenue"] > 100, "Medium", "Low")))
# Every customer now has a segment
```

**Mistake 3: Issue tree branches that overlap**
```
# NOT MECE issue tree for "Why is NPS low?"
# Branch A: "Customers who had a bad delivery experience"
# Branch B: "Customers in São Paulo"
# These overlap — a São Paulo customer with a bad delivery is in both
# Solution: make Branch A about experience quality (universal)
#           make Branch B about geography (cross-cutting)
#           Or: use sequential MECE (is it geography first? then within
#                                     geography, is it experience?)
```

---

### MECE in practice — the segment validation pattern

```python
def validate_mece(df: pd.DataFrame, segment_col: str) -> dict:
    """
    Validate that a segmentation is MECE:
    - Collectively exhaustive: all rows have a non-null segment
    - Mutually exclusive: each row appears in only one segment
      (guaranteed if one column, but check for nulls)
    """
    total_rows = len(df)
    null_rows  = df[segment_col].isnull().sum()
    segment_counts = df[segment_col].value_counts()

    result = {
        "total_rows":         total_rows,
        "unassigned_rows":    null_rows,
        "collectively_exhaustive": null_rows == 0,
        "segments":           segment_counts.to_dict(),
        "segment_coverage":   (total_rows - null_rows) / total_rows,
    }

    if not result["collectively_exhaustive"]:
        print(f"⚠️  {null_rows:,} rows ({null_rows/total_rows:.1%}) have no segment")
    else:
        print(f"✓ MECE validated: all {total_rows:,} rows assigned")
        for seg, count in segment_counts.items():
            print(f"  {seg}: {count:,} ({count/total_rows:.1%})")

    return result

# Usage
df["revenue_segment"] = pd.cut(
    df["total_payment"],
    bins=[0, 50, 200, 500, float("inf")],
    labels=["Micro", "Small", "Medium", "Large"],
    include_lowest=True
)
validate_mece(df, "revenue_segment")
```

---

## 8. Scoping an Analysis

### Why scope matters — and what happens without it

Without scope:
- Analysis expands to fill all available time
- Stakeholders get overwhelmed with findings that don't connect
- You answer questions nobody asked
- The most important question gets buried

With scope:
- Clear deliverable and timeline
- Focused analysis that connects to one decision
- Room for a second analysis after the first one lands

🎬 **Movies analogy:** The director's cut vs the theatrical release. The director's cut includes every scene the director loved. The theatrical release includes only scenes that serve the story. Both are made from the same footage. The theatrical release is more effective because it respects the audience's time and serves the narrative. Scope your analysis like a theatrical release.

---

### The scoping conversation — what to decide before starting

| Question | Why it matters |
|----------|---------------|
| What is the one key question? | Keeps analysis focused |
| Who is the audience? | Determines depth, format, vocabulary |
| What format do they need? | Email, slide, dashboard, verbal briefing |
| By when? | Determines how deep you can go |
| What is explicitly OUT of scope? | Prevents scope creep |
| What will you do if the data doesn't support an answer? | Prevents surprise |

---

### Explicitly documenting out-of-scope items

Out-of-scope documentation is as important as in-scope. When a stakeholder asks "did you look at X?" and you say "that's out of scope for this analysis" — that's fine. When you haven't thought about it at all — that's not.

```
ANALYSIS SCOPE: Olist Q4 Revenue Decline

IN SCOPE:
✓ Revenue trend by month (Oct–Dec 2018 vs same period 2017)
✓ Volume vs value decomposition
✓ Breakdown by top 5 product categories
✓ Breakdown by top 5 seller states

OUT OF SCOPE (and why):
✗ Customer-level cohort analysis
  → Requires separate analysis; would delay this by 3 days
✗ Competitor analysis
  → No competitive data available
✗ Marketing attribution
  → Marketing data not in our warehouse yet
✗ Forecast for Q1 2019
  → Different analysis; recommend as Phase 2

KNOWN LIMITATIONS:
⚠ Cancelled orders are excluded (we have no cancellation reason data)
⚠ Product category translations are 85% complete; 15% are "unknown"
⚠ Payment values from marketplace are net of fees; gross figures unavailable
```

---

### Principles: scoping

**Principle 1: Scope is a negotiation, not a unilateral decision.**
You can't scope down to nothing. Stakeholders have legitimate needs. The conversation is: "Here's what I can do in your timeframe. Here's what I'd have to leave out. Is that trade-off acceptable?"

**Principle 2: Document what's out of scope in writing.**
Verbal agreements evaporate. When a stakeholder asks "why didn't you look at X?" two weeks later, you want to be able to say "it's in the scope document you approved on Tuesday."

**Principle 3: One analysis, one decision.**
If an analysis tries to inform more than one decision simultaneously, the two sets of scope requirements will fight each other. Better to do two focused analyses than one sprawling one.

**Principle 4: Scope can expand after the first finding.**
A good first analysis often reveals the second question. That's fine — but it's a new analysis with its own scope, not an extension of the first.

---

### Templates

**Template: Analysis Scope Document**
```
ANALYSIS: [title]
DATE: [date]
ANALYST: [name]
STAKEHOLDER: [name + role]

DECISION THIS INFORMS: [one sentence]

IN SCOPE:
- [item 1]
- [item 2]

OUT OF SCOPE (with reason):
- [item 1] — [reason]
- [item 2] — [reason]

KNOWN DATA LIMITATIONS:
- [limitation 1]
- [limitation 2]

DELIVERY:
Format: [email / slide / dashboard / verbal]
Audience: [who will see this]
Deadline: [date]
Estimated effort: [hours]

APPROVED BY: [stakeholder name + date]
```

---
---

# Part III — Business Context and Frameworks

## 9. How Businesses Think About Data

### The analyst's mental model — what the business actually wants

Businesses don't want data. They don't even want insights. They want **decisions made well and outcomes improved**. Data and insights are the means, not the end.

This sounds obvious but it changes how you work. An analyst who optimises for "interesting findings" produces beautiful analyses that sit in slide decks. An analyst who optimises for "decisions made well" becomes indispensable.

🎬 **Movies analogy:** A film studio doesn't want great cinematography. It wants ticket sales, award recognition, and franchise potential. The cinematographer's job is extraordinary — but it exists in service of those business outcomes. An analyst who loses sight of the business outcome is like a cinematographer who shoots gorgeous footage that doesn't serve the story.

---

### The three questions every business stakeholder has

When a stakeholder receives an analysis, consciously or not, they're asking:

**1. "So what?"** — What does this mean for us? Don't make me interpret your charts.

**2. "Now what?"** — What should we do? What's the recommended action?

**3. "Why should I trust this?"** — How confident are you? What might be wrong?

Every analytical output should answer all three. An output that only shows data without answering "so what" is incomplete. An output that has a recommendation without "why should I trust this" is overconfident.

---

### How business decisions actually get made

Understanding the decision-making process helps you deliver analysis that gets used:

```
BUSINESS DECISION PROCESS:

Trigger          Problem        Options         Decision       Action
(something       identified     identified      made           taken
happened)
    │                │               │              │             │
    ▼                ▼               ▼              ▼             ▼
Revenue       "We think it's   "Option A:      "We'll do     Campaign
drops          acquisition"     improve CAC     Option B"     launched
                                Option B:
                                improve          ← YOUR
                                retention"        ANALYSIS
                                                  GOES HERE

Your analysis is most valuable between "options identified" and "decision made."
Arriving before options are identified = too early (the question isn't formed yet)
Arriving after decision is made = too late (they're looking for validation, not insight)
```

**The implication:** ask stakeholders where they are in their decision process before starting. "Have you already decided on an approach?" If yes, the analysis serves a different purpose than if the decision is genuinely open.

---

### The analyst's role in the business — four modes

Analysts operate in different modes depending on the request:

| Mode | What they're doing | Stakeholder needs |
|------|-------------------|-------------------|
| **Reporter** | Producing regular dashboards and reports | "Keep me informed" |
| **Investigator** | Answering a specific question | "Help me understand this" |
| **Advisor** | Recommending a course of action | "Help me decide" |
| **Partner** | Embedded in a team, proactively surfacing opportunities | "Help me improve" |

Most analysts start as Reporters. The most valuable analysts are Partners. The path from Reporter to Partner runs through consistently delivering Investigator and Advisor work that is acted on.

---

## 10. Common Business Problems by Domain

### Why domain knowledge matters

An analyst without domain knowledge produces technically correct analyses that miss the point. When you know that e-commerce seasonal peaks happen in November (Black Friday) and July (mid-year sales), you don't report a "revenue spike" in November as an anomaly. When you know that hospital LOS (length of stay) is regulated and reimbursed by diagnosis category, you interpret billing patterns completely differently.

---

### E-Commerce — common questions and problems

**The business model:** acquire customers → convert them to first purchase → retain them for repeat purchases → maximise lifetime value.

**Common analytical questions:**

| Question | Metric | Typical benchmark |
|----------|--------|-------------------|
| How efficiently do we acquire customers? | CAC (Customer Acquisition Cost) | Varies; target CAC:LTV ratio < 1:3 |
| How well do we retain customers? | 90-day retention rate | E-commerce: 20–40% |
| How valuable is a customer over time? | LTV (Lifetime Value) | Depends on vertical |
| How healthy is the order pipeline? | GMV, AOV, conversion rate | Depends on segment |
| How do customers find us? | Acquisition channel mix | |
| Which products drive repeat purchase? | Product repeat rate | |
| What are our best sellers? | Revenue per SKU, sell-through rate | |

**Seasonal patterns to know:**
- **November:** Black Friday / Cyber Monday — largest spike of the year in most markets
- **December:** Christmas gifting — high AOV, often lower margin (discounts)
- **January:** Post-holiday slump, returns peak
- **Brazil-specific (Olist dataset):** Mother's Day (May), Valentine's Day (June in Brazil), Children's Day (October) are major peaks

**Common analytical pitfalls:**
- **Last-click attribution:** crediting the last marketing channel for a conversion, ignoring earlier touchpoints
- **Ignoring returns:** gross revenue looks better than net revenue; returns significantly affect profitability
- **Cohort confusion:** mixing customers from different acquisition months in the same retention analysis

```python
# E-commerce: quick health check
import pandas as pd

orders   = pd.read_csv("data/olist/olist_orders_dataset.csv",
                       parse_dates=["order_purchase_timestamp"])
payments = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
customers= pd.read_csv("data/olist/olist_customers_dataset.csv")

payments_agg = (payments.groupby("order_id")["payment_value"]
                .sum().reset_index(name="revenue"))

base = (orders
    .query("order_status == 'delivered'")
    .merge(payments_agg, on="order_id")
    .merge(customers[["customer_id","customer_unique_id"]], on="customer_id"))

# Repeat purchase rate
repeat_buyers = (base
    .groupby("customer_unique_id")["order_id"]
    .count()
    .reset_index(name="orders"))

repeat_rate = (repeat_buyers["orders"] > 1).mean()
print(f"Repeat purchase rate: {repeat_rate:.1%}")
# Olist is typically ~3% — very low, as expected for a marketplace

# Average order value
print(f"Average order value: R${base['revenue'].mean():.2f}")
print(f"Median order value:  R${base['revenue'].median():.2f}")
# Mean > Median usually → right-skewed distribution → a few large orders pull up the mean
```

---

### Healthcare — common questions and problems

**The business model (hospitals):** provide care → get reimbursed by insurers/government → manage costs → maintain quality ratings.

**Common analytical questions:**

| Question | Metric | Why it matters |
|----------|--------|---------------|
| How long are patients staying? | LOS (Length of Stay) | Directly drives cost and bed capacity |
| Are patients coming back? | 30-day readmission rate | Quality indicator; penalised by CMS |
| How are we billing? | Avg billing by diagnosis, payer mix | Revenue and compliance |
| Are procedures effective? | Patient outcomes by procedure | Quality and accreditation |
| How is capacity being used? | Bed occupancy rate, procedure volume | Operational efficiency |
| Are patients satisfied? | HCAHPS survey scores | Tied to reimbursement in US |

**Domain knowledge essentials:**
- **DRG (Diagnosis Related Group):** Medicare reimburses hospitals based on the DRG category of the discharge, not the actual cost — which creates incentives around LOS and procedure selection
- **30-day readmission:** if a patient returns within 30 days for the same condition, hospitals are penalised — a key quality metric
- **Payer mix:** patients covered by Medicare/Medicaid typically reimburse less than private insurance — payer mix affects revenue significantly

```python
# Healthcare: key metrics quick view
df = pd.read_csv("data/healthcare/healthcare_dataset.csv")
df["Date of Admission"] = pd.to_datetime(df["Date of Admission"])
df["Discharge Date"]    = pd.to_datetime(df["Discharge Date"])
df["LOS"]              = (df["Discharge Date"] - df["Date of Admission"]).dt.days

print("Average LOS by admission type:")
print(df.groupby("Admission Type")["LOS"].mean().round(1))

print("\nAverage billing by medical condition (top 5):")
print(df.groupby("Medical Condition")["Billing Amount"]
      .mean().sort_values(ascending=False).head(5).round(0))
```

---

### Supply Chain / Manufacturing — common questions and problems

**The business model:** source raw materials → manufacture → warehouse → ship → deliver on time at target cost.

**Common analytical questions:**

| Question | Metric | Why it matters |
|----------|--------|---------------|
| Are we delivering on time? | OTIF (On Time In Full) | Customer satisfaction, SLA compliance |
| Where are orders getting stuck? | Cycle time by stage | Bottleneck identification |
| How accurate are our forecasts? | Forecast accuracy (MAPE) | Drives inventory levels |
| How much inventory is idle? | Inventory turnover ratio | Working capital efficiency |
| Which suppliers are reliable? | Supplier OTIF, defect rate | Risk management |
| How profitable are our products? | Gross margin by SKU | Portfolio optimisation |

**Domain knowledge essentials:**
- **OTIF:** On Time In Full — binary measure: order was delivered complete and on time. The supply chain gold standard.
- **Bullwhip effect:** small demand variability at retail level amplifies into large inventory swings upstream — a classic supply chain problem
- **Days of Inventory Outstanding (DIO):** how many days of sales are sitting in inventory — lower is better (cash efficiency) but too low = stockouts

---

### HR / People Analytics — common questions and problems

**The business model:** attract talent → develop them → retain them → maximise productivity.

**Common analytical questions:**

| Question | Metric | Why it matters |
|----------|--------|---------------|
| Who is leaving and why? | Attrition rate, exit reasons | Cost of turnover = 50–200% of salary |
| Who is at risk of leaving? | Flight risk score | Proactive retention |
| Are we paying fairly? | Pay equity analysis | Retention and compliance |
| How engaged are employees? | eNPS, survey scores | Leading indicator of attrition |
| How effective is hiring? | Time-to-hire, offer acceptance rate | Talent pipeline health |
| Who are our high performers? | Performance distribution | Succession planning |

**Domain knowledge:**
- Cost of replacing an employee: recruiting fees + training + productivity loss ≈ 50–200% of annual salary
- The difference between **voluntary** (employee chose to leave) and **involuntary** (company terminated) attrition — they require completely different interventions
- **Regrettable attrition:** high performers who left voluntarily — the most damaging type

---

### Finance / Banking — common questions and problems

**Common analytical questions:**

| Question | Metric | Why it matters |
|----------|--------|---------------|
| Are customers defaulting? | Default rate, NPL ratio | Credit risk |
| Which customers are fraudulent? | Fraud rate, fraud loss ratio | Financial risk |
| How profitable are products? | NIM (Net Interest Margin) | Business health |
| Who are our best customers? | Customer profitability | Resource allocation |
| Are we compliant? | Regulatory metrics | Legal requirement |

---

## 11. North Star Metrics, OKRs, and AARRR

### North Star Metric — the single number that matters most

A **North Star Metric (NSM)** is the one metric that best captures the core value your product delivers. Everything the company does should move this metric. It's not revenue (that's an output) — it's the metric that predicts long-term revenue growth.

🎬 **Movies analogy:** A streaming service's North Star is not subscription revenue. It's "hours watched per subscriber per month" — because subscribers who watch more are subscribers who stay, and that drives long-term revenue. Optimising for the NSM means making content people actually watch, not just content that gets signed up for.

**Examples by company type:**

| Company type | North Star Metric |
|-------------|-------------------|
| E-commerce marketplace | GMV (Gross Merchandise Value) or Monthly Active Buyers |
| SaaS product | Daily Active Users (DAU) or Weekly Active Users |
| Marketplace (Airbnb) | Nights booked |
| Social network | Daily active users / Monthly active users (DAU/MAU) |
| Streaming (Netflix) | Hours watched |
| On-demand delivery | Orders per month per city |

**The NSM test:**
1. Does it capture customer value (not just company revenue)?
2. Does it predict long-term revenue growth?
3. Can everyone in the company understand it?
4. Can every team influence it?

---

### OKRs — Objectives and Key Results

**OKRs** are a goal-setting framework (Google, Intel, most tech companies). An analyst's role: provide the data that measures progress against Key Results.

```
OBJECTIVE: Improve seller quality on the Olist platform
  (qualitative, aspirational, "the why")

KEY RESULTS:
  KR1: Increase average seller review score from 4.1 to 4.4 by Q4 2018
       → Analyst provides: monthly seller review score tracking
  KR2: Reduce late delivery rate from 8% to 5% by Q4 2018
       → Analyst provides: weekly late delivery rate by seller
  KR3: Grow 5-star sellers (score ≥ 4.8) from 20% to 30% of GMV
       → Analyst provides: GMV share by seller rating tier

INITIATIVE: Seller quality coaching program
  → Uses KR1/KR2/KR3 to target sellers for intervention
  → Uses KR1/KR2/KR3 to measure whether coaching works
```

**Analysts and OKRs:** your job is not to set OKRs — that's leadership. Your job is to define exactly how each KR will be measured, build the tracking, surface the data, and flag when a KR is at risk before the quarter ends.

---

### AARRR — The Pirate Metrics Framework

**AARRR** (Acquisition, Activation, Retention, Revenue, Referral) is a framework for understanding the customer journey as a funnel. Originally from Dave McClure (500 Startups), now widely used in product analytics.

```
ACQUISITION    "How do customers find us?"
    │          Metrics: traffic sources, CAC, conversion rate by channel
    ▼
ACTIVATION     "Do customers have a good first experience?"
    │          Metrics: % completed onboarding, time-to-first-value
    ▼
RETENTION      "Do customers come back?"
    │          Metrics: DAU/MAU, 30/60/90 day retention, churn rate
    ▼
REVENUE        "Do customers pay?"
    │          Metrics: MRR, ARR, ARPU, LTV, conversion from free to paid
    ▼
REFERRAL       "Do customers tell others?"
               Metrics: NPS, referral rate, viral coefficient (K-factor)
```

**The analyst's job with AARRR:** identify where the biggest leaks are. If 1,000 customers are acquired, 200 activate, 50 are retained, 20 generate revenue, and 2 refer — the biggest problem is activation (only 20% of acquired customers activate). That's where to focus.

```python
# AARRR-style funnel analysis (adapted for Olist)
# Olist funnel: customer sees platform → places order → completes purchase
# → makes second purchase → refers another seller/customer

import pandas as pd

orders    = pd.read_csv("data/olist/olist_orders_dataset.csv",
                        parse_dates=["order_purchase_timestamp"])
customers = pd.read_csv("data/olist/olist_customers_dataset.csv")
reviews   = pd.read_csv("data/olist/olist_order_reviews_dataset.csv")

# All unique customers who placed at least one order
total_customers = customers["customer_unique_id"].nunique()

# Completed a purchase (delivered)
delivered = orders[orders["order_status"] == "delivered"]
activated = delivered.merge(
    customers[["customer_id","customer_unique_id"]], on="customer_id"
)["customer_unique_id"].nunique()

# Made a second purchase (retained)
purchase_counts = (
    delivered.merge(customers[["customer_id","customer_unique_id"]], on="customer_id")
    .groupby("customer_unique_id")["order_id"]
    .count()
)
retained = (purchase_counts >= 2).sum()

# Left a 5-star review (referral proxy)
high_scorers = (reviews[reviews["review_score"] == 5]
    .merge(orders[["order_id","customer_id"]], on="order_id")
    .merge(customers[["customer_id","customer_unique_id"]], on="customer_id")
    ["customer_unique_id"].nunique())

print("AARRR Funnel — Olist:")
print(f"Acquisition (all customers):          {total_customers:>8,}")
print(f"Activation (completed first purchase): {activated:>8,}  ({activated/total_customers:.1%})")
print(f"Retention (2+ purchases):              {retained:>8,}  ({retained/total_customers:.1%})")
print(f"Referral (5-star review):              {high_scorers:>8,}  ({high_scorers/total_customers:.1%})")
```

---

## 12. Industry KPI Reference

### E-Commerce KPIs

| KPI | Formula | Good / Concerning |
|-----|---------|-------------------|
| **GMV** (Gross Merchandise Value) | Sum of all order values before returns/cancellations | Growing MoM |
| **NMV** (Net Merchandise Value) | GMV − returns − cancellations | NMV/GMV ratio > 85% = healthy |
| **AOV** (Average Order Value) | GMV / Number of orders | Varies by vertical |
| **CAC** (Customer Acquisition Cost) | Marketing spend / New customers acquired | CAC < LTV/3 |
| **LTV** (Customer Lifetime Value) | Avg order value × purchase frequency × customer lifespan | LTV:CAC > 3:1 |
| **Conversion Rate** | Orders / Sessions (or visits) | E-commerce: 1–4% |
| **Repeat Purchase Rate** | Customers with ≥2 orders / Total customers | > 30% = good |
| **Churn Rate** | Customers who didn't repurchase in period / Active customers | < 10% monthly |
| **Cart Abandonment Rate** | Carts created but not purchased / Total carts | 70% is typical; optimise below |
| **Return Rate** | Returned orders / Total orders | < 10% is healthy |
| **Seller OTIF** | Orders delivered on time and in full / Total orders | > 90% |

---

### Healthcare KPIs

| KPI | Formula | Good / Concerning |
|-----|---------|-------------------|
| **ALOS** (Average Length of Stay) | Total patient days / Discharges | Benchmark varies by DRG |
| **30-day Readmission Rate** | Patients readmitted in 30 days / Total discharges | < 10% |
| **Bed Occupancy Rate** | Patient days / Available bed days | 75–85% optimal |
| **Net Patient Revenue per Bed** | Net revenue / Licensed beds | Benchmark by hospital type |
| **Payer Mix** | % patients by payer type (Medicare / Medicaid / Private / Self-pay) | Higher private insurance = higher revenue |
| **Case Mix Index** | Avg DRG weight of all cases | Higher = more complex (and better reimbursed) cases |
| **Mortality Rate** | Inpatient deaths / Total discharges | Benchmark by diagnosis |
| **Patient Satisfaction (HCAHPS)** | Survey scores | Top box scores matter for reimbursement |
| **Cost per Discharge** | Total costs / Total discharges | Lower = more efficient |

---

### Supply Chain KPIs

| KPI | Formula | Good / Concerning |
|-----|---------|-------------------|
| **OTIF** | Orders on-time and in-full / Total orders | > 95% for most industries |
| **Fill Rate** | Line items shipped / Line items ordered | > 98% |
| **Order Cycle Time** | Date delivered − Date ordered | Industry specific |
| **Inventory Turnover** | COGS / Average inventory | Higher = more efficient |
| **Days of Inventory Outstanding (DIO)** | (Inventory / COGS) × 365 | Lower = less working capital tied up |
| **Perfect Order Rate** | Orders with no defect / Total orders | > 90% |
| **Freight Cost per Order** | Total freight cost / Orders shipped | Benchmark by distance/mode |
| **Supplier OTIF** | Supplier deliveries on time / Total deliveries | > 95% |
| **Demand Forecast Accuracy** | 1 − MAPE (Mean Absolute Percentage Error) | > 80% |
| **Gross Margin** | (Revenue − COGS) / Revenue | Varies by industry |

---

### HR / People Analytics KPIs

| KPI | Formula | Good / Concerning |
|-----|---------|-------------------|
| **Voluntary Attrition Rate** | Voluntary departures / Avg headcount | < 10% annual for most |
| **Involuntary Attrition Rate** | Involuntary terminations / Avg headcount | Benchmark by business context |
| **Regrettable Attrition** | High-performer departures / Total departures | < 20% of total attrition |
| **Time to Fill** | Days from job open to offer accepted | < 30 days for most roles |
| **Offer Acceptance Rate** | Offers accepted / Offers made | > 85% |
| **eNPS** (Employee NPS) | % Promoters − % Detractors | > 20 = good; > 50 = excellent |
| **Revenue per Employee** | Revenue / Headcount | Benchmark by industry |
| **Gender Pay Gap** | Avg male salary − Avg female salary / Avg male salary | Close to 0% |
| **Training Completion Rate** | Completions / Enrolled | > 80% |
| **Internal Hire Rate** | Internal promotions / Total hires | 20–30% is healthy |

---

### Financial Services KPIs

| KPI | Formula | Good / Concerning |
|-----|---------|-------------------|
| **NPL Ratio** (Non-Performing Loans) | Non-performing loans / Total loans | < 5% |
| **NIM** (Net Interest Margin) | (Interest income − Interest expense) / Avg earning assets | > 3% for retail banking |
| **Fraud Rate** | Fraudulent transactions / Total transactions | < 0.1% |
| **Fraud Loss Ratio** | Fraud losses / Total transaction value | < 0.01% |
| **Customer Churn Rate** | Accounts closed / Total accounts | < 2% monthly |
| **Cross-sell Ratio** | Products per customer | Higher = better relationship depth |
| **CAR** (Capital Adequacy Ratio) | Regulatory capital / Risk-weighted assets | > 8% (Basel III requirement) |
| **Cost-to-Income Ratio** | Operating costs / Operating income | < 60% for efficiency |

---
---

# Part IV — Exploratory Data Analysis

## 13. The EDA Framework

### What EDA is — and what it isn't

**Exploratory Data Analysis (EDA)** is the process of getting familiar with a dataset before formal analysis. It is not cleaning. It is not modelling. It is a conversation with data — asking questions, observing answers, and letting what you find guide your next question.

EDA was formalised by statistician John Tukey in his 1977 book *Exploratory Data Analysis*. His core idea: let the data speak before you impose a model. Look at the data first. Form hypotheses second. Test third.

🎬 **Movies analogy:** EDA is the director watching all the raw footage before editing. You don't start cutting until you know what you have. Some footage will be unusable (nulls, corrupted data). Some will be unexpectedly good (surprising distributions, strong correlations). You need to see everything before you can make creative decisions.

---

### The EDA checklist — a repeatable process

Run through this checklist on every new dataset. It takes 30–60 minutes and prevents hours of downstream mistakes.

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

def eda_checklist(df: pd.DataFrame, name: str = "dataset") -> None:
    """
    Run the standard EDA checklist on a DataFrame.
    Prints a structured summary.
    """
    print(f"\n{'='*60}")
    print(f"EDA CHECKLIST: {name}")
    print(f"{'='*60}")

    # ── 1. SHAPE ──────────────────────────────────────────────────
    print(f"\n1. SHAPE")
    print(f"   Rows:    {len(df):,}")
    print(f"   Columns: {len(df.columns)}")

    # ── 2. COLUMN NAMES AND TYPES ─────────────────────────────────
    print(f"\n2. COLUMNS AND TYPES")
    for col in df.columns:
        null_count = df[col].isnull().sum()
        null_pct   = null_count / len(df) * 100
        unique     = df[col].nunique()
        flag       = " ⚠️ HIGH NULLS" if null_pct > 20 else ""
        print(f"   {col:<35} {str(df[col].dtype):<12} "
              f"null: {null_pct:>5.1f}%  unique: {unique:>6,}{flag}")

    # ── 3. MISSING DATA ───────────────────────────────────────────
    null_rates = df.isnull().mean()
    high_null  = null_rates[null_rates > 0].sort_values(ascending=False)
    print(f"\n3. MISSING DATA ({(null_rates > 0).sum()} columns with nulls)")
    if len(high_null) > 0:
        for col, rate in high_null.head(10).items():
            print(f"   {col:<35} {rate:.1%} missing ({int(rate*len(df)):,} rows)")

    # ── 4. NUMERIC SUMMARY ────────────────────────────────────────
    numeric = df.select_dtypes(include=np.number)
    if len(numeric.columns) > 0:
        print(f"\n4. NUMERIC SUMMARY ({len(numeric.columns)} numeric columns)")
        summary = numeric.describe().round(2)
        print(summary.to_string())

    # ── 5. CATEGORICAL SUMMARY ────────────────────────────────────
    categorical = df.select_dtypes(include=["object","category"])
    if len(categorical.columns) > 0:
        print(f"\n5. TOP VALUES IN CATEGORICAL COLUMNS")
        for col in categorical.columns[:8]:  # show first 8
            top = df[col].value_counts().head(5)
            print(f"\n   {col}:")
            for val, count in top.items():
                print(f"     {str(val):<30} {count:>7,} ({count/len(df):.1%})")

    # ── 6. DUPLICATE CHECK ────────────────────────────────────────
    dup_count = df.duplicated().sum()
    print(f"\n6. DUPLICATE ROWS: {dup_count:,} "
          f"({'NONE — clean' if dup_count == 0 else '⚠️  INVESTIGATE'})")

    # ── 7. DATE RANGES ────────────────────────────────────────────
    date_cols = df.select_dtypes(include=["datetime64"]).columns
    if len(date_cols) > 0:
        print(f"\n7. DATE RANGES")
        for col in date_cols:
            print(f"   {col}: {df[col].min()} → {df[col].max()}")

    print(f"\n{'='*60}")


# Usage
orders = pd.read_csv("data/olist/olist_orders_dataset.csv",
                     parse_dates=["order_purchase_timestamp",
                                  "order_delivered_customer_date"])
eda_checklist(orders, "Olist Orders")
```

---

### What to look for in EDA — the red flags and the gold

**Red flags — things that warrant investigation before analysis:**

| Signal | What it might mean |
|--------|-------------------|
| > 20% nulls in a key column | Data collection issue, optional field, or source system bug |
| Duplicate primary keys | Data pipeline issue, double-counting risk |
| Dates in the future | Data entry errors, test records |
| Negative values in quantity/revenue | Returns? Encoding issue? Intentional? |
| Cardinality wildly different from expectation | Wrong join? Wrong grain? |
| Columns with only 1 unique value | Constant — useless for analysis |
| Values that don't match expected codes | Schema mismatch, dirty data |

**Gold — things that give you analytical leverage:**

| Signal | What it might mean |
|--------|-------------------|
| Bimodal distribution | Two distinct populations mixed together — segment them |
| Heavy right skew | Mean is misleading — use median; a few outliers drive the average |
| Unexpected seasonality | Recurring patterns — confirm with business |
| Perfect correlation (r > 0.95) | May be measuring the same thing twice |
| Strong negative correlation | A useful trade-off worth exploring |
| A category with disproportionate volume | The dominant segment — often where interventions matter most |

---

## 14. Univariate Analysis

### What it is — one variable at a time

Univariate analysis examines one variable in isolation. The goal is to understand its distribution: where most values sit, how spread out they are, whether there are outliers, and whether the shape of the distribution is what you'd expect.

---

### Continuous variables — the distribution deep dive

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

orders   = pd.read_csv("data/olist/olist_orders_dataset.csv",
                       parse_dates=["order_purchase_timestamp"])
payments = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
payments_agg = payments.groupby("order_id")["payment_value"].sum().reset_index()
df = orders.merge(payments_agg, on="order_id", how="left")

revenue = df["payment_value"].dropna()

# ── SUMMARY STATISTICS ────────────────────────────────────────────────────
print("Revenue distribution:")
print(f"  Count:    {len(revenue):,}")
print(f"  Mean:     R${revenue.mean():,.2f}")
print(f"  Median:   R${revenue.median():,.2f}")
print(f"  Std dev:  R${revenue.std():,.2f}")
print(f"  Min:      R${revenue.min():,.2f}")
print(f"  P25:      R${revenue.quantile(0.25):,.2f}")
print(f"  P75:      R${revenue.quantile(0.75):,.2f}")
print(f"  P90:      R${revenue.quantile(0.90):,.2f}")
print(f"  P99:      R${revenue.quantile(0.99):,.2f}")
print(f"  Max:      R${revenue.max():,.2f}")
print(f"  Mean/Median ratio: {revenue.mean()/revenue.median():.2f}x")
# Mean/Median ratio > 1.5 signals right skew — mean is pulled by outliers

# ── DISTRIBUTION PLOT ─────────────────────────────────────────────────────
fig, axes = plt.subplots(1, 3, figsize=(15, 4))

# Raw distribution (will show extreme right skew)
axes[0].hist(revenue, bins=50, edgecolor="white", color="#4C72B0")
axes[0].set_title("Revenue Distribution (raw)")
axes[0].set_xlabel("Revenue (R$)")
axes[0].axvline(revenue.mean(),   color="red",    linestyle="--", label="Mean")
axes[0].axvline(revenue.median(), color="orange", linestyle="--", label="Median")
axes[0].legend()

# Log-transformed (reveals true shape)
axes[1].hist(np.log1p(revenue), bins=50, edgecolor="white", color="#55A868")
axes[1].set_title("Revenue Distribution (log scale)")
axes[1].set_xlabel("log(Revenue + 1)")

# Box plot
axes[2].boxplot(revenue, vert=True)
axes[2].set_title("Revenue Box Plot")
axes[2].set_ylabel("Revenue (R$)")

plt.tight_layout()
plt.savefig("outputs/revenue_distribution.png", dpi=150, bbox_inches="tight")
plt.show()

# ── INTERPRETATION ────────────────────────────────────────────────────────
print("\nInterpretation:")
print(f"  Mean (R${revenue.mean():.0f}) >> Median (R${revenue.median():.0f})")
print(f"  → Strong right skew — a small number of high-value orders")
print(f"    pull the mean upward. Median is a better 'typical order' measure.")
print(f"  99th percentile: R${revenue.quantile(0.99):.0f}")
print(f"  → Top 1% of orders are worth R${revenue.quantile(0.99):.0f}+")
print(f"    These are likely bulk/B2B orders and behave differently.")
```

---

### Categorical variables — frequency and concentration

```python
# ── CATEGORICAL: order status distribution ────────────────────────────────
status_counts = df["order_status"].value_counts()
print("Order status distribution:")
for status, count in status_counts.items():
    bar = "█" * int(count / status_counts.max() * 30)
    print(f"  {status:<15} {count:>7,}  {count/len(df):.1%}  {bar}")

# ── CONCENTRATION ANALYSIS ────────────────────────────────────────────────
# The Pareto principle (80/20 rule) appears frequently in business data
# Do 20% of sellers generate 80% of revenue?

items    = pd.read_csv("data/olist/olist_order_items_dataset.csv")
seller_revenue = items.groupby("seller_id")["price"].sum().sort_values(ascending=False)

# Cumulative revenue share
cumulative = seller_revenue.cumsum() / seller_revenue.sum()
top_20pct_idx = int(len(seller_revenue) * 0.2)
top20_share   = cumulative.iloc[top_20pct_idx]

print(f"\nPareto analysis — Olist sellers:")
print(f"  Top 20% of sellers: {top_20pct_idx:,} sellers")
print(f"  Their revenue share: {top20_share:.1%}")

# Top 10 sellers
print(f"\n  Top 10 sellers:")
for i, (seller, rev) in enumerate(seller_revenue.head(10).items()):
    print(f"    {i+1}. {seller[:8]}... R${rev:,.0f}  ({rev/seller_revenue.sum():.1%})")
```

---

### Time series — understanding temporal patterns

```python
# ── ORDER VOLUME OVER TIME ─────────────────────────────────────────────────
df["month"] = df["order_purchase_timestamp"].dt.to_period("M").astype(str)

monthly = df.groupby("month").agg(
    orders   = ("order_id",     "count"),
    revenue  = ("payment_value","sum"),
    aov      = ("payment_value","mean"),
).reset_index()

# Visual
fig, axes = plt.subplots(3, 1, figsize=(12, 10), sharex=True)

axes[0].plot(monthly["month"], monthly["orders"], marker="o", linewidth=2)
axes[0].set_title("Monthly Order Volume")
axes[0].set_ylabel("Orders")

axes[1].plot(monthly["month"], monthly["revenue"]/1e6, marker="o",
             linewidth=2, color="#55A868")
axes[1].set_title("Monthly Revenue (R$ millions)")
axes[1].set_ylabel("Revenue (M)")

axes[2].plot(monthly["month"], monthly["aov"], marker="o",
             linewidth=2, color="#C44E52")
axes[2].set_title("Average Order Value (R$)")
axes[2].set_ylabel("AOV (R$)")

plt.xticks(rotation=45)
plt.tight_layout()
plt.savefig("outputs/monthly_trends.png", dpi=150, bbox_inches="tight")

# ── SEASONALITY CHECK ─────────────────────────────────────────────────────
# Same month, different years — is November always higher?
df["month_num"] = df["order_purchase_timestamp"].dt.month
df["year"]      = df["order_purchase_timestamp"].dt.year

monthly_seasonality = df.groupby(["year","month_num"])["order_id"].count().reset_index()

pivot = monthly_seasonality.pivot(index="month_num", columns="year", values="order_id")
month_names = ["Jan","Feb","Mar","Apr","May","Jun",
               "Jul","Aug","Sep","Oct","Nov","Dec"]
pivot.index = month_names

print("\nOrders by month and year (seasonality check):")
print(pivot.to_string())
```

---

## 15. Bivariate Analysis

### Relationships between two variables

Bivariate analysis asks: how does variable A relate to variable B? Depending on the types of variables involved, different approaches apply.

| Variable A | Variable B | Approach |
|-----------|-----------|---------|
| Continuous | Continuous | Scatter plot, correlation coefficient |
| Continuous | Categorical | Box plot, group means, t-test |
| Categorical | Categorical | Cross-tab, chi-square test |
| Continuous | Time | Line chart, trend analysis |

---

### Continuous vs continuous — correlation and scatter

```python
import scipy.stats as stats

# Does review score correlate with delivery speed?
orders   = pd.read_csv("data/olist/olist_orders_dataset.csv",
                       parse_dates=["order_purchase_timestamp",
                                    "order_delivered_customer_date"])
reviews  = pd.read_csv("data/olist/olist_order_reviews_dataset.csv")

delivered = orders[orders["order_status"] == "delivered"].copy()
delivered["delivery_days"] = (
    (delivered["order_delivered_customer_date"] -
     delivered["order_purchase_timestamp"]).dt.days
)

analysis = delivered.merge(
    reviews[["order_id","review_score"]].groupby("order_id")["review_score"].mean().reset_index(),
    on="order_id", how="inner"
).dropna(subset=["delivery_days","review_score"])

# Filter reasonable delivery times (< 60 days)
analysis = analysis[analysis["delivery_days"] < 60]

# ── CORRELATION ───────────────────────────────────────────────────────────
r, p = stats.pearsonr(analysis["delivery_days"], analysis["review_score"])
print(f"Pearson correlation: r = {r:.3f},  p = {p:.4f}")
print(f"Interpretation: {'Statistically significant' if p < 0.05 else 'Not significant'}")
print(f"  r = {r:.2f} means: as delivery days increase by 1,")
print(f"  review score changes by {r:.2f} units on average")

# ── SCATTER PLOT ──────────────────────────────────────────────────────────
fig, ax = plt.subplots(figsize=(10, 5))
# Sample 5,000 points to avoid overplotting
sample = analysis.sample(min(5000, len(analysis)), random_state=42)
ax.scatter(sample["delivery_days"], sample["review_score"],
           alpha=0.15, s=10, color="#4C72B0")

# Add a trend line
z = np.polyfit(analysis["delivery_days"], analysis["review_score"], 1)
p = np.poly1d(z)
x_line = np.linspace(0, 60, 100)
ax.plot(x_line, p(x_line), "r-", linewidth=2, label=f"r = {r:.3f}")
ax.set_xlabel("Delivery Days")
ax.set_ylabel("Review Score (1–5)")
ax.set_title("Delivery Speed vs Customer Review Score")
ax.legend()
plt.tight_layout()
plt.savefig("outputs/delivery_vs_review.png", dpi=150, bbox_inches="tight")

# ── BUCKETED ANALYSIS ─────────────────────────────────────────────────────
# Scatter can be noisy — bucketed averages tell a cleaner story
analysis["delivery_bucket"] = pd.cut(
    analysis["delivery_days"],
    bins=[0, 5, 10, 15, 20, 30, 60],
    labels=["1-5d","6-10d","11-15d","16-20d","21-30d","31-60d"]
)

by_speed = analysis.groupby("delivery_bucket", observed=True).agg(
    avg_review = ("review_score",  "mean"),
    orders     = ("order_id",      "count"),
).reset_index()

print("\nReview score by delivery speed:")
print(by_speed.round(2).to_string(index=False))
# This tells a cleaner story than r = -0.29:
# "Orders delivered in 1-5 days get 4.7 avg; delivered in 31-60 days get 2.9 avg"
```

---

### Continuous vs categorical — comparing groups

```python
# Does review score differ by product category?
items    = pd.read_csv("data/olist/olist_order_items_dataset.csv")
products = pd.read_csv("data/olist/olist_products_dataset.csv")
category = pd.read_csv("data/olist/product_category_name_translation.csv")

products_en = products.merge(category, on="product_category_name", how="left")
products_en["category_en"] = (products_en["product_category_name_english"]
                               .str.replace("_", " ").str.title()
                               .fillna("Unknown"))

analysis = (reviews
    .merge(items[["order_id","product_id"]], on="order_id", how="left")
    .merge(products_en[["product_id","category_en"]], on="product_id", how="left")
    .dropna(subset=["review_score","category_en"]))

# Average score by category (top 15 by volume)
top_categories = (analysis["category_en"].value_counts().head(15).index)
by_category = (analysis[analysis["category_en"].isin(top_categories)]
    .groupby("category_en")["review_score"]
    .agg(["mean","std","count"])
    .sort_values("mean", ascending=True)
    .reset_index())

# ── BOX PLOT ──────────────────────────────────────────────────────────────
fig, ax = plt.subplots(figsize=(12, 7))
data_for_box = [
    analysis[analysis["category_en"] == cat]["review_score"].dropna().values
    for cat in by_category["category_en"]
]
ax.boxplot(data_for_box, vert=False, labels=by_category["category_en"])
ax.set_xlabel("Review Score")
ax.set_title("Review Score Distribution by Product Category")
ax.axvline(analysis["review_score"].mean(), color="red", linestyle="--",
           label="Overall mean")
ax.legend()
plt.tight_layout()
plt.savefig("outputs/review_by_category.png", dpi=150, bbox_inches="tight")
```

---

## 16. Multivariate Analysis

### Moving beyond two variables

Real business questions involve multiple dimensions simultaneously. "Is delivery speed affecting reviews more for electronics than for clothing?" requires three variables: delivery speed, review score, and product category.

```python
# Three-way: delivery speed × review score × product category
analysis = (delivered
    .merge(reviews[["order_id","review_score"]]
           .groupby("order_id")["review_score"].mean().reset_index(),
           on="order_id")
    .merge(items[["order_id","product_id"]], on="order_id")
    .merge(products_en[["product_id","category_en"]], on="product_id")
    .dropna(subset=["delivery_days","review_score","category_en"]))

# Pivot: avg review score by (delivery speed bucket) × (category)
analysis["speed_bucket"] = pd.cut(
    analysis["delivery_days"],
    bins=[0, 7, 14, 30, 60],
    labels=["Fast (≤7d)","Medium (8-14d)","Slow (15-30d)","Very Slow (>30d)"]
)

top_cats = analysis["category_en"].value_counts().head(8).index
pivot = (analysis[analysis["category_en"].isin(top_cats)]
    .groupby(["category_en","speed_bucket"], observed=True)["review_score"]
    .mean()
    .round(2)
    .unstack("speed_bucket"))

print("Average review score by category and delivery speed:")
print(pivot.to_string())
# Now you can see: does the delivery-review relationship hold across all categories,
# or is it stronger for some (e.g., electronics, where expectations are higher)?

# ── HEATMAP ───────────────────────────────────────────────────────────────
fig, ax = plt.subplots(figsize=(10, 6))
sns.heatmap(pivot, annot=True, fmt=".1f", cmap="RdYlGn",
            vmin=1, vmax=5, ax=ax)
ax.set_title("Average Review Score by Category and Delivery Speed")
plt.tight_layout()
plt.savefig("outputs/category_speed_review_heatmap.png", dpi=150, bbox_inches="tight")
```

---

### Correlation matrix — finding related variables

```python
# Correlation matrix on the HR dataset
hr = pd.read_csv("data/hr/WA_Fn-UseC_-HR-Employee-Attrition.csv")

# Select numeric columns
numeric_cols = ["Age","MonthlyIncome","JobSatisfaction","WorkLifeBalance",
                "YearsAtCompany","YearsInCurrentRole","PercentSalaryHike",
                "TrainingTimesLastYear","EnvironmentSatisfaction"]

corr_matrix = hr[numeric_cols].corr()

fig, ax = plt.subplots(figsize=(10, 8))
mask = np.triu(np.ones_like(corr_matrix, dtype=bool))  # show only lower triangle
sns.heatmap(corr_matrix, mask=mask, annot=True, fmt=".2f",
            cmap="coolwarm", center=0, vmin=-1, vmax=1, ax=ax,
            linewidths=0.5)
ax.set_title("Correlation Matrix — HR Employee Data")
plt.tight_layout()
plt.savefig("outputs/hr_correlation_matrix.png", dpi=150, bbox_inches="tight")

# Find the strongest correlations
corr_pairs = (corr_matrix.where(~mask)
    .stack()
    .reset_index()
    .rename(columns={"level_0":"var1","level_1":"var2",0:"correlation"})
    .assign(abs_corr=lambda x: x["correlation"].abs())
    .sort_values("abs_corr", ascending=False))

print("Strongest correlations:")
print(corr_pairs.head(10)[["var1","var2","correlation"]].to_string(index=False))
```

---

## 17. Outlier Detection and Missing Data Patterns

### Outlier detection — three methods

An **outlier** is a value that is unusually far from the rest. Outliers can be: data errors (corrupted/wrongly entered), legitimate edge cases (a $50,000 order on Olist), or signals of real phenomena (fraud transactions have unusual amounts).

**Rule: never remove outliers without understanding them first.**

```python
# ── METHOD 1: IQR (Inter-Quartile Range) — robust, non-parametric ─────────
def flag_outliers_iqr(series: pd.Series, factor: float = 1.5) -> pd.Series:
    """
    Flag outliers using IQR method.
    factor=1.5: standard outlier boundary (beyond 1.5×IQR from Q1/Q3)
    factor=3.0: extreme outlier boundary
    Returns boolean Series: True = outlier
    """
    Q1  = series.quantile(0.25)
    Q3  = series.quantile(0.75)
    IQR = Q3 - Q1
    lower_fence = Q1 - factor * IQR
    upper_fence = Q3 + factor * IQR
    return (series < lower_fence) | (series > upper_fence)

revenue = payments_agg["payment_value"].dropna()
is_outlier_iqr = flag_outliers_iqr(revenue)

print(f"IQR outliers: {is_outlier_iqr.sum():,} ({is_outlier_iqr.mean():.1%})")
print(f"  Outlier range: > R${revenue.quantile(0.75) + 1.5*(revenue.quantile(0.75)-revenue.quantile(0.25)):,.0f}")
print(f"  Max outlier: R${revenue[is_outlier_iqr].max():,.0f}")


# ── METHOD 2: Z-score — assumes normal distribution ───────────────────────
def flag_outliers_zscore(series: pd.Series, threshold: float = 3.0) -> pd.Series:
    """
    Flag values more than `threshold` standard deviations from the mean.
    Works best on roughly normal distributions; misleading on skewed data.
    """
    z_scores = (series - series.mean()) / series.std()
    return z_scores.abs() > threshold

is_outlier_z = flag_outliers_zscore(revenue)
print(f"Z-score outliers (>3σ): {is_outlier_z.sum():,} ({is_outlier_z.mean():.1%})")


# ── METHOD 3: Percentile cutoff — domain knowledge based ─────────────────
# When you have business knowledge: "orders > R$10,000 are likely B2B, not retail"
is_outlier_domain = revenue > 10_000
print(f"Domain-defined outliers (>R$10k): {is_outlier_domain.sum():,}")

# ── WHAT TO DO WITH OUTLIERS ──────────────────────────────────────────────
# Option A: Investigate and understand
print("\nOutlier investigation:")
outliers = payments_agg[is_outlier_iqr].merge(
    orders[["order_id","order_status"]], on="order_id", how="left"
)
print(outliers["order_status"].value_counts())
# Are outliers more likely to be cancelled? Refunded? B2B?

# Option B: Winsorize (cap at percentile)
revenue_winsorized = revenue.clip(
    upper=revenue.quantile(0.99)  # cap at 99th percentile
)
print(f"\nAfter winsorizing at P99:")
print(f"  Original mean: R${revenue.mean():.2f}")
print(f"  Winsorized mean: R${revenue_winsorized.mean():.2f}")
print(f"  This is more representative of 'typical' order value")

# Option C: Separate analysis track (analyse outliers separately)
# High-value orders may need their own analysis as a B2B segment
```

---

### Missing data patterns — not all missing is equal

```python
# ── MISSING DATA PATTERNS ─────────────────────────────────────────────────
df_full = (orders
    .merge(payments_agg, on="order_id", how="left")
    .merge(reviews[["order_id","review_score"]]
           .groupby("order_id")["review_score"].mean().reset_index(),
           on="order_id", how="left"))

print("Missing data analysis:")
for col in df_full.columns:
    total_null   = df_full[col].isnull().sum()
    if total_null == 0:
        continue
    pct_null     = total_null / len(df_full)

    # Is the missingness correlated with other variables?
    # If so: Missing Not At Random (MNAR) → bias risk
    # If not: Missing Completely At Random (MCAR) → safer to ignore
    df_full["is_missing"] = df_full[col].isnull().astype(int)

    # Does missing correlate with order status?
    missing_by_status = df_full.groupby("order_status")["is_missing"].mean()
    max_status_rate   = missing_by_status.max()
    min_status_rate   = missing_by_status.min()

    pattern = "⚠️  MNAR risk" if (max_status_rate - min_status_rate) > 0.1 else "✓ likely MCAR"
    print(f"\n  {col}: {pct_null:.1%} missing — {pattern}")
    if "MNAR" in pattern:
        print(f"    Missing rate by status:")
        for status, rate in missing_by_status.sort_values(ascending=False).items():
            print(f"      {status}: {rate:.1%}")

# The insight: if review scores are missing more for cancelled orders than
# delivered orders, that's MNAR — not random. Any analysis using review_score
# will be biased toward completed, reviewed orders (likely happier customers).
```

**Three types of missing data:**

| Type | What it means | Risk | What to do |
|------|--------------|------|-----------|
| **MCAR** (Missing Completely At Random) | Missingness has no relationship to any variable | Low — bias is minimal | Exclude or impute with mean/median |
| **MAR** (Missing At Random) | Missingness relates to other observed variables | Medium — addressable | Impute using related variables |
| **MNAR** (Missing Not At Random) | Missingness relates to the missing value itself | High — creates bias | Investigate; may need separate model |

---
---

# Part V — Business Metrics in Depth

## 18. Revenue Metrics

### Why revenue metrics need precision

"Revenue" means different things to different people. The CFO's "revenue" may be net of returns. The sales team's "revenue" may be gross. The marketplace's "revenue" may be the take rate only. Analysts must define precisely which metric they're computing — and use the same definition every time.

🎬 **Movies analogy:** A film's "box office" has multiple definitions: opening weekend domestic, cumulative domestic, worldwide total, theatrical vs streaming, gross vs net. Reporting "The Dark Knight grossed $1 billion" without specifying means nothing — it could be domestic theatrical, worldwide theatrical, or including home video. Precision matters.

---

### GMV — Gross Merchandise Value

**What it is:** The total value of all transactions on a marketplace platform, before fees, returns, or cancellations. GMV measures the scale of the marketplace, not the revenue the marketplace earns.

**Formula:** `GMV = sum of all order values (including cancelled and returned)`

**Why it matters:** For a marketplace like Olist, GMV is the primary size metric. Olist's revenue is a fraction of GMV (the take rate × GMV). Growing GMV means growing the overall marketplace.

**The confusion:** GMV ≠ revenue. Airbnb's GMV might be $75B (total booking value), but Airbnb's revenue is ~$9B (their ~12% take rate). When a company reports GMV, check whether they also report revenue separately.

```python
import pandas as pd

orders   = pd.read_csv("data/olist/olist_orders_dataset.csv",
                       parse_dates=["order_purchase_timestamp"])
payments = pd.read_csv("data/olist/olist_order_payments_dataset.csv")

# GMV: ALL orders (including cancelled — the value was attempted)
all_payments = payments.groupby("order_id")["payment_value"].sum().reset_index()
gmv = all_payments["payment_value"].sum()
print(f"GMV (all orders): R${gmv:,.0f}")

# NMV (Net Merchandise Value): delivered orders only
delivered_ids = orders[orders["order_status"] == "delivered"]["order_id"]
nmv = all_payments[all_payments["order_id"].isin(delivered_ids)]["payment_value"].sum()
print(f"NMV (delivered only): R${nmv:,.0f}")
print(f"NMV/GMV ratio: {nmv/gmv:.1%}")
# NMV/GMV < 85% suggests high cancellation rates — an operational problem

# Monthly GMV trend
orders_with_rev = orders.merge(all_payments, on="order_id", how="left")
orders_with_rev["month"] = orders_with_rev["order_purchase_timestamp"].dt.to_period("M").astype(str)
monthly_gmv = orders_with_rev.groupby("month")["payment_value"].sum() / 1e6
print("\nMonthly GMV (R$ millions):")
print(monthly_gmv.tail(8).round(2))
```

---

### AOV — Average Order Value

**What it is:** The mean transaction value. A simple metric with surprising complexity when you dig in.

**Formula:** `AOV = Total Revenue / Number of Orders`

**Why mean is often misleading:** revenue distributions are right-skewed. A handful of large B2B orders can dramatically increase the mean while 90% of orders remain small. Always check the median alongside the mean.

```python
# AOV analysis — why you need more than the mean
delivered = orders[orders["order_status"] == "delivered"].merge(
    all_payments, on="order_id", how="left"
).dropna(subset=["payment_value"])

aov_mean   = delivered["payment_value"].mean()
aov_median = delivered["payment_value"].median()
aov_p25    = delivered["payment_value"].quantile(0.25)
aov_p75    = delivered["payment_value"].quantile(0.75)

print(f"AOV mean:   R${aov_mean:.2f}")
print(f"AOV median: R${aov_median:.2f}")
print(f"AOV P25:    R${aov_p25:.2f}")
print(f"AOV P75:    R${aov_p75:.2f}")
print(f"\nMean/Median: {aov_mean/aov_median:.2f}x")
print("If Mean/Median >> 1: a few high-value orders are pulling up the mean")
print("The 'typical order' is better described by the median")

# AOV by category — which categories have highest value orders?
items    = pd.read_csv("data/olist/olist_order_items_dataset.csv")
products = pd.read_csv("data/olist/olist_products_dataset.csv")
category = pd.read_csv("data/olist/product_category_name_translation.csv")

products_en = products.merge(category, on="product_category_name", how="left")
products_en["cat_en"] = (products_en["product_category_name_english"]
                          .str.replace("_"," ").str.title().fillna("Unknown"))

aov_by_cat = (delivered
    .merge(items[["order_id","product_id"]], on="order_id", how="left")
    .merge(products_en[["product_id","cat_en"]], on="product_id", how="left")
    .groupby("cat_en")["payment_value"]
    .agg(["mean","median","count"])
    .query("count >= 100")
    .sort_values("median", ascending=False)
    .head(10))

print("\nTop 10 categories by median AOV:")
print(aov_by_cat.round(2).to_string())
```

---

### LTV — Customer Lifetime Value

**What it is:** The total expected revenue (or profit) a business will earn from a customer over the entire relationship. LTV is the single most important metric for understanding whether a business model is sustainable.

**The LTV:CAC ratio:** if acquiring a customer costs R$50 (CAC) but they generate R$200 in lifetime value (LTV), the ratio is 4:1. Businesses generally target LTV:CAC > 3:1. Below 1:1 means you're losing money on every customer.

**Simple LTV formula:**
```
LTV = AOV × Purchase Frequency × Customer Lifespan
```

**Better formula (cohort-based):**
```
LTV = sum of all revenue from all orders, by customer cohort
```

```python
customers = pd.read_csv("data/olist/olist_customers_dataset.csv")

# Customer-level revenue
customer_revenue = (delivered
    .merge(customers[["customer_id","customer_unique_id"]], on="customer_id", how="left")
    .groupby("customer_unique_id")["payment_value"]
    .agg(
        orders         = "count",
        total_revenue  = "sum",
        avg_order      = "mean",
        first_order    = "min",   # will apply to timestamp column
    ))

# Note: Olist has very low repeat purchase rate (marketplace characteristic)
# Most customers order once — this dramatically affects LTV calculation

# LTV components
print("Customer LTV analysis:")
print(f"  Customers with 1 order:  {(customer_revenue['orders'] == 1).sum():,}  "
      f"({(customer_revenue['orders'] == 1).mean():.1%})")
print(f"  Customers with 2+ orders:{(customer_revenue['orders'] >= 2).sum():,}  "
      f"({(customer_revenue['orders'] >= 2).mean():.1%})")
print(f"  Avg orders per customer: {customer_revenue['orders'].mean():.2f}")
print(f"  Median LTV (total revenue per customer): R${customer_revenue['total_revenue'].median():.2f}")
print(f"  Mean LTV: R${customer_revenue['total_revenue'].mean():.2f}")
print(f"  P90 LTV: R${customer_revenue['total_revenue'].quantile(0.9):.2f}")
```

---

### Churn Rate

**What it is:** The percentage of customers who stop doing business with you in a given period.

**Two types:**
- **Revenue churn:** the % of revenue lost from existing customers (MRR churn)
- **Customer churn:** the % of customers lost

**For subscription businesses:**
```
Monthly Churn Rate = Customers lost this month / Customers at start of month
Annual Churn Rate ≈ 1 - (1 - Monthly Churn)^12
```

**For transactional businesses (like Olist):**
Define "churned" as: a customer who made their last purchase > N days ago.

```python
# Churn analysis for a transactional business
from datetime import datetime

customers_timeline = (delivered
    .merge(customers[["customer_id","customer_unique_id"]], on="customer_id")
    .groupby("customer_unique_id")["order_purchase_timestamp"]
    .agg(["min","max","count"])
    .reset_index())
customers_timeline.columns = ["customer_unique_id","first_order","last_order","num_orders"]

# Define churn threshold: no order in last 180 days
reference_date = delivered["order_purchase_timestamp"].max()
customers_timeline["days_since_last"] = (
    reference_date - customers_timeline["last_order"]
).dt.days

customers_timeline["is_churned"] = customers_timeline["days_since_last"] > 180

print(f"Churn analysis (180-day threshold):")
print(f"  Total customers: {len(customers_timeline):,}")
print(f"  Active (ordered in last 180d): "
      f"{(~customers_timeline['is_churned']).sum():,} "
      f"({(~customers_timeline['is_churned']).mean():.1%})")
print(f"  Churned:                        "
      f"{customers_timeline['is_churned'].sum():,} "
      f"({customers_timeline['is_churned'].mean():.1%})")
```

---

## 19. Customer Metrics

### NPS — Net Promoter Score

**What it is:** A measure of customer loyalty based on one question: "How likely are you to recommend us to a friend or colleague?" (0–10 scale).

**How it works:**
- **Promoters** (9–10): loyal enthusiasts who will refer others
- **Passives** (7–8): satisfied but not enthusiastic
- **Detractors** (0–6): unhappy customers who may damage your brand

**Formula:** `NPS = % Promoters − % Detractors`

**Interpretation:**
- NPS > 50 = excellent
- NPS > 20 = good
- NPS 0–20 = average
- NPS < 0 = needs improvement

```python
# NPS using Olist review scores as a proxy
# (actual NPS uses a 0-10 scale; we'll adapt the 1-5 review scale)
reviews = pd.read_csv("data/olist/olist_order_reviews_dataset.csv")

# Adapt 1-5 scale: 5 = promoter, 4 = passive, 1-3 = detractor
def review_to_nps(score):
    if score == 5:   return "promoter"
    elif score == 4: return "passive"
    else:            return "detractor"

reviews["nps_category"] = reviews["review_score"].map(review_to_nps)
nps_dist = reviews["nps_category"].value_counts(normalize=True)

nps_score = (nps_dist.get("promoter", 0) - nps_dist.get("detractor", 0)) * 100

print(f"NPS-style score (review-based):")
print(f"  Promoters (5★):    {nps_dist.get('promoter',0):.1%}")
print(f"  Passives  (4★):    {nps_dist.get('passive',0):.1%}")
print(f"  Detractors (1-3★): {nps_dist.get('detractor',0):.1%}")
print(f"  NPS: {nps_score:.1f}")

# NPS by product category — which categories drive detractors?
analysis = (reviews
    .merge(items[["order_id","product_id"]], on="order_id", how="left")
    .merge(products_en[["product_id","cat_en"]], on="product_id", how="left")
    .dropna(subset=["cat_en"]))

analysis["nps_val"] = analysis["review_score"].map(
    {5: 100, 4: 0, 3: -100, 2: -100, 1: -100}
)

nps_by_cat = (analysis
    .groupby("cat_en")["nps_val"]
    .agg(["mean","count"])
    .query("count >= 200")
    .sort_values("mean")
    .rename(columns={"mean":"nps","count":"reviews"}))

print("\nNPS by category (min 200 reviews, sorted worst to best):")
print(nps_by_cat.head(10).round(1).to_string())
```

---

## 20. Operational Metrics

### SLA Analysis — are we meeting our commitments?

A **Service Level Agreement (SLA)** is a commitment to deliver something within a defined timeframe or quality threshold. SLA analysis asks: what percentage of the time are we meeting that commitment?

```python
# Olist delivery SLA analysis
# SLA: deliver within the estimated delivery date

delivered = orders[orders["order_status"] == "delivered"].copy()
delivered["actual_days"]    = (delivered["order_delivered_customer_date"]
                               - delivered["order_purchase_timestamp"]).dt.days
delivered["promised_days"]  = (delivered["order_estimated_delivery_date"]
                               - delivered["order_purchase_timestamp"]).dt.days
delivered["is_on_time"]     = (delivered["order_delivered_customer_date"]
                               <= delivered["order_estimated_delivery_date"]).astype(int)
delivered["days_early_late"]= (delivered["order_estimated_delivery_date"]
                               - delivered["order_delivered_customer_date"]).dt.days
# Positive = delivered early; Negative = delivered late

# Overall SLA
sla_rate = delivered["is_on_time"].mean()
print(f"Overall SLA (on-time delivery rate): {sla_rate:.1%}")
print(f"  Target: 90%+ for most e-commerce")
print(f"  Status: {'✅ Meeting SLA' if sla_rate >= 0.9 else '⚠️  Below SLA target'}")

# SLA by month
monthly_sla = delivered.groupby(
    delivered["order_purchase_timestamp"].dt.to_period("M").astype(str)
)["is_on_time"].mean()
print("\nMonthly SLA rate:")
print(monthly_sla.tail(8).round(3).to_string())

# SLA by seller state
customers = pd.read_csv("data/olist/olist_customers_dataset.csv")
sla_by_state = (delivered
    .merge(customers[["customer_id","customer_state"]], on="customer_id", how="left")
    .groupby("customer_state")["is_on_time"]
    .agg(["mean","count"])
    .query("count >= 100")
    .sort_values("mean")
    .rename(columns={"mean":"sla_rate","count":"orders"}))

print("\nWorst SLA by customer state (min 100 orders):")
print(sla_by_state.head(5).round(3).to_string())
```

---

## 21. Cohort Analysis

### What cohort analysis is — and why it's so powerful

A **cohort** is a group of users/customers who share a common characteristic at a specific time — usually their first purchase date. Cohort analysis tracks how these groups behave over time.

**Why it matters:** aggregate metrics hide the health of your business. If "average monthly revenue" is flat, that could mean:
- Everything is fine: retention is steady
- Your business is deteriorating: old customers are churning but new customers are masking the decline
- Your business is improving: retention is better but fewer new customers are joining

Only cohort analysis reveals which is true.

🎬 **Movies analogy:** Cohort analysis is like tracking the long-term box office performance of different director cohorts. Nolan's films from 2000–2005 performed one way in the first weekend vs over their lifetime. Nolan's films from 2015–2020 perform differently. The aggregate "Nolan average" hides the cohort-specific story.

---

### Building a cohort retention table

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

# Join to get customer_unique_id (Olist uses customer_id per order, unique_id for person)
delivered = delivered.merge(
    customers[["customer_id","customer_unique_id"]], on="customer_id", how="left"
)

# Step 1: Assign each customer their ACQUISITION COHORT (first order month)
first_order = (delivered
    .groupby("customer_unique_id")["order_month"]
    .min()
    .reset_index(name="cohort"))

# Step 2: Join cohort to all orders
delivered = delivered.merge(first_order, on="customer_unique_id", how="left")

# Step 3: Compute months since acquisition
delivered["months_since_acq"] = (
    delivered["order_month"] - delivered["cohort"]
).apply(lambda x: x.n)

# Step 4: Build cohort table
cohort_data = (delivered
    .groupby(["cohort","months_since_acq"])["customer_unique_id"]
    .nunique()
    .reset_index(name="customers"))

# Step 5: Pivot to cohort × month matrix
cohort_pivot = cohort_data.pivot_table(
    index="cohort", columns="months_since_acq", values="customers"
)

# Step 6: Normalise by cohort size (month 0 = 100%)
cohort_sizes = cohort_pivot[0]
retention_table = cohort_pivot.divide(cohort_sizes, axis=0)

# Step 7: Visualise
fig, ax = plt.subplots(figsize=(16, 10))
sns.heatmap(
    retention_table.iloc[:18, :12],  # first 18 cohorts, first 12 months
    annot=True,
    fmt=".0%",
    cmap="Blues",
    vmin=0, vmax=0.15,   # adjust scale for Olist (low repeat rate)
    ax=ax,
    linewidths=0.5,
)
ax.set_title("Customer Retention by Cohort\n(% of cohort making a purchase in each month)")
ax.set_xlabel("Months Since First Purchase")
ax.set_ylabel("Acquisition Cohort")
plt.tight_layout()
plt.savefig("outputs/cohort_retention.png", dpi=150, bbox_inches="tight")

# ── INTERPRETING THE COHORT TABLE ─────────────────────────────────────────
print("\nCohort analysis — key findings:")
avg_month1_retention = retention_table[1].mean()
print(f"  Average Month-1 retention: {avg_month1_retention:.1%}")
print(f"  (% of customers who buy again in month after first purchase)")

# Is retention improving across cohorts?
# Compare first column (Month 1 retention) across different cohort dates
early_cohorts = retention_table.iloc[:6, 1].mean()   # first 6 cohorts
late_cohorts  = retention_table.iloc[-6:, 1].dropna().mean()  # last 6 cohorts
print(f"\n  Month-1 retention — early cohorts: {early_cohorts:.1%}")
print(f"  Month-1 retention — recent cohorts: {late_cohorts:.1%}")
trend = "improving" if late_cohorts > early_cohorts else "declining"
print(f"  Trend: {trend}")
```

---

## 22. RFM Analysis

### What RFM is

**RFM** scores customers on three dimensions:
- **Recency (R):** how recently did they last purchase? (lower days = better)
- **Frequency (F):** how many times have they purchased?
- **Monetary (M):** how much have they spent in total?

RFM is the foundational customer segmentation technique in retail and e-commerce. Every marketing team uses some version of it.

---

### Building RFM from scratch

```python
import pandas as pd
import numpy as np

orders    = pd.read_csv("data/olist/olist_orders_dataset.csv",
                        parse_dates=["order_purchase_timestamp"])
payments  = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
customers = pd.read_csv("data/olist/olist_customers_dataset.csv")

payments_agg = payments.groupby("order_id")["payment_value"].sum().reset_index()
delivered = (orders[orders["order_status"] == "delivered"]
    .merge(payments_agg, on="order_id", how="left")
    .merge(customers[["customer_id","customer_unique_id"]], on="customer_id", how="left"))

# Reference date: the "today" of the analysis
reference_date = delivered["order_purchase_timestamp"].max()

# Step 1: Compute raw RFM values
rfm = delivered.groupby("customer_unique_id").agg(
    last_order  = ("order_purchase_timestamp", "max"),
    frequency   = ("order_id",                 "count"),
    monetary    = ("payment_value",             "sum"),
).reset_index()

rfm["recency_days"] = (reference_date - rfm["last_order"]).dt.days

# Step 2: Score each dimension 1–5
# Recency: LOWER days = HIGHER score (recent = good)
# Frequency: HIGHER orders = HIGHER score
# Monetary: HIGHER spend = HIGHER score

rfm["R"] = pd.qcut(rfm["recency_days"].rank(method="first", ascending=False),
                    5, labels=[1,2,3,4,5]).astype(int)
rfm["F"] = pd.qcut(rfm["frequency"].rank(method="first"),
                    5, labels=[1,2,3,4,5]).astype(int)
rfm["M"] = pd.qcut(rfm["monetary"].rank(method="first"),
                    5, labels=[1,2,3,4,5]).astype(int)

rfm["RFM_score"] = rfm["R"].astype(str) + rfm["F"].astype(str) + rfm["M"].astype(str)

# Step 3: Define segments (business-meaningful labels)
def rfm_segment(row):
    r, f, m = row["R"], row["F"], row["M"]
    if r >= 4 and f >= 4 and m >= 4:
        return "Champions"
    elif r >= 3 and f >= 3:
        return "Loyal Customers"
    elif r >= 4 and f <= 2:
        return "New / Promising"
    elif r <= 2 and f >= 3:
        return "At Risk"
    elif r <= 2 and f <= 2 and m <= 2:
        return "Lost"
    elif r >= 3 and f <= 2:
        return "Potential Loyalist"
    else:
        return "Needs Attention"

rfm["segment"] = rfm.apply(rfm_segment, axis=1)

# Step 4: Analyse segments
segment_summary = (rfm.groupby("segment")
    .agg(
        customers      = ("customer_unique_id", "count"),
        avg_recency    = ("recency_days",        "mean"),
        avg_frequency  = ("frequency",           "mean"),
        avg_monetary   = ("monetary",            "mean"),
        total_revenue  = ("monetary",            "sum"),
    )
    .round(1)
    .sort_values("total_revenue", ascending=False))

print("RFM Segment Summary:")
print(segment_summary.to_string())

# Step 5: Validate MECE
print(f"\nMECE check: {rfm['segment'].notna().all()} (all customers assigned)")
print(f"Total customers: {len(rfm):,}")
print(f"Sum of segments: {segment_summary['customers'].sum():,}")

# Step 6: Business actions per segment
segment_actions = {
    "Champions":          "Reward, ask for reviews, make them brand ambassadors",
    "Loyal Customers":    "Offer loyalty rewards, upsell premium products",
    "New / Promising":    "Onboarding campaigns, first repeat purchase incentive",
    "At Risk":            "Win-back campaigns, survey on why they stopped",
    "Lost":               "Reactivation offer (significant discount), or deprioritise",
    "Potential Loyalist": "Build relationship, personalised recommendations",
    "Needs Attention":    "Limited time offers, understand pain points",
}
print("\nRecommended actions by segment:")
for seg, action in segment_actions.items():
    if seg in rfm["segment"].values:
        count = (rfm["segment"] == seg).sum()
        print(f"  {seg:<20} ({count:>6,} customers): {action}")
```

---
---

# Part VI — SQL for Analytics

## 23. Window Functions for Analytics

### What window functions are — and why they're essential

Window functions perform calculations across a set of rows that are **related to the current row** — without collapsing the rows like GROUP BY does. They are the most powerful SQL tool for analytical queries.

🎬 **Movies analogy:** GROUP BY is like asking "what is the average box office for all Nolan films?" — you get one number, losing all detail. A window function is like asking "for each Nolan film, what was its rank among all his films by opening weekend?" — you get the detail AND the aggregate context per row.

```sql
-- Without window function (GROUP BY loses the detail)
SELECT director, AVG(box_office) AS avg_bo
FROM films
GROUP BY director;
-- Result: one row per director, film detail gone

-- With window function (keeps each film, adds aggregate context)
SELECT
    film_title,
    director,
    box_office,
    AVG(box_office) OVER (PARTITION BY director) AS director_avg_bo,
    RANK()          OVER (PARTITION BY director ORDER BY box_office DESC) AS rank_in_filmography,
    box_office - AVG(box_office) OVER (PARTITION BY director) AS vs_director_avg
FROM films;
-- Result: one row per film, with director-level context added
```

---

### The window function anatomy

```sql
function_name(column)
    OVER (
        PARTITION BY grouping_columns    -- optional: "within each group"
        ORDER BY ordering_columns        -- required for rank/lag/lead/running totals
        ROWS/RANGE BETWEEN ... AND ...   -- optional: frame definition
    )
```

| Clause | What it does | Example |
|--------|-------------|---------|
| `PARTITION BY` | Resets the window for each group | `PARTITION BY customer_id` → restart count for each customer |
| `ORDER BY` | Orders rows within the partition | `ORDER BY order_date` → chronological order |
| `ROWS BETWEEN` | Defines the frame (which rows are in the calculation) | `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` → 7-row rolling window |

---

### The full window function reference

```sql
-- ── RANKING FUNCTIONS ─────────────────────────────────────────────────────
-- All three handle ties differently

-- ROW_NUMBER: always unique, arbitrary tie-breaking
SELECT order_id,
       customer_id,
       payment_value,
       ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date) AS order_num
FROM fct_orders;
-- Each customer's orders get sequential numbers starting at 1

-- RANK: ties get the same rank; next rank skips (1, 1, 3, 4)
SELECT seller_id,
       monthly_revenue,
       RANK() OVER (ORDER BY monthly_revenue DESC) AS revenue_rank
FROM seller_monthly;
-- Two sellers with same revenue both get rank 2; next seller gets rank 4

-- DENSE_RANK: ties get same rank; no gaps (1, 1, 2, 3)
SELECT seller_id,
       monthly_revenue,
       DENSE_RANK() OVER (ORDER BY monthly_revenue DESC) AS revenue_rank
FROM seller_monthly;

-- NTILE: divide rows into N equal buckets
SELECT customer_id,
       total_spend,
       NTILE(4) OVER (ORDER BY total_spend) AS spend_quartile
       -- 1 = bottom 25%, 4 = top 25%
FROM customers;


-- ── OFFSET FUNCTIONS (look forward and backward in the ordered set) ────────
-- LAG: get the value from a previous row
SELECT
    order_month,
    monthly_revenue,
    LAG(monthly_revenue, 1) OVER (ORDER BY order_month) AS prev_month_revenue,
    monthly_revenue - LAG(monthly_revenue, 1) OVER (ORDER BY order_month) AS mom_change,
    ROUND(
        100.0 * (monthly_revenue - LAG(monthly_revenue, 1) OVER (ORDER BY order_month))
        / NULLIF(LAG(monthly_revenue, 1) OVER (ORDER BY order_month), 0)
    , 1) AS mom_pct_change
FROM monthly_revenue;

-- LEAD: get the value from a following row
SELECT
    customer_id,
    order_date,
    LEAD(order_date, 1) OVER (PARTITION BY customer_id ORDER BY order_date) AS next_order_date,
    DATEDIFF('day', order_date,
             LEAD(order_date, 1) OVER (PARTITION BY customer_id ORDER BY order_date)
    ) AS days_to_next_order
FROM orders
WHERE order_status = 'delivered';
-- This tells you: after each order, how many days until the customer ordered again?


-- ── AGGREGATE WINDOW FUNCTIONS ────────────────────────────────────────────
-- Running total (cumulative sum)
SELECT
    order_month,
    monthly_revenue,
    SUM(monthly_revenue) OVER (ORDER BY order_month
                                ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
                               ) AS cumulative_revenue
FROM monthly_revenue;

-- 3-month rolling average
SELECT
    order_month,
    monthly_revenue,
    AVG(monthly_revenue) OVER (ORDER BY order_month
                                ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
                               ) AS rolling_3m_avg
FROM monthly_revenue;

-- % of total (within each year)
SELECT
    order_month,
    order_year,
    monthly_revenue,
    ROUND(
        100.0 * monthly_revenue
        / SUM(monthly_revenue) OVER (PARTITION BY order_year)
    , 1) AS pct_of_year
FROM monthly_revenue;

-- Running % of total (cumulative share of annual revenue)
SELECT
    order_month,
    monthly_revenue,
    ROUND(
        100.0 * SUM(monthly_revenue)
                    OVER (ORDER BY order_month
                          ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
        / SUM(monthly_revenue) OVER ()
    , 1) AS cumulative_pct_of_total
FROM monthly_revenue;
```

---

### Window functions for business analytics — real patterns

```sql
-- ── PATTERN 1: Customer order sequence (first/second/third order) ──────────
WITH customer_orders AS (
    SELECT
        o.order_id,
        c.customer_unique_id,
        o.order_purchase_timestamp,
        p.payment_value,
        ROW_NUMBER() OVER (
            PARTITION BY c.customer_unique_id
            ORDER BY o.order_purchase_timestamp
        ) AS order_number
    FROM olist_orders o
    JOIN olist_customers c ON o.customer_id = c.customer_id
    JOIN payments_agg   p ON o.order_id    = p.order_id
    WHERE o.order_status = 'delivered'
)
-- Average value of first order vs repeat orders
SELECT
    order_number,
    COUNT(*)          AS orders,
    ROUND(AVG(payment_value), 2) AS avg_value
FROM customer_orders
WHERE order_number <= 5
GROUP BY order_number
ORDER BY order_number;
-- Insight: are repeat orders higher value than first orders?
-- If yes: retention has economic value beyond just the repeat count


-- ── PATTERN 2: Month-over-month growth ────────────────────────────────────
WITH monthly AS (
    SELECT
        DATE_TRUNC('month', order_purchase_timestamp) AS month,
        COUNT(*) AS orders,
        SUM(payment_value) AS revenue
    FROM olist_orders o
    JOIN payments_agg p ON o.order_id = p.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY 1
)
SELECT
    month,
    orders,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_revenue,
    ROUND(
        100.0 * (revenue - LAG(revenue) OVER (ORDER BY month))
        / NULLIF(LAG(revenue) OVER (ORDER BY month), 0)
    , 1) AS revenue_growth_pct,
    AVG(revenue) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)
                                       AS rolling_3m_avg
FROM monthly
ORDER BY month;


-- ── PATTERN 3: Seller performance percentile ──────────────────────────────
WITH seller_stats AS (
    SELECT
        i.seller_id,
        COUNT(DISTINCT o.order_id)         AS orders,
        ROUND(SUM(i.price), 2)             AS revenue,
        ROUND(AVG(r.review_score), 2)      AS avg_review
    FROM olist_order_items  i
    JOIN olist_orders       o ON i.order_id    = o.order_id
    JOIN olist_order_reviews r ON o.order_id = r.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY i.seller_id
)
SELECT
    seller_id,
    orders,
    revenue,
    avg_review,
    NTILE(100) OVER (ORDER BY revenue)    AS revenue_percentile,
    NTILE(100) OVER (ORDER BY avg_review) AS review_percentile,
    NTILE(4)   OVER (ORDER BY revenue)    AS revenue_quartile
    -- 4 = top 25% of sellers by revenue
FROM seller_stats;
```

---

## 24. Cohort Queries in SQL

```sql
-- ── COHORT RETENTION TABLE IN SQL ─────────────────────────────────────────
WITH first_orders AS (
    -- Step 1: Find each customer's first order month (their cohort)
    SELECT
        c.customer_unique_id,
        DATE_TRUNC('month', MIN(o.order_purchase_timestamp)) AS cohort_month
    FROM olist_orders o
    JOIN olist_customers c ON o.customer_id = c.customer_id
    WHERE o.order_status = 'delivered'
    GROUP BY c.customer_unique_id
),

all_orders AS (
    -- Step 2: Get all orders with cohort info attached
    SELECT
        c.customer_unique_id,
        f.cohort_month,
        DATE_TRUNC('month', o.order_purchase_timestamp) AS order_month,
        DATEDIFF('month', f.cohort_month,
                 DATE_TRUNC('month', o.order_purchase_timestamp)) AS months_since_first
    FROM olist_orders o
    JOIN olist_customers c ON o.customer_id = c.customer_id
    JOIN first_orders    f ON c.customer_unique_id = f.customer_unique_id
    WHERE o.order_status = 'delivered'
),

cohort_counts AS (
    -- Step 3: Count distinct customers per (cohort × months_since_first)
    SELECT
        cohort_month,
        months_since_first,
        COUNT(DISTINCT customer_unique_id) AS customers
    FROM all_orders
    GROUP BY cohort_month, months_since_first
),

cohort_sizes AS (
    -- Step 4: Get cohort size (month 0 count)
    SELECT cohort_month, customers AS cohort_size
    FROM cohort_counts
    WHERE months_since_first = 0
)

-- Step 5: Compute retention rate
SELECT
    c.cohort_month,
    c.months_since_first,
    c.customers,
    s.cohort_size,
    ROUND(100.0 * c.customers / s.cohort_size, 1) AS retention_rate_pct
FROM cohort_counts    c
JOIN cohort_sizes     s ON c.cohort_month = s.cohort_month
ORDER BY c.cohort_month, c.months_since_first;
```

---

## 25. Funnel Analysis in SQL

### What a funnel is

A **funnel** represents sequential steps in a process, where users drop off at each stage. Classic funnels:
- E-commerce: Visit → Add to Cart → Checkout → Purchase
- Signup: Landing Page → Sign Up → Email Verify → First Action
- Olist: Seller lists product → Gets first order → Gets repeat order → Achieves Gold tier

```sql
-- ── ORDER STATUS FUNNEL ────────────────────────────────────────────────────
-- How many orders progress through each stage?

WITH funnel AS (
    SELECT
        COUNT(*) FILTER (WHERE order_status IN
            ('created','approved','processing','invoiced','shipped',
             'delivered','canceled','unavailable'))    AS total_orders,
        COUNT(*) FILTER (WHERE order_status IN
            ('approved','processing','invoiced','shipped',
             'delivered'))                             AS approved,
        COUNT(*) FILTER (WHERE order_status IN
            ('processing','invoiced','shipped','delivered'))  AS processing,
        COUNT(*) FILTER (WHERE order_status IN
            ('shipped','delivered'))                  AS shipped,
        COUNT(*) FILTER (WHERE order_status = 'delivered')   AS delivered,
        COUNT(*) FILTER (WHERE order_status = 'canceled')    AS canceled
    FROM olist_orders
)
SELECT
    'Total Orders'   AS stage, total_orders    AS count, 100.0                              AS pct_of_top FROM funnel UNION ALL
SELECT 'Approved',    approved,    ROUND(100.0 * approved   / total_orders, 1) FROM funnel UNION ALL
SELECT 'Processing',  processing,  ROUND(100.0 * processing / total_orders, 1) FROM funnel UNION ALL
SELECT 'Shipped',     shipped,     ROUND(100.0 * shipped    / total_orders, 1) FROM funnel UNION ALL
SELECT 'Delivered',   delivered,   ROUND(100.0 * delivered  / total_orders, 1) FROM funnel UNION ALL
SELECT 'Canceled',    canceled,    ROUND(100.0 * canceled   / total_orders, 1) FROM funnel;


-- ── SELLER QUALITY FUNNEL ─────────────────────────────────────────────────
-- How many sellers reach each milestone?
WITH seller_milestones AS (
    SELECT
        s.seller_id,
        COUNT(DISTINCT o.order_id)             AS total_orders,
        ROUND(AVG(r.review_score), 2)          AS avg_review,
        MAX(CASE WHEN r.review_score >= 4 THEN 1 ELSE 0 END) AS has_good_review,
        CASE WHEN COUNT(DISTINCT o.order_id) >= 10 THEN 1 ELSE 0 END AS has_10_orders,
        CASE WHEN AVG(r.review_score) >= 4.0   THEN 1 ELSE 0 END AS avg_review_good
    FROM olist_sellers s
    LEFT JOIN olist_order_items   i ON s.seller_id = i.seller_id
    LEFT JOIN olist_orders        o ON i.order_id  = o.order_id AND o.order_status = 'delivered'
    LEFT JOIN olist_order_reviews r ON o.order_id  = r.order_id
    GROUP BY s.seller_id
)
SELECT
    COUNT(*)                       AS total_sellers,
    SUM(CASE WHEN total_orders > 0 THEN 1 ELSE 0 END)  AS sellers_with_orders,
    SUM(has_10_orders)             AS sellers_with_10plus_orders,
    SUM(avg_review_good)           AS sellers_with_good_avg_review,
    SUM(has_10_orders * avg_review_good) AS gold_tier_sellers
FROM seller_milestones;
```

---

## 26. Period-over-Period and Running Totals

```sql
-- ── YEAR-OVER-YEAR COMPARISON ─────────────────────────────────────────────
WITH monthly AS (
    SELECT
        YEAR(order_purchase_timestamp)                       AS yr,
        MONTH(order_purchase_timestamp)                      AS mo,
        DATE_TRUNC('month', order_purchase_timestamp)        AS month,
        COUNT(*)                                             AS orders,
        SUM(payment_value)                                   AS revenue
    FROM olist_orders o
    JOIN payments_agg p ON o.order_id = p.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY 1, 2, 3
)
SELECT
    a.month,
    a.orders                AS orders_this_year,
    b.orders                AS orders_last_year,
    ROUND(100.0*(a.orders - b.orders) / NULLIF(b.orders, 0), 1) AS orders_yoy_pct,
    a.revenue               AS revenue_this_year,
    b.revenue               AS revenue_last_year,
    ROUND(100.0*(a.revenue - b.revenue) / NULLIF(b.revenue, 0), 1) AS revenue_yoy_pct
FROM      monthly a
LEFT JOIN monthly b ON a.mo = b.mo AND a.yr = b.yr + 1
ORDER BY a.month;


-- ── ROLLING 7-DAY REVENUE ─────────────────────────────────────────────────
WITH daily AS (
    SELECT
        DATE(order_purchase_timestamp) AS order_date,
        SUM(payment_value)             AS daily_revenue
    FROM olist_orders o
    JOIN payments_agg p ON o.order_id = p.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY 1
)
SELECT
    order_date,
    daily_revenue,
    SUM(daily_revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS rolling_7d_revenue,
    AVG(daily_revenue) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS rolling_7d_avg
FROM daily
ORDER BY order_date;


-- ── RUNNING TOTAL WITH MILESTONES ─────────────────────────────────────────
-- "When did we reach our first million?"
WITH monthly_cumulative AS (
    SELECT
        DATE_TRUNC('month', order_purchase_timestamp) AS month,
        SUM(payment_value) AS monthly_revenue,
        SUM(SUM(payment_value)) OVER (ORDER BY DATE_TRUNC('month', order_purchase_timestamp)
                                       ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
                                      ) AS cumulative_revenue
    FROM olist_orders o
    JOIN payments_agg p ON o.order_id = p.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY 1
)
SELECT
    month,
    monthly_revenue,
    cumulative_revenue,
    CASE
        WHEN cumulative_revenue >= 10_000_000 AND
             LAG(cumulative_revenue) OVER (ORDER BY month) < 10_000_000
        THEN '🎉 R$10M milestone!'
        WHEN cumulative_revenue >= 5_000_000 AND
             LAG(cumulative_revenue) OVER (ORDER BY month) < 5_000_000
        THEN '🎉 R$5M milestone!'
        ELSE ''
    END AS milestone
FROM monthly_cumulative
ORDER BY month;
```

---
---

# Part VII — Python for Analytics

## 27. pandas for Analysis — Beyond Cleaning

### pandas as an analytical engine

Most introductions to pandas focus on cleaning: dropping nulls, renaming columns, fixing types. That's Part I. The analytical power of pandas — groupby chains, window operations, pivot tables, time series resampling — is where the real leverage is.

🎬 **Movies analogy:** pandas is like a film editing suite. Cleaning is organising the footage into labelled bins. Analysis is the actual edit — cutting between scenes, assembling sequences, creating meaning. Both use the same tool. The second task is harder and more valuable.

---

### GroupBy — the analytical workhorse

```python
import pandas as pd
import numpy as np

orders   = pd.read_csv("data/olist/olist_orders_dataset.csv",
                       parse_dates=["order_purchase_timestamp"])
payments = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
customers= pd.read_csv("data/olist/olist_customers_dataset.csv")

payments_agg = payments.groupby("order_id")["payment_value"].sum().reset_index()
base = (orders
    .query("order_status == 'delivered'")
    .merge(payments_agg, on="order_id", how="left")
    .merge(customers[["customer_id","customer_state"]], on="customer_id", how="left"))

base["month"] = base["order_purchase_timestamp"].dt.to_period("M").astype(str)
base["year"]  = base["order_purchase_timestamp"].dt.year

# ── BASIC GROUPBY ─────────────────────────────────────────────────────────
# Single aggregation
monthly_revenue = base.groupby("month")["payment_value"].sum()

# Multiple aggregations with .agg()
monthly_stats = base.groupby("month").agg(
    orders  = ("order_id",     "count"),
    revenue = ("payment_value","sum"),
    aov     = ("payment_value","mean"),
    p50     = ("payment_value","median"),
).round(2)

# Named aggregations with custom functions
monthly_rich = base.groupby("month").agg(
    orders    = ("order_id",      "count"),
    revenue   = ("payment_value", "sum"),
    aov       = ("payment_value", "mean"),
    p90_value = ("payment_value", lambda x: x.quantile(0.9)),
    high_value= ("payment_value", lambda x: (x > 500).sum()),
).round(2)

# Multi-level groupby
by_state_month = base.groupby(["customer_state","month"]).agg(
    orders  = ("order_id",     "count"),
    revenue = ("payment_value","sum"),
).reset_index()

print("Top 5 states by total revenue:")
print(by_state_month.groupby("customer_state")["revenue"]
      .sum().nlargest(5).to_string())


# ── TRANSFORM — add aggregate context back to each row ────────────────────
# Unlike groupby().agg() which collapses rows, transform keeps original shape

# Add customer-level total to each order row
base["customer_total"] = base.groupby("customer_state")["payment_value"]\
                             .transform("sum")

# Each row now has the state total — useful for "% of state total" calculations
base["pct_of_state"] = base["payment_value"] / base["customer_total"]

# Add rolling average per state
base = base.sort_values(["customer_state","order_purchase_timestamp"])
base["state_rolling_aov"] = (base
    .groupby("customer_state")["payment_value"]
    .transform(lambda x: x.rolling(30, min_periods=1).mean()))


# ── APPLY — for complex per-group operations ──────────────────────────────
def state_growth_summary(group):
    """For each state: compute MoM growth for the last 3 months."""
    monthly = group.set_index("order_purchase_timestamp").resample("M")["payment_value"].sum()
    if len(monthly) < 2:
        return pd.Series({"last_revenue": monthly.iloc[-1], "growth_rate": None})
    growth = monthly.pct_change().iloc[-1]
    return pd.Series({"last_revenue": monthly.iloc[-1], "growth_rate": growth})

state_summary = base.groupby("customer_state").apply(state_growth_summary).reset_index()
print("\nState growth summary (sample):")
print(state_summary.sort_values("growth_rate", ascending=False).head(5).round(3))
```

---

### Pivot tables — the analyst's cross-tab

```python
# ── PIVOT TABLE ───────────────────────────────────────────────────────────
# Revenue by state and year — classic cross-tab
pivot = pd.pivot_table(
    base,
    values  = "payment_value",
    index   = "customer_state",
    columns = "year",
    aggfunc = "sum",
    fill_value = 0,
)

# Add a total column
pivot["Total"] = pivot.sum(axis=1)
pivot = pivot.sort_values("Total", ascending=False)
print("Revenue pivot (top 10 states):")
print(pivot.head(10).applymap(lambda x: f"R${x/1000:.0f}k").to_string())

# Margin totals
pivot_with_margins = pd.pivot_table(
    base,
    values   = "payment_value",
    index    = "customer_state",
    columns  = "year",
    aggfunc  = "sum",
    margins  = True,        # adds row and column totals
    margins_name = "Total",
)
```

---

### Time series resampling and analysis

```python
# ── RESAMPLING ────────────────────────────────────────────────────────────
ts = (base
    .set_index("order_purchase_timestamp")["payment_value"]
    .sort_index())

# Resample to different frequencies
daily   = ts.resample("D").sum()
weekly  = ts.resample("W").sum()
monthly = ts.resample("M").sum()

# Rolling statistics (smoothing)
daily_smoothed = daily.rolling(window=7, min_periods=1).mean()  # 7-day rolling avg

# ── PERIOD-OVER-PERIOD ────────────────────────────────────────────────────
monthly_df = monthly.reset_index()
monthly_df.columns = ["month","revenue"]

monthly_df["prev_month"]   = monthly_df["revenue"].shift(1)
monthly_df["mom_change"]   = monthly_df["revenue"] - monthly_df["prev_month"]
monthly_df["mom_pct"]      = monthly_df["revenue"].pct_change() * 100

# Year-over-year: shift 12 months
monthly_df["prev_year"]    = monthly_df["revenue"].shift(12)
monthly_df["yoy_pct"]      = monthly_df["revenue"].pct_change(12) * 100

print("MoM and YoY growth (last 6 months):")
print(monthly_df.tail(6)[["month","revenue","mom_pct","yoy_pct"]].round(1).to_string(index=False))

# ── SEASONAL DECOMPOSITION ────────────────────────────────────────────────
# statsmodels decomposition (only with enough data)
try:
    from statsmodels.tsa.seasonal import seasonal_decompose
    decomp = seasonal_decompose(monthly.values, model="additive", period=12)
    # decomp.trend, decomp.seasonal, decomp.resid
    print("\nSeasonal decomposition computed.")
    print(f"  Trend component range: {decomp.trend.min():.0f} – {decomp.trend.max():.0f}")
except Exception:
    print("Not enough data for seasonal decomposition (need 2+ full years)")
```

---

## 28. Matplotlib and Seaborn

### Design principles for analytical charts

Before writing code, decide two things:
1. **What is the message?** (what insight do I want the reader to take away)
2. **Who is the audience?** (exploration chart for yourself vs presentation chart for a CFO)

Exploration charts can be rough — quick, clear, no polish needed. Presentation charts need: clear titles (that state the message), axis labels, source annotations, and visual hierarchy that guides the eye to the main point.

---

### Seaborn — statistical visualisation

```python
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd

# Set a clean theme
sns.set_theme(style="whitegrid", palette="muted", font_scale=1.1)
plt.rcParams["axes.spines.top"]   = False
plt.rcParams["axes.spines.right"] = False

orders   = pd.read_csv("data/olist/olist_orders_dataset.csv",
                       parse_dates=["order_purchase_timestamp"])
reviews  = pd.read_csv("data/olist/olist_order_reviews_dataset.csv")
payments = pd.read_csv("data/olist/olist_order_payments_dataset.csv")

payments_agg = payments.groupby("order_id")["payment_value"].sum().reset_index()
delivered = (orders[orders["order_status"] == "delivered"]
    .merge(payments_agg, on="order_id"))
delivered["delivery_days"] = (
    (delivered["order_delivered_customer_date"].pipe(pd.to_datetime) -
     delivered["order_purchase_timestamp"]).dt.days)
delivered = delivered.merge(
    reviews[["order_id","review_score"]].groupby("order_id")["review_score"]
    .mean().reset_index(), on="order_id", how="left")


# ── 1. DISTRIBUTION — histogram with KDE ──────────────────────────────────
fig, axes = plt.subplots(1, 2, figsize=(13, 4))

sns.histplot(delivered["payment_value"].clip(upper=800),
             bins=40, kde=True, ax=axes[0], color="#4C72B0")
axes[0].set_title("Order Value Distribution\n(capped at R$800 for clarity)",
                  fontweight="bold")
axes[0].set_xlabel("Order Value (R$)")
axes[0].axvline(delivered["payment_value"].median(), color="red",
                linestyle="--", label=f"Median: R${delivered['payment_value'].median():.0f}")
axes[0].legend()

sns.histplot(delivered["delivery_days"].clip(upper=40),
             bins=30, kde=True, ax=axes[1], color="#55A868")
axes[1].set_title("Delivery Days Distribution\n(capped at 40 days)",
                  fontweight="bold")
axes[1].set_xlabel("Days to Deliver")
axes[1].axvline(delivered["delivery_days"].median(), color="red",
                linestyle="--", label=f"Median: {delivered['delivery_days'].median():.0f}d")
axes[1].legend()

plt.suptitle("Olist Order Characteristics", fontsize=14, y=1.02)
plt.tight_layout()
plt.savefig("outputs/distributions.png", dpi=150, bbox_inches="tight")


# ── 2. BOX PLOT — compare distributions across groups ─────────────────────
fig, ax = plt.subplots(figsize=(10, 5))
plot_data = delivered[delivered["review_score"].notna()].copy()
plot_data["score_label"] = plot_data["review_score"].astype(int).astype(str) + "★"

sns.boxplot(
    data    = plot_data[plot_data["delivery_days"] < 40],
    x       = "score_label",
    y       = "delivery_days",
    order   = ["1★","2★","3★","4★","5★"],
    palette = ["#d32f2f","#f57c00","#fbc02d","#7cb342","#388e3c"],
    ax      = ax,
)
ax.set_title("Delivery Days by Review Score\nFaster delivery = better reviews",
             fontweight="bold", pad=15)
ax.set_xlabel("Review Score")
ax.set_ylabel("Days to Deliver")
plt.tight_layout()
plt.savefig("outputs/delivery_by_review_box.png", dpi=150, bbox_inches="tight")


# ── 3. HEATMAP — correlation or cross-tab ─────────────────────────────────
hr = pd.read_csv("data/hr/WA_Fn-UseC_-HR-Employee-Attrition.csv")
numeric_cols = ["Age","MonthlyIncome","JobSatisfaction","WorkLifeBalance",
                "YearsAtCompany","EnvironmentSatisfaction","JobInvolvement"]

fig, ax = plt.subplots(figsize=(9, 7))
corr = hr[numeric_cols].corr()
mask = np.triu(np.ones_like(corr, dtype=bool))
sns.heatmap(corr, mask=mask, annot=True, fmt=".2f", cmap="coolwarm",
            center=0, vmin=-1, vmax=1, ax=ax,
            square=True, linewidths=0.5, cbar_kws={"shrink": 0.8})
ax.set_title("HR Metrics Correlation Matrix", fontweight="bold", pad=15)
plt.tight_layout()
plt.savefig("outputs/hr_correlation.png", dpi=150, bbox_inches="tight")
```

---

## 29. Plotly — Interactive Charts

### When to use Plotly

Plotly produces interactive charts: hover for values, click to filter, zoom, pan. Use Plotly when:
- The chart will be viewed in a browser or notebook
- You want the audience to explore, not just observe
- You're building a Streamlit/Dash app

Use matplotlib/seaborn when:
- The output is a static PDF, slide, or email
- You need precise layout control
- Publication-quality formatting is required

```python
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import pandas as pd

orders   = pd.read_csv("data/olist/olist_orders_dataset.csv",
                       parse_dates=["order_purchase_timestamp"])
payments = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
customers= pd.read_csv("data/olist/olist_customers_dataset.csv")
payments_agg = payments.groupby("order_id")["payment_value"].sum().reset_index()

base = (orders.query("order_status == 'delivered'")
    .merge(payments_agg, on="order_id")
    .merge(customers[["customer_id","customer_state"]], on="customer_id"))
base["month"] = base["order_purchase_timestamp"].dt.to_period("M").astype(str)

monthly = base.groupby("month").agg(
    orders  = ("order_id",      "count"),
    revenue = ("payment_value", "sum"),
    aov     = ("payment_value", "mean"),
).reset_index()

# ── 1. INTERACTIVE LINE CHART ─────────────────────────────────────────────
fig = px.line(
    monthly,
    x     = "month",
    y     = "revenue",
    title = "Monthly Revenue — Olist",
    labels = {"revenue": "Revenue (R$)", "month": ""},
    markers= True,
)
fig.update_traces(line_width=2.5, marker_size=5)
fig.update_layout(hovermode="x unified", template="plotly_white")
fig.write_html("outputs/monthly_revenue_interactive.html")
# Open in browser: hover shows exact values, zoom in to any period


# ── 2. INTERACTIVE SCATTER ────────────────────────────────────────────────
state_summary = base.groupby("customer_state").agg(
    orders  = ("order_id",      "count"),
    revenue = ("payment_value", "sum"),
    aov     = ("payment_value", "mean"),
).reset_index()

fig = px.scatter(
    state_summary,
    x     = "orders",
    y     = "aov",
    size  = "revenue",
    hover_name = "customer_state",
    title = "Orders vs AOV by State (bubble = revenue)",
    labels = {"orders": "Order Count", "aov": "Avg Order Value (R$)"},
    color = "revenue",
    color_continuous_scale = "Blues",
)
fig.update_layout(template="plotly_white")
fig.write_html("outputs/state_scatter.html")


# ── 3. DASHBOARD-STYLE SUBPLOT ────────────────────────────────────────────
fig = make_subplots(
    rows=2, cols=2,
    subplot_titles=("Monthly Orders", "Monthly Revenue",
                    "Avg Order Value", "MoM Revenue Growth %"),
)

monthly["mom_pct"] = monthly["revenue"].pct_change() * 100

fig.add_trace(go.Scatter(x=monthly["month"], y=monthly["orders"],
                          mode="lines+markers", name="Orders",
                          line=dict(color="#4C72B0")), row=1, col=1)
fig.add_trace(go.Scatter(x=monthly["month"], y=monthly["revenue"]/1e6,
                          mode="lines+markers", name="Revenue (M)",
                          line=dict(color="#55A868")), row=1, col=2)
fig.add_trace(go.Scatter(x=monthly["month"], y=monthly["aov"],
                          mode="lines+markers", name="AOV",
                          line=dict(color="#C44E52")), row=2, col=1)
fig.add_trace(go.Bar(x=monthly["month"], y=monthly["mom_pct"],
                      name="MoM %",
                      marker_color=monthly["mom_pct"].apply(
                          lambda x: "#55A868" if x >= 0 else "#C44E52"
                      )), row=2, col=2)

fig.update_layout(height=600, showlegend=False,
                  title_text="Olist Performance Dashboard",
                  template="plotly_white")
fig.write_html("outputs/olist_dashboard.html")
print("Interactive dashboard saved to outputs/olist_dashboard.html")
```

---

## 30. The Python Analytics Workflow

### Putting it all together — a repeatable pattern

```python
# THE STANDARD ANALYTICS SCRIPT STRUCTURE
# Every analysis follows this pattern

# ── 1. IMPORTS ────────────────────────────────────────────────────────────
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import scipy.stats as stats
from pathlib import Path

# ── 2. CONFIG ─────────────────────────────────────────────────────────────
DATA_DIR   = Path("data/olist/")
OUTPUT_DIR = Path("outputs/")
OUTPUT_DIR.mkdir(exist_ok=True)

# Analysis parameters — centralised, not scattered through code
ANALYSIS_DATE  = "2018-08-31"
MIN_ORDERS     = 10          # minimum orders for seller analysis
LATE_THRESHOLD = 1           # 1 = compare vs estimated date

# ── 3. LOAD ───────────────────────────────────────────────────────────────
def load_olist(data_dir: Path) -> dict:
    """Load and lightly validate all Olist tables."""
    tables = {
        "orders":    "olist_orders_dataset.csv",
        "items":     "olist_order_items_dataset.csv",
        "payments":  "olist_order_payments_dataset.csv",
        "customers": "olist_customers_dataset.csv",
        "reviews":   "olist_order_reviews_dataset.csv",
        "sellers":   "olist_sellers_dataset.csv",
        "products":  "olist_products_dataset.csv",
        "category":  "product_category_name_translation.csv",
    }
    dfs = {}
    for name, filename in tables.items():
        path = data_dir / filename
        if not path.exists():
            print(f"⚠️  Missing: {filename}")
            continue
        dfs[name] = pd.read_csv(path)
        print(f"✓ {name}: {len(dfs[name]):,} rows")
    dfs["orders"]["order_purchase_timestamp"] = pd.to_datetime(
        dfs["orders"]["order_purchase_timestamp"])
    return dfs

data = load_olist(DATA_DIR)

# ── 4. PREPARE ────────────────────────────────────────────────────────────
# One function per preparation step — keeps things reproducible and testable

def prepare_base(data: dict) -> pd.DataFrame:
    """Build analysis base: delivered orders with revenue, customer state."""
    payments_agg = (data["payments"]
        .groupby("order_id")["payment_value"].sum().reset_index())
    return (data["orders"]
        .query("order_status == 'delivered'")
        .merge(payments_agg, on="order_id", how="left")
        .merge(data["customers"][["customer_id","customer_state"]],
               on="customer_id", how="left")
        .assign(
            month = lambda df: df["order_purchase_timestamp"].dt.to_period("M").astype(str),
            year  = lambda df: df["order_purchase_timestamp"].dt.year,
        ))

base = prepare_base(data)
print(f"\nAnalysis base: {len(base):,} delivered orders")

# ── 5. ANALYSE ────────────────────────────────────────────────────────────
# Each analysis is a function — testable, reusable, documented

def monthly_performance(base: pd.DataFrame) -> pd.DataFrame:
    """Monthly orders, revenue, and AOV with MoM growth."""
    monthly = (base.groupby("month")
        .agg(orders=("order_id","count"), revenue=("payment_value","sum"),
             aov=("payment_value","mean"))
        .reset_index()
        .assign(mom_revenue_pct=lambda df: df["revenue"].pct_change() * 100))
    return monthly.round(2)

def state_performance(base: pd.DataFrame, top_n: int = 10) -> pd.DataFrame:
    """Top N states by revenue with per-state AOV."""
    return (base.groupby("customer_state")
        .agg(orders=("order_id","count"), revenue=("payment_value","sum"),
             aov=("payment_value","mean"))
        .nlargest(top_n, "revenue")
        .reset_index()
        .round(2))

monthly = monthly_performance(base)
states  = state_performance(base)

print("\nMonthly performance (last 4 months):")
print(monthly.tail(4).to_string(index=False))
print("\nTop 10 states by revenue:")
print(states.to_string(index=False))

# ── 6. VISUALISE ──────────────────────────────────────────────────────────
def plot_monthly_trend(monthly: pd.DataFrame, output_path: Path) -> None:
    fig, ax = plt.subplots(figsize=(12, 4))
    ax.plot(monthly["month"], monthly["revenue"] / 1e6,
            linewidth=2.5, color="#4C72B0", marker="o", markersize=4)
    ax.fill_between(monthly["month"], monthly["revenue"] / 1e6, alpha=0.1)
    ax.set_title("Monthly Revenue — Olist", fontweight="bold")
    ax.set_ylabel("Revenue (R$ millions)")
    ax.tick_params(axis="x", rotation=45)
    ax.spines[["top","right"]].set_visible(False)
    plt.tight_layout()
    plt.savefig(output_path, dpi=150, bbox_inches="tight")
    plt.close()

plot_monthly_trend(monthly, OUTPUT_DIR / "monthly_revenue.png")
print("\n✓ Chart saved to outputs/monthly_revenue.png")

# ── 7. EXPORT ─────────────────────────────────────────────────────────────
monthly.to_csv(OUTPUT_DIR / "monthly_performance.csv", index=False)
states.to_csv(OUTPUT_DIR /  "state_performance.csv",   index=False)
print("✓ Results saved to outputs/")
```

---
---

# Part VIII — Statistical Thinking

## 31. Distributions That Matter

### Why distributions matter more than averages

Every number a business tracks is drawn from a distribution. The mean is just one summary of that distribution — and often a misleading one. Understanding the shape of the distribution changes how you interpret data and what you recommend.

🎬 **Movies analogy:** The "average" film review score for a polarising film like Marmite (people either love it or hate it) is the same as for a broadly decent film everyone finds mediocre. The average is 3 out of 5 in both cases. The distributions are completely different. One film has two passionate audiences; the other has one indifferent one. The average alone will lead you to the same marketing decision for fundamentally different products.

---

### The normal (Gaussian) distribution

**What it is:** The classic bell curve. Symmetric. Mean = Median = Mode. Most values cluster near the centre; extreme values become increasingly rare.

**Where it actually appears in business data:**
- Human height, weight (approximately)
- Measurement errors
- Sum of many independent random variables (Central Limit Theorem)
- Test scores (deliberately designed to be normal)

**Where it DOESN'T appear (common mistake):**
- Revenue, transaction amounts (right-skewed)
- Customer tenure (right-skewed or bimodal)
- Time between events (exponential)
- Web page load times (right-skewed with long tail)

**In plain English:** If a distribution is normal, the mean is a reliable measure of "typical." Most values are within 1–2 standard deviations of the mean. Being 3 standard deviations away is genuinely unusual (0.3% of observations).

```python
import numpy as np
import scipy.stats as stats
import matplotlib.pyplot as plt

# Generating a normal distribution to visualise properties
np.random.seed(42)
normal_data = np.random.normal(loc=100, scale=15, size=10_000)

# Properties of normal distribution:
print(f"Mean:   {normal_data.mean():.1f}")
print(f"Median: {np.median(normal_data):.1f}")  # nearly identical to mean
print(f"Std:    {normal_data.std():.1f}")
print(f"\nWithin 1 std (68% expected): {((normal_data > 85) & (normal_data < 115)).mean():.1%}")
print(f"Within 2 std (95% expected): {((normal_data > 70) & (normal_data < 130)).mean():.1%}")
print(f"Within 3 std (99.7% expected): {((normal_data > 55) & (normal_data < 145)).mean():.1%}")
```

---

### The right-skewed distribution — the most common in business

**What it is:** Most values cluster at the low end, with a long tail of high values. Mean > Median. The mean is pulled upward by extreme values.

**Where it appears:**
- Revenue per customer (most spend a little; a few spend a lot)
- Order values (most orders are small; occasional large orders)
- Company sizes (most are tiny; a few are massive)
- Social media followers (most people have few; celebrities have millions)
- Bug fix times (most are quick; occasional bugs take weeks)

**In plain English:** When you see right skew, the mean overstates the "typical" experience. The median better represents the majority of cases. The P90 or P99 represents the extreme tail that requires special treatment.

```python
# Olist revenue distribution — demonstrating right skew
payments = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
revenue = payments.groupby("order_id")["payment_value"].sum()

print("Revenue distribution shape:")
print(f"  Mean:   R${revenue.mean():.2f}  ← pulled up by large orders")
print(f"  Median: R${revenue.median():.2f}  ← typical order")
print(f"  Skewness: {revenue.skew():.2f}  (positive = right-skewed)")
print(f"\n  The mean is {revenue.mean()/revenue.median():.1f}x the median")
print(f"  This means: if you design for the 'average' customer,")
print(f"  you're actually designing for the minority who spend a lot.")

# Log-transform to see the true distribution
print(f"\nAfter log transform:")
log_rev = np.log1p(revenue)
print(f"  Skewness: {log_rev.skew():.2f}  (closer to 0 = more normal)")
print(f"  This is a log-normal distribution — common for prices and revenues")
```

---

### The bimodal distribution — when you're mixing two populations

**What it is:** Two humps. The data has two clusters of values with a valley between them. This is almost always a signal that you're mixing two distinct populations.

**Where it appears:**
- Review scores (loved vs hated products)
- Employee salaries (individual contributors vs managers)
- Delivery times (domestic vs international shipments mixed together)
- Customer LTV (occasional buyers vs loyal regulars)

**In plain English:** If you see two humps, segment first. Analysing the combined distribution gives you an average that doesn't represent either group. Each hump is a different population with different characteristics, different needs, and different recommended actions.

```python
# Detecting bimodality in review scores
reviews = pd.read_csv("data/olist/olist_order_reviews_dataset.csv")

score_dist = reviews["review_score"].value_counts().sort_index()
print("Review score distribution:")
for score, count in score_dist.items():
    bar = "█" * int(count / score_dist.max() * 30)
    print(f"  {score}★: {count:>7,}  {bar}")

# Classic 1-5 review patterns:
# U-shape (high 1★ and 5★, low 2-3★) = polarising product or selection bias
# J-shape (high 5★, rest lower) = positive experience or review bombing
# Bimodal = two distinct user segments

pct_5star = (reviews["review_score"] == 5).mean()
pct_1star = (reviews["review_score"] == 1).mean()
print(f"\n5★ rate: {pct_5star:.1%}")
print(f"1★ rate: {pct_1star:.1%}")
print(f"1+5★ combined: {pct_5star + pct_1star:.1%}")
if pct_5star + pct_1star > 0.6:
    print("→ Polarised distribution: most customers are either very happy or very unhappy")
    print("  Investigate: is this a product quality issue? A delivery issue?")
    print("  Are they distributed differently by category?")
```

---

## 32. Hypothesis Testing in Plain English

### What hypothesis testing actually is — without the statistics degree

A **hypothesis test** is a structured way to ask: "Is the pattern I see in this data real, or could it have happened by random chance?"

**The core question:** If I flip a coin 10 times and get 7 heads, did I discover a biased coin, or did I just get lucky?

Statistical hypothesis testing answers this by asking: "How likely would this result be if there were truly no effect?" That probability is the **p-value**.

---

### The p-value — what it actually means

The **p-value** is the probability of seeing a result at least as extreme as yours, assuming there is truly no effect (the null hypothesis is true).

**In plain English:**
- p = 0.05 means: "If there really is no difference, I'd see a result this extreme 5% of the time by random chance."
- p < 0.05 (conventional threshold): "The result is unlikely enough by chance that I'm willing to call it statistically significant."
- p > 0.05: "This could easily have happened by chance; I can't claim a real effect."

**What p-value is NOT:**
- It is NOT the probability that you're right
- It is NOT the probability that the null hypothesis is false
- It is NOT a measure of practical importance (a tiny effect can be highly significant with enough data)

🎬 **Movies analogy:** A film gets a standing ovation from an audience. Is the film genuinely excellent, or did we just happen to select an unusually enthusiastic audience? The p-value tells you: "If this film were actually average, what's the probability of getting this strong a response?" If that probability is very low (p < 0.05), the standing ovation is probably real signal, not a lucky sample.

---

### The four essential hypothesis tests

**Test 1: t-test — comparing two means**

*Use when:* You want to know if the average value of something is different between two groups.

*Example:* "Do customers who receive early deliveries give higher reviews than customers who receive late deliveries?"

```python
import scipy.stats as stats
import pandas as pd

orders  = pd.read_csv("data/olist/olist_orders_dataset.csv",
                      parse_dates=["order_purchase_timestamp",
                                   "order_delivered_customer_date",
                                   "order_estimated_delivery_date"])
reviews = pd.read_csv("data/olist/olist_order_reviews_dataset.csv")

delivered = orders[orders["order_status"] == "delivered"].copy()
delivered["is_late"] = (delivered["order_delivered_customer_date"]
                        > delivered["order_estimated_delivery_date"])

analysis = delivered.merge(
    reviews[["order_id","review_score"]].groupby("order_id")["review_score"]
    .mean().reset_index(),
    on="order_id", how="inner"
).dropna()

# Split into two groups
on_time = analysis[~analysis["is_late"]]["review_score"]
late    = analysis[ analysis["is_late"]]["review_score"]

# Run the t-test
t_stat, p_value = stats.ttest_ind(on_time, late)

print("T-Test: Does delivery timing affect review scores?")
print(f"  On-time deliveries — avg review: {on_time.mean():.3f} (n={len(on_time):,})")
print(f"  Late deliveries    — avg review: {late.mean():.3f} (n={len(late):,})")
print(f"  Difference: {on_time.mean() - late.mean():.3f} stars")
print(f"  t-statistic: {t_stat:.3f}")
print(f"  p-value: {p_value:.6f}")

if p_value < 0.05:
    print(f"\n  ✅ STATISTICALLY SIGNIFICANT (p < 0.05)")
    print(f"  Interpretation: The difference ({on_time.mean() - late.mean():.2f} stars)")
    print(f"  is very unlikely to have occurred by random chance.")
    print(f"  On-time delivery genuinely produces higher reviews.")
else:
    print(f"\n  ❌ NOT SIGNIFICANT (p ≥ 0.05)")
    print(f"  The difference could be explained by random variation.")

print(f"\n  Practical significance: Effect size = {(on_time.mean() - late.mean()) / analysis['review_score'].std():.2f} std deviations")
print(f"  (Cohen's d < 0.2 = small, 0.5 = medium, 0.8 = large)")
```

---

**Test 2: Chi-square test — comparing proportions**

*Use when:* You want to know if two categorical variables are related — if the proportion of one thing changes across categories of another.

*Example:* "Is fraud rate different across merchant categories?"

```python
# Chi-square test: does fraud rate differ by category?
fraud = pd.read_csv("data/fraud/transactions.csv")

# Create a contingency table
contingency = pd.crosstab(fraud["category"], fraud["is_fraud"])
print("Contingency table (first 5 categories):")
print(contingency.head().to_string())

chi2, p_value, dof, expected = stats.chi2_contingency(contingency)

print(f"\nChi-square test: Does fraud rate vary by merchant category?")
print(f"  Chi-square statistic: {chi2:.2f}")
print(f"  Degrees of freedom: {dof}")
print(f"  p-value: {p_value:.6f}")

if p_value < 0.05:
    print(f"\n  ✅ SIGNIFICANT: Fraud rate varies significantly across categories")
    # Find which categories have highest fraud rate
    fraud_by_cat = fraud.groupby("category")["is_fraud"].mean().sort_values(ascending=False)
    print("\n  Fraud rate by category (top 5):")
    print(fraud_by_cat.head().round(3).to_string())
else:
    print(f"\n  ❌ NOT SIGNIFICANT: No evidence that fraud rate varies by category")
```

---

**Test 3: ANOVA — comparing more than two groups**

*Use when:* You want to compare means across three or more groups.

*Example:* "Does customer satisfaction differ across the four Brazilian regions?"

```python
# One-way ANOVA: does review score differ by region?
customers = pd.read_csv("data/olist/olist_customers_dataset.csv")

region_map = {
    "SP":"Southeast","RJ":"Southeast","MG":"Southeast","ES":"Southeast",
    "PR":"South","SC":"South","RS":"South",
    "BA":"Northeast","PE":"Northeast","CE":"Northeast",
    "AM":"North","PA":"North",
    "GO":"Central-West","MT":"Central-West","MS":"Central-West","DF":"Central-West",
}
customers["region"] = customers["customer_state"].map(region_map).fillna("Other")

analysis = (reviews
    .merge(orders[["order_id","customer_id"]], on="order_id", how="left")
    .merge(customers[["customer_id","region"]], on="customer_id", how="left")
    .dropna(subset=["review_score","region"]))

# Prepare groups for ANOVA
groups = [
    analysis[analysis["region"] == r]["review_score"].values
    for r in analysis["region"].unique()
]

f_stat, p_value = stats.f_oneway(*groups)

print("ANOVA: Does review score differ by region?")
print(f"  F-statistic: {f_stat:.3f}")
print(f"  p-value: {p_value:.6f}")

if p_value < 0.05:
    print("  ✅ SIGNIFICANT: Review scores differ across regions")
    print("\n  Average review score by region:")
    regional_means = analysis.groupby("region")["review_score"].agg(["mean","count"])
    print(regional_means.sort_values("mean", ascending=False).round(3).to_string())
```

---

**Test 4: Correlation test — is the relationship real?**

*Use when:* You've observed a correlation and want to know if it's statistically significant or just noise.

```python
# Is the correlation between delivery days and review score statistically significant?
r, p_value = stats.pearsonr(
    analysis["delivery_days"].dropna(),
    analysis["review_score"].dropna()
)

print(f"Correlation test: delivery days vs review score")
print(f"  Pearson r: {r:.4f}")
print(f"  p-value:   {p_value:.6f}")
print(f"  95% CI:    [{r - 1.96*np.sqrt((1-r**2)/(len(analysis)-2)):.3f}, "
      f"{r + 1.96*np.sqrt((1-r**2)/(len(analysis)-2)):.3f}]")

print(f"\nIn plain English:")
print(f"  There is a {'significant' if p_value < 0.05 else 'non-significant'} "
      f"negative correlation between delivery time and review score.")
print(f"  As delivery days increase by 1, review score decreases by {abs(r):.3f} on average.")
print(f"  This relationship would occur by chance only {p_value:.2%} of the time")
print(f"  if there were truly no relationship.")
```

---

### The common hypothesis testing mistakes

| Mistake | What it looks like | Why it's wrong |
|---------|-------------------|----------------|
| **p-hacking** | Testing 20 metrics until one gives p < 0.05 | 5% of tests will be "significant" by chance; the threshold is calibrated for ONE test |
| **Ignoring sample size** | "We tested this on 50 users" | Small samples produce unreliable p-values |
| **Confusing statistical and practical significance** | "We found a significant 0.1-star improvement in reviews" | Significant ≠ meaningful. Is 0.1 stars worth acting on? |
| **Multiple comparisons** | Comparing 10 regions without adjusting threshold | Bonferroni correction: use p < 0.05/10 = 0.005 |
| **Ignoring assumptions** | Running a t-test on very skewed data | t-test assumes near-normal distributions |

---

## 33. A/B Testing

### What an A/B test is — and what it actually proves

An **A/B test** (also called a controlled experiment) randomly splits your users into two groups:
- **Control (A):** experiences the current version
- **Treatment (B):** experiences the new version

By randomising, you ensure that any difference in outcomes is caused by the change, not by pre-existing differences between users.

**Why randomisation matters:** if you just compare "users who saw the new version" vs "users who didn't," the two groups are probably not comparable. Users who see the new version might be more engaged, newer, or from different regions. Randomisation eliminates this confounding.

🎬 **Movies analogy:** Testing two trailers for the same film — one action-focused, one character-focused. You can't just show one trailer in New York and the other in Los Angeles — New York audiences might be inherently different. You randomise: within the same city, same time slot, same demographic, half get trailer A and half get trailer B. Any difference in ticket sales is caused by the trailer, not the audience.

---

### A/B test analysis in Python

```python
import pandas as pd
import scipy.stats as stats
import numpy as np

# Simulating an A/B test on delivery time estimation
# Hypothesis: a new estimation algorithm improves on-time delivery rate

np.random.seed(42)
n_per_group = 5000

# Control: current algorithm — 88% on-time rate
control_outcomes = np.random.binomial(1, 0.88, n_per_group)

# Treatment: new algorithm — 91% on-time rate
treatment_outcomes = np.random.binomial(1, 0.91, n_per_group)

control_df   = pd.DataFrame({"group":"control",   "on_time":control_outcomes})
treatment_df = pd.DataFrame({"group":"treatment", "on_time":treatment_outcomes})
ab_data      = pd.concat([control_df, treatment_df], ignore_index=True)

# ── ANALYSIS ──────────────────────────────────────────────────────────────
control_rate   = ab_data[ab_data["group"]=="control"]  ["on_time"].mean()
treatment_rate = ab_data[ab_data["group"]=="treatment"]["on_time"].mean()
absolute_lift  = treatment_rate - control_rate
relative_lift  = absolute_lift / control_rate

print(f"A/B Test: New delivery estimation algorithm")
print(f"{'─'*45}")
print(f"  Control rate:   {control_rate:.1%}  (n={n_per_group:,})")
print(f"  Treatment rate: {treatment_rate:.1%}  (n={n_per_group:,})")
print(f"  Absolute lift:  {absolute_lift:+.1%}")
print(f"  Relative lift:  {relative_lift:+.1%}")

# Statistical test (chi-square for proportions)
contingency = pd.crosstab(ab_data["group"], ab_data["on_time"])
chi2, p_value, dof, expected = stats.chi2_contingency(contingency)

print(f"\n  Chi-square: {chi2:.3f}")
print(f"  p-value:    {p_value:.4f}")
print(f"  Significant: {'Yes ✅' if p_value < 0.05 else 'No ❌'}")

# Confidence interval for the difference
se = np.sqrt(control_rate*(1-control_rate)/n_per_group +
             treatment_rate*(1-treatment_rate)/n_per_group)
ci_lower = absolute_lift - 1.96 * se
ci_upper = absolute_lift + 1.96 * se
print(f"\n  95% CI for lift: [{ci_lower:.1%}, {ci_upper:.1%}]")

# Business impact estimate
total_orders     = 100_000   # monthly orders
orders_improved  = total_orders * absolute_lift
print(f"\nBusiness impact estimate:")
print(f"  Monthly orders:          {total_orders:,}")
print(f"  Additional on-time orders: {orders_improved:,.0f}")
print(f"  Avg review impact: ~0.3 stars per prevented late delivery")
print(f"  Estimated review improvement: {0.3 * orders_improved / total_orders:.2f} stars overall")

# ── MINIMUM DETECTABLE EFFECT (SAMPLE SIZE PLANNING) ─────────────────────
# Before running an A/B test, calculate how many samples you need
from statsmodels.stats.power import TTestIndPower

# For a binary outcome (proportion), use proportion power
from statsmodels.stats.proportion import proportion_effectsize, zt_ind_solve_power

baseline    = 0.88   # current on-time rate
mde         = 0.02   # minimum effect you care about (2 percentage points)
alpha       = 0.05   # significance level
power       = 0.80   # probability of detecting a real effect (1 - Type II error)

effect_size = proportion_effectsize(baseline + mde, baseline)
n_required  = zt_ind_solve_power(effect_size=effect_size,
                                  alpha=alpha, power=power,
                                  alternative='two-sided')

print(f"\nSample size planning:")
print(f"  Baseline rate:  {baseline:.0%}")
print(f"  Min effect:     +{mde:.0%}")
print(f"  Required n per group: {int(np.ceil(n_required)):,}")
print(f"  At 1,000 orders/day, test runs: {int(np.ceil(n_required)) * 2 / 1000:.0f} days per group")
```

---

## 34. Correlation and Causation

### The most important distinction in analytics

**Correlation** means two variables move together. **Causation** means one variable *causes* the other to change.

Correlation is observable from data. Causation requires either a controlled experiment (A/B test) or careful causal reasoning. Most business data shows correlations. Assuming they are causal is one of the most common and costly analytical mistakes.

🎬 **Movies analogy:** Films released in summer earn more at the box office. Does summer *cause* high box office? No — studios *choose* to release their biggest films in summer because schools are out and people go to cinemas more. The correlation (summer + high box office) exists, but the causal chain is: summer holiday → more cinema attendance → studios strategically target this period → high box office. Misreading this as "summer weather causes people to like films more" would lead to a completely wrong strategy.

---

### Common spurious correlations in business data

| Observed correlation | Naive causal claim | What's actually happening |
|---------------------|-------------------|--------------------------|
| Customers with more reviews give higher NPS | "Reviews cause satisfaction" | Happy customers leave reviews AND give high NPS — happiness causes both |
| High ad spend → high revenue | "Ads cause revenue" | Companies with successful products spend more on ads. Revenue may cause ad spend, not vice versa. |
| Employees who attend more training are promoted more | "Training causes promotion" | High performers seek training AND get promoted — performance causes both |
| Sellers with lower prices get better reviews | "Lower prices cause better reviews" | Lower-priced items are bought more impulsively and generate less disappointment |

---

### How to think about causation — Simpson's Paradox

**Simpson's Paradox** occurs when an aggregate correlation reverses (or disappears) when you segment the data. It's a dramatic illustration of why segmentation matters.

```python
# Simpson's Paradox example — Olist (constructed illustration)
# Overall: large sellers have lower review scores
# Within each category: large sellers have HIGHER review scores
# The paradox is caused by: large sellers concentrate in electronics,
# which has overall lower review scores regardless of seller size

import pandas as pd
import numpy as np

# Simulate the paradox (since real data may not be perfectly clean)
np.random.seed(42)
n = 1000

# Small sellers: 60% in high-quality categories, 40% in low-quality
# Large sellers: 20% in high-quality categories, 80% in low-quality
# Within each category, large sellers score 0.3 better

data = []
for i in range(n):
    is_large = np.random.rand() < 0.5
    if is_large:
        category = "electronics" if np.random.rand() < 0.8 else "toys"
    else:
        category = "electronics" if np.random.rand() < 0.4 else "toys"

    base_score = 3.5 if category == "electronics" else 4.5
    size_bonus = 0.3 if is_large else 0
    score = base_score + size_bonus + np.random.normal(0, 0.3)
    score = np.clip(score, 1, 5)

    data.append({"is_large": is_large, "category": category, "score": score})

df_sim = pd.DataFrame(data)

print("SIMPSON'S PARADOX DEMONSTRATION")
print(f"\nOverall (aggregate — misleading):")
for size, grp in df_sim.groupby("is_large"):
    label = "Large sellers" if size else "Small sellers"
    print(f"  {label}: avg review = {grp['score'].mean():.2f}")

print(f"\nWithin each category (correct view):")
for cat, cat_grp in df_sim.groupby("category"):
    print(f"  {cat}:")
    for size, grp in cat_grp.groupby("is_large"):
        label = "Large" if size else "Small"
        print(f"    {label}: avg review = {grp['score'].mean():.2f}")

print("\nExplanation:")
print("  Large sellers are concentrated in electronics (lower-scoring category).")
print("  Within electronics: large > small. Within toys: large > small.")
print("  But overall: large < small (because of category mix).")
print("  Decision: do NOT penalise large sellers based on aggregate data.")
print("  Compare within categories, not across.")
```

---

# Part IX — Data Storytelling

## 35. Chart Selection

### Choosing the right visual for your message

Every chart type answers a different question. Using the wrong chart type makes your message harder to see, not easier.

| Message you want to convey | Right chart type |
|---------------------------|-----------------|
| How something changes over time | Line chart |
| Comparing values across categories | Bar chart (horizontal for many categories) |
| Part-to-whole relationship | Stacked bar or pie chart (pie only if ≤5 categories) |
| Distribution of a single variable | Histogram or box plot |
| Relationship between two continuous variables | Scatter plot |
| Ranking | Sorted bar chart |
| Geographic data | Choropleth map |
| Correlation matrix | Heatmap |
| Multiple metrics across multiple categories | Small multiples |

---

### The chart selection flowchart

```
What is your message?
│
├── "This changed over time"
│   → LINE CHART (one or few series)
│   → AREA CHART (if cumulative or stacked)
│
├── "A is bigger/smaller than B"
│   → BAR CHART (few items, < 10)
│   → HORIZONTAL BAR (many items, > 10, or long labels)
│   → DOT PLOT (many items, precise values matter)
│
├── "This is made up of these parts"
│   → STACKED BAR (compare composition across categories)
│   → PIE/DONUT (only if ≤5 parts and sum = 100%)
│   → TREEMAP (many parts, hierarchical)
│
├── "Here is how this is distributed"
│   → HISTOGRAM (continuous variable)
│   → BOX PLOT (compare distributions across groups)
│   → VIOLIN PLOT (show full distribution AND comparison)
│
├── "These two things are related"
│   → SCATTER PLOT (two continuous variables)
│   → BUBBLE CHART (third variable as size)
│   → HEATMAP (many pairs, grid format)
│
└── "Here is the geographic pattern"
    → CHOROPLETH MAP (values by region)
    → BUBBLE MAP (values as circle size at locations)
```

---

### Python implementation — the most useful charts

```python
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
import numpy as np

# Use a clean style
plt.rcParams.update({
    "figure.facecolor": "white",
    "axes.spines.top":  False,
    "axes.spines.right":False,
    "font.size":        11,
})

# ── 1. LINE CHART — trend over time ───────────────────────────────────────
orders  = pd.read_csv("data/olist/olist_orders_dataset.csv",
                      parse_dates=["order_purchase_timestamp"])
payments= pd.read_csv("data/olist/olist_order_payments_dataset.csv")
payments_agg = payments.groupby("order_id")["payment_value"].sum().reset_index()
monthly = (orders.merge(payments_agg, on="order_id", how="left")
           .groupby(orders["order_purchase_timestamp"].dt.to_period("M").astype(str))
           ["payment_value"].sum() / 1e6)

fig, ax = plt.subplots(figsize=(12, 4))
ax.plot(monthly.index, monthly.values, linewidth=2.5, color="#1f77b4", marker="o", markersize=4)
ax.fill_between(monthly.index, monthly.values, alpha=0.15, color="#1f77b4")
ax.set_title("Monthly Revenue — Olist", fontsize=14, fontweight="bold", pad=15)
ax.set_xlabel("")
ax.set_ylabel("Revenue (R$ millions)")
ax.tick_params(axis="x", rotation=45)
# Annotate peak
peak_month = monthly.idxmax()
peak_val   = monthly.max()
ax.annotate(f"Peak: R${peak_val:.1f}M",
            xy=(peak_month, peak_val),
            xytext=(peak_month, peak_val * 1.05),
            ha="center", fontsize=9, color="darkblue")
plt.tight_layout()
plt.savefig("outputs/monthly_revenue_line.png", dpi=150, bbox_inches="tight")


# ── 2. HORIZONTAL BAR CHART — category comparison ─────────────────────────
items    = pd.read_csv("data/olist/olist_order_items_dataset.csv")
products = pd.read_csv("data/olist/olist_products_dataset.csv")
category = pd.read_csv("data/olist/product_category_name_translation.csv")

products_en = products.merge(category, on="product_category_name", how="left")
products_en["cat_en"] = (products_en["product_category_name_english"]
                          .str.replace("_"," ").str.title().fillna("Unknown"))

cat_revenue = (items.merge(products_en[["product_id","cat_en"]], on="product_id")
               .groupby("cat_en")["price"].sum()
               .sort_values()
               .tail(12))  # top 12

fig, ax = plt.subplots(figsize=(9, 6))
bars = ax.barh(cat_revenue.index, cat_revenue.values / 1e6,
               color="#4C72B0", edgecolor="white", height=0.7)
ax.set_xlabel("Revenue (R$ millions)")
ax.set_title("Top 12 Categories by Revenue", fontsize=14, fontweight="bold")

# Add value labels
for bar, val in zip(bars, cat_revenue.values):
    ax.text(bar.get_width() + 0.05, bar.get_y() + bar.get_height()/2,
            f"R${val/1e6:.1f}M", va="center", fontsize=9)

plt.tight_layout()
plt.savefig("outputs/category_revenue_bar.png", dpi=150, bbox_inches="tight")
```

---

## 36. The Pyramid Principle

### What the Pyramid Principle is

The **Pyramid Principle** was developed by Barbara Minto at McKinsey in the 1970s. The core idea: lead with the conclusion, then support it with arguments, then support each argument with data.

**Why it matters:** most people present in the opposite order — they walk through their analysis chronologically, building to a conclusion at the end. This makes audiences wait through minutes of context before learning what they should care about. Executives especially hate this.

🎬 **Movies analogy:** Most films start at the beginning and end at the conclusion. A Pyramid Principle film would open with the final scene — "here is who survived the battle and what the world looks like now" — and then explain how we got there. This sounds counterintuitive for storytelling, but for business communication, the audience already knows there's a problem and needs the answer first.

---

### The pyramid structure

```
         ┌──────────────────────────────────────┐
         │                                      │
         │          GOVERNING THOUGHT           │
         │    (the conclusion, stated upfront)  │
         │                                      │
         └──────────────────────────────────────┘
                        /     |     \
          ┌────────────┐  ┌────────┐  ┌────────────┐
          │  Argument  │  │Argument│  │  Argument  │
          │     1      │  │   2    │  │     3      │
          └─────┬──────┘  └───┬────┘  └─────┬──────┘
                │             │             │
           ┌────┴───┐    ┌────┴───┐    ┌────┴───┐
           │ Data   │    │ Data   │    │ Data   │
           │ Data   │    │ Data   │    │ Data   │
           └────────┘    └────────┘    └────────┘
```

**The governing thought** is your single most important message. Everything else exists to support it.
**Arguments** are the 3–5 reasons why the governing thought is true.
**Data** is the evidence supporting each argument.

---

### Worked example — Olist

**The scenario:** You've analysed the delivery performance decline. Here's how to structure it using the Pyramid Principle.

**Pyramid structure:**

```
GOVERNING THOUGHT:
"Our on-time delivery rate dropped 40% in Q4 because we onboarded a logistics
partner that systematically underperforms on heavy-item categories."

ARGUMENT 1:                 ARGUMENT 2:                 ARGUMENT 3:
The drop is concentrated    The new partner handles      Reverting to the old
in specific regions and      80% of heavy-item           partner or renegotiating
item categories.             shipments in those areas.   the SLA will recover rate.

DATA:                       DATA:                       DATA:
- SP/RJ: -45%               - Partner Y handles         - Partner X had 93% OTIF
- Electronics: -52%           electronics in SP/RJ        in the same period
- Furniture: -48%           - Partner Y OTIF: 72%       - Cost delta: +R$2.50/order
- Other categories: -3%     - Partner X OTIF: 91%       - Customer review impact:
                            - Partner Y SLA: none          -0.8 stars on late orders
```

**The output — how you'd write this in an email or slide:**

```
Subject: Q4 On-Time Delivery Root Cause — Action Required

[Governing thought, immediately]:
Our Q4 on-time delivery rate fell from 91% to 73%, and we've identified
the cause: our new logistics partner (Partner Y) has a 72% OTIF rate on
heavy items, versus 91% for our previous provider.

[Argument 1]:
The decline is not broad — it's concentrated in São Paulo and Rio, in
electronics and furniture categories. Other states and categories are
within 1% of prior performance.

[Argument 2]:
Partner Y handles 80% of heavy-item shipments in SP/RJ. Their OTIF
is 72%, versus Partner X's 91% on the same routes.

[Argument 3]:
We recommend reverting Partner Y allocation for heavy items in SP/RJ.
Cost impact: +R$2.50/order (estimated R$40k/month), offset by
estimated 0.8-star review improvement on affected orders.

Requested action: approval to renegotiate Partner Y contract and
restore Partner X for electronics/furniture in SP/RJ by January 15.
```

Notice: the executive knows the conclusion in the first sentence. They can stop reading there if they trust you, or read on for the evidence. Their choice.

---

### Pyramid Principle template

```
GOVERNING THOUGHT: [One sentence. The conclusion. The recommendation. The finding.]

WHY IS THAT?
  ARGUMENT 1: [One sentence]
    - Evidence 1.1: [data point]
    - Evidence 1.2: [data point]

  ARGUMENT 2: [One sentence]
    - Evidence 2.1: [data point]
    - Evidence 2.2: [data point]

  ARGUMENT 3: [One sentence]
    - Evidence 3.1: [data point]
    - Evidence 3.2: [data point]

THEREFORE:
  RECOMMENDATION: [What should be done, by whom, by when]
  RISK IF NO ACTION: [What happens if nothing changes]
  ALTERNATIVE: [If recommendation isn't feasible, what else]
```

---

## 37. SCR Framework

### Situation, Complication, Resolution

**SCR** (Situation → Complication → Resolution) is a narrative structure for analytical presentations. It creates context (Situation), creates tension (Complication), and then releases it with your finding (Resolution).

It works because the human brain is wired for narrative structure. A presentation with no tension has no reason to hold attention.

🎬 **Movies analogy:** Every film uses this structure. *Situation:* a detective is assigned a routine case. *Complication:* the evidence points to someone who couldn't possibly be the killer. *Resolution:* the detective discovers a double — the impossible suspect was framed. Remove the complication and the film is just a list of events.

---

### SCR template and worked example

**Template:**
```
SITUATION: [The world as it was / the accepted reality]
           Should be agreed-upon facts — nothing controversial here.

COMPLICATION: [Something has changed, or there's a problem with the situation]
              This is why we're having this conversation.
              This creates the need for a decision or action.

RESOLUTION: [What you found / what you recommend]
            The answer to the complication.
```

**Worked example:**

```
SITUATION:
Olist's delivery performance has been our strongest competitive advantage.
Our on-time delivery rate was consistently above 90% in H1 2018,
and our average review score for delivery experience was 4.6/5.

COMPLICATION:
In Q4 2018, on-time delivery rate dropped to 73%, and delivery-related
review scores fell to 3.8/5. Customer complaints about delivery
increased 3× month-on-month. If unaddressed, our review score advantage
will erode and affect seller acquisition — our primary growth driver.

RESOLUTION:
We've identified the root cause: our new logistics partner (onboarded
October 1) has a 72% on-time rate on heavy items, vs 91% for our
previous provider. Reverting to the previous provider for electronics
and furniture in SP/RJ will recover performance in 30 days at an
estimated cost of R$40k/month — less than the revenue risk from
continued review score decline.
```

---

### Principles: storytelling

**Principle 1: One message per slide/page.**
If a slide has three important messages, it has no important messages. The audience remembers the last thing they read. Give them one thing.

**Principle 2: The title IS the message.**
Slide titles like "Q4 Delivery Performance" tell the audience nothing. "On-Time Rate Fell 18pp in Q4 — New Logistics Partner is the Root Cause" tells them everything before they look at the chart.

**Principle 3: Design for skimming.**
Executives will skim your output before reading it. Make the main message visible without reading the body text: headline title, callout annotations on charts, bold key numbers.

**Principle 4: Separate insight from data.**
The chart shows data. The annotation or title communicates the insight. "Figure 1: Monthly Revenue" is data. "Revenue Stalled After Partner Change in October" is insight.

**Principle 5: Anticipate the "so what" for every chart.**
Before including any chart, ask: "What does this chart prove? What would someone conclude from looking at this?" If you can't answer in one sentence, the chart may not belong in the final output.

---

### Templates

**Template: Stakeholder-Facing Insight Summary**
```
THE FINDING:
[One sentence — the single most important thing you discovered]

WHY THIS MATTERS:
[One sentence — business consequence or opportunity]

THE EVIDENCE:
• [Data point 1 — most compelling]
• [Data point 2 — corroborating]
• [Data point 3 — quantifying the scale]

WHAT WE RECOMMEND:
[Specific action + who + by when]

WHAT HAPPENS IF WE DO NOTHING:
[The risk of inaction, quantified if possible]

CONFIDENCE LEVEL:
[High / Medium / Low + why]
```

**Template: Executive Summary Email (5 sentences max)**
```
[Sentence 1: The finding — answer first]
[Sentence 2: The magnitude — how big is this?]
[Sentence 3: The root cause — why is this happening?]
[Sentence 4: The recommendation — what should we do?]
[Sentence 5: The ask — what do you need from the reader?]
```

---
---


---

## 38. Communicating to Non-Technical Stakeholders

### The translation problem

The most common failure mode for analysts is not bad analysis — it's good analysis communicated badly to the wrong audience. A technically rigorous explanation of p-values to a CFO who needs a budget decision in 10 minutes is a communication failure, not an analytical success.

🎬 **Movies analogy:** A director who explains the Kuleshov effect and lens focal lengths to a first-time cinema audience has failed as a communicator, even if the film itself is brilliant. The job is to create an experience the audience can receive, not demonstrate technical mastery.

---

### Know your audience — four stakeholder types

| Stakeholder type | What they care about | What they don't need | How to pitch |
|-----------------|---------------------|---------------------|-------------|
| **Executive / C-suite** | Decision, risk, cost, strategic impact | Methodology, statistical details, data caveats | Conclusion first, one number, one ask |
| **Functional manager** | Team-level impact, operational changes, timeline | Statistical theory, underlying data model | Impact on their team + what you need from them |
| **Technical peer** | Methodology, assumptions, data quality | Business simplification | Full detail; welcome questions; share code |
| **Non-data business partner** | What this means for their work | Any statistics | Analogy + plain-language consequence |

---

### Language translation — analytical to business

| What you'd say technically | What the stakeholder needs to hear |
|--------------------------|-----------------------------------|
| "The p-value is 0.003, rejecting the null hypothesis" | "We're confident this is a real effect, not random noise" |
| "The 95% confidence interval is [R$1.1M, R$1.4M]" | "Our best estimate is R$1.25M, and we'd expect to be within R$150k of that" |
| "There's a negative correlation of -0.43 between delivery days and review score" | "Customers who wait longer give significantly worse reviews — specifically, each extra week of wait costs about half a star" |
| "The model has an AUC of 0.82" | "The model correctly identifies high-risk customers 82% of the time" |
| "We found Simpson's Paradox — the aggregate reverses when segmented" | "The overall number is misleading — let me show you why the category-level view tells a different story" |

---

### The one-page principle

Any finding worth presenting can be communicated in one page. If you need more than one page to communicate the insight (not the supporting evidence — the insight), the communication needs work.

One-page structure:
```
Title: [The finding — not "Q4 Analysis" but "Late Deliveries Drove Q4 Review Decline"]

The finding:     [2 sentences maximum]
Why it matters:  [1 sentence — business consequence]
The evidence:    [3 bullet points with specific numbers]
Recommendation:  [1 specific action + owner + date]
Confidence:      [High / Medium / Low + why]
```

---

### Handling "but what does this mean?"

When a stakeholder asks "but what does this mean?" after you've presented, one of three things happened:
1. You led with data and never stated the conclusion
2. You stated the conclusion but not the implication for their specific work
3. The implication is genuinely ambiguous and requires their business judgement

For (1): restart with the conclusion. "What this means is: late delivery is the primary driver of poor reviews, not product quality."

For (2): connect to their decision. "For the logistics team specifically: this means the Partner Y contract needs renegotiating before Q1."

For (3): be explicit. "The data tells us X happened. Whether to respond with Y or Z depends on your read of the competitive situation — that's a business call I can inform but not make."

---

### Principles: communicating findings

**Principle 1: Answer first.** Never build to a conclusion. State it upfront. Executives don't have patience for a narrative — they want the answer, then they'll decide if they want the detail.

**Principle 2: Translate uncertainty into risk language.** "p < 0.05" means nothing. "There's less than a 5% chance this is random noise — we're confident enough to act on it" means something.

**Principle 3: One number per slide.** If you're presenting a 0.8-star drop in review scores, that's the number. Don't also show the overall NPS trend and the category breakdown on the same slide. One number. One message.

**Principle 4: Anticipate the "so what."** Before every chart and every finding, ask yourself: "What should the audience do differently because of this?" If you can't answer, either the finding isn't actionable or you haven't connected it to the decision.

**Principle 5: Own your uncertainty explicitly.** "I'm confident in this finding" and "this is my best estimate but the data is noisy" are both legitimate — but you must say which one it is. Unexplained uncertainty erodes trust. Named, bounded uncertainty builds it.

---

# Part X — BI Tools

## 39. Tableau

### What Tableau is — and why it matters

**Tableau** is the dominant enterprise data visualisation and BI platform. Founded in 2003 from a Stanford research project, acquired by Salesforce in 2019 for $15.7B. It is not a programming environment — it's a visual, drag-and-drop interface for exploring and presenting data.

Tableau's core value proposition: business users (non-coders) can build interactive dashboards themselves, without SQL or Python. A finance manager should be able to drag "Sales" onto a chart and "Region" onto a colour shelf and immediately see a segmented view — no engineering required.

---

### How Tableau works — the architecture

```
TABLEAU ARCHITECTURE:

Data Sources                    Tableau Desktop
(connect to any)                (build it here)
────────────────                ────────────────
Spreadsheets (Excel, CSV)  ───▶ Drag fields to shelves
SQL Databases              ───▶ VizQL engine renders charts
Cloud DW (BigQuery, Snow)  ───▶ Calculated fields, filters
Web APIs                   ───▶ Dashboards (multiple views)
                                Stories (narrative sequence)
         │
         ▼
Tableau Server / Tableau Cloud
(share and govern it here)
─────────────────────────────
Publish workbooks and data sources
Schedule refreshes
Row-level security (who sees what)
Usage analytics
         │
         ▼
Business Users (consume)
─────────────────────────
Interact with dashboards in browser
Filter, drill down, export
No Tableau Desktop license needed to view
```

---

### Key Tableau concepts

**Data Source:**
The connection to your data. Tableau supports 70+ connectors. A single workbook can connect to multiple data sources.

**Live vs Extract:**
- **Live connection:** queries the database in real-time on every interaction. Shows current data but can be slow on large databases.
- **Extract:** Tableau downloads a snapshot of the data into its proprietary `.hyper` format. Faster interactions but data is only as fresh as the last extract.

**Dimensions vs Measures:**
- **Dimensions:** categorical data (text, dates, geographies). Drive the structure of the view — create rows, columns, colours.
- **Measures:** numeric data (revenue, count, LTV). Get aggregated (SUM, AVG, COUNT) in the view.
- Tableau automatically classifies fields as dimension or measure when you connect to data. You can change it.

**Marks card:**
Controls how the data is displayed: as bars, lines, circles, shapes, text. Drag a dimension to Colour and each category gets a different colour. Drag a measure to Size and points scale by value.

**Shelves:**
The Tableau interface is built around shelves:
- **Rows / Columns:** what appears on each axis
- **Color:** colour encode by a dimension or measure
- **Size:** size encode marks
- **Label:** add text labels
- **Detail:** add context without changing the chart type
- **Tooltip:** what appears when you hover

**Calculated fields:**
Tableau's formula language. Create new fields from existing ones:
```
// Profit margin
[Profit] / [Revenue]

// Year-over-year growth
(ZN(SUM([Revenue])) - LOOKUP(ZN(SUM([Revenue])), -1)) / ABS(LOOKUP(ZN(SUM([Revenue])), -1))

// Customer segment
IF [LTV] > 1000 THEN "Premium"
ELSEIF [LTV] > 200 THEN "Standard"
ELSE "Basic" END

// Days since last order
DATEDIFF('day', [Last Order Date], TODAY())
```

**Table calculations:**
Calculations that run on the result set *after* aggregation — like window functions in SQL. Common table calculations:
- **Running total:** cumulative sum of a measure
- **Percent of total:** each row as % of the whole
- **Rank:** rank each row by a measure
- **Moving average:** N-period rolling average
- **Year-over-year:** compare to previous period

**LOD (Level of Detail) expressions:**
The most powerful Tableau feature. LOD expressions let you compute at a different level of granularity than the view.

```
// FIXED LOD: compute regardless of view filters
{ FIXED [Customer ID] : SUM([Revenue]) }
// Always computes per customer, even if the view shows by month

// INCLUDE LOD: add a dimension to the current view's aggregation
{ INCLUDE [Product Category] : AVG([Review Score]) }

// EXCLUDE LOD: remove a dimension from the current view's aggregation
{ EXCLUDE [Month] : SUM([Revenue]) }
// Annual revenue, even when the view is monthly
```

---

### Tableau vs Python — side by side

| Task | Tableau approach | Python equivalent |
|------|-----------------|-------------------|
| Revenue by month | Drag date to columns, revenue to rows | `df.groupby("month")["revenue"].sum()` |
| Top 10 categories by revenue | Drag category to rows, revenue to columns, filter Top N | `df.groupby("cat")["revenue"].sum().nlargest(10)` |
| YoY comparison | Table calculation: "Percent difference from previous year" | `LAG()` window function or `pct_change()` |
| Customer segment colours | Drag segment to Color shelf | `df.plot(kind="bar", color=segment_colors)` |
| Interactive filter | Add filter shelf, check "Show Filter" | Plotly/Dash dropdown widgets |
| LOD per-customer total | FIXED LOD expression | `df.groupby("customer")["revenue"].transform("sum")` |
| Dashboard | Drag multiple views onto dashboard canvas | Plotly Dash / Streamlit app |

**When to use Tableau vs Python:**
- Tableau: stakeholders need to self-serve, filter, explore interactively. No coding.
- Python: complex statistical analysis, custom ML scoring, data prep before visualisation, automation.
- Both: build in Python, present in Tableau (export clean CSV/Parquet, connect Tableau to it).

---

## 40. Power BI

### What Power BI is

**Power BI** is Microsoft's business intelligence platform. Released in 2015, it is the dominant BI tool in organisations that run on the Microsoft stack (Azure, Office 365, Excel, SQL Server). It is split into:
- **Power BI Desktop:** free Windows application for building reports
- **Power BI Service:** cloud platform for sharing, collaboration, and governance
- **Power BI Mobile:** viewing on phones/tablets

**Power BI vs Tableau:**

| | Power BI | Tableau |
|--|---------|---------|
| Price | Free Desktop; ~$10/user/month for sharing | ~$70/user/month (Creator) |
| Ecosystem | Microsoft (Azure, Excel, Teams, Dynamics) | Salesforce (CRM integration) |
| Scripting | DAX (Data Analysis Expressions), M (Power Query) | Tableau Calculated Fields |
| Strengths | Cost, Microsoft integration, Excel-like feel | Visualisation quality, flexibility |
| Weaknesses | Steeper DAX learning curve, less flexible layout | Higher cost, no Microsoft ecosystem |
| Users | Finance teams, Microsoft shops | Analysts, data teams |

---

### The Power BI data model

Power BI's power comes from its data model. Unlike Tableau (which works with flat tables), Power BI builds a **star schema data model** within the tool. You define relationships between tables, and measures (calculated fields) automatically follow those relationships.

```
Power BI Data Model:

dim_date ──────────────────────────────────────────┐
    date_id (PK)                                    │
                                                    │
dim_customer ──────────────────────────────────────┤
    customer_id (PK)                                │ Relationships
    customer_state                                  │ (one-to-many)
    region                                          │
                                                    ├──▶ fct_orders (fact table)
dim_product ───────────────────────────────────────┤    order_id (PK)
    product_id (PK)                                 │    date_id (FK)
    category_en                                     │    customer_id (FK)
                                                    │    product_id (FK)
dim_seller ────────────────────────────────────────┘    revenue
    seller_id (PK)                                      delivery_days
    seller_state                                        is_late
```

This structure means: when a user filters by `customer_state`, Power BI automatically knows to filter `fct_orders` through the relationship — you don't need to write the join yourself.

---

### DAX — Data Analysis Expressions

**DAX** is Power BI's formula language. It is more powerful than Tableau's calculated fields but significantly harder to learn. The key concept: DAX operates on the **filter context** — the set of filters currently applied to the model.

```dax
-- Basic measures (the equivalent of SQL aggregate functions)
Total Revenue = SUM(fct_orders[revenue])
Order Count   = COUNTROWS(fct_orders)
Avg Order     = AVERAGEX(fct_orders, fct_orders[revenue])

-- Conditional aggregation
Revenue Delivered = CALCULATE(
    SUM(fct_orders[revenue]),
    fct_orders[order_status] = "delivered"
)

-- Time intelligence (built-in DAX functions for time comparison)
Revenue YTD = TOTALYTD(SUM(fct_orders[revenue]), dim_date[date])
Revenue LY  = CALCULATE(SUM(fct_orders[revenue]), SAMEPERIODLASTYEAR(dim_date[date]))
Revenue MoM = DIVIDE([Total Revenue] - [Revenue Last Month], [Revenue Last Month])

-- YoY growth %
Revenue YoY % = 
    VAR current_revenue = SUM(fct_orders[revenue])
    VAR prior_revenue   = CALCULATE(
        SUM(fct_orders[revenue]),
        DATEADD(dim_date[date], -1, YEAR)
    )
    RETURN DIVIDE(current_revenue - prior_revenue, prior_revenue)

-- Running total
Revenue Running Total = 
    CALCULATE(
        SUM(fct_orders[revenue]),
        FILTER(
            ALL(dim_date[month]),
            dim_date[month] <= MAX(dim_date[month])
        )
    )

-- Customer LTV (calculated across the entire customer relationship)
Customer LTV = 
    CALCULATE(
        SUM(fct_orders[revenue]),
        ALLEXCEPT(dim_customer, dim_customer[customer_id])
    )
    -- ALLEXCEPT removes all filters except customer_id
    -- meaning: total revenue for this customer, regardless of other view filters
```

---

### Power BI vs Python — side by side

| Task | Power BI / DAX | Python equivalent |
|------|---------------|-------------------|
| Total revenue | `SUM(fct_orders[revenue])` | `df["revenue"].sum()` |
| Revenue by category | Drag to visual + SUM measure | `df.groupby("category")["revenue"].sum()` |
| YTD revenue | `TOTALYTD(SUM(...), date)` | `df[df["month"] <= current_month]["revenue"].sum()` |
| LY comparison | `SAMEPERIODLASTYEAR(...)` | `LAG()` or `merge()` on same month - 1 year |
| Filter context | Implicit in every measure (filter propagates) | Explicit `.query()` or `.loc[]` filter |
| Row-level security | Role definition in data model | Applied at dashboard/API level |

---

## 41. Looker and LookML

### What Looker is

**Looker** (acquired by Google in 2019 for $2.6B) is a BI platform with a fundamentally different approach from Tableau and Power BI. Looker's key differentiator: **LookML** — a code-based semantic layer that defines business logic centrally.

**The problem Looker solves:** in most organisations, the same metric is computed differently by different teams. The marketing team calculates "revenue" as gross sales. Finance calculates it as net of returns. Sales calculates it as booked but not yet delivered. When different departments report different numbers, credibility collapses.

Looker's solution: define "revenue" once in LookML. Every chart, every dashboard, every analyst who queries the data gets the same definition. It's the single source of truth, enforced in code.

---

### LookML concepts

**Model:** the top-level file that defines database connections and includes views.

**View:** defines the fields (dimensions and measures) for one table.

**Explore:** defines which views can be joined together and how. An Explore is what analysts see in the Looker UI — the set of dimensions and measures available to query.

**Dimension:** an attribute to group or filter by (equivalent to Tableau dimension, SQL GROUP BY column).

**Measure:** an aggregation over rows (equivalent to Tableau measure, SQL aggregate function).

```yaml
# LookML view file: views/fct_orders.view.lkml

view: fct_orders {
  sql_table_name: analytics.fct_orders ;;

  # Dimensions (GROUP BY / filter columns)
  dimension: order_id {
    primary_key: yes
    type:        string
    sql:         ${TABLE}.order_id ;;
  }

  dimension: order_status {
    type:        string
    sql:         ${TABLE}.order_status ;;
  }

  dimension_group: ordered_at {
    type:        time
    timeframes:  [date, week, month, quarter, year]
    sql:         ${TABLE}.ordered_at ;;
    # Creates: ordered_at_date, ordered_at_week, ordered_at_month, etc.
  }

  dimension: is_late {
    type:        yesno
    sql:         ${TABLE}.is_late = 1 ;;
    label:       "Late Delivery"
  }

  # Measures (aggregate functions — the business metrics)
  measure: total_revenue {
    type:        sum
    sql:         ${TABLE}.item_revenue ;;
    value_format: "R$ #,##0.00"
    description: "Sum of revenue from delivered orders"
    filters:     [order_status: "delivered"]
    # This filter is applied automatically whenever this measure is used
    # No analyst needs to remember to filter — it's built into the definition
  }

  measure: order_count {
    type:        count_distinct
    sql:         ${TABLE}.order_id ;;
  }

  measure: avg_order_value {
    type:        number
    sql:         ${total_revenue} / NULLIF(${order_count}, 0) ;;
    value_format: "R$ #,##0.00"
  }

  measure: late_delivery_rate {
    type:        average
    sql:         ${TABLE}.is_late ;;
    value_format: "0.0%"
    description: "% of delivered orders that arrived after estimated date"
  }

  measure: on_time_rate {
    type:        number
    sql:         1 - ${late_delivery_rate} ;;
    value_format: "0.0%"
  }
}
```

```yaml
# LookML explore file: models/olist.model.lkml
connection: "snowflake_prod"

include: "/views/*.view.lkml"

explore: fct_orders {
  label: "Order Analytics"

  join: dim_customer {
    type:        left_outer
    sql_on:      ${fct_orders.customer_sk} = ${dim_customer.customer_sk} ;;
    relationship: many_to_one
  }

  join: dim_product {
    type:        left_outer
    sql_on:      ${fct_orders.product_sk} = ${dim_product.product_sk} ;;
    relationship: many_to_one
  }
}
```

**The analyst experience in Looker:** the analyst opens the "Order Analytics" explore, sees a list of dimensions and measures. They select "Customer State" (dimension) and "Total Revenue" (measure). Looker generates the SQL, runs it, and shows a table. They add a date filter. They export to a dashboard. They never wrote SQL — but the SQL generated is precisely correct because the LookML is precisely correct.

---

## 42. BI Tool Decision Framework

### Which tool to use when

```
DECISION TREE:

Your organisation already uses...
│
├── Microsoft 365 / Azure → Power BI (licensing included, AAD integration)
│
├── Salesforce → Tableau (same vendor, native CRM integration)
│
└── Google / GCP → Looker Studio (free) or Looker (enterprise)


Your primary user is...
│
├── Business user, non-technical, needs to self-serve → Tableau or Power BI
│
├── SQL-proficient analyst who wants a governed semantic layer → Looker
│
├── Developer who wants to code dashboards → Evidence.dev, Observable, Streamlit
│
└── Data scientist who wants full control → Python (Plotly, Seaborn, Altair)


Your budget is...
│
├── Minimal → Tableau Public (free, public only), Power BI Desktop (free),
│             Looker Studio (free), Metabase (open source), Superset (open source)
│
├── $10–30/user/month → Power BI Pro, Metabase Cloud
│
└── $50+/user/month → Tableau Creator, Looker, Sigma Computing
```

---

### Complete BI tool comparison

| | Tableau | Power BI | Looker | Metabase | Python (Plotly/Dash) |
|--|---------|---------|--------|---------|---------------------|
| **Type** | Visual analytics | BI + data model | Governed analytics | Self-serve analytics | Code-based |
| **Best for** | Visual storytelling | Finance, Microsoft shops | Enterprise governance | Small teams, SQL users | Data scientists |
| **Learning curve** | Medium | Medium-High (DAX) | High (LookML) | Low | Medium (Python) |
| **Interactivity** | High | High | High | Medium | High (Dash) |
| **Semantic layer** | Limited | Data model (partial) | LookML (best-in-class) | None | None |
| **Governance** | Tableau Server | Power BI Service | Built-in | Limited | Custom |
| **Cost** | High | Low-Medium | High | Low | Free |
| **Cloud native** | Tableau Cloud | Power BI Service | GCP native | Self-hosted or cloud | Any |
| **SQL needed?** | No | No | Yes (for LookML) | Yes (optional) | Yes |
| **Python integration** | TabPy | Python visuals | No | No | Native |

---
---

# Part XI — Analytics in Practice

## 43. End-to-End Analysis Walkthrough

### The scenario

**Request (as received):**
> *"Hey, can you look at our seller performance? We're thinking about launching a seller coaching program and want to know if it's worth it."*

This is the kind of request analysts receive constantly. No metric. No time period. No definition of "worth it." Let's work through the full process.

---

### Step 1: Frame the question

**Applying the framing template:**

```
DECISION: Whether to invest in a seller coaching program
PRECISE QUESTION: What is the relationship between seller performance
                  metrics (review score, late delivery rate) and GMV
                  outcomes? Specifically: would improving underperforming
                  sellers to average performance materially increase GMV?

METRICS: Review score, late delivery rate, GMV per seller
DIMENSIONS: Seller tier (by performance), time period
PERIOD: Last 12 months
BASELINE: Top-quartile sellers vs bottom-quartile sellers
SUCCESS CRITERIA:
  If bottom-quartile sellers generate significantly less GMV AND
  there's a large gap vs top-quartile → coaching could recover material GMV
  If gap is small OR bottom-quartile sellers are few → ROI questionable

DATA: olist_orders, olist_order_items, olist_sellers, olist_reviews
```

---

### Step 2: Build the analysis

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
import scipy.stats as stats

# Load
orders   = pd.read_csv("data/olist/olist_orders_dataset.csv",
                       parse_dates=["order_purchase_timestamp"])
items    = pd.read_csv("data/olist/olist_order_items_dataset.csv")
sellers  = pd.read_csv("data/olist/olist_sellers_dataset.csv")
reviews  = pd.read_csv("data/olist/olist_order_reviews_dataset.csv")

# Build seller-level performance table
delivered = orders[orders["order_status"] == "delivered"].copy()
delivered["is_late"] = (delivered["order_delivered_customer_date"]
                        > delivered["order_estimated_delivery_date"]).astype(int)

# Seller metrics
seller_orders = (items
    .merge(delivered[["order_id","is_late"]], on="order_id", how="inner"))

seller_reviews = (reviews
    .merge(items[["order_id","seller_id"]], on="order_id", how="left")
    .groupby("seller_id")["review_score"]
    .agg(avg_review="mean", review_count="count")
    .reset_index())

seller_perf = (seller_orders
    .groupby("seller_id")
    .agg(
        gmv         = ("price",    "sum"),
        orders      = ("order_id", "count"),
        late_rate   = ("is_late",  "mean"),
    )
    .reset_index()
    .merge(seller_reviews, on="seller_id", how="left")
    .query("orders >= 10")  # minimum 10 orders for stability
    .query("review_count >= 5"))  # minimum 5 reviews

print(f"Sellers in analysis: {len(seller_perf):,}")
print(f"\nPerformance distribution:")
print(seller_perf[["gmv","avg_review","late_rate"]].describe().round(3))

# Segment sellers into quartiles
seller_perf["review_quartile"] = pd.qcut(
    seller_perf["avg_review"], 4, labels=["Q1 (worst)","Q2","Q3","Q4 (best)"]
)

# How does GMV vary by review quartile?
by_quartile = seller_perf.groupby("review_quartile", observed=True).agg(
    sellers     = ("seller_id",   "count"),
    total_gmv   = ("gmv",         "sum"),
    avg_gmv     = ("gmv",         "mean"),
    avg_review  = ("avg_review",  "mean"),
    avg_late    = ("late_rate",   "mean"),
).reset_index()

print("\nPerformance by review score quartile:")
print(by_quartile.round(3).to_string(index=False))

# Statistical test: does top quartile generate more GMV than bottom?
q4_gmv = seller_perf[seller_perf["review_quartile"] == "Q4 (best)"]["gmv"]
q1_gmv = seller_perf[seller_perf["review_quartile"] == "Q1 (worst)"]["gmv"]
t_stat, p_val = stats.ttest_ind(q4_gmv, q1_gmv)
print(f"\nt-test: Q4 vs Q1 GMV: t={t_stat:.2f}, p={p_val:.4f}")

# The ROI calculation: what if Q1 sellers performed like Q3?
q1_avg_gmv  = q1_gmv.mean()
q3_avg_gmv  = seller_perf[seller_perf["review_quartile"] == "Q3"]["gmv"].mean()
n_q1        = len(seller_perf[seller_perf["review_quartile"] == "Q1 (worst)"])
gmv_upside  = (q3_avg_gmv - q1_avg_gmv) * n_q1

print(f"\nROI estimation:")
print(f"  Q1 sellers: {n_q1} sellers, avg GMV: R${q1_avg_gmv:,.0f}")
print(f"  Q3 sellers avg GMV: R${q3_avg_gmv:,.0f}")
print(f"  Potential GMV uplift if Q1 → Q3: R${gmv_upside:,.0f}")
print(f"  At 20% take rate: R${gmv_upside * 0.20:,.0f} additional revenue")
```

---

### Step 3: Validate

```python
# Sanity check 1: do our seller counts make sense?
total_sellers_raw = items["seller_id"].nunique()
print(f"Total unique sellers (all time): {total_sellers_raw:,}")
print(f"Sellers in analysis (10+ orders, 5+ reviews): {len(seller_perf):,}")
print(f"Coverage: {len(seller_perf)/total_sellers_raw:.1%}")

# Sanity check 2: does the GMV in our table match total marketplace GMV?
our_total_gmv = seller_perf["gmv"].sum()
actual_gmv    = items.merge(delivered[["order_id"]], on="order_id")["price"].sum()
print(f"\nGMV sanity check:")
print(f"  Our analysis GMV: R${our_total_gmv:,.0f}")
print(f"  Total delivered GMV: R${actual_gmv:,.0f}")
print(f"  Coverage: {our_total_gmv/actual_gmv:.1%}")
# If < 80%, our seller filter is too aggressive

# Sanity check 3: are the quartile group sizes equal?
print(f"\nQuartile group sizes:")
print(seller_perf["review_quartile"].value_counts().sort_index())
```

---

### Step 4: Communicate — applying the Pyramid Principle

**The structured output:**

```
GOVERNING THOUGHT:
Improving the bottom 25% of sellers to average performance could recover
R$XX million in annual GMV, making a seller coaching program financially
justified if cost per seller is below R$YY.

ARGUMENT 1: Bottom-quartile sellers generate 40% less GMV than average.
  - Q1 sellers (avg review < 3.5): avg R$1,200 GMV/year
  - Q3 sellers (avg review 4.0-4.5): avg R$2,100 GMV/year
  - Difference: R$900/seller/year

ARGUMENT 2: The performance gap is driven by delivery issues, not product quality.
  - Q1 sellers: 18% late delivery rate
  - Q3 sellers: 5% late delivery rate
  - Review scores correlate with late delivery (r = -0.62, p < 0.001)
  - Coaching on shipping practices is more tractable than product quality

ARGUMENT 3: The opportunity is significant but concentrated.
  - 387 sellers are in Q1 (bottom quartile)
  - Combined GMV gap vs Q3 average: R$348k/year
  - At Olist's 20% take rate: R$70k additional revenue
  - Break-even: coaching program costs < R$180/seller/year

RECOMMENDATION:
Pilot coaching program with 50 Q1 sellers who have good product ratings
but high late delivery rates (the most improvable cohort). Measure
on-time delivery rate and review score at 90 days. If improvement is
≥5pp delivery rate, scale to all Q1 sellers.
```

---

## 44. Root Cause Analysis Workflow

### A repeatable 5-step process

Root cause analysis is one of the most valuable analytical skills. The template:

```
STEP 1: QUANTIFY THE SYMPTOM
  What exactly is the problem? How big is it? When did it start?
  → Gives you something specific to explain

STEP 2: FORM HYPOTHESES
  Using your issue tree, list possible causes
  → MECE list of potential root causes

STEP 3: RULE OUT THE IMPOSSIBLE FIRST
  Which hypotheses can you eliminate immediately with one data check?
  → Narrows the search quickly

STEP 4: TEST THE MOST LIKELY
  Run the data analysis that would confirm or deny the most likely cause
  → Efficient — test the highest-probability branch first

STEP 5: CONFIRM THE ROOT CAUSE
  Once you think you've found it, verify with a second independent test
  → Never share a root cause finding without corroborating evidence
```

---

## 45. From Analysis to Recommendation

### The last mile — where most analysts fail

Many analysts produce excellent analyses but weak recommendations. The recommendation is where the value is transferred from analyst to organisation. An analysis with no recommendation is an interesting document. An analysis with a clear, justified recommendation is a decision tool.

**The anatomy of a strong recommendation:**

```
RECOMMENDATION STRUCTURE:

1. THE ACTION
   What specifically should be done?
   Who should do it?
   By when?
   [Vague: "Improve seller performance"]
   [Strong: "Suspend the bottom 5% of sellers by review score (< 3.0)
             on February 1st; provide 60 days to appeal with evidence
             of corrective action."]

2. THE EXPECTED OUTCOME
   If we take this action, what specific change do we expect?
   How will we measure it?
   What is the timeline for seeing the effect?
   ["Expected: 0.2 average review score improvement across platform
     within 90 days of implementation"]

3. THE EVIDENCE BASE
   Why do we believe this will work?
   What analogous evidence exists?
   How strong is the causal chain?
   ["Sellers with review score > 4.0 generate 40% more GMV.
     Removing low-score sellers shifts the mix toward high-score sellers.
     The relationship is statistically significant (p < 0.001)
     and consistent across product categories."]

4. THE RISKS
   What could go wrong?
   What assumptions are we making?
   What would change your recommendation?
   ["Risk: suspended sellers may not be replaceable in some categories,
     leading to inventory gaps. Mitigation: check category coverage
     before suspension — don't suspend if coverage drops below 3 sellers."]

5. THE ALTERNATIVES CONSIDERED
   What other options did you evaluate?
   Why did you recommend this one and not another?
   ["Alternative: coaching program. Rejected because 90-day coaching
     timeline is longer than the competitive window, and data shows
     low-review sellers have been on platform 12+ months without improvement."]
```

---

## Appendix A: Analytics Tool Stack Reference

```
EXPLORATORY ANALYSIS:
  Python:      pandas, numpy (data manipulation)
  Viz:         matplotlib, seaborn (static), plotly (interactive)
  SQL:         DuckDB (local), any warehouse (cloud)
  Notebooks:   Jupyter, VS Code, Google Colab

STATISTICAL ANALYSIS:
  scipy.stats  — t-tests, chi-square, ANOVA, correlations
  statsmodels  — regression, time series, econometrics
  pingouin     — cleaner API for statistical tests

BUSINESS INTELLIGENCE:
  Tableau      — visual analytics, drag-and-drop, enterprise
  Power BI     — Microsoft ecosystem, DAX, star schema model
  Looker       — LookML semantic layer, governance
  Metabase     — open source, self-hosted, SQL-friendly
  Evidence.dev — code-first BI (Markdown + SQL)

PRESENTATION / COMMUNICATION:
  PowerPoint / Google Slides — traditional deck
  Notion / Confluence        — written analysis
  Streamlit / Dash           — Python-built interactive apps
  Observable / Quarto        — code-based analytical narratives
```

---

## Appendix B: Python Libraries Quick Reference

```bash
# Core analytics stack
pip install pandas numpy scipy matplotlib seaborn plotly

# Statistical analysis
pip install statsmodels pingouin

# Interactive dashboards
pip install streamlit plotly-dash

# Data connection
pip install duckdb sqlalchemy pyarrow

# All at once
pip install pandas numpy scipy matplotlib seaborn plotly \
            statsmodels pingouin duckdb sqlalchemy pyarrow \
            streamlit
```

---

## Appendix C: Metrics Glossary

| Term | Definition |
|------|-----------|
| **A/B Test** | Randomised experiment comparing two versions; measures causal effect |
| **AARRR** | Acquisition, Activation, Retention, Revenue, Referral — funnel framework |
| **AOV** | Average Order Value = Revenue / Orders |
| **ARPU** | Average Revenue Per User = Total Revenue / Users |
| **Attrition** | Employees leaving an organisation (voluntary + involuntary) |
| **Bimodal** | Distribution with two peaks — signals two mixed populations |
| **CAC** | Customer Acquisition Cost = Marketing Spend / New Customers |
| **Churn** | Rate at which customers stop doing business |
| **Cohort** | Group sharing a common characteristic at a specific time (e.g., acquisition month) |
| **Confidence Interval** | Range within which the true value falls with stated probability |
| **Control Group** | In A/B test: group experiencing the unchanged version |
| **Correlation** | Statistical relationship between two variables — NOT causation |
| **DAU/MAU** | Daily Active Users / Monthly Active Users — engagement ratio |
| **Diagnostic Analytics** | Explaining why something happened (drill-down, root cause) |
| **DIO** | Days of Inventory Outstanding = (Inventory / COGS) × 365 |
| **Effect Size** | Practical magnitude of a difference (Cohen's d) — vs statistical significance |
| **eNPS** | Employee Net Promoter Score |
| **Exploratory Analysis** | Open-ended investigation of a dataset before formal analysis |
| **Fill Rate** | % of order items shipped as ordered |
| **Funnel** | Sequential steps in a process; measures drop-off at each stage |
| **GMV** | Gross Merchandise Value — total transaction value on a marketplace |
| **Governing Thought** | The single most important conclusion in a Pyramid Principle document |
| **IQR** | Interquartile Range = P75 − P25; used for outlier detection |
| **Issue Tree** | MECE hierarchical decomposition of a complex problem |
| **KPI** | Key Performance Indicator — a metric tied to a strategic objective |
| **LAG/LEAD** | SQL window functions for accessing previous/next row values |
| **LOS** | Length of Stay — average hospital admission days |
| **LTV** | Lifetime Value — total expected revenue from a customer |
| **MCAR/MAR/MNAR** | Missing Completely/At/Not At Random — missing data taxonomy |
| **MECE** | Mutually Exclusive, Collectively Exhaustive |
| **MRR** | Monthly Recurring Revenue — subscription business metric |
| **NMV** | Net Merchandise Value — GMV minus returns and cancellations |
| **NPS** | Net Promoter Score = % Promoters − % Detractors |
| **North Star Metric** | The one metric that best captures core product value |
| **NPL Ratio** | Non-Performing Loan Ratio — credit risk metric |
| **Null Hypothesis** | The assumption that there is no effect; hypothesis testing tries to reject this |
| **OKR** | Objectives and Key Results — goal-setting framework |
| **OTIF** | On Time In Full — supply chain performance metric |
| **Outlier** | Value unusually far from the rest of the distribution |
| **p-value** | Probability of observing a result as extreme as yours if null hypothesis is true |
| **Pareto Principle** | 80/20 rule — 20% of causes drive 80% of effects |
| **Predictive Analytics** | Forecasting future outcomes using models |
| **Prescriptive Analytics** | Recommending optimal actions based on data and models |
| **Pyramid Principle** | Structuring communication: conclusion first, then supporting arguments |
| **RFM** | Recency, Frequency, Monetary — customer segmentation framework |
| **ROAS** | Return on Ad Spend = Revenue from ads / Ad spend |
| **SCR** | Situation, Complication, Resolution — narrative framework |
| **SLA** | Service Level Agreement — defined performance commitment |
| **Simpson's Paradox** | Aggregate trend reverses when data is segmented |
| **Skewness** | Asymmetry of a distribution; positive = right tail; negative = left tail |
| **Statistical Significance** | Result unlikely to be due to random chance (p < 0.05 convention) |
| **Surrogate Metric** | A measurable proxy for a harder-to-measure outcome |
| **Treatment Group** | In A/B test: group experiencing the new version |
| **Type I Error** | False positive — rejecting null hypothesis when it's actually true |
| **Type II Error** | False negative — failing to reject null hypothesis when it's actually false |
| **Univariate Analysis** | Examining one variable in isolation |
| **Winsorizing** | Capping extreme values at a percentile rather than removing them |
| **Window Function** | SQL function computing across related rows without collapsing them |
| **YoY** | Year-over-Year — comparison to same period prior year |

---

## Appendix D: SQL Analytics Patterns Cheatsheet

```sql
-- RUNNING TOTAL
SUM(revenue) OVER (ORDER BY month ROWS UNBOUNDED PRECEDING)

-- ROLLING 3-MONTH AVERAGE
AVG(revenue) OVER (ORDER BY month ROWS BETWEEN 2 PRECEDING AND CURRENT ROW)

-- RANK WITHIN GROUP
RANK() OVER (PARTITION BY category ORDER BY revenue DESC)

-- MONTH-OVER-MONTH CHANGE
revenue - LAG(revenue, 1) OVER (ORDER BY month)

-- % OF TOTAL
revenue / SUM(revenue) OVER ()

-- % OF GROUP TOTAL
revenue / SUM(revenue) OVER (PARTITION BY category)

-- YEAR-OVER-YEAR COMPARISON
revenue / LAG(revenue, 12) OVER (ORDER BY month) - 1

-- FIRST ORDER DATE PER CUSTOMER
MIN(order_date) OVER (PARTITION BY customer_id)

-- CUSTOMER ORDER SEQUENCE
ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date)

-- DAYS SINCE PREVIOUS ORDER
order_date - LAG(order_date) OVER (PARTITION BY customer_id ORDER BY order_date)

-- PERCENTILE BUCKET
NTILE(4) OVER (ORDER BY revenue)  -- 1=bottom 25%, 4=top 25%

-- CUMULATIVE % OF TOTAL
100.0 * SUM(revenue) OVER (ORDER BY revenue DESC ROWS UNBOUNDED PRECEDING)
      / SUM(revenue) OVER ()
```

---

*End of Data Analytics 101*
*Chat #4 · June 2026*
