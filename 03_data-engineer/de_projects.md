# Data Engineering Projects — 15 Practice Projects

> **How to use this file**
> Same 7 datasets as SQL and Python tracks. Projects build progressively from single-file pipelines to full multi-table orchestrated workflows.
> Each project has a **Business Context** (the "why"), **Setup** code, and 15 questions of increasing difficulty.
> Q1–Q5: foundational (format, extraction, basic transform)
> Q6–Q10: intermediate (schema design, quality checks, loading)
> Q11–Q15: advanced (pipeline architecture, orchestration, production patterns)
> Answers at the bottom — attempt first.
>
> **Chat #3 · June 2026**

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

## Standard Setup

```python
import pandas as pd
import numpy as np
import duckdb
import json
import time
import logging
from pathlib import Path
from datetime import date, timedelta
from sqlalchemy import create_engine

logging.basicConfig(level=logging.INFO,
                    format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger(__name__)

DATA_DIR = Path("data")
OUT_DIR  = Path("outputs")
OUT_DIR.mkdir(exist_ok=True)
```

---

## Project 1 🟢 — File Format Conversion Pipeline
**Dataset:** D1 — Olist | **Focus:** File formats, Parquet, partitioning

### Business Context
Your team receives Olist data as CSVs every day. Your job: convert them to Parquet, apply partitioning, and verify the gains.

**Q1.** Load `olist_orders_dataset.csv`. Print: file size (KB), shape, and all column names. How many date columns are stored as strings?

**Q2.** Parse all date columns. Save the DataFrame as Parquet (uncompressed). Compare file size vs CSV. What is the compression ratio?

**Q3.** Save the same data as Parquet with `snappy` compression. Then with `gzip`. Then with `zstd`. Print file sizes for all four formats. Which gives the smallest file? Which is fastest to write?
> *Hint: Use `time.perf_counter()` to measure write time.*

**Q4.** Load the snappy Parquet back. Verify: same row count, same column names, same dtypes as original CSV. What happened to the date columns' dtype?

**Q5.** Read only 3 columns from the Parquet file using `columns=["order_id","order_status","order_purchase_timestamp"]`. How long does this take vs reading the full file? Why is it faster?

**Q6.** Add `year` and `month` integer columns extracted from `order_purchase_timestamp`. Write as partitioned Parquet using `pyarrow.parquet.write_to_dataset` with `partition_cols=["year","month"]`. What directory structure is created?

**Q7.** Read ONLY January 2018 orders from the partitioned dataset using `filters=[("year","==",2018),("month","==",1)]`. How many rows? How does read time compare to reading all data and filtering in pandas?

**Q8.** The small file problem: your partition by year+month creates how many files? Calculate the average file size. Is this too small? What is the recommended target file size for Parquet?

**Q9.** Load `olist_order_payments_dataset.csv`. Convert to Parquet. Now do a DuckDB query joining orders Parquet + payments Parquet:
```python
con = duckdb.connect()
con.execute("SELECT ... FROM read_parquet('orders.parquet') o JOIN read_parquet('payments.parquet') p ...")
```
Compare query time vs doing the join in pandas.

**Q10.** Simulate schema evolution: add a `currency` column with value `"BRL"` to the orders DataFrame. Write a new Parquet file. Now try to read BOTH old and new files together — does pandas handle the missing `currency` column in old files gracefully?

**Q11.** Build a `convert_csv_to_parquet(input_dir, output_dir, partition_col=None)` function that:
1. Scans input_dir for all CSV files
2. Reads each, cleans column names (lowercase, no spaces)
3. Writes as snappy Parquet (partitioned if partition_col given)
4. Returns a summary dict: `{filename: {"rows": n, "size_ratio": x.x}}`

**Q12.** Implement compression benchmarking: for all Olist CSVs, run `convert_csv_to_parquet` and print a table: `filename | csv_size_KB | parquet_size_KB | ratio | write_ms`. Which dataset compresses most? Least?

**Q13.** Write a `ParquetWriter` class with:
- `write(df, path, partition_cols=None)` — write with snappy
- `append(df, path)` — read existing, concat, rewrite (simulate append)
- `compact(path, max_file_size_mb=100)` — merge small files in a partition

**Q14.** Implement a file registry: after writing each Parquet file, append a row to `data/registry.jsonl` with: `{filename, rows, size_bytes, created_at, schema, partition_cols}`. Write a `read_registry()` function that loads the JSONL and returns a DataFrame.

**Q15.** End-to-end: write a `raw_to_staging_pipeline(data_dir, staging_dir, run_date)` that:
1. Reads all CSVs from `data_dir`
2. Validates each has expected columns (from a hardcoded schema dict)
3. Writes partitioned Parquet to `staging_dir/tablename/year=.../month=.../`
4. Logs a conversion report
5. Is idempotent (running twice produces the same output)

---

## Project 2 🟢 — Extraction Patterns
**Dataset:** D2 — UK Online Retail II | **Focus:** Robust extraction, error handling, incremental

### Business Context
You're building an ingestion layer for the UK Retail dataset. It arrives as a messy Excel file and must be extracted reliably with proper error handling.

**Q1.** Load `online_retail_II.xlsx` (or .csv). Print shape, dtypes, null counts. How many sheets does the Excel file have? Load both sheets and combine.

**Q2.** The `InvoiceDate` column has mixed formats in some versions of this file. Write a robust date parser:
```python
def parse_dates_safely(series: pd.Series) -> pd.Series:
    # Try multiple formats, fall back to coerce
    ...
```
How many dates couldn't be parsed (NaT after coerce)?

**Q3.** Write an `ExtractConfig` dataclass that holds: `source_path`, `date_columns`, `required_columns`, `encoding`, `null_values`. Instantiate one for the UK Retail dataset.

**Q4.** Implement `extract_with_retry(config, max_retries=3)` that: reads the file, catches `FileNotFoundError` and `UnicodeDecodeError` specifically, retries up to 3 times with 1s delay, raises after all retries fail.

**Q5.** Incremental extraction simulation: pretend the file updates daily. Write `extract_incremental(df_full, watermark_date)` that returns only rows with `InvoiceDate > watermark_date`. Verify: what % of rows is "new" for each hypothetical daily run?

**Q6.** Chunk reading: the UK Retail file has 1M+ rows. Read it in chunks of 100k rows using `pd.read_csv(chunksize=100000)`. Compute total revenue across all chunks WITHOUT loading the full file into memory at once.

**Q7.** Write an `extract_all_formats(path)` function that detects the file extension and calls the right reader (`.csv` → `read_csv`, `.xlsx` → `read_excel`, `.parquet` → `read_parquet`, `.json` → `read_json`). The function signature should not change based on file format.

**Q8.** Schema validation on extract: after loading, call `validate_schema(df, expected_schema)`. If schema mismatches are found, write them to `logs/schema_drift.jsonl` (don't crash — just log).

**Q9.** Source data profiling: after extract, automatically compute and save a profile:
```python
profile = {
    "table": "uk_retail",
    "rows": len(df),
    "columns": len(df.columns),
    "null_rates": df.isnull().mean().to_dict(),
    "dtypes": df.dtypes.astype(str).to_dict(),
    "sampled_values": {col: df[col].dropna().head(3).tolist() for col in df.columns},
}
```
Save as `data/profiles/uk_retail_profile.json`.

**Q10.** Extract audit trail: for every extraction, append a row to `data/extract_audit.jsonl`:
```python
{
    "table":           "uk_retail",
    "extracted_at":    "2024-01-15T06:00:00",
    "rows_extracted":  1_067_371,
    "source_path":     "data/uk_retail.xlsx",
    "watermark_used":  "2024-01-14",
    "new_watermark":   "2024-01-15",
}
```

**Q11.** Detect source anomalies on extract: flag and log if:
- Row count drops > 20% vs the previous extract (from audit trail)
- Any "required" column is entirely null
- `InvoiceDate` has future dates (> today)

**Q12.** Write `extract_with_validation(config)` that wraps extraction + schema check + anomaly detection. Returns `(df, extraction_report)` — a dataclass with `rows`, `null_rates`, `anomalies`, `schema_issues`, `is_ok`.

**Q13.** Multi-source extract: the UK Retail data is split across two files: `online_retail_II_2009_2010.xlsx` and `online_retail_II_2010_2011.xlsx`. Write `extract_multi_source(file_list)` that:
1. Extracts each file
2. Tags each row with `source_file`
3. Detects and logs any schema differences between files
4. Concatenates and returns combined DataFrame with `ignore_index=True`

**Q14.** Simulate an API extract: generate mock API pages of UK Retail data:
```python
def mock_api_get(page: int, size: int = 1000) -> dict:
    df_slice = df.iloc[page*size:(page+1)*size]
    return {
        "data": df_slice.to_dict(orient="records"),
        "total": len(df),
        "page": page,
        "has_next": (page+1)*size < len(df),
    }
```
Write `extract_paginated(base_fn, page_size=1000)` that calls `mock_api_get` repeatedly until `has_next=False` and returns the combined DataFrame.

**Q15.** Full extraction layer: write an `ExtractionLayer` class with:
- `register_source(name, config)` — add a source
- `extract_all(run_date)` — extract all registered sources
- `get_status()` — return DataFrame of last extract time, row counts, is_ok per source
- All extracts are logged to `extract_audit.jsonl`
- Failed extracts are logged but don't block successful ones

---

## Project 3 🟢 — Transform: Healthcare Billing Pipeline
**Dataset:** D3 — Healthcare | **Focus:** Business logic transforms, SCD prep

### Business Context
A healthcare analytics team needs a clean, enriched billing table for reporting. You'll apply clinical domain transformations.

**Q1.** Load `healthcare_dataset.csv`. Parse `Date of Admission` and `Discharge Date`. Add `length_of_stay = (Discharge - Admission).days`.

**Q2.** Standardise all string columns: strip whitespace, title case for `Name`, `Doctor`, `Hospital`; uppercase for coded values. How many rows change?

**Q3.** Create `admission_type_code`: `Emergency→E`, `Elective→L`, `Urgent→U`. Create a reverse lookup function.

**Q4.** Create `age_band` using `pd.cut`: `[0,17)→Pediatric`, `[17,40)→Young Adult`, `[40,65)→Adult`, `[65,+)→Senior`.

**Q5.** Create a `billing_tier` column using `pd.qcut` into 4 quartiles: `Budget`, `Standard`, `Premium`, `Luxury`. Verify approximately equal counts per tier.

**Q6.** Anonymise PII: replace `Name` with a hash (use `hashlib.sha256(name.encode()).hexdigest()[:12]`). Why would you do this before loading to a warehouse?

**Q7.** Add a `data_quality_score` per row: 0–100 based on:
- +30 if no null in critical columns (patient_id, admission_date, billing)
- +20 if billing > 0
- +20 if length_of_stay >= 1
- +30 if test_results is in expected values

**Q8.** Detect outlier billing amounts using IQR. Add `is_billing_outlier` flag. How many outliers? Do they cluster in any particular admission type or condition?

**Q9.** SCD Type 2 prep: identify patients who appear multiple times (multiple admissions). Create a `visit_number` column = chronological rank of admissions per patient.

**Q10.** Build a doctor performance table:
```python
doctor_stats = df.groupby("Doctor").agg(
    patients           = ("Name", "nunique"),
    avg_billing        = ("Billing Amount", "mean"),
    avg_los            = ("length_of_stay", "mean"),
    normal_result_rate = ("Test Results", lambda x: (x=="Normal").mean()),
    emergency_rate     = ("Admission Type", lambda x: (x=="Emergency").mean()),
)
```
Add a `performance_tier`: `Gold/Silver/Bronze` based on composite score.

**Q11.** Write `transform_healthcare(df)` as a pure function (no side effects) that applies all Q1–Q10 transforms and returns the clean DataFrame. Test with `df.copy()` to confirm original is unchanged.

**Q12.** Transform audit: wrap `transform_healthcare` so it logs at each step: rows before, rows after, new columns added, nulls introduced. Return `(df_clean, audit_log)`.

**Q13.** Billing category prep: using `pd.pivot_table`, create a wide-format summary: patient_hash as rows, medical_condition as columns, values = sum of billing per condition. This is the format used for ML feature tables.

**Q14.** Temporal validation: add assertions that:
- `admission_date` <= `discharge_date` (no negative stays)
- `admission_date` <= today
- `length_of_stay` <= 365 (no year-long stays without investigation)
Log violations and continue rather than crash.

**Q15.** Build a `HealthcareTransformer` class with methods for each transform stage. Use `__call__` to run the full pipeline in sequence:
```python
transformer = HealthcareTransformer()
df_clean, report = transformer(df_raw)
```
The class should track which transforms were applied and their row/column impact.

---

## Project 4 🟢 — Load Patterns: Fraud Data
**Dataset:** D4 — Banking/Fraud | **Focus:** Load modes, idempotency, DuckDB

### Business Context
You're building the load layer for a fraud analytics warehouse. Data must be loadable reliably, idempotently, and with proper audit trails.

**Q1.** Load the fraud transactions CSV. What is the fraud rate? Print schema and first 5 rows.

**Q2.** Load to DuckDB — full table replace:
```python
con = duckdb.connect("data/fraud_warehouse.ddb")
con.execute("DROP TABLE IF EXISTS transactions")
con.register("_df", df)
con.execute("CREATE TABLE transactions AS SELECT * FROM _df")
```
Verify row count matches. How large is the `.ddb` file vs the original CSV?

**Q3.** Append mode: split the DataFrame into two halves (simulate two daily batches). Load first half. Then append second half. Verify total row count = original.

**Q4.** Idempotent load: implement delete-then-insert for a date partition:
```python
def load_partition(df, date_col, partition_date, table, con):
    con.execute(f"DELETE FROM {table} WHERE DATE({date_col}) = ?", [partition_date])
    # ... then insert
```
Run it twice for the same date. Confirm row count is the same both times.

**Q5.** Upsert: simulate an update where 100 transactions get their `is_fraud` flag corrected. Implement upsert using `INSERT OR REPLACE` (SQLite/DuckDB) semantics. Verify the corrections applied.

**Q6.** Load with schema validation: before loading, check that all required columns exist and have correct types. Raise a `ValueError` if not. Log what was checked.

**Q7.** Partition-aware load: split transactions by `year` and `month`. Load each partition as a separate table `transactions_YYYY_MM`. Then create a `transactions_all` view that `UNION ALL`s all partition tables.

**Q8.** Load performance: compare loading times for:
- `df.to_sql(engine, method=None)` (default, row by row)
- `df.to_sql(engine, method="multi")` (multi-row INSERT)
- DuckDB direct register
Which is fastest? By how much?

**Q9.** Post-load verification: after each load, automatically run:
- Row count check (must match source)
- Null count check for critical columns
- Sum of `amount` check (must match source sum)
Log pass/fail for each.

**Q10.** Write a `LoadResult` dataclass and update `load_partition` to return it:
```python
@dataclass
class LoadResult:
    table:       str
    rows_loaded: int
    rows_deleted: int
    duration_s:  float
    is_ok:       bool
    errors:      list
```

**Q11.** Simulate a failed load: introduce a deliberate type mismatch (e.g., make `amount` a string). The load should fail. Implement a rollback: if load fails, restore the table to its pre-load state. *Hint: save old data to a temp table before deleting.*

**Q12.** Multi-table transactional load: load `transactions`, `customers`, and `cards` tables inside a single DuckDB transaction. If any table fails, roll back ALL three. Verify atomicity by simulating a failure in the middle table.

**Q13.** Incremental load with watermarking: read the max `transaction_date` from the warehouse. Load only transactions newer than that watermark. Store the new watermark in a `pipeline_state` table.

**Q14.** Write `WatermarkManager`:
```python
class WatermarkManager:
    def get_watermark(self, table: str) -> str: ...
    def set_watermark(self, table: str, value: str) -> None: ...
    def get_all(self) -> pd.DataFrame: ...
```
Back it by a `pipeline_state` DuckDB table.

**Q15.** Full load layer: write `LoadLayer` class with:
- `full_refresh(df, table)` — drop and recreate
- `append(df, table)` — append rows
- `upsert(df, table, key_cols)` — insert or update
- `partitioned_load(df, table, partition_col)` — delete partition then insert
- All operations log to `load_audit` table in DuckDB
- All return `LoadResult`

---

## Project 5 🟢 — Simple ETL Pipeline: Supply Chain
**Dataset:** D5 — DataCo Supply Chain | **Focus:** Complete ETL, first full pipeline

### Business Context
Build a complete ETL pipeline for supply chain analytics. This is your first end-to-end pipeline.

**Q1.** Load `DataCoSupplyChainDataset.csv` (note: `encoding="latin-1"`). Print shape and column count. Standardise column names (lowercase, underscores).

**Q2.** Extract phase: write `extract_supply_chain(path)` that loads, standardises column names, and returns `(df, metadata)` where metadata = `{rows, cols, source, extracted_at}`.

**Q3.** Parse the order date column. How many rows have unparseable dates? Handle them with `errors="coerce"`.

**Q4.** Transform phase: write `transform_supply_chain(df)` that:
- Parses dates
- Adds `profit_margin = profit / sales` (handle division by zero)
- Adds `late_flag = 1 if late_delivery_risk == 1 else 0`
- Adds `order_month` as YYYY-MM string
- Drops columns with > 50% nulls

**Q5.** Load phase: write `load_supply_chain(df, db_path)` that loads to DuckDB with `DROP + CREATE` (full replace). Return `LoadResult`.

**Q6.** Wire up extract → transform → load in a `run_pipeline(date_str)` function. Log timing for each step.

**Q7.** Add data quality checks between transform and load:
- No null `order_id`
- `profit_margin` between -1 and 2
- `late_flag` is only 0 or 1
Fail the pipeline if critical checks don't pass.

**Q8.** After loading, run a summary query via DuckDB:
```sql
SELECT department_name, COUNT(*) as orders, SUM(sales) AS revenue,
       AVG(profit_margin) AS avg_margin, AVG(late_flag) AS late_rate
FROM supply_chain GROUP BY 1 ORDER BY 3 DESC
```
Print the result.

**Q9.** Idempotency: the pipeline must produce identical output if run multiple times for the same date. Implement and prove this by running twice and comparing row counts.

**Q10.** Add a `pipeline_log` table to DuckDB that records every run:
```
run_id | run_date | status | rows_extracted | rows_loaded | duration_s | error_msg
```

**Q11.** Parameterise the pipeline: accept `config.json`:
```json
{
  "source_path": "data/supply_chain.csv",
  "db_path":     "data/warehouse/supply.ddb",
  "encoding":    "latin-1",
  "partition_col": "order_month"
}
```
Load config with `json.load()` and use it throughout.

**Q12.** Error recovery: if the transform step raises an exception, the pipeline should:
1. Log the full traceback
2. Write `status=FAILED` to `pipeline_log`
3. NOT corrupt the existing warehouse data
4. Return `success=False`

**Q13.** Add a `--backfill` mode: re-run the pipeline for the last 7 days. Each day's data is a slice of the original CSV filtered by `order_month`. Use a loop over a date list.

**Q14.** Post-pipeline report: after a successful run, generate `outputs/supply_chain_report.md` with:
```markdown
# Supply Chain Pipeline Report — YYYY-MM-DD
## Run Summary
- Status: SUCCESS
- Rows loaded: N
- Duration: Xs
## KPIs
| Department | Revenue | Margin | Late Rate |
...
```

**Q15.** Package the full pipeline:
- `pipeline/config.py` — `PipelineConfig` dataclass
- `pipeline/extract.py` — `extract_supply_chain()`
- `pipeline/transform.py` — `transform_supply_chain()`
- `pipeline/load.py` — `load_supply_chain()`
- `pipeline/quality.py` — quality check functions
- `pipeline/run.py` — `run_pipeline(config)` that ties everything together
- `README.md` — how to run it

---

## Project 6 🟡 — Star Schema Build: Olist
**Dataset:** D1 — Olist (all 8 tables) | **Focus:** Dimensional modelling, full star schema

### Business Context
Design and build a production star schema for Olist analytics. Analysts should be able to answer any business question with a single fact table + dimension joins.

**Q1.** Load all 8 Olist CSV files. Print the grain (what one row represents) for each table. Which tables have a 1:N relationship with `orders`?

**Q2.** Design the star schema on paper (or in code comments):
```
fct_orders (grain: order line item)
  dim_date, dim_customer, dim_product, dim_seller
```
For each dimension, list: natural key, surrogate key, attributes.

**Q3.** Build `dim_date` for 2016–2026 with: `date_id (YYYYMMDD int)`, `full_date`, `year`, `quarter`, `month`, `month_name`, `week`, `day_of_week`, `day_name`, `is_weekend`. How many rows?

**Q4.** Build `dim_customer`: deduplicate on `customer_unique_id`, add `region` (derived from state), add `customer_sk` surrogate key. How many unique customers?

**Q5.** Build `dim_product`: merge with category translation, add English category name. Add `product_sk`. How many distinct products?

**Q6.** Build `dim_seller`: standardise state, add `seller_sk`. How many sellers?

**Q7.** Build `fct_orders` (grain: order_item):
- Join items ← orders ← customers ← products ← sellers
- Aggregate payments to order level
- Add: `item_revenue`, `freight_value`, `total_payment`, `delivery_days`, `is_late`, `avg_review`
- Replace natural keys with surrogate keys
- Add `date_id` from order timestamp

**Q8.** Validate the fact table:
- No null `order_id`
- All `product_sk` values exist in `dim_product`
- All `customer_sk` values exist in `dim_customer`
- `total_payment >= 0`
Log any violations.

**Q9.** Load all dimensions and the fact table to DuckDB. Verify: `SELECT COUNT(*) FROM fct_orders` matches expected count.

**Q10.** Write the SQL for these analytical queries against your star schema:
1. Monthly revenue (use `dim_date.month_name`, `dim_date.year`, `fct_orders.item_revenue`)
2. Top 10 product categories by revenue
3. Late delivery rate by seller state
4. Average review score by customer region

Compare query complexity: star schema SQL vs what it would take against raw CSVs.

**Q11.** Performance test: run query #1 above against (a) DuckDB with Parquet files, (b) DuckDB with your loaded star schema. Which is faster? Why?

**Q12.** Add a `fct_orders_monthly_agg` table — pre-aggregated fact:
```sql
CREATE TABLE fct_orders_monthly_agg AS
SELECT date_id, customer_sk, product_sk,
       COUNT(*) as orders, SUM(item_revenue) as revenue,
       AVG(avg_review) as avg_review, AVG(is_late) as late_rate
FROM fct_orders GROUP BY 1,2,3
```
How much smaller is this vs the full fact table?

**Q13.** Data freshness tracking: add a `pipeline_run` table with `run_id`, `run_ts`, `table_name`, `row_count`, `max_date_loaded`. Query it to see when each table was last refreshed.

**Q14.** Incremental fact loading: rewrite the `fct_orders` build to only process orders newer than the last loaded `order_purchase_timestamp`. Load with delete-partition-then-insert.

**Q15.** Full star schema pipeline: write `build_star_schema(raw_dir, warehouse_path)` that:
1. Extracts all CSVs
2. Builds all 4 dimensions
3. Builds the fact table
4. Loads to DuckDB
5. Runs validation checks
6. Logs everything to `pipeline_runs` table
7. Returns `{success: bool, tables: {name: rows}, duration_s: float}`

---

## Project 7 🟡 — SCD Implementation
**Dataset:** D1 — Olist + D7 — IBM HR | **Focus:** Slowly Changing Dimensions Types 1, 2, 3

### Business Context
Sellers change their performance tier over time. Employees change departments. History matters for accurate reporting.

**Q1.** Load Olist sellers. Build a `dim_seller_v1` with: `seller_id`, `seller_state`, `seller_sk`, initial `tier = "Bronze"` for all, `effective_from = "2016-01-01"`, `effective_to = None`, `is_current = 1`.

**Q2.** Simulate a monthly tier assignment: write `assign_seller_tier(sales_df, month)` that computes:
- Top 10% by revenue → `"Gold"`
- Next 20% → `"Silver"`
- Rest → `"Bronze"`

**Q3.** Apply tier assignment for Jan 2018. How many sellers changed tier? What % upgraded vs downgraded?

**Q4.** SCD Type 1 update: apply the tier change by overwriting in place. Before and after: verify that historical tier is gone.

**Q5.** Rebuild `dim_seller_v2` with SCD Type 2. Apply Jan 2018 tier changes. For changed sellers:
- Expire the old row: `effective_to = "2018-01-31"`, `is_current = 0`
- Add a new row: `effective_from = "2018-02-01"`, `effective_to = None`, `is_current = 1`
How many total rows now vs before?

**Q6.** Apply Feb 2018 tier changes with SCD Type 2. Verify that sellers who changed again now have 3 rows: original → first change → second change.

**Q7.** Historical accuracy check: join `fct_orders` (filtered to Jan 2018 orders) with `dim_seller_v2` using:
```sql
JOIN dim_seller_v2 s ON f.seller_id = s.seller_id
    AND f.order_date BETWEEN s.effective_from AND COALESCE(s.effective_to, '9999-12-31')
```
Do Jan orders use the correct (pre-Feb) tier?

**Q8.** SCD Type 3: rebuild `dim_seller_v3` with `current_tier` and `prev_tier` columns. Apply one tier change and show that prev_tier captures the old value.

**Q9.** Load IBM HR `WA_Fn-UseC_-HR-Employee-Attrition.csv`. Build `dim_employee` with: `employee_id`, `department`, `job_role`, `monthly_income`, `current_attrition`, `effective_from`, `effective_to`, `is_current`.

**Q10.** Simulate an employee getting a promotion: `job_role` changes and `monthly_income` increases. Apply SCD Type 2. Verify that historical reports would show the old job role and salary.

**Q11.** Write a generic `apply_scd2(existing_df, new_snapshot, natural_key, tracked_cols, effective_date)` function. Test it on both sellers and employees.

**Q12.** SCD2 query helper: write `get_current_dimension(df)` that returns only `is_current == 1` rows, and `get_as_of(df, as_of_date)` that returns rows effective on a given date.

**Q13.** Dimension bridge: some analysts want a flat view that shows all seller tier changes:
```python
seller_changes = dim_seller_v2.query("effective_to.notna()").copy()
seller_changes["changed_from"] = seller_changes["tier"]
# ... join next row's tier as changed_to
```
Build this change history view.

**Q14.** SCD monitoring: detect sellers who have changed tier more than 3 times (volatile performers). These might indicate data quality issues or genuinely unstable sellers.

**Q15.** Full SCD pipeline: `run_monthly_scd_update(dim_df, new_data, month_str)` that:
1. Computes new tier assignments
2. Detects changes vs current `is_current=1` rows
3. Applies SCD Type 2 updates
4. Logs: how many new records, how many expired, how many unchanged
5. Validates no duplicates in current records
6. Returns updated dimension DataFrame

---

## Project 8 🟡 — Data Quality Framework
**Dataset:** D4 — Banking/Fraud | **Focus:** GX-style checks, data contracts, automated profiling

### Business Context
The fraud analytics team has been burned by bad data. You're building a reusable quality framework that runs on every pipeline and alerts on violations.

**Q1.** Load the fraud transactions dataset. Write a `profile_dataframe(df, name)` function that returns a dict: `{col: {dtype, null_pct, unique_count, min, max, top5_values}}` for every column. Print it cleanly.

**Q2.** Implement a `QualityRule` dataclass:
```python
@dataclass
class QualityRule:
    name:        str
    column:      str
    check:       Callable[[pd.Series], bool]
    severity:    str = "error"    # "error" | "warning"
    description: str = ""
```
Write 10 rules for the fraud dataset (null checks, range checks, allowed values, uniqueness).

**Q3.** Implement `QualityEngine.run(df, rules)` → returns `QualityReport` with lists of passed, warnings, errors. Print a formatted summary table.

**Q4.** Add a `threshold` parameter to each rule: some checks are OK if < 1% of rows fail (e.g., a small % of missing zip codes is acceptable). Update the engine to handle `allowed_failure_rate`.

**Q5.** Great Expectations style — expectation suite as a config file. Write `rules.yaml`:
```yaml
table: transactions
rules:
  - name: transaction_id_unique
    column: transaction_id
    check: unique
    severity: error
  - name: amount_positive
    column: amount
    check: min_value
    params: {min: 0}
    severity: error
```
Write `load_rules_from_yaml(path)` that parses this into `QualityRule` objects.

**Q6.** Trend detection: run the quality engine on each month of data separately. Track `null_pct` for `merchant_category` over time. Plot the trend. Does null rate increase in certain months?

**Q7.** Cross-table checks: write rules that span two tables:
- Every `customer_id` in `transactions` must exist in `customers` table
- Every `card_id` in `transactions` must exist in `cards` table
Implement `CrossTableRule` that takes two DataFrames.

**Q8.** Anomaly detection as a quality check: flag days where transaction count is > 3 standard deviations from the rolling 7-day mean. Add this as a `QualityRule` on the aggregated daily table.

**Q9.** Quality check timing: add `@monitor_task` timing to each check. Which check takes longest? Optimise the slowest one (hint: vectorised operations are faster than `.apply()`).

**Q10.** Quality report as HTML: write `generate_quality_report(report, output_path)` that saves an HTML file with a colour-coded table (green = passed, yellow = warning, red = error). Use only stdlib (no external HTML libraries).

**Q11.** Data contract validation: define the fraud data contract in a Python dict (as in the tutorial). Write `validate_contract(df, contract)` that checks all schema, type, range, and completeness rules. Return a `ContractValidationResult` with pass/fail per clause.

**Q12.** Quarantine bad rows: instead of failing the whole pipeline when quality checks fail, implement quarantine:
- Rows that fail critical checks → `data/quarantine/transactions_YYYY_MM_DD.parquet`
- Clean rows → proceed to load
- Write a quarantine summary log

**Q13.** Quality gate: wrap the full pipeline with a quality gate:
```python
def quality_gate(df, rules, fail_on_error=True):
    report = QualityEngine().run(df, rules)
    if fail_on_error and report.has_errors:
        raise QualityGateError(f"Quality gate failed: {report.errors}")
    return report
```

**Q14.** Historical quality tracking: save every quality report run to `data/quality_history.jsonl`. Write `quality_trends(table_name, window_days=30)` that loads history and plots null rates and check pass rates over time.

**Q15.** Full quality framework as a package:
- `quality/rules.py` — `QualityRule`, `CrossTableRule`, `load_from_yaml()`
- `quality/engine.py` — `QualityEngine`, `QualityReport`
- `quality/quarantine.py` — `QuarantineWriter`
- `quality/contracts.py` — `DataContract`, `validate_contract()`
- `quality/reporting.py` — `generate_html_report()`, `quality_trends()`
Write a `README.md` for the package explaining how to use it.

---

## Project 9 🟡 — Airflow DAG Build
**Dataset:** D2 — UK Online Retail II | **Focus:** Full Airflow DAG, sensors, XCom

### Business Context
You're productionising the UK Retail pipeline. It needs proper orchestration: schedule, retry, sensor, and alerting.

> **Note:** For this project, write the DAG code as if deploying to Airflow, even if running locally you'll execute the task functions directly. All code must be syntactically correct Airflow DAGs.

**Q1.** Design the DAG on paper first. Draw the task graph:
```
sense_source_file → extract → validate → transform → load → dq_check → notify
```
What dependencies exist? Which tasks could theoretically run in parallel?

**Q2.** Write the full Airflow DAG file `dag_uk_retail_daily.py` with:
- `schedule_interval="0 7 * * 1-5"` (weekdays at 7am)
- `catchup=False`
- `max_active_runs=1`
- `default_args` with 2 retries, 5-minute retry delay

**Q3.** Write the `extract` task as a `PythonOperator`. It should:
- Accept `execution_date` from context
- Load UK Retail data
- Push the output path to XCom
- Log row count and file size

**Q4.** Write the `validate` task. Pull the raw path from XCom. Run at least 5 quality checks. Push `validation_passed: bool` to XCom. If validation fails, raise `AirflowException`.

**Q5.** Write the `transform` task. Pull raw path from XCom. Apply these transforms:
- Parse `InvoiceDate`
- Add `line_total = Quantity * Price`
- Filter: `Quantity > 0`, `Price > 0`
- Add `year_month`
Push clean path to XCom.

**Q6.** Write the `load` task. Pull clean path from XCom. Load to DuckDB. Push `rows_loaded` to XCom.

**Q7.** Write the `dq_check` task. Pull `rows_loaded` from XCom. Query DuckDB to verify the data landed correctly. Check: row count matches, sum of `line_total` matches.

**Q8.** Add a `FileSensor` before `extract` that waits for the source file to exist. Set `poke_interval=300` (5 min), `timeout=7200` (2hr), `mode="reschedule"`.

**Q9.** Add a `BranchPythonOperator` after `validate`: if validation passes → `transform` → `load`. If validation fails → `notify_failure` (a separate email task that doesn't block the pipeline from ending gracefully).

**Q10.** Write the `notify_success` and `notify_failure` tasks using `EmailOperator`. The success email should include rows loaded and duration. The failure email should include which quality check failed.

**Q11.** Add SLA monitoring to the DAG:
```python
sla_miss_callback=lambda dag, task_list, blocking_task_list, slas, blocking_tis: \
    send_slack_alert(f"SLA MISS: {[s.task_id for s in slas]}")
```
Set SLA = 30 minutes per task.

**Q12.** Write a test file `tests/test_dag_uk_retail.py` that:
- Imports the DAG
- Verifies it has exactly 8 tasks
- Verifies the task order is correct
- Verifies `catchup=False`
These are "DAG integrity tests" — fast, no actual data needed.

**Q13.** Write an `XComHelper` class that wraps `context["ti"].xcom_push/pull` with type safety:
```python
class XComHelper:
    def push(self, ti, key, value): ...
    def pull(self, ti, task_id, key): ...
    def pull_required(self, ti, task_id, key):  # raises if None
        ...
```

**Q14.** Backfill simulation: write `run_dag_for_dates(dag_fn, start_date, end_date)` that calls each task function directly (no Airflow server needed) for each date in the range. Use a mock context `{"ds": date_str, "ti": MockTaskInstance()}`.

**Q15.** Package the full Airflow DAG project:
```
dags/
  dag_uk_retail_daily.py
  dag_uk_retail_weekly.py     ← weekly summary DAG
pipeline/
  extract.py
  transform.py
  load.py
  quality.py
tests/
  test_dag_uk_retail.py
  test_transforms.py
README.md                      ← setup instructions
```

---

## Project 10 🟡 — Incremental Pipeline
**Dataset:** D1 — Olist | **Focus:** Watermarking, backfill, incremental patterns

### Business Context
The Olist pipeline currently does a full reload every day. With 100k+ orders and growing, you need to switch to incremental loading — only process what changed.

**Q1.** Understand the current data: what is the date range in `order_purchase_timestamp`? How many orders per month? Simulate that each month's data "arrives" on the last day of that month.

**Q2.** Write a `WatermarkStore` backed by a JSON file:
```python
class WatermarkStore:
    def __init__(self, path: str): ...
    def get(self, pipeline: str) -> str | None: ...
    def set(self, pipeline: str, value: str) -> None: ...
```
On first run, watermark is `None` → do a full load.

**Q3.** Implement `extract_incremental(df_full, watermark)`:
- If `watermark is None`: return all rows
- Else: return rows where `order_purchase_timestamp > watermark`
Print how many rows are "new" for each monthly increment.

**Q4.** Incremental transform: your transform function must work on any subset of data, not just the full dataset. Verify `transform_orders(df_new)` works correctly on a single month's data.

**Q5.** Incremental load: implement `append_to_warehouse(df_new, table, db_path)` that:
1. Checks for overlapping `order_id` values (idempotency guard)
2. Deletes overlapping rows from warehouse before inserting
3. Inserts new rows
4. Updates the watermark

**Q6.** Simulate 12 monthly runs: loop from 2017-01 to 2017-12, extracting only that month's data and appending. After all 12 runs, verify total warehouse row count = original full-year row count.

**Q7.** Backfill mode: write `run_backfill(start_date, end_date, batch_size_days=30)` that:
- Splits the date range into batches
- Runs the incremental pipeline for each batch
- Logs progress
- Is resumable (if interrupted, continue from last successful batch)

**Q8.** Late-arriving data problem: some orders arrive with timestamps from 3 days ago. Add a `lookback_days=3` parameter: `extract_incremental` always re-processes the last 3 days to catch late arrivals. Make the load idempotent to handle re-processing.

**Q9.** Watermark drift detection: if the new watermark is more than 7 days ahead of the old one, log a warning — this suggests the pipeline missed some days.

**Q10.** Add a `pipeline_runs` table to DuckDB:
```
run_id | run_date | mode (full/incremental) | rows_in | rows_out | watermark_before | watermark_after | duration_s | status
```
Every run appends one row.

**Q11.** Catchup: if the pipeline didn't run for 5 days, it needs to process all 5 days in one run. Write `determine_runs_needed(last_watermark, today)` that returns a list of date ranges to process.

**Q12.** Schema migration handling: when a new column is added to the source, the incremental load must not break. Write `safe_append(df_new, table, con)` that:
- Checks if `df_new` has columns not in the warehouse table
- If yes: `ALTER TABLE` to add the new columns
- Then proceeds with the append

**Q13.** Incremental RFM: instead of recomputing all customer RFM scores daily, compute only for customers who had a transaction in the last 30 days. Merge with existing RFM scores. Show the compute time saving.

**Q14.** Parallel incremental: if you have 4 months to backfill, process them in parallel using `concurrent.futures.ThreadPoolExecutor`. Add proper locking so the watermark isn't updated incorrectly.

**Q15.** Production-ready incremental pipeline:
```python
def run_incremental_pipeline(
    config: PipelineConfig,
    force_full: bool = False,
    backfill_from: str = None,
) -> PipelineResult:
    """
    - force_full=True: ignore watermark, reload everything
    - backfill_from: re-process from a given date
    - Otherwise: standard incremental from last watermark
    Returns PipelineResult with run metadata.
    """
```

---

## Project 11 🔴 — dbt Project: Olist Semantic Layer
**Dataset:** D1 — Olist | **Focus:** dbt models, tests, docs, incremental materialisation

### Business Context
You're building the dbt project that powers Olist's analytics warehouse. Analysts should be able to query clean, tested, documented tables — never touching raw data.

> **Setup:** Install dbt-duckdb: `pip install dbt-core dbt-duckdb`. Initialise: `dbt init olist_dbt`. Load raw CSVs into a DuckDB database as `raw_*` tables first.

**Q1.** Set up `profiles.yml` to connect to your DuckDB warehouse file. Verify `dbt debug` passes.

**Q2.** Create `sources.yml` declaring all 8 Olist tables as sources:
```yaml
sources:
  - name: olist
    schema: main
    tables:
      - name: raw_orders
        loaded_at_field: order_purchase_timestamp
        freshness:
          warn_after: {count: 1, period: day}
          error_after: {count: 3, period: day}
```

**Q3.** Write 4 staging models (views) in `models/staging/`:
- `stg_orders.sql` — clean + rename orders columns
- `stg_customers.sql` — deduplicate, add region
- `stg_payments.sql` — aggregate to order level
- `stg_reviews.sql` — aggregate to order level

**Q4.** Add `schema.yml` for the staging models with `unique` and `not_null` tests on primary keys. Run `dbt test --select staging`. Fix any failures.

**Q5.** Write `models/marts/dim_customer.sql` — customer dimension with surrogate key, state, region. Materialise as `table`.

**Q6.** Write `models/marts/fct_orders.sql` — orders fact. Materialise as `incremental` with `unique_key="order_id"`.
```sql
{% if is_incremental() %}
WHERE ordered_at > (SELECT MAX(ordered_at) FROM {{ this }})
{% endif %}
```

**Q7.** Write `models/marts/agg_monthly_revenue.sql` — pre-aggregated monthly KPI table:
```sql
SELECT
    DATE_TRUNC('month', ordered_at)  AS month,
    COUNT(DISTINCT order_id)          AS orders,
    COUNT(DISTINCT customer_id)       AS unique_customers,
    SUM(total_payment)                AS revenue,
    AVG(avg_review)                   AS avg_review,
    AVG(is_late)                      AS late_rate
FROM {{ ref('fct_orders') }}
GROUP BY 1
```

**Q8.** Add a custom `dbt` test: `test_no_negative_revenue`. Write the test SQL in `tests/` and apply it to `fct_orders.total_payment`.

**Q9.** Write a `dbt` macro `cents_to_dollars(column)` and a `safe_divide(numerator, denominator)` macro. Use them in your models.

**Q10.** Add model documentation: write `description:` fields in `schema.yml` for every model and every column in `fct_orders`. Run `dbt docs generate && dbt docs serve` — screenshot (or describe) what the lineage graph shows.

**Q11.** Add `tags` to models: `staging` tag for staging models, `daily` tag for incremental models, `weekly` for aggregation models. Run `dbt run --select tag:staging`.

**Q12.** Write a `dbt` seed: create `seeds/seller_tiers.csv` with manual tier assignments for top 10 sellers. Reference it in a model: `{{ ref('seller_tiers') }}`.

**Q13.** Exposure: define a `exposures.yml` for a "Monthly Revenue Dashboard" that consumes `agg_monthly_revenue`. This documents who uses your data.

**Q14.** Run `dbt build` (run + test + snapshot) on the full project. How many models ran? How many tests passed? How long did it take?

**Q15.** Write `dbt_project.yml` with:
- Model-level configs (staging = view, marts = table, incremental = incremental)
- Variables for the project: `run_date`, `lookback_days`
- On-run-start hook: log to a `dbt_run_log` table
- On-run-end hook: update `dbt_run_summary` with pass/fail counts

---

## Project 12 🔴 — Streaming Simulation
**Dataset:** D4 — Banking/Fraud | **Focus:** Micro-batch processing, event simulation, real-time patterns

### Business Context
The fraud team needs near-real-time detection. You'll simulate a Kafka-style streaming pipeline using micro-batches.

**Q1.** Sort fraud transactions by `transaction_timestamp`. This is your "event stream" — events in chronological order.

**Q2.** Write an `EventStream` class that simulates streaming:
```python
class EventStream:
    def __init__(self, df: pd.DataFrame, timestamp_col: str):
        self.data = df.sort_values(timestamp_col)
        self.pointer = 0

    def get_batch(self, n: int = 100) -> pd.DataFrame:
        """Return next n events from the stream."""
        batch = self.data.iloc[self.pointer:self.pointer + n]
        self.pointer += n
        return batch

    def has_more(self) -> bool:
        return self.pointer < len(self.data)
```

**Q3.** Write a `process_batch(batch_df, state)` function that:
- Detects fraud in the current batch using a simple rule: `amount > 3 × customer_avg_amount`
- Maintains `state` (a dict of `customer_id → running avg amount`) across batches
- Returns `(processed_df, updated_state, fraud_count)`

**Q4.** Windowed aggregation: for each batch, compute a 5-minute rolling window of transaction counts per customer. Flag customers with > 3 transactions in 5 minutes. *Hint: use the `transaction_timestamp` to determine the window boundary.*

**Q5.** Tumbling window: aggregate all transactions in each 10-minute window:
```
[00:00-00:10) → count, sum, fraud_count
[00:10-00:20) → count, sum, fraud_count
...
```
Visualise: plot fraud rate per 10-minute window over 24 hours.

**Q6.** Micro-batch pipeline: simulate a pipeline that processes a new batch every 30 seconds:
```python
while stream.has_more():
    batch = stream.get_batch(100)
    results = process_batch(batch, state)
    load_to_duckdb(results, "streaming_results")
    time.sleep(0.001)   # simulate delay
```
After processing all batches, verify total rows in DuckDB = original dataset.

**Q7.** State management: your `state` dict grows over time. Implement `state_checkpoint(state, path)` that saves the state to JSON every 1000 events, so the pipeline can resume after a crash.

**Q8.** Late-arriving events: some transactions arrive with timestamps 2 minutes in the past. Add a watermark: accept events up to 2 minutes late. Discard events older than 2 minutes (log them to a late-events file).

**Q9.** Real-time alerting simulation: when fraud is detected in a batch, call `alert(transaction)`:
```python
def alert(txn):
    print(f"🚨 FRAUD ALERT: customer={txn['customer_id']}, "
          f"amount=${txn['amount']:.2f}, "
          f"merchant={txn['merchant_category']}, "
          f"time={txn['timestamp']}")
```
Log all alerts to `data/alerts.jsonl`.

**Q10.** Throughput measurement: process all batches and compute:
- Total events processed
- Total processing time
- Events per second
- Average batch processing time
- Max batch processing time (the bottleneck)

**Q11.** Dead letter queue: events that fail processing (e.g., missing `customer_id`, null `amount`) should go to a dead-letter queue (`data/dlq.jsonl`) rather than crashing the pipeline. Simulate 1% bad events.

**Q12.** Simulated Kafka topics: implement:
```python
class MockKafkaTopic:
    def __init__(self, name: str): ...
    def produce(self, event: dict): ...
    def consume(self, n: int) -> list: ...
```
Use this to decouple your event generator from your event processor.

**Q13.** Multi-topic: create two topics: `transactions` and `fraud_alerts`. The processor reads from `transactions`, and writes detected fraud to `fraud_alerts`. Write a separate consumer that reads `fraud_alerts` and generates a report.

**Q14.** Exactly-once simulation: implement a processed-event log (`data/processed_ids.jsonl`). Before processing each batch, check which events were already processed and skip them. This prevents double-counting on pipeline restart.

**Q15.** Full streaming pipeline:
```python
def run_streaming_pipeline(
    source_df:     pd.DataFrame,
    batch_size:    int = 100,
    checkpoint_every: int = 1000,
    output_db:     str = "data/streaming.ddb",
) -> StreamingResult:
```
Returns `StreamingResult` with: batches_processed, total_events, fraud_detected, alerts_sent, processing_time_s, throughput_eps.

---

## Project 13 🔴 — Multi-Source Pipeline
**Dataset:** D1 + D3 + D6 | **Focus:** Cross-domain integration, unified schema, master pipeline

### Business Context
A consulting firm has three clients: Olist (e-commerce), a hospital (healthcare), and an airline. You're building a unified analytics pipeline that processes all three and outputs comparable KPIs.

**Q1.** Define a `UnifiedKPI` schema that works across all three domains:
```python
@dataclass
class UnifiedKPI:
    domain:           str   # "ecommerce" | "healthcare" | "airline"
    date:             str
    volume:           int   # orders / admissions / flights
    revenue:          float # payment / billing / N/A
    quality_score:    float # review / test_normal_rate / satisfaction
    operational_kpi:  float # late_delivery_rate / avg_LOS / avg_delay_min
    run_date:         str
```

**Q2.** Write `extract_ecommerce()`, `extract_healthcare()`, `extract_airline()` — each returns raw DataFrames with a `source` column tagging the domain.

**Q3.** Write `transform_to_unified(df, domain)` that converts each domain's DataFrame into the `UnifiedKPI` schema. Each function handles its own column mapping.

**Q4.** Monthly aggregation: for each domain, compute monthly KPIs and output a `unified_monthly_kpis` table with one row per (domain, month).

**Q5.** Cross-domain comparison: after loading all three domains, write a DuckDB query that ranks domains by their `quality_score` each month. Does airline quality correlate with e-commerce quality over time?

**Q6.** Schema registry: maintain a `schema_registry.json` that describes each domain's raw schema and its mapping to the unified schema. Write `load_schema(domain)` and `validate_against_registry(df, domain)`.

**Q7.** Error isolation: if one domain's pipeline fails, the others should still complete. Implement with `try/except` per domain and a summary report at the end.

**Q8.** Write `UnifiedPipelineOrchestrator`:
```python
class UnifiedPipelineOrchestrator:
    def register(self, domain: str, extract_fn, transform_fn): ...
    def run_all(self, run_date: str) -> dict: ...  # {domain: PipelineResult}
    def run_one(self, domain: str, run_date: str) -> PipelineResult: ...
```

**Q9.** Reconciliation: after loading all three domains, write a reconciliation query:
```sql
SELECT domain, COUNT(*) as months_loaded, MIN(date) as earliest, MAX(date) as latest
FROM unified_monthly_kpis GROUP BY 1
```
Are all domains covering the same date range? Flag gaps.

**Q10.** Unified KPI dashboard query: write a DuckDB query that produces a report showing all three domains side by side for the most recent 3 months.

**Q11.** Cross-domain anomaly detection: flag any month where a domain's `quality_score` drops > 15% from the prior month. These are "quality incidents" requiring investigation.

**Q12.** Write `generate_unified_report(db_path, output_path)` that queries the unified table and outputs a markdown report with sections per domain and a cross-domain comparison table.

**Q13.** Data freshness check: before running the unified comparison, verify each domain's data is current (latest date within last 7 days). If any domain is stale, flag it in the report.

**Q14.** Metadata store: add a `domain_metadata` table:
```
domain | last_run | rows_loaded | earliest_date | latest_date | quality_checks_passed | status
```
Update it after each domain run.

**Q15.** Full multi-domain pipeline `run_unified_pipeline(config, run_date)`:
1. Extract all three domains in parallel
2. Validate each against schema registry
3. Transform to unified schema
4. Load to `unified_monthly_kpis`
5. Run cross-domain reconciliation
6. Generate markdown report
7. Update metadata store
8. Return `{domain: PipelineResult}` dict

---

## Project 14 🔴 — Data Lakehouse Pattern
**Dataset:** D5 — DataCo Supply Chain | **Focus:** Delta-style partitioned lake, compaction, time travel simulation

### Business Context
You're migrating the supply chain pipeline from a flat Parquet setup to a lakehouse-style architecture with versioning, compaction, and audit trails.

**Q1.** Create a `Lakehouse` class backed by a directory of Parquet files:
```python
class Lakehouse:
    def __init__(self, base_path: str):
        self.path = Path(base_path)
        self.log_path = self.path / "_delta_log"
        self.log_path.mkdir(parents=True, exist_ok=True)
```

**Q2.** Implement `Lakehouse.write(df, mode="append")`:
- Generates a new part file: `part-00000001.parquet`
- Logs the write to `_delta_log/00000001.json`:
```json
{"version": 1, "timestamp": "...", "operation": "WRITE",
 "rows_added": 100, "file": "part-00000001.parquet"}
```

**Q3.** Implement `Lakehouse.read(as_of_version=None)`:
- If `as_of_version=None`: read all current part files
- If `as_of_version=N`: read only files present at version N (time travel!)

**Q4.** Implement `Lakehouse.delete(filter_condition)`:
- Identify rows matching condition
- Write a new file with matching rows removed
- Log the delete operation
- Do NOT physically delete old files (like Delta Lake — write-only log)

**Q5.** Load the supply chain CSV in monthly batches (12 months). Each batch = one `Lakehouse.write()` call. After all 12 writes, verify total row count.

**Q6.** Time travel: using `read(as_of_version=3)`, verify you can see the state after the 3rd write. Row count should be 3 months' worth.

**Q7.** Implement `Lakehouse.get_history()` — returns a DataFrame of all log entries:
```
version | timestamp | operation | rows_added | rows_removed | file
```

**Q8.** Compaction: after 12 writes you have 12 small files. Implement `Lakehouse.compact()`:
- Read all current data
- Write as one large file
- Log as `COMPACT` operation
- Verify row count unchanged

**Q9.** Partitioned lakehouse: upgrade `Lakehouse.write()` to support `partition_by=["year","month"]`. Files are written to `year=2023/month=01/part-00000001.parquet`. Partition-aware reads skip irrelevant directories.

**Q10.** Schema evolution: add a `discount_pct` column to new batches. Implement `Lakehouse._merge_schemas()` that handles reading old files (missing column → fill with NULL) and new files together.

**Q11.** Vacuum: in Delta Lake, old files are physically deleted after retention period. Implement `Lakehouse.vacuum(retain_hours=168)`:
- Find files not referenced in the last N log entries AND older than retain_hours
- Delete them
- Log the vacuum operation

**Q12.** ACID simulation: wrap `write` in a "transaction":
```python
def write_atomic(self, df, mode):
    temp_file = self._write_temp(df)   # write to temp first
    self._commit(temp_file, mode)      # only then log the commit
    # If crash between write and commit → temp file is ignored on next read
```

**Q13.** Lakehouse analytics: load the full supply chain into the lakehouse. Then write a DuckDB query that reads directly from the partitioned Parquet files to answer: "Monthly profit margin by department, last 6 months."

**Q14.** Migrate existing pipeline: update `Project 5`'s supply chain pipeline to use `Lakehouse` instead of direct Parquet writes. All pipeline code should be unchanged except the load function.

**Q15.** Full lakehouse layer:
- `lakehouse/core.py` — `Lakehouse` class
- `lakehouse/log.py` — `DeltaLog` (log reader/writer)
- `lakehouse/reader.py` — partition-aware reader with filter pushdown
- `lakehouse/vacuum.py` — `vacuum()`
- `tests/test_lakehouse.py` — unit tests for each operation
- `README.md` — with diagram of the file structure

---

## Project 15 🔴 — Full DE Capstone
**Datasets:** D1 + D4 + D5 | **Focus:** End-to-end production pipeline with all DE concepts

### Build a production-grade, multi-source data engineering system.

**Q1.** Architecture diagram: draw (in ASCII or comments) the complete system:
```
Sources → Extract Layer → Staging Zone → Transform Layer → Warehouse → Serve Layer
                ↓               ↓               ↓               ↓
           Audit Log      Quality Check    Lineage Track    Pipeline Log
```

**Q2.** Implement `SourceRegistry` — tracks all data sources:
```python
@dataclass
class DataSource:
    name:         str
    path:         str
    format:       str    # csv | parquet | json
    grain:        str    # "one row = one order"
    frequency:    str    # daily | monthly
    schema:       dict
    extract_fn:   Callable
```

**Q3.** Implement `StagingZone`:
- Writes raw extracts to `staging/<source>/<date>/raw.parquet`
- Validates against schema registry
- Never overwrites — each run writes to a new timestamped directory

**Q4.** Implement `TransformRegistry` — maps source tables to transform functions. Use dependency injection so transforms can be swapped without changing orchestration code.

**Q5.** Implement `WarehouseLayer` with DuckDB:
- `load(df, table, mode)` — append | replace | upsert | partition
- `query(sql)` → DataFrame
- `table_exists(name)` → bool
- `get_stats(table)` → {rows, size_mb, last_updated}

**Q6.** Implement `LineageTracker`:
```python
@dataclass
class LineageEntry:
    source:         str
    target:         str
    transform:      str
    run_id:         str
    rows_in:        int
    rows_out:       int
    timestamp:      str
    duration_s:     float
```
Save all entries to `warehouse/lineage_log` table.

**Q7.** Implement `PipelineOrchestrator`:
```python
class PipelineOrchestrator:
    def __init__(self, source_registry, transform_registry, warehouse, quality_engine, lineage):
        ...

    def run(self, sources: list, run_date: str) -> dict:
        # For each source:
        # 1. Extract → staging
        # 2. Quality check
        # 3. Transform
        # 4. Load to warehouse
        # 5. Track lineage
        ...
```

**Q8.** Add retry logic to the orchestrator: if a source fails, retry up to 3 times with exponential backoff. After 3 failures, mark that source as `FAILED` and continue with others.

**Q9.** Implement `PipelineMonitor`:
- `record_run(result)` — saves to `pipeline_runs` table
- `get_dashboard()` — returns summary DataFrame: source, last_run_status, last_run_date, rows_loaded, avg_duration_s
- `detect_anomalies()` — flags runs where rows_loaded < 50% of rolling average

**Q10.** Implement `ReportGenerator`:
```python
def generate_executive_report(warehouse, run_date) -> str:
    """Generate a markdown report with KPIs from all three domains."""
```
The report should include: Olist monthly revenue trend, fraud summary, supply chain scorecard.

**Q11.** Write `main.py`:
```python
if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("--run-date", default=str(date.today() - timedelta(1)))
    parser.add_argument("--sources",  nargs="+", default=["olist","fraud","supply"])
    parser.add_argument("--config",   default="config.json")
    parser.add_argument("--report",   action="store_true")
    args = parser.parse_args()

    config   = load_config(args.config)
    pipeline = build_pipeline(config)
    result   = pipeline.run(args.sources, args.run_date)
    if args.report:
        generate_executive_report(pipeline.warehouse, args.run_date)
```

**Q12.** Write unit tests (`pytest`) for:
- `WatermarkManager.get/set`
- `QualityEngine.run` with a known-bad DataFrame
- `LineageTracker.log_transform`
- `transform_orders` (from Project 3 patterns)
All tests must pass with `pytest tests/ -v`.

**Q13.** Write integration test `tests/test_full_pipeline.py`:
1. Copy 3 months of Olist data to a temp directory
2. Run `pipeline.run(["olist"], run_date="2018-03-01")`
3. Query warehouse: assert row count > 0
4. Query lineage log: assert 1 entry per transform step
5. Query pipeline_runs: assert status="SUCCESS"

**Q14.** Performance profiling: add `cProfile` instrumentation to the orchestrator. Run the full pipeline and print the top 10 slowest function calls. Identify the bottleneck and optimise it.

**Q15.** Final package structure — your complete DE portfolio project:
```
de_capstone/
├── config.json                 ← all configurable parameters
├── main.py                     ← CLI entrypoint
├── pipeline/
│   ├── __init__.py
│   ├── orchestrator.py         ← PipelineOrchestrator
│   ├── registry.py             ← SourceRegistry, TransformRegistry
│   ├── sources/
│   │   ├── olist.py            ← extract_olist()
│   │   ├── fraud.py            ← extract_fraud()
│   │   └── supply_chain.py     ← extract_supply()
│   ├── transforms/
│   │   ├── olist.py            ← transform_orders(), compute_rfm()
│   │   ├── fraud.py            ← transform_fraud()
│   │   └── supply_chain.py     ← transform_supply()
│   ├── warehouse.py            ← WarehouseLayer (DuckDB)
│   ├── staging.py              ← StagingZone
│   ├── quality.py              ← QualityEngine
│   ├── lineage.py              ← LineageTracker
│   ├── monitoring.py           ← PipelineMonitor
│   └── reporting.py            ← ReportGenerator
├── lakehouse/
│   └── core.py                 ← Lakehouse (from Project 14)
├── tests/
│   ├── test_transforms.py
│   ├── test_quality.py
│   ├── test_pipeline.py
│   └── test_full_pipeline.py
└── README.md                   ← setup, usage, architecture diagram
```

---


---

## Project 16 🔴 — Kafka Event Pipeline
**Dataset:** D4 — Banking/Fraud | **Focus:** Kafka producers, consumers, Schema Registry, CDC simulation

### Business Context
The fraud team is moving from daily batch detection to real-time alerting. Every transaction must be evaluated within seconds of being placed. You'll build the event-driven pipeline that makes this possible.

### Setup
```python
# pip install confluent-kafka fastavro duckdb pandas

# We'll simulate Kafka using an in-process queue so no Kafka server is needed.
# All code is structured to be Kafka-compatible — swap MockKafka for a real
# Confluent/Kafka client and it runs unchanged.

import json
import time
import threading
import queue
from dataclasses import dataclass, field
from typing import Callable, Optional
import pandas as pd

class MockKafkaBroker:
    """
    In-process Kafka simulation. Supports multiple topics,
    consumer groups with offset tracking, and basic partition logic.
    """
    def __init__(self):
        self._topics:  dict[str, list] = {}
        self._offsets: dict[str, dict[str, int]] = {}  # group → topic → offset

    def create_topic(self, name: str) -> None:
        if name not in self._topics:
            self._topics[name] = []

    def produce(self, topic: str, key: str, value: dict) -> int:
        if topic not in self._topics:
            self.create_topic(topic)
        msg = {"key": key, "value": value, "offset": len(self._topics[topic]),
               "timestamp": time.time()}
        self._topics[topic].append(msg)
        return msg["offset"]

    def consume(self, topic: str, group: str, max_messages: int = 10) -> list:
        if topic not in self._topics:
            return []
        if group not in self._offsets:
            self._offsets[group] = {}
        current = self._offsets[group].get(topic, 0)
        messages = self._topics[topic][current:current + max_messages]
        self._offsets[group][topic] = current + len(messages)
        return messages

    def lag(self, topic: str, group: str) -> int:
        current = self._offsets.get(group, {}).get(topic, 0)
        return len(self._topics.get(topic, [])) - current

broker = MockKafkaBroker()
```

---

### Questions

**Q1.** Load the fraud transactions dataset. Sort by transaction timestamp. Write a `TransactionProducer` class with a `produce(transaction: dict) -> int` method that:
- Serialises the transaction as JSON
- Writes to the `"transactions"` topic using `broker.produce()`
- Uses `customer_id` as the Kafka key (same customer → same partition)
- Returns the offset

Produce the first 1,000 transactions. Print: total produced, first offset, last offset.

**Q2.** Write a `TransactionConsumer` class with a `consume_batch(max: int) -> list` method that reads from the `"transactions"` topic. Implement `get_lag() -> int`. Consume the first 100 messages and verify: messages arrive in the same order they were produced (offsets 0–99).

**Q3.** Schema definition. Define an Avro schema for a transaction event:
```python
TRANSACTION_SCHEMA = {
    "type": "record",
    "name": "Transaction",
    "fields": [
        {"name": "transaction_id", "type": "string"},
        {"name": "customer_id",    "type": "string"},
        {"name": "amount",         "type": "double"},
        {"name": "merchant_category", "type": "string"},
        {"name": "is_fraud",       "type": "int"},
        {"name": "timestamp",      "type": "long"},
    ]
}
```
Write `serialize(event: dict) -> bytes` using fastavro and `deserialize(data: bytes) -> dict`. Verify round-trip: `deserialize(serialize(event)) == event`.

**Q4.** Schema evolution. Add a new optional field `"card_type"` with default `None` to the schema (v2). Write a message using v2. Read it back using the v1 schema. Does it crash? What value does `card_type` have when read by a v1 consumer? This is backward compatibility in action.

**Q5.** Real-time fraud detection consumer. Write `FraudDetectionConsumer` that:
- Consumes from `"transactions"` in batches of 50
- Flags any transaction where `amount > 3 × customer_avg_amount` as suspicious
- Maintains a running dict `{customer_id: rolling_avg}` across batches (stateful processing)
- Produces flagged transactions to a `"fraud.alerts"` topic

Run it over all 1,000 produced transactions. How many alerts were generated?

**Q6.** Consumer groups and parallelism. The fraud detection service and the analytics service both need to read all transactions independently. Create two consumer groups: `"fraud-detection"` and `"analytics"`. Consume 100 messages with each. Verify: both groups get the same messages (both read from offset 0), and each group's offset is tracked independently.

**Q7.** Dead letter queue. Transactions with `amount <= 0` or missing `customer_id` are malformed. Modify your consumer to:
- Route malformed messages to a `"transactions.dlq"` (dead letter queue) topic instead of processing them
- Log: `{offset, reason, raw_value}`
- Continue processing valid messages without interruption

Inject 10 malformed transactions into the stream. Verify they land in the DLQ and don't appear in fraud detection output.

**Q8.** Exactly-once simulation. Your consumer crashes mid-batch. To avoid double-processing:
- Only commit the offset after successfully processing AND writing results to DuckDB
- On restart, the consumer reads from the last committed offset (no events skipped, none double-processed)

Implement this by: committing offset to a `pipeline_state` DuckDB table, not in-memory. Simulate a crash by raising an exception mid-batch. Restart and verify: no duplicates in the output table.

**Q9.** Windowed aggregation. Every 100 messages (a "window"), compute:
- Window number
- Message count
- Total transaction amount
- Fraud rate (% flagged)
- Top 3 merchant categories by volume

Write results to a `"window.stats"` topic AND to a DuckDB `window_aggregates` table. After processing all 1,000 transactions, query the table and print the fraud rate trend across windows.

**Q10.** CDC simulation — Change Data Capture. The fraud team also needs to know when customer records change (new phone number, new address — classic fraud pattern). Simulate CDC by:
1. Loading the customers table as the "baseline"
2. Creating 50 "updated" customer records with changed `city`
3. Producing each change as a CDC event to `"customers.cdc"` topic:
```python
{"op": "UPDATE", "before": {...old record...}, "after": {...new record...}, "ts": ...}
```
4. Writing a CDC consumer that applies changes to a `dim_customer_live` DuckDB table

**Q11.** Kafka Connect simulation. Write a `JdbcSourceConnector` class that:
- Reads rows from a DuckDB table where `updated_at > last_poll_watermark`
- Produces each row as a Kafka message
- Updates the watermark after successful produce
- Runs in a loop (simulating continuous polling)

Use the supply chain dataset: simulate new orders arriving every 100ms.

**Q12.** Multi-topic fan-out. One order event should trigger three downstream consumers:
1. `InventoryConsumer` — decrements stock levels
2. `NotificationConsumer` — simulates sending an email
3. `AnalyticsConsumer` — writes to DuckDB

Implement all three reading from the same `"orders"` topic with different consumer group IDs. Run them concurrently using `threading.Thread`. Verify: all three processed the same messages independently.

**Q13.** Backpressure simulation. Your analytics consumer is slow (add `time.sleep(0.01)` per message). The producer is fast. Implement lag monitoring:
- Every 10 seconds, print consumer lag for each group
- If lag exceeds 500 messages, log a `BACKPRESSURE_WARNING`
- Implement a "pause producer" mechanism when lag is critical (> 1,000 messages)

**Q14.** Schema Registry implementation. Build a `MockSchemaRegistry`:
```python
class MockSchemaRegistry:
    def register(self, subject: str, schema: dict) -> int: ...  # returns schema_id
    def get_schema(self, schema_id: int) -> dict: ...
    def get_latest(self, subject: str) -> tuple[int, dict]: ...
    def check_compatibility(self, subject: str, new_schema: dict) -> bool: ...
        # BACKWARD: new schema can read data written by old schema
        # Check: all fields in old schema exist in new (or have defaults)
```
Register the transaction schema v1, then v2 (with `card_type`). Verify `check_compatibility` returns `True` for backward-compatible change and `False` for a breaking change (removing a required field).

**Q15.** Full real-time fraud pipeline. Assemble everything into `run_realtime_fraud_pipeline(transactions_df, runtime_seconds)`:
1. Producer thread: produces transactions at 100/second
2. Fraud detection consumer thread: consumes, evaluates, produces alerts
3. Analytics consumer thread: consumes, writes to `window_aggregates`
4. Monitor thread: prints lag every 5 seconds
5. After `runtime_seconds`, stop all threads gracefully (no data loss)
6. Print final report: transactions produced, processed, alerts generated, windows computed, DLQ count, processing lag

---

## Project 17 🔴 — Delta Lakehouse Architecture
**Dataset:** D5 — DataCo Supply Chain | **Focus:** Delta Lake, ACID transactions, time travel, MERGE, compaction

### Business Context
The supply chain team has been burned by partial writes corrupting their Parquet lake. You're migrating their pipeline to a proper lakehouse architecture with ACID guarantees, history, and self-healing.

### Setup
```python
# We'll build a pure-Python Delta Lake simulation that mirrors real Delta Lake behaviour.
# If you have PySpark + delta-spark installed, the actual PySpark code in each answer
# is directly runnable. The simulation lets you learn the concepts without cluster setup.

import json
import time
import shutil
from pathlib import Path
from datetime import datetime
import pandas as pd
import pyarrow as pa
import pyarrow.parquet as pq

class DeltaTable:
    """
    Pure-Python Delta Lake simulation.
    Implements: write, append, delete, merge, time travel, vacuum, history.
    """
    def __init__(self, base_path: str):
        self.path     = Path(base_path)
        self.log_path = self.path / "_delta_log"
        self.path.mkdir(parents=True, exist_ok=True)
        self.log_path.mkdir(exist_ok=True)
        self._version = self._load_current_version()

    def _load_current_version(self) -> int:
        logs = sorted(self.log_path.glob("*.json"))
        return len(logs)

    def _write_log(self, operation: str, metadata: dict) -> int:
        version = self._version
        entry = {
            "version":   version,
            "timestamp": datetime.now().isoformat(),
            "operation": operation,
            **metadata
        }
        log_file = self.log_path / f"{version:020d}.json"
        log_file.write_text(json.dumps(entry, indent=2))
        self._version += 1
        return version

    def write(self, df: pd.DataFrame, mode: str = "overwrite") -> int:
        """Write DataFrame. mode: 'overwrite' or 'append'."""
        part_name = f"part-{self._version:08d}.parquet"
        part_path = self.path / part_name

        if mode == "overwrite":
            # Mark all existing data files as removed
            existing = [f.name for f in self.path.glob("part-*.parquet")]
            removed  = [{"path": f, "rows": -1} for f in existing]
        else:
            removed = []

        table = pa.Table.from_pandas(df)
        pq.write_table(table, part_path, compression="snappy")

        return self._write_log("WRITE", {
            "mode":        mode,
            "rows_added":  len(df),
            "file_added":  part_name,
            "files_removed": removed,
            "schema":      {c: str(t) for c, t in zip(df.columns, df.dtypes)},
        })

    def read(self, as_of_version: int = None) -> pd.DataFrame:
        """Read current data or as of a specific version."""
        target = as_of_version if as_of_version is not None else self._version - 1
        active_files = set()

        for v in range(target + 1):
            log_file = self.log_path / f"{v:020d}.json"
            if not log_file.exists():
                continue
            entry = json.loads(log_file.read_text())
            if "file_added" in entry:
                active_files.add(entry["file_added"])
            for removed in entry.get("files_removed", []):
                active_files.discard(removed["path"])

        if not active_files:
            return pd.DataFrame()

        dfs = []
        for fname in active_files:
            fpath = self.path / fname
            if fpath.exists():
                dfs.append(pd.read_parquet(fpath))
        return pd.concat(dfs, ignore_index=True) if dfs else pd.DataFrame()

    def history(self) -> pd.DataFrame:
        """Return operation history."""
        rows = []
        for log_file in sorted(self.log_path.glob("*.json")):
            entry = json.loads(log_file.read_text())
            rows.append({
                "version":   entry["version"],
                "timestamp": entry["timestamp"],
                "operation": entry["operation"],
                "rows_added":entry.get("rows_added", 0),
            })
        return pd.DataFrame(rows)
```

---

### Questions

**Q1.** Load the supply chain CSV. Write the first 3 months of data to a `DeltaTable` at `data/delta/supply_chain/` using `mode="overwrite"`. Check: what files exist in the directory? What does the `_delta_log/` folder contain? Print the `history()`.

**Q2.** Append the next 3 months using `mode="append"`. Verify: total row count = 6 months. Check history — how many log entries? How many Parquet files?

**Q3.** Time travel. Read the table `as_of_version=0` (after the first write). Row count should equal the first 3 months only. Read `as_of_version=1` (after the append). Row count should equal 6 months. This is time travel — no extra storage cost for the history.

**Q4.** Implement `DeltaTable.delete(condition: Callable[[pd.DataFrame], pd.Series]) -> int` that:
- Reads current data
- Splits into rows matching condition (to delete) and rows not matching (to keep)
- Writes the kept rows as a new Parquet file
- Logs the operation with `operation="DELETE"`, `rows_removed=N`
- Returns rows deleted

Test: delete all orders with `delivery_status == "Shipping canceled"`. Verify row count decreased.

**Q5.** Implement `DeltaTable.merge(source_df, join_key, update_cols) -> dict` that:
- For rows in source_df where `join_key` exists in current data → update `update_cols`
- For rows in source_df where `join_key` doesn't exist → insert
- Returns `{"updated": N, "inserted": M}`

This is the UPSERT (INSERT OR UPDATE) pattern. Test: create 50 updated orders and 20 new orders. Merge them. Verify counts.

**Q6.** Schema evolution. Add a `discount_pct` column to a new batch of supply chain data. Implement `DeltaTable._merge_schemas(existing_cols, new_cols)` that:
- Detects new columns
- Reads existing files and fills missing columns with `None`
- Allows the write to proceed
- Logs `operation="SCHEMA_EVOLUTION"` with the added columns

**Q7.** Compaction. After 10 individual append operations, you have 10 small Parquet files. Implement `DeltaTable.optimize()` that:
- Reads all current data
- Writes as a single optimised file
- Logs `operation="OPTIMIZE"` with: files before, files after, rows
- Old files are marked as removed in the log (but NOT physically deleted — that's vacuum)

Verify: after `optimize()`, querying is faster (fewer files to open).

**Q8.** Vacuum. Implement `DeltaTable.vacuum(retain_hours: int = 168)` that:
- Finds files referenced only in log entries older than `retain_hours`
- Physically deletes those files
- Logs `operation="VACUUM"` with files deleted and space freed

After calling `vacuum(retain_hours=0)` (aggressive), verify: time travel to old versions now raises `FileNotFoundError` (the files are gone). This is the trade-off: space savings vs history.

**Q9.** Concurrent write safety. Implement optimistic concurrency control:
- Before writing, read the current version number
- After preparing the write but before committing, check if the version changed
- If it changed (another writer snuck in), retry the write
- If it's the same, commit atomically

Simulate two concurrent writers using `threading.Thread`. Verify: both writes complete, no data is lost, no partial writes.

**Q10.** Partition evolution. Your current table is not partitioned. Implement `DeltaTable.repartition(partition_cols)` that:
- Reads all data
- Rewrites as a partitioned dataset in `year=.../month=.../` structure
- Logs the operation
- Future reads use partition pruning

Verify: `read(filters=[("year", "==", "2018")])` is faster than reading all data (measure with `time.perf_counter()`).

**Q11.** Delta vs plain Parquet — the ACID test. Simulate the "partial write" problem:
1. Start writing 10,000 rows to plain Parquet
2. Raise an exception halfway through
3. Show that the destination now has corrupted/partial data

Then repeat with your `DeltaTable`:
1. Start writing 10,000 rows
2. Raise an exception halfway
3. Show that `read()` returns the previous complete version — no corruption

This is the core ACID value proposition.

**Q12.** Write `SupplyChainLakehouse` wrapping `DeltaTable` with:
- `raw`: DeltaTable at `data/lakehouse/raw/supply/`
- `clean`: DeltaTable at `data/lakehouse/clean/supply/`
- `aggregate`: DeltaTable at `data/lakehouse/gold/supply_monthly/`
- `ingest(df)`: write to raw
- `transform()`: read raw, apply transformations, write to clean
- `aggregate()`: read clean, compute monthly KPIs, write to aggregate

This is the Medallion Architecture (Bronze/Silver/Gold) implemented with ACID guarantees at every layer.

**Q13.** Streaming writes simulation. Simulate micro-batch writes to the lakehouse:
- Every 30 seconds (simulated): new supply chain data arrives
- Write to the raw DeltaTable (append)
- Run transform() → write to clean
- Run aggregate() → update gold layer
- Each cycle is atomic — either all three layers update or none do

After 5 cycles, verify: history() on each layer shows 5 write operations; time travel lets you see the state after cycle 2.

**Q14.** Delta Sharing simulation. Implement `DeltaTable.create_share(recipient, tables, expiry_days)` that:
- Creates a read-only "share token" (a JSON file)
- The token encodes: recipient, accessible tables, expiry timestamp
- Implement `read_shared(token_path)` that validates the token and returns the data

This simulates Databricks Delta Sharing / Snowflake Data Marketplace — sharing data across organisations without copying it.

**Q15.** Full lakehouse pipeline. Write `run_lakehouse_pipeline(config)` that:
1. Extracts supply chain CSV (or simulates daily arrival)
2. Writes raw data to Bronze DeltaTable (append, with schema validation)
3. Transforms to Silver (clean types, business logic, quality checks)
4. Builds Gold aggregations (department scorecard, monthly trends)
5. Runs OPTIMIZE on all three layers weekly
6. Runs VACUUM on raw layer (retain 30 days), clean layer (retain 90 days)
7. Logs every operation to a `pipeline_runs` DuckDB table

Print the full lakehouse history after running 4 weeks of simulated data.

---

## Project 18 🔴 — Cloud Architecture Design
**Dataset:** All datasets | **Focus:** Cloud service selection, architecture diagrams, presales thinking

### Business Context
You are a data engineering consultant. Three clients have described their situations. For each, you will design a cloud data architecture, justify every tool choice, estimate rough costs, and anticipate objections. There is no single correct answer — the goal is to think like an architect.

> **Note:** This is a design project — the deliverable is diagrams (ASCII), justifications, and written analysis, not running code. Where code is asked for, it illustrates the design rather than implements a full system.

### Client Profiles

**Client A — MedCore Hospital Group (Healthcare)**
- 15 hospitals, 200k patient admissions per year
- Data currently in: Oracle EHR (on-premises), Excel reports per hospital, no central warehouse
- Goals: unified patient analytics, regulatory reporting (HIPAA), readmission prediction
- Constraints: strict HIPAA compliance, data cannot leave EU region, IT team is Microsoft-heavy
- Budget: $15k/month

**Client B — Retailit (E-Commerce Startup)**
- 2M orders/year, growing 3× annually
- Data currently in: PostgreSQL (RDS on AWS), Stripe for payments, Segment for clickstream
- Goals: real-time inventory management, customer churn prediction, daily revenue dashboard
- Constraints: already on AWS, 2-person data team, need results in 6 weeks
- Budget: $5k/month

**Client C — Globalix Logistics (Manufacturing)**
- 50 warehouses across 12 countries, 10M shipments/year
- Data currently in: SAP (on-premises), IoT sensors on vehicles (100k events/minute), Excel
- Goals: predictive maintenance, supply chain optimisation, executive KPI dashboard
- Constraints: multi-cloud (AWS in Americas, Azure in Europe, GCP in APAC), SAP integration required
- Budget: $80k/month

---

### Questions

**Q1.** For each client, identify their **maturity stage** (Stage 1–4 from the tutorial). Write 2–3 sentences explaining your reasoning.

**Q2.** **Client A — Tool selection.** Given the Microsoft-heavy IT team, HIPAA requirement, and EU data residency: which cloud platform would you recommend and why? List the specific Azure services you'd use for each stage of the data journey (ingest, store, process, serve). Are there any services you'd explicitly NOT recommend for this client?

**Q3.** **Client A — Architecture diagram.** Draw an ASCII architecture diagram showing data flow from Oracle EHR → central warehouse → analytics + ML. Include: ingestion method, raw storage, transformation, warehouse, BI tool, governance layer.

**Q4.** **Client A — Compliance architecture.** HIPAA requires: data encryption at rest and in transit, access logging, column-level masking of PII (patient names, SSNs), and audit trails. Which Azure services handle each requirement? What would you tell the client about shared responsibility (what Microsoft handles vs what the client must configure)?

**Q5.** **Client B — Tool selection.** Already on AWS, 2-person team, 6-week deadline, growing fast. Design the simplest possible stack that works today and scales to 10× data volume. What would you start with? What would you migrate to at 10×? Justify every choice based on the team size and timeline constraint.

**Q6.** **Client B — Real-time vs batch.** The client says they need "real-time inventory management." Ask the right clarifying question: what latency is actually needed? Map three possible answers (< 1 minute / < 1 hour / daily) to three different architectures. Which architecture would you recommend first and why?

**Q7.** **Client B — Cost breakdown.** Estimate monthly costs for your recommended stack:
- S3 storage at 500GB
- RDS PostgreSQL → Fivetran connection (1 source)
- Redshift dc2.large (1 node)
- Airflow on a t3.medium EC2
- Looker Studio (free) for BI

Show your math. Is this within their $5k/month budget?

**Q8.** **Client C — Multi-cloud architecture.** Data is spread across 3 clouds. Design a multi-cloud strategy that:
- Keeps regional data in its cloud (GDPR compliance)
- Enables global executive reporting
- Uses a tool that runs on all three clouds natively

Which tool is the natural choice for the cross-cloud layer? Draw the data flow.

**Q9.** **Client C — IoT streaming.** 100,000 events per minute from vehicle sensors. That's 1,666 events per second, 144M events per day. Design the streaming architecture:
- What Kafka/streaming service on each cloud?
- How do you handle a vehicle going offline for 4 hours and then sending all buffered events?
- What's the processing latency target for predictive maintenance alerts?
- How do you store time-series data cost-effectively?

**Q10.** **Client C — SAP integration.** SAP on-premises is notoriously difficult to integrate. What are the two main approaches (CDC vs bulk extract)? What specific connector/service would you use for each cloud? What are the risks of SAP integration that you should flag in a presales conversation?

**Q11.** **Objection handling — Client A.** The IT director says: *"We already have SQL Server Reporting Services. Why do we need Azure Synapse? We'd rather just upgrade SSRS."* Write a response that: acknowledges the valid concern, explains the specific limitation they'll hit, and makes the case for moving forward without being dismissive.

**Q12.** **Objection handling — Client B.** The CTO says: *"Why do we need Fivetran at $500/month? Can't we just write Python scripts to pull from Stripe?"* Write a response that honestly answers the build vs buy question for their specific situation (2-person team, 6-week deadline).

**Q13.** **Objection handling — Client C.** The CFO says: *"$80k/month seems like a lot. Our current setup costs us $12k/month in server maintenance."* Write a response that reframes cost, quantifies the opportunity cost of the current setup, and explains what they get for $80k that they can't get for $12k.

**Q14.** **Migration planning — Client A.** The hospital currently generates reports from Oracle. Migrating to Azure Synapse will break their existing reports. Write a 4-phase migration plan that:
- Phase 1: parallel run (new platform built, validated, reports still from Oracle)
- Phase 2: cut over non-critical reports
- Phase 3: cut over critical reports with sign-off
- Phase 4: Oracle decommission

For each phase: timeline estimate, who is responsible, how you define "done."

**Q15.** **Full proposal.** Choose one client (A, B, or C). Write a 1-page (in markdown) architecture proposal covering:
- Executive summary (3 sentences: problem, solution, outcome)
- Recommended architecture with tool stack
- Phase 1 scope (what you'd deliver in 90 days)
- Cost estimate
- Top 3 risks and mitigations

This mirrors what a real presales document looks like.

---

## Project 19 🔴 — Presales Case Studies
**Dataset:** All datasets | **Focus:** Presales conversations, solution design, objection handling, client communication

### Business Context
You are senior enough to lead presales calls. A client has described their situation. Your job is to ask the right questions, diagnose the real problem, and articulate a solution — without overselling or underselling. This project tests whether you understand the business context of everything you've learned technically.

> **Note:** All deliverables are written — analysis, conversation scripts, and documents. This is the track where data engineering knowledge becomes business value.

---

### Questions

**Q1.** **Discovery call — the right questions.** A client says: *"We want to build a data lake."* That tells you almost nothing useful. Write 10 discovery questions you would ask to understand: what problem they're actually solving, what data they have, who consumes it, what they've tried before, and what success looks like in 6 months.

**Q2.** **Diagnosing maturity.** Read these client statements and identify the maturity stage (1–4) for each:
- *"Our BI team exports CSVs from our CRM every Monday and builds reports in Excel."*
- *"We have a Redshift cluster but our pipelines break every time a source schema changes."*
- *"Our data warehouse is working well but we want to do real-time fraud detection."*
- *"We have 50 data engineers and our cloud bill is $200k/month. We need better governance."*

For each: name the stage, identify the top 2 problems, and name the first thing you'd recommend.

**Q3.** **The "we just need a dashboard" client.** A retail company says they want *"a dashboard showing daily sales."* After asking questions, you discover:
- Data is in: Shopify (orders), QuickBooks (accounting), a custom inventory system
- 3 people will use the dashboard
- They've tried Google Looker Studio but "it's too complicated"
- Budget: $500/month

Design the simplest possible solution. What tools? What's the architecture? What do you NOT build? Write the 5-sentence summary you'd give to a non-technical stakeholder.

**Q4.** **Scope creep.** A client started with "we need a daily orders pipeline." Over 3 weeks, the scope has grown to include: real-time streaming, ML predictions, a data catalog, a self-service BI platform, and governance policies. This is a $50k project that's turning into a $500k project.

Write the conversation you'd have with the client to reset scope, protect the original timeline, and create a phased roadmap. What do you build now vs what goes in Phase 2?

**Q5.** **The technical client.** A client's internal data engineer says in a call: *"We're evaluating whether to use Iceberg or Delta Lake as our table format. We're currently on AWS but may move some workloads to Azure. We also need to support both Spark and Trino as query engines."*

Write your response. What questions do you ask? What's the answer? What does the choice depend on? Show that you understand the technical decision without being dismissive of either option.

**Q6.** **Estimating effort.** A client asks: *"How long would it take to build a data warehouse for us?"* You know nothing about their data yet.

Write the response you'd give to set expectations properly, and list the 8 variables that determine timeline. Then: given the following inputs, estimate effort ranges — 5 data sources / 3 analysts / no existing DW / no cloud account yet / moderate data quality issues.

**Q7.** **Build vs buy.** A client is deciding between:
- Option A: Build custom Python ETL + Airflow + Redshift (~$8k/month infrastructure)
- Option B: Fivetran + dbt Cloud + Snowflake (~$12k/month including tools)

The client leans toward Option A because it's cheaper. Write the analysis that helps them make the right decision. Include: total cost of ownership (not just infrastructure), maintenance burden, scalability, team skills needed. What would you recommend and why?

**Q8.** **The "we tried this before" client.** A client says: *"We built a data warehouse 3 years ago. It failed. Our analysts stopped using it because the data was always wrong and late. Why would this time be different?"*

This is a trust problem, not a technology problem. Write your response. What caused the previous failure (ask questions)? What do you commit to doing differently? What does success look like in 30 days?

**Q9.** **Stakeholder alignment.** Three stakeholders have conflicting views:
- CTO: *"We should build everything in-house for maximum control"*
- CFO: *"We need this done in 3 months, not 12"*
- Head of Analytics: *"I just need clean data I can trust"*

Write a proposed solution that genuinely addresses all three concerns. It won't fully satisfy anyone — explain the trade-offs explicitly.

**Q10.** **Data contract negotiation.** You're presenting a data contract to the engineering team that owns the source system:
```yaml
source: orders_service
freshness: updated within 1 hour
schema_change_notice: 2 weeks advance notice
null_rate_max: 1% on order_id
```
The source team pushes back: *"We can't guarantee 1-hour freshness — our service does weekly releases. And 2 weeks notice for schema changes is impossible — we move fast."*

Write the negotiation. What do you compromise on? What do you hold firm on and why? What's the final agreed contract?

**Q11.** **ROI conversation.** A client's CFO asks: *"How do I justify $15,000/month to my board?"* 

Build the ROI case using:
- 5 analysts spending 40% of time on data preparation (avg salary ₹18L/year)
- 2 incorrect business decisions per year estimated at ₹50L impact each
- Current data latency: 3 days (reports show 3-day-old data)

Calculate: cost of the problem, cost of the solution, payback period. Present this as a business case, not a technology pitch.

**Q12.** **Handling "we can do this ourselves."** A client says their intern *"knows Python and can build the pipelines."* Without being condescending, explain what professional data engineering involves that a Python-literate intern would likely miss.

Write 6 specific things that separate a production-grade pipeline from a script that works in dev.

**Q13.** **The "we don't trust the cloud" client.** A financial services client says: *"We can't put customer data in the cloud. It's against our policy."* 

After asking questions, you discover: they already use Office 365 (Azure AD), their CRM is Salesforce (cloud), and their HR system is Workday (cloud). Write the conversation that reframes their actual concern (probably compliance, not cloud per se) and maps their real requirements to a solution.

**Q14.** **Post-implementation review.** A project you delivered 6 months ago has issues:
- Pipelines run 40% slower than the SLA
- 3 data quality incidents in the last month
- Two analysts say they "don't trust the data"

The client is unhappy. Write the agenda for a post-implementation review meeting. What do you investigate before the meeting? What do you commit to in the meeting? How do you rebuild trust?

**Q15.** **Capstone — full presales document.** A new prospect has sent you this brief:
> *"We are a 500-person logistics company. We have data in SAP, a legacy Oracle warehouse from 2012, and 15 Excel-based reports that finance runs manually each month. We want to modernise. We've heard about 'data mesh' and want to implement it. Budget is flexible but we need to show ROI in 12 months."*

Write a 2-page presales response document (in markdown) covering:
1. Your understanding of their situation (restate the problem in your words)
2. What "data mesh" actually means and whether they need it right now (honest assessment)
3. Recommended phased approach (Phase 1: 90 days / Phase 2: 6 months / Phase 3: 12 months)
4. Technology stack with justifications
5. What success looks like at 12 months (measurable outcomes)
6. Top 3 risks and how you'd mitigate them
7. Why your team (feel free to be generic — "a team with these capabilities") is the right choice


---

## Additional Questions — Existing Projects

### Project 1 — Additional Questions (Formats → Delta Lake)

**Q16.** Convert the partitioned Parquet dataset you built in Q6 into a Delta Lake table using your `DeltaTable` class from Project 17. Read the same January 2018 filter from Delta. Does the result match? What extra capabilities do you now have that plain Parquet didn't provide?

**Q17.** Simulate an accidental overwrite. Write the correct January 2018 data to the Delta table. Then accidentally overwrite with wrong data (all amounts set to 0). Use time travel to recover: read `as_of_version=0` (the correct data) and write it back as a new version. Verify: the current table now has correct data again, AND the history shows the accident and recovery.

---

### Project 6 — Additional Questions (Star Schema → dbt)

**Q16.** Write dbt staging models for the star schema you built in Q1–Q10. Create:
- `models/staging/stg_orders.sql` — cleans and renames raw orders
- `models/staging/stg_payments.sql` — aggregates payments to order level
- `models/staging/stg_customers.sql` — deduplicates, adds region

Each model should use `{{ source('olist', 'raw_tablename') }}` as the source reference. Add `schema.yml` with `unique` and `not_null` tests on primary keys.

**Q17.** Write `models/marts/fct_orders.sql` as an incremental dbt model:
```sql
{{ config(materialized='incremental', unique_key='order_id') }}
...
{% if is_incremental() %}
WHERE ordered_at > (SELECT MAX(ordered_at) FROM {{ this }})
{% endif %}
```
Run `dbt run --select fct_orders` followed by `dbt test --select fct_orders`. Fix any test failures. Run `dbt run --full-refresh` and confirm row count matches your DuckDB star schema from Q9.

---

### Project 9 — Additional Questions (Airflow → Prefect)

**Q16.** Rewrite the UK Retail pipeline from Project 9 using Prefect. Use `@flow` and `@task` decorators. The pipeline structure should be identical — same steps, same data — but implemented in Prefect's model. Compare:
- Lines of code (Airflow DAG vs Prefect flow)
- How you pass data between steps (XCom vs return values)
- How you run it locally (Airflow requires a server vs `python my_flow.py`)

**Q17.** Add a Dagster asset version. Define `raw_uk_retail`, `clean_uk_retail`, and `monthly_revenue_uk` as Dagster assets using `@asset`. Run `dagster dev` and observe the asset lineage graph in the UI. What information does the Dagster UI show that the Airflow/Prefect UIs don't? When would you choose Dagster over the others?

---

### Project 12 — Additional Questions (Streaming → Schema Registry)

**Q16.** Implement schema validation on your fraud event stream using the `MockSchemaRegistry` from Project 16 Q14. Before producing each transaction event, validate it against the registered schema. Produce 1,000 transactions, then introduce 50 "schema-breaking" events (missing a required field). Verify: the 50 bad events go to the DLQ, the 950 valid ones process normally.

**Q17.** Schema evolution in production. Your fraud detection schema adds a new field `"device_fingerprint"` (optional, string, default null). Some events have it, some don't. Implement the backward-compatible schema change: register v2 schema, produce a mix of v1 and v2 events, consume all of them. Verify: v1 events (missing `device_fingerprint`) deserialise successfully with `device_fingerprint=None`. This is how real Kafka schema evolution works in production.

---

## Answer Key

---

### Project 1 — Selected Answers

```python
# Q2: CSV vs Parquet comparison
import pandas as pd
import time
from pathlib import Path

df = pd.read_csv("data/olist/olist_orders_dataset.csv",
                 parse_dates=["order_purchase_timestamp",
                               "order_delivered_customer_date",
                               "order_estimated_delivery_date"])

csv_path = "data/olist/olist_orders_dataset.csv"
csv_size  = Path(csv_path).stat().st_size / 1024
print(f"CSV:             {csv_size:,.0f} KB")

for codec in [None, "snappy", "gzip", "zstd"]:
    path = f"/tmp/orders_{codec or 'none'}.parquet"
    t    = time.perf_counter()
    df.to_parquet(path, compression=codec)
    write_ms = (time.perf_counter() - t) * 1000
    size_kb  = Path(path).stat().st_size / 1024
    print(f"{str(codec):<8}: {size_kb:>8,.0f} KB  "
          f"({csv_size/size_kb:>5.1f}× smaller)  "
          f"{write_ms:>6.0f}ms write")

# Typical output:
# None    : 7,890 KB  ( 1.0× smaller)    18ms write
# snappy  : 1,920 KB  ( 4.1× smaller)    55ms write
# gzip    : 1,490 KB  ( 5.3× smaller)   310ms write
# zstd    : 1,180 KB  ( 6.7× smaller)   160ms write


# Q6: Partitioned Parquet
import pyarrow as pa
import pyarrow.parquet as pq

df["year"]  = df["order_purchase_timestamp"].dt.year
df["month"] = df["order_purchase_timestamp"].dt.month

pq.write_to_dataset(
    pa.Table.from_pandas(df),
    root_path="data/warehouse/orders/",
    partition_cols=["year", "month"],
    compression="snappy",
    existing_data_behavior="delete_matching",
)

# Directory structure created:
# data/warehouse/orders/
#   year=2016/month=9/part-00000000.parquet
#   year=2016/month=10/part-00000000.parquet
#   ...
#   year=2018/month=8/part-00000000.parquet

# Q7: Partition pruning read
import time

# Without partitioning
t = time.perf_counter()
df_full = pd.read_parquet("data/warehouse/orders/")
df_jan18_slow = df_full[
    (df_full["year"] == 2018) & (df_full["month"] == 1)
]
slow_ms = (time.perf_counter() - t) * 1000

# With partition pruning
t = time.perf_counter()
df_jan18_fast = pd.read_parquet(
    "data/warehouse/orders/",
    filters=[("year", "==", 2018), ("month", "==", 1)]
)
fast_ms = (time.perf_counter() - t) * 1000

print(f"Full scan + filter: {slow_ms:.0f}ms")
print(f"Partition pruning:  {fast_ms:.0f}ms")
print(f"Speedup: {slow_ms/fast_ms:.1f}×")
# Typical: 3–10× speedup depending on data size
```

---

### Project 5 — Full Pipeline Answer

```python
# pipeline/run.py — complete supply chain ETL pipeline
import json, logging, sys, time
from dataclasses import dataclass, field
from datetime import date, timedelta
from pathlib import Path

import duckdb, numpy as np, pandas as pd

logger = logging.getLogger("supply_pipeline")

@dataclass
class PipelineConfig:
    source_path:   str = "data/supply_chain.csv"
    db_path:       str = "data/warehouse/supply.ddb"
    encoding:      str = "latin-1"
    log_dir:       str = "logs"

    @classmethod
    def from_json(cls, path: str) -> "PipelineConfig":
        with open(path) as f:
            return cls(**json.load(f))

@dataclass
class PipelineResult:
    run_id:         str
    status:         str
    rows_extracted: int  = 0
    rows_loaded:    int  = 0
    duration_s:     float = 0.0
    error_msg:      str  = ""


def extract(config: PipelineConfig) -> pd.DataFrame:
    path = Path(config.source_path)
    if not path.exists():
        raise FileNotFoundError(f"Source not found: {path}")
    df = pd.read_csv(path, encoding=config.encoding, low_memory=False)
    df.columns = df.columns.str.lower().str.strip().str.replace(" ", "_")
    logger.info(f"Extracted {len(df):,} rows")
    return df


def transform(df: pd.DataFrame) -> pd.DataFrame:
    df = df.copy()
    date_col = next((c for c in df.columns if "date" in c and "order" in c), None)
    if date_col:
        df[date_col] = pd.to_datetime(df[date_col], errors="coerce")
        df["order_month"] = df[date_col].dt.strftime("%Y-%m")
    if "order_profit_per_order" in df.columns and "sales" in df.columns:
        df["profit_margin"] = (df["order_profit_per_order"]
                               / df["sales"].replace(0, np.nan)).round(4)
    if "late_delivery_risk" in df.columns:
        df["late_flag"] = df["late_delivery_risk"].fillna(0).astype(int)
    null_rates = df.isnull().mean()
    drop_cols  = null_rates[null_rates > 0.5].index.tolist()
    if drop_cols:
        logger.warning(f"Dropping high-null columns: {drop_cols}")
        df = df.drop(columns=drop_cols)
    logger.info(f"Transformed {len(df):,} rows")
    return df


def quality_check(df: pd.DataFrame) -> dict:
    checks = {
        "has_rows":           len(df) >= 100,
        "profit_margin_range":df.get("profit_margin", pd.Series([0])).dropna().between(-1, 2).all(),
        "late_flag_binary":   df.get("late_flag", pd.Series([0])).isin([0, 1]).all(),
    }
    for name, ok in checks.items():
        logger.info(f"  {'✓' if ok else '✗'} {name}")
    return checks


def load(df: pd.DataFrame, config: PipelineConfig) -> int:
    Path(config.db_path).parent.mkdir(parents=True, exist_ok=True)
    con = duckdb.connect(config.db_path)
    con.execute("DROP TABLE IF EXISTS supply_chain")
    con.register("_df", df)
    con.execute("CREATE TABLE supply_chain AS SELECT * FROM _df")
    rows = con.execute("SELECT COUNT(*) FROM supply_chain").fetchone()[0]
    con.close()
    return rows


def run_pipeline(config: PipelineConfig = None, run_date: str = None) -> PipelineResult:
    config   = config or PipelineConfig()
    run_date = run_date or str(date.today() - timedelta(1))
    run_id   = f"supply_{run_date.replace('-', '')}"
    start    = time.perf_counter()

    Path(config.log_dir).mkdir(exist_ok=True)
    logging.basicConfig(level=logging.INFO, handlers=[
        logging.StreamHandler(),
        logging.FileHandler(f"{config.log_dir}/supply_{run_date}.log"),
    ])

    result = PipelineResult(run_id=run_id, status="RUNNING")
    try:
        df_raw = extract(config)
        result.rows_extracted = len(df_raw)

        df_clean = transform(df_raw)

        checks = quality_check(df_clean)
        if not all(checks.values()):
            failed = [k for k, v in checks.items() if not v]
            raise ValueError(f"Quality gate failed: {failed}")

        result.rows_loaded = load(df_clean, config)
        result.status      = "SUCCESS"

    except Exception as e:
        result.status    = "FAILED"
        result.error_msg = str(e)
        logger.exception(f"Pipeline failed: {e}")

    result.duration_s = time.perf_counter() - start
    logger.info(f"Pipeline {result.status} in {result.duration_s:.1f}s "
                f"({result.rows_loaded:,} rows)")
    return result


if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument("--date",   default=str(date.today() - timedelta(1)))
    parser.add_argument("--config", default=None)
    args   = parser.parse_args()
    config = PipelineConfig.from_json(args.config) if args.config else PipelineConfig()
    result = run_pipeline(config, args.date)
    sys.exit(0 if result.status == "SUCCESS" else 1)
```

---

### Project 6 — Key Answers

```python
# Q3: Build dim_date
def build_date_dim(start: str = "2016-01-01", end: str = "2026-12-31") -> pd.DataFrame:
    dates = pd.date_range(start, end, freq="D")
    df = pd.DataFrame({"full_date": dates})
    df["date_id"]    = df["full_date"].dt.strftime("%Y%m%d").astype(int)
    df["year"]       = df["full_date"].dt.year
    df["quarter"]    = df["full_date"].dt.quarter
    df["month"]      = df["full_date"].dt.month
    df["month_name"] = df["full_date"].dt.month_name()
    df["week"]       = df["full_date"].dt.isocalendar().week.astype(int)
    df["day_of_week"]= df["full_date"].dt.dayofweek
    df["day_name"]   = df["full_date"].dt.day_name()
    df["is_weekend"] = df["day_of_week"] >= 5
    df["year_month"] = df["full_date"].dt.strftime("%Y-%m")
    return df

dim_date = build_date_dim()
print(f"dim_date: {len(dim_date):,} rows")  # 3,987 rows


# Q10: Analytical queries on star schema
import duckdb

con = duckdb.connect("data/warehouse/olist.ddb")

# Monthly revenue
monthly = con.execute("""
    SELECT
        d.year,
        d.month_name,
        COUNT(DISTINCT f.order_id)       AS orders,
        ROUND(SUM(f.item_revenue), 2)    AS revenue
    FROM fct_orders f
    JOIN dim_date d ON f.date_id = d.date_id
    GROUP BY d.year, d.month, d.month_name
    ORDER BY d.year, d.month
""").df()

# Late delivery rate by seller state
late_by_state = con.execute("""
    SELECT
        s.seller_state,
        COUNT(*) AS orders,
        ROUND(AVG(f.is_late) * 100, 1) AS late_pct
    FROM fct_orders f
    JOIN dim_seller s ON f.seller_sk = s.seller_sk
    WHERE f.order_status = 'delivered'
    GROUP BY s.seller_state
    ORDER BY late_pct DESC
    LIMIT 10
""").df()
print(late_by_state)
```

---

### Project 9 — Key Answers

```python
# Q3: Extract task with XCom
def extract(**context):
    import pandas as pd
    ds = context["ds"]
    ti = context["ti"]

    df = pd.read_csv("data/uk_retail/online_retail_II.csv",
                     encoding="latin-1", low_memory=False)
    df.columns = df.columns.str.lower().str.strip().str.replace(" ", "_")
    df["invoicedate"] = pd.to_datetime(df["invoicedate"], errors="coerce")

    output_path = f"/tmp/uk_retail_raw_{ds}.parquet"
    df.to_parquet(output_path, index=False)
    ti.xcom_push(key="raw_path",   value=output_path)
    ti.xcom_push(key="row_count",  value=len(df))
    print(f"Extracted {len(df):,} rows for {ds}")


# Q5: Transform task
def transform(**context):
    import pandas as pd, numpy as np
    ds       = context["ds"]
    ti       = context["ti"]
    raw_path = ti.xcom_pull(task_ids="extract", key="raw_path")

    df = pd.read_parquet(raw_path)

    # Filter out returns and bad rows
    df = df[df["quantity"] > 0]
    df = df[df["price"]    > 0]
    df = df.dropna(subset=["customer_id"])

    df["line_total"]  = df["quantity"] * df["price"]
    df["year_month"]  = df["invoicedate"].dt.strftime("%Y-%m")

    clean_path = f"/tmp/uk_retail_clean_{ds}.parquet"
    df.to_parquet(clean_path, index=False)
    ti.xcom_push(key="clean_path", value=clean_path)
    ti.xcom_push(key="rows_clean", value=len(df))
    print(f"Transformed: {len(df):,} clean rows")


# Q12: DAG integrity tests
def test_dag_integrity():
    import importlib.util, sys
    spec = importlib.util.spec_from_file_location(
        "dag_uk_retail", "dags/dag_uk_retail_daily.py"
    )
    mod = importlib.util.module_from_spec(spec)
    spec.loader.exec_module(mod)

    dag = mod.dag
    assert dag.dag_id == "uk_retail_daily_pipeline"
    assert dag.catchup == False
    assert dag.max_active_runs == 1
    assert len(dag.tasks) == 8, f"Expected 8 tasks, got {len(dag.tasks)}"

    task_ids = {t.task_id for t in dag.tasks}
    assert "extract"         in task_ids
    assert "validate"        in task_ids
    assert "transform"       in task_ids
    assert "load"            in task_ids
    assert "post_load_check" in task_ids
    print("All DAG integrity tests passed")
```

---

### Project 12 — Key Answers

```python
# Q3: Micro-batch processing with state
def process_batch(batch_df: pd.DataFrame, state: dict) -> tuple:
    """
    Process one micro-batch. Returns (processed_df, updated_state, fraud_count).
    state = {customer_id: running_avg_amount}
    """
    results   = []
    fraud_count = 0

    for _, row in batch_df.iterrows():
        cid    = str(row.get("customer_id", "unknown"))
        amount = float(row.get("amount", 0))

        # Get or initialise customer average
        avg = state.get(cid, amount)

        # Fraud rule: amount > 3× customer average
        is_suspicious = amount > 3 * avg

        if is_suspicious:
            fraud_count += 1

        # Update running average (exponential moving average, α=0.3)
        state[cid] = 0.3 * amount + 0.7 * avg

        results.append({
            **row.to_dict(),
            "is_suspicious": int(is_suspicious),
            "customer_avg":  round(avg, 2),
        })

    return pd.DataFrame(results), state, fraud_count


# Q6: Micro-batch loop
import duckdb

con   = duckdb.connect("data/streaming.ddb")
con.execute("""
    CREATE TABLE IF NOT EXISTS streaming_results (
        transaction_id  VARCHAR,
        customer_id     VARCHAR,
        amount          DOUBLE,
        is_suspicious   INTEGER,
        customer_avg    DOUBLE,
        batch_number    INTEGER
    )
""")

df    = pd.read_csv("data/fraud/transactions.csv").sort_values("transaction_date")
state = {}
batch_num   = 0
total_fraud = 0

stream = EventStream(df, timestamp_col="transaction_date")
while stream.has_more():
    batch = stream.get_batch(100)
    if len(batch) == 0:
        break

    processed, state, fraud_in_batch = process_batch(batch, state)
    total_fraud += fraud_in_batch
    batch_num   += 1

    processed["batch_number"] = batch_num
    con.register("_batch", processed[["transaction_id","customer_id",
                                      "amount","is_suspicious",
                                      "customer_avg","batch_number"]])
    con.execute("INSERT INTO streaming_results SELECT * FROM _batch")

total_in_db = con.execute("SELECT COUNT(*) FROM streaming_results").fetchone()[0]
print(f"Batches processed: {batch_num}")
print(f"Total events: {total_in_db:,}")
print(f"Fraud alerts: {total_fraud:,} ({100*total_fraud/total_in_db:.1f}%)")
```

---

### Project 15 — Orchestrator Skeleton Answer

```python
# pipeline/orchestrator.py
import logging, time
from dataclasses import dataclass, field
from typing import Callable, Dict, List

logger = logging.getLogger("orchestrator")

@dataclass
class PipelineResult:
    source:     str
    status:     str      # SUCCESS | FAILED | SKIPPED
    rows_in:    int   = 0
    rows_out:   int   = 0
    duration_s: float = 0.0
    error_msg:  str   = ""


class PipelineOrchestrator:
    def __init__(self, warehouse, quality_engine, lineage_tracker):
        self.warehouse = warehouse
        self.quality   = quality_engine
        self.lineage   = lineage_tracker
        self._sources: Dict[str, dict] = {}
        self._transforms: Dict[str, Callable] = {}

    def register_source(self, name: str, extract_fn: Callable,
                        transform_fn: Callable) -> None:
        self._sources[name]    = {"extract": extract_fn}
        self._transforms[name] = transform_fn

    def run(self, sources: List[str], run_date: str) -> Dict[str, PipelineResult]:
        results = {s: self._run_one(s, run_date) for s in sources}
        self._log_summary(results, run_date)
        return results

    def _run_one(self, source: str, run_date: str,
                 max_retries: int = 3) -> PipelineResult:
        if source not in self._sources:
            return PipelineResult(source=source, status="SKIPPED",
                                  error_msg="Not registered")
        start = time.perf_counter()

        for attempt in range(1, max_retries + 1):
            try:
                df_raw   = self._sources[source]["extract"](run_date)
                qr       = self.quality.run(df_raw, source)
                if not qr.is_ok:
                    raise ValueError(f"Quality gate: {qr.errors}")
                df_clean = self._transforms[source](df_raw)
                self.warehouse.load(df_clean, source, mode="replace")
                self.lineage.log_transform(
                    f"raw_{source}", source, f"transform_{source}",
                    len(df_raw), len(df_clean), run_date
                )
                return PipelineResult(source=source, status="SUCCESS",
                                      rows_in=len(df_raw), rows_out=len(df_clean),
                                      duration_s=time.perf_counter()-start)
            except Exception as e:
                if attempt < max_retries:
                    time.sleep(2 ** attempt)
                else:
                    return PipelineResult(source=source, status="FAILED",
                                          duration_s=time.perf_counter()-start,
                                          error_msg=str(e))

    def _log_summary(self, results, run_date):
        logger.info(f"\n{'='*50}\nPipeline Summary — {run_date}\n{'='*50}")
        for s, r in results.items():
            icon = "✓" if r.status == "SUCCESS" else "✗"
            logger.info(f"  {icon} {s:<20} {r.status:<10} "
                        f"{r.rows_out:>8,} rows  {r.duration_s:.1f}s")
```

---

### Project 16 — Key Answers

```python
# Q1: TransactionProducer
class TransactionProducer:
    def __init__(self, broker: MockKafkaBroker, topic: str = "transactions"):
        self.broker = broker
        self.topic  = topic
        broker.create_topic(topic)

    def produce(self, transaction: dict) -> int:
        key   = str(transaction.get("customer_id", "unknown"))
        value = transaction.copy()
        return self.broker.produce(self.topic, key, value)

df = pd.read_csv("data/fraud/transactions.csv").head(1000)
producer  = TransactionProducer(broker)
offsets   = [producer.produce(row.to_dict()) for _, row in df.iterrows()]
print(f"Produced {len(offsets):,} messages  "
      f"| first offset: {offsets[0]}  "
      f"| last offset: {offsets[-1]}")
# Output: Produced 1,000 messages | first offset: 0 | last offset: 999


# Q5: Stateful fraud detection consumer
class FraudDetectionConsumer:
    def __init__(self, broker, source_topic="transactions",
                 alert_topic="fraud.alerts", group="fraud-detection"):
        self.broker       = broker
        self.source_topic = source_topic
        self.alert_topic  = alert_topic
        self.group        = group
        self.state        = {}   # customer_id → running avg amount
        broker.create_topic(alert_topic)

    def process_all(self, batch_size: int = 50) -> int:
        total_alerts = 0
        while True:
            messages = self.broker.consume(self.source_topic, self.group, batch_size)
            if not messages:
                break
            for msg in messages:
                txn    = msg["value"]
                cid    = str(txn.get("customer_id", ""))
                amount = float(txn.get("amount", 0))
                avg    = self.state.get(cid, amount)

                if amount > 3 * avg:
                    self.broker.produce(self.alert_topic, cid, {
                        "transaction_id": txn.get("transaction_id"),
                        "customer_id":    cid,
                        "amount":         amount,
                        "customer_avg":   round(avg, 2),
                        "reason":         "amount_exceeds_3x_avg",
                    })
                    total_alerts += 1

                self.state[cid] = 0.3 * amount + 0.7 * avg

        return total_alerts

fdc     = FraudDetectionConsumer(broker)
alerts  = fdc.process_all()
print(f"Fraud alerts generated: {alerts}")
print(f"Fraud rate: {100*alerts/1000:.1f}%")
print(f"Alert topic size: {len(broker._topics.get('fraud.alerts', []))}")


# Q8: Exactly-once with DuckDB offset tracking
import duckdb

def create_state_db(db_path: str) -> duckdb.DuckDBPyConnection:
    con = duckdb.connect(db_path)
    con.execute("""
        CREATE TABLE IF NOT EXISTS consumer_offsets (
            group_id  VARCHAR,
            topic     VARCHAR,
            offset    INTEGER,
            updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            PRIMARY KEY (group_id, topic)
        )
    """)
    con.execute("""
        CREATE TABLE IF NOT EXISTS processed_transactions (
            transaction_id VARCHAR PRIMARY KEY,
            customer_id    VARCHAR,
            amount         DOUBLE,
            is_suspicious  INTEGER,
            processed_at   TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        )
    """)
    return con

def get_committed_offset(con, group: str, topic: str) -> int:
    result = con.execute(
        "SELECT offset FROM consumer_offsets WHERE group_id=? AND topic=?",
        [group, topic]
    ).fetchone()
    return result[0] if result else 0

def commit_offset(con, group: str, topic: str, offset: int) -> None:
    con.execute("""
        INSERT INTO consumer_offsets (group_id, topic, offset)
        VALUES (?, ?, ?)
        ON CONFLICT (group_id, topic)
        DO UPDATE SET offset=excluded.offset, updated_at=CURRENT_TIMESTAMP
    """, [group, topic, offset])

# Exactly-once consumer loop
con   = create_state_db("data/exactly_once.ddb")
group = "exactly-once-consumer"
topic = "transactions"

# Resume from committed offset
start_offset = get_committed_offset(con, group, topic)
# Manually set broker offset to committed position
broker._offsets.setdefault(group, {})[topic] = start_offset

batch_size = 50
while True:
    messages = broker.consume(topic, group, batch_size)
    if not messages:
        break
    last_offset = messages[-1]["offset"]

    try:
        rows = []
        for msg in messages:
            txn = msg["value"]
            rows.append({
                "transaction_id": str(txn.get("transaction_id", "")),
                "customer_id":    str(txn.get("customer_id", "")),
                "amount":         float(txn.get("amount", 0)),
                "is_suspicious":  0,
            })
        batch_df = pd.DataFrame(rows)
        con.register("_batch", batch_df)
        con.execute("""
            INSERT OR IGNORE INTO processed_transactions
                (transaction_id, customer_id, amount, is_suspicious)
            SELECT transaction_id, customer_id, amount, is_suspicious
            FROM _batch
        """)
        # Only commit offset after successful write
        commit_offset(con, group, topic, last_offset + 1)
    except Exception as e:
        print(f"Batch failed at offset {last_offset}: {e}")
        # On restart, consumer resumes from last committed offset
        break

total = con.execute("SELECT COUNT(*) FROM processed_transactions").fetchone()[0]
print(f"Total processed (no duplicates): {total:,}")
```

---

### Project 17 — Key Answers

```python
# Q3: Time travel
table = DeltaTable("data/delta/supply/")

# Write 3 months (version 0)
df_first3 = df[df["order_month"].isin(["2016-09","2016-10","2016-11"])]
table.write(df_first3, mode="overwrite")

# Write next 3 months (version 1)
df_next3 = df[df["order_month"].isin(["2016-12","2017-01","2017-02"])]
table.write(df_next3, mode="append")

v0 = table.read(as_of_version=0)
v1 = table.read(as_of_version=1)
v_current = table.read()

print(f"Version 0: {len(v0):,} rows (first 3 months)")
print(f"Version 1: {len(v1):,} rows (6 months combined)")
print(f"Current:   {len(v_current):,} rows")
assert len(v0) < len(v1)  # time travel to earlier = fewer rows


# Q4: Delete with log
def delete(self, condition: Callable) -> int:
    df_current  = self.read()
    mask        = condition(df_current)
    df_delete   = df_current[mask]
    df_keep     = df_current[~mask]
    rows_deleted= len(df_delete)

    if rows_deleted == 0:
        return 0

    # Write kept rows as new file
    part_name = f"part-{self._version:08d}.parquet"
    part_path = self.path / part_name
    table     = pa.Table.from_pandas(df_keep)
    pq.write_table(table, part_path, compression="snappy")

    # Log: add new file, mark all previous files as removed
    existing_files = [f.name for f in self.path.glob("part-*.parquet")
                      if f.name != part_name]
    self._write_log("DELETE", {
        "rows_removed":  rows_deleted,
        "rows_kept":     len(df_keep),
        "file_added":    part_name,
        "files_removed": [{"path": f} for f in existing_files],
    })
    return rows_deleted

DeltaTable.delete = delete

table = DeltaTable("data/delta/supply/")
table.write(df, mode="overwrite")
rows_before = len(table.read())
deleted     = table.delete(lambda df: df["delivery_status"] == "Shipping canceled")
rows_after  = len(table.read())

print(f"Rows before: {rows_before:,}")
print(f"Deleted:     {deleted:,}")
print(f"Rows after:  {rows_after:,}")
assert rows_after == rows_before - deleted
```

---

*End of Data Engineering Projects*
*Chat #3 · June 2026 (v2 — Expanded Edition)*
