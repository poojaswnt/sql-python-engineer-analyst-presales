# Data Engineering 101 — A Complete Tutorial

> **How to use this document**
> Read end to end like a book. Every section builds on the previous one.
> All code runs with Python 3.8+ and standard data stack libraries.
> No API keys needed for any core example.
> Concept examples use 🎬 **Movies** throughout — same as SQL 101 and Python 101.
> Practice examples use the 7 Kaggle datasets from `de_projects.md`.
> SQL and Python equivalents are called out wherever they connect back to prior tutorials.
> Technology deep dives in Part IX give standalone reference for each major tool.
>
> **Chat #3 · June 2026 (v2 — Expanded Edition)**

---

## Table of Contents

### Part I — What Is Data Engineering?
1. [The Data Ecosystem — Roles, Tools, and Where DE Fits](#1-the-data-ecosystem)
2. [Data Pipeline Anatomy — What a Pipeline Actually Is](#2-data-pipeline-anatomy)
3. [Batch vs Streaming — The Fundamental Split](#3-batch-vs-streaming)
4. [The Modern Data Stack — A Map of the Toolscape](#4-the-modern-data-stack)

### Part II — Data Formats
5. [File Formats — CSV, JSON, Parquet, Avro, ORC, Delta, Iceberg](#5-file-formats)
6. [Compression, Partitioning & Storage Layout](#6-compression-partitioning-and-storage-layout)
7. [Schema Management and Evolution](#7-schema-management-and-evolution)

### Part III — Extract, Transform, Load
8. [Extract — Reading from Every Source](#8-extract)
9. [Transform — Cleaning, Joining, Business Logic](#9-transform)
10. [Load — Writing to Targets, All Patterns](#10-load)
11. [ETL vs ELT — When and Why](#11-etl-vs-elt)

### Part IV — Data Modelling for Pipelines
12. [Dimensional Modelling — Star Schema in Practice](#12-dimensional-modelling)
13. [Slowly Changing Dimensions — All Types in Code](#13-slowly-changing-dimensions)
14. [Data Vault — Architecture and Working Example](#14-data-vault)

### Part V — Orchestration
15. [DAGs and Workflow Concepts](#15-dags-and-workflow-concepts)
16. [Apache Airflow — Architecture, DAGs, Operators, Pitfalls](#16-apache-airflow)
17. [Prefect — Python-Native Orchestration](#17-prefect)
18. [Dagster — Asset-Oriented Orchestration](#18-dagster)
19. [Scheduling, Retries, Sensors, Monitoring](#19-scheduling-retries-dependencies)

### Part VI — Data Quality & Testing
20. [Why Data Quality Matters — and How to Measure It](#20-data-quality)
21. [Testing Pipelines — Unit, Integration, Contract Tests](#21-testing-pipelines)
22. [Data Lineage and Observability](#22-data-lineage-and-observability)

### Part VII — Warehouses and the Modern Stack
23. [Data Warehouses vs Data Lakes vs Lakehouses](#23-warehouses-vs-lakes-vs-lakehouses)
24. [DuckDB — Local In-Process Analytics](#24-duckdb)
25. [Snowflake — Cloud Warehouse Deep Dive](#25-snowflake)
26. [BigQuery — Google's Serverless Analytics Engine](#26-bigquery)
27. [dbt — Transform in the Warehouse](#27-dbt)

### Part VIII — Real Pipelines
28. [End-to-End Pipeline Walkthrough — Olist](#28-end-to-end-pipeline)
29. [Apache Spark — Architecture and Distributed Processing](#29-apache-spark)

### Part IX — Technology Deep Dives
30. [Apache Kafka — Event Streaming Platform](#30-kafka-deep-dive)
31. [Apache Spark — Architecture and Internals](#31-spark-deep-dive)
32. [Orchestration Deep Dive — Airflow, Prefect, Dagster Compared](#32-orchestration-deep-dive)
33. [Delta Lake and Apache Iceberg — Lakehouse Table Formats](#33-lakehouse-table-formats)
34. [dbt — How It Actually Works](#34-dbt-deep-dive)
35. [DuckDB — In-Process OLAP Engine Internals](#35-duckdb-deep-dive)
36. [Great Expectations — Data Quality Architecture](#36-great-expectations-deep-dive)
37. [Snowflake — Full Architecture Deep Dive](#37-snowflake-deep-dive)
38. [BigQuery — Full Architecture Deep Dive](#38-bigquery-deep-dive)

### Part X — Cloud Platforms for Data Engineering
39. [The Data Journey on Cloud Platforms — Service Map](#39-cloud-service-map)
40. [AWS for Data Engineering — Complete Service Guide](#40-aws)
41. [GCP for Data Engineering — Complete Service Guide](#41-gcp)
42. [Azure for Data Engineering — Complete Service Guide](#42-azure)
43. [Databricks — Unified Analytics Platform](#43-databricks)
44. [Cross-Cloud Comparison and Decision Framework](#44-cross-cloud-comparison)
45. [Data Engineering in Presales Conversations](#45-de-in-presales)

### Appendix
- [DE Tool Comparison Matrix](#appendix-a-tool-comparison-matrix)
- [pip Install Guide for DE](#appendix-b-pip-install-guide)
- [Glossary — 100+ Terms](#appendix-c-glossary)

---
---

## 1. The Data Ecosystem

### What does a data engineer actually do?

A **Data Engineer** builds and maintains the infrastructure that makes data usable. If a data analyst is the person who reads and interprets a book, the data engineer is the person who built the library — organised the shelves, ensured new books arrive on time every day, and verified that every book is the correct edition with no missing pages.

Concretely, a data engineer:
- Designs and builds **pipelines** — automated workflows that move data from sources to destinations
- Manages **data storage** — databases, data lakes, data warehouses, and the schemas within them
- Ensures **data quality** — data arriving downstream is correct, complete, and timely
- Creates the **infrastructure** that data analysts and data scientists depend on
- Thinks in systems — pipelines fail, schemas drift, sources go down; the job is to make the system resilient

---

### The data team roles — how they fit together

```
                Raw data sources
          (APIs, DBs, logs, files, events)
                        │
                        ▼
            ┌───────────────────────┐
            │   DATA ENGINEER       │  ← builds pipelines, models, infrastructure
            │   "moves and prepares │
            │    data reliably"     │
            └───────────┬───────────┘
                        │  clean, modelled, reliable data
                        ▼
            ┌───────────────────────────────────────┐
            │   DATA WAREHOUSE / DATA LAKE           │
            │   (BigQuery, Snowflake, S3+Parquet)   │
            └───────┬──────────────────────┬────────┘
                    │                      │
                    ▼                      ▼
        ┌──────────────────┐  ┌──────────────────────┐
        │  DATA ANALYST    │  │  DATA SCIENTIST /     │
        │  SQL + BI tools  │  │  ANALYTICS ENGINEER   │
        │  "what happened" │  │  "what will happen"   │
        └──────────────────┘  └──────────────────────┘
```

**The key insight about DE's role:** Data engineers are rarely the ones making business decisions from data. They are the enablers — every insight an analyst surfaces, every model a data scientist trains, depends entirely on whether the data engineer has built reliable, correct, timely pipelines. A 2% error in a pipeline can invalidate an entire quarter of reporting.

---

### Role comparison — what each role cares about

| Concern | Data Engineer | Data Analyst | Data Scientist |
|---------|--------------|--------------|----------------|
| Primary question | "Is the data correct, fresh, and accessible?" | "What happened and why?" | "What will happen? What should we do?" |
| Primary tools | Python, SQL, Spark, Airflow, dbt, cloud platforms | SQL, Python, Tableau, Power BI | Python, SQL, ML frameworks, notebooks |
| Thinks in | Pipelines, schemas, reliability, scale | Reports, dashboards, trends | Models, features, experiments |
| Success metric | Pipeline uptime, data freshness, row counts matching | Insight delivered, decision influenced | Model accuracy, experiment result |
| Career path | → Platform Engineer, Data Architect, ML Engineer | → Analytics Manager, BI Lead | → ML Engineer, Applied Scientist |

---

### The data journey — from raw to insight

🎬 **Movies analogy:** Think of a movie studio. Raw footage arrives from many cameras (source systems). Editors (data engineers) cut, clean, colour-grade, and assemble it into a final film (clean, modelled data). Directors (analysts) then use that film to tell a story. Without the editors, the footage is unusable. If the editors make a cut error, the story falls apart.

```
SOURCE SYSTEMS         INGESTION         STORAGE               SERVING
───────────────       ──────────         ─────────────         ──────────────
Olist orders DB ──┐                   ┌─ Raw Layer            ┌─ Analyst SQL
UK Retail CSVs  ──┤  Extract (E)  ────┤  (data lake,          │
Banking API     ──┤                   │   staging zone)       ├─ BI Dashboard
Healthcare EHR  ──┤  Transform (T) ───┤                       │
IoT sensors     ──┘                   ├─ Transformed Layer    ├─ ML feature store
                                      │  (cleaned, joined,    │
                                      │   business logic)     └─ Reports
                   Load (L)           │
                                      └─ Serving Layer
                                         (star schema DW,
                                          aggregated tables)
```

---

### Key terminology you need before anything else

| Term | Plain-English definition | Where it appears |
|------|--------------------------|-----------------|
| **Pipeline** | An automated sequence of steps: read data → transform it → write it somewhere | Everywhere in DE |
| **Ingestion** | The act of reading data from a source and bringing it into your system | First step |
| **Transformation** | Cleaning, reshaping, joining, applying business logic | Middle step |
| **Data Lake** | Raw storage — all formats accepted, cheap, schema applied at read time (S3, GCS) | Storage layer |
| **Data Warehouse** | Structured SQL-queryable storage optimised for analytics (BigQuery, Snowflake) | Serving layer |
| **Data Lakehouse** | Hybrid: lake storage + warehouse query semantics + ACID transactions (Delta Lake, Iceberg) | Modern pattern |
| **ETL** | Extract → Transform → Load: transform before loading | Traditional pattern |
| **ELT** | Extract → Load → Transform: load raw, transform inside the warehouse | Modern pattern |
| **Orchestration** | Scheduling and coordinating when pipeline steps run and in what order | Operational layer |
| **Idempotency** | Running a pipeline twice produces the same result as running it once | Critical property |
| **Backfill** | Re-running a pipeline for historical dates it missed | Recovery operation |
| **SLA** | Service Level Agreement — "data must be ready by 6am" | Operational commitment |
| **Lineage** | A record of where data came from and every transformation it went through | Governance/debugging |
| **Schema** | The structure of data: column names, types, constraints | Data contract |
| **Grain** | What one row in a table represents — the most important question to ask of any table | Data modelling |

---

## 2. Data Pipeline Anatomy

### What is a pipeline, really?

A **data pipeline** is an automated program that:
1. Reads data from one or more **sources** (databases, APIs, files, event streams)
2. Applies **transformations** (cleaning, joining, aggregating, applying business logic)
3. Writes to a **destination** (database table, Parquet file, data warehouse, downstream API)
4. Is triggered on a **schedule or event**
5. Has **error handling**, **logging**, and **monitoring** built in

The word "pipeline" is used loosely — it could be a 20-line Python script or a 50-step distributed Spark job. What makes it a pipeline is that it runs automatically, reliably, and produces a predictable output.

---

### Pipeline components — anatomy of a real pipeline

```python
# A minimal but complete pipeline — all concepts visible
from pathlib import Path
import pandas as pd
from datetime import date, timedelta
import logging

logger = logging.getLogger(__name__)

# ── COMPONENT 1: CONFIGURATION ────────────────────────────────────────────
# Never hardcode paths, dates, or environment-specific values
# A real pipeline accepts these as parameters or reads from config files

def get_config(run_date: date = None) -> dict:
    run_date = run_date or date.today() - timedelta(days=1)
    return {
        "run_date":    run_date,
        "source_path": f"data/raw/orders/{run_date}.csv",
        "output_path": f"data/warehouse/orders/date={run_date}/orders.parquet",
        "min_rows":    10,        # quality gate
    }

# ── COMPONENT 2: EXTRACT ──────────────────────────────────────────────────
# Read data from source; handle errors; return raw DataFrame

def extract(config: dict) -> pd.DataFrame:
    path = Path(config["source_path"])
    if not path.exists():
        raise FileNotFoundError(f"Source missing: {path}")
    df = pd.read_csv(path)
    logger.info(f"Extracted {len(df):,} rows from {path.name}")
    return df

# ── COMPONENT 3: VALIDATE ─────────────────────────────────────────────────
# Check raw data before doing any work on it
# Fail fast: better to fail here than discover corruption after loading

def validate(df: pd.DataFrame, config: dict) -> None:
    if len(df) < config["min_rows"]:
        raise ValueError(f"Only {len(df)} rows — expected ≥ {config['min_rows']}")
    required = ["order_id", "customer_id", "order_date", "revenue"]
    missing  = [c for c in required if c not in df.columns]
    if missing:
        raise ValueError(f"Missing required columns: {missing}")
    if df["order_id"].duplicated().sum() > 0:
        raise ValueError("Duplicate order_ids in source")
    logger.info("Validation passed")

# ── COMPONENT 4: TRANSFORM ────────────────────────────────────────────────
# Apply business logic; produce analytics-ready data
# Pure function: same input always gives same output (no side effects)

def transform(df: pd.DataFrame) -> pd.DataFrame:
    df = df.copy()                                          # never mutate the input
    df["order_date"]   = pd.to_datetime(df["order_date"])
    df["order_month"]  = df["order_date"].dt.to_period("M").astype(str)
    df["revenue"]      = pd.to_numeric(df["revenue"], errors="coerce")
    df["is_high_value"]= (df["revenue"] > 500).astype(int)
    logger.info(f"Transformed {len(df):,} rows")
    return df

# ── COMPONENT 5: LOAD ─────────────────────────────────────────────────────
# Write to destination; must be idempotent

def load(df: pd.DataFrame, config: dict) -> None:
    path = Path(config["output_path"])
    path.parent.mkdir(parents=True, exist_ok=True)
    df.to_parquet(path, index=False, compression="snappy")
    logger.info(f"Loaded {len(df):,} rows → {path}")

# ── COMPONENT 6: ORCHESTRATION ────────────────────────────────────────────
# Tie all steps together; handle failures; log the run

def run_pipeline(run_date: date = None) -> bool:
    config = get_config(run_date)
    logger.info(f"Pipeline start: {config['run_date']}")

    try:
        df_raw   = extract(config)
        validate(df_raw, config)
        df_clean = transform(df_raw)
        load(df_clean, config)
        logger.info("Pipeline complete: SUCCESS")
        return True
    except Exception as e:
        logger.error(f"Pipeline FAILED: {e}")
        return False

if __name__ == "__main__":
    run_pipeline()
```

---

### Pipeline properties — what separates a good pipeline from a bad one

| Property | What it means | Consequence of missing it |
|----------|--------------|--------------------------|
| **Idempotent** | Running twice = same result as running once | Re-runs after failures double-count data |
| **Atomic** | Either fully succeeds or fully rolls back | Half-written data corrupts downstream |
| **Incremental** | Only processes new/changed data | Full-reload of 2 years of history daily at 3am |
| **Observable** | Logs, metrics, and alerts on failures | You don't know something broke until an analyst complains |
| **Testable** | Each step can be tested in isolation | Bugs hide until they hit production |
| **Parameterised** | Accepts date/config as input, no hardcoded values | Can't rerun for any historical date (backfill impossible) |
| **Self-healing** | Retries transient failures automatically | One network blip kills the whole pipeline |

---

### The data pipeline landscape — tools by category

Before going further, here is a map of the tools you will encounter. Everything in this tutorial fits into one of these categories:

```
PIPELINE CATEGORY      TOOLS (sorted by adoption)
─────────────────────────────────────────────────────────────────────────
File Formats           Parquet · CSV · JSON/JSONL · Avro · ORC
                       Delta Lake · Apache Iceberg

Compute / Processing   pandas · PySpark (Apache Spark) · DuckDB
                       Polars · Dask · Flink

Workflow Orchestration Apache Airflow · Prefect · Dagster
                       dbt Cloud · AWS Step Functions

Transformation         dbt · Python/pandas · Spark SQL
                       SQL (in-warehouse)

Ingestion Connectors   Fivetran · Airbyte · Stitch
                       Custom Python scripts

Data Quality           Great Expectations · dbt tests
                       Pandera · Monte Carlo (observability)

Storage — Lake         Amazon S3 · Google Cloud Storage (GCS)
                       Azure Data Lake Storage Gen2 (ADLS)

Storage — Warehouse    Snowflake · Google BigQuery · Amazon Redshift
                       Azure Synapse · Databricks (Delta Lake)
                       DuckDB (local) · PostgreSQL (small scale)

Streaming              Apache Kafka · Apache Flink
                       AWS Kinesis · Google Pub/Sub · Azure Event Hubs

Serving / BI           Tableau · Power BI · Looker · Metabase
                       Apache Superset · Evidence
```

> **Don't panic at this list.** You don't need to know all of them. You need to understand the *category* each tool belongs to, so when a client or colleague says "we use Databricks for processing and Fivetran for ingestion" you understand the architecture without knowing those tools deeply.

---

## 3. Batch vs Streaming

### The fundamental split — explained properly

Every data pipeline is either batch or streaming (or a hybrid). The choice is not about technology preference — it is about answering the question: **how quickly does a decision need to be made after data is generated?**

**Batch processing:** collect data over a period, then process it all at once.
**Streaming processing:** process data event-by-event as it arrives.

```
BATCH                                      STREAMING
────────────────────────────────           ───────────────────────────────────
"Process yesterday's orders at 2am"        "Process each transaction as it lands"

Time to result: hours                      Time to result: milliseconds to seconds
Latency:        high (hours)               Latency:        low (ms)
Complexity:     low                        Complexity:     high
Cost:           lower (can use spot VMs)   Cost:           higher (always running)
Failure mode:   rerun yesterday's job      Failure mode:   events may be lost/duplicated
Use for:        most analytics             Use for:        fraud detection, live alerts

🎬 Movie analogy:
Batch   = editing a film offline in a studio, released when done
Stream  = broadcasting live TV — every frame processed the moment the camera captures it
```

---

### When you genuinely need streaming (and when you don't)

This is a judgment call that trips up many early-career engineers. Streaming is technically impressive but expensive and complex. Most analytics problems do not need it.

**Use streaming when:**
- A business decision must be made **in under a minute** of the event happening
- Fraud detection — if you wait an hour to detect a fraudulent transaction, the money is gone
- Live operational dashboards — "how many tickets are being processed right now?"
- Alerting — "notify the on-call engineer the moment error rate exceeds 5%"
- IoT sensor data — machine temperature must trigger an alert before equipment fails

**Batch is perfectly fine when:**
- Revenue reports are refreshed daily — nobody needs yesterday's revenue at 11:59pm
- Customer segmentation runs weekly — RFM scores don't need to update by the minute
- ML model training — models are retrained on batches of historical data
- Finance reconciliation — month-end reports are inherently batch

> **Rule of thumb:** If the word "yesterday" or "daily" appears in the requirement, use batch. If the word "real-time", "immediately", or "as it happens" appears, investigate whether streaming is truly needed — often "hourly batch" satisfies the actual business need at 10% of the complexity.

---

### Micro-batch — the middle ground

Most real-world "streaming" systems are actually **micro-batch**: small batches processed every few minutes. Apache Spark's Structured Streaming and Kafka Streams both offer this model. It gives near-real-time latency with batch-like simplicity.

```python
# Batch: run once per day, process 24 hours of data
def daily_batch(run_date):
    df = pd.read_parquet(f"data/orders/date={run_date}/")
    result = df.groupby("category")["revenue"].sum()
    load_to_warehouse(result, run_date)

# Micro-batch: run every 5 minutes, process last 5 minutes of data
import time

def run_micro_batch_loop(interval_seconds: int = 300):
    """
    Simulates a micro-batch pipeline.
    In production, Spark Structured Streaming or Flink handles this.
    """
    while True:
        batch_start = time.time()
        end_time    = pd.Timestamp.now()
        start_time  = end_time - pd.Timedelta(seconds=interval_seconds)

        # Only read events in the last interval
        df = read_events_between(start_time, end_time)

        if len(df) > 0:
            result = process_micro_batch(df, end_time)
            append_to_warehouse(result)
            print(f"Micro-batch: {len(df)} events in "
                  f"{time.time()-batch_start:.1f}s")

        # Sleep until next interval
        time.sleep(max(0, interval_seconds - (time.time() - batch_start)))
```

---

### Key streaming concepts — for conversations and interviews

Even if you rarely build streaming pipelines yourself, you will encounter these terms constantly. Understanding them is important for presales and architectural conversations.

| Concept | What it means | Why it matters |
|---------|---------------|----------------|
| **Event** | A single unit of data with a timestamp | The atomic unit of streaming |
| **Stream** | An unbounded (never-ending) sequence of events | Contrast with batch: a finite file |
| **Topic** | In Kafka: a named category for a stream of events | "orders", "payments", "fraud-alerts" |
| **Partition** | A topic is split into partitions for parallelism | More partitions = more throughput |
| **Offset** | A cursor: where a consumer is in a partition | Enables replay and recovery |
| **Consumer group** | A set of consumers that share processing of a topic | Scale out: each consumer handles different partitions |
| **Producer** | A service that writes events to a stream | The order service publishes to "orders" topic |
| **Consumer** | A service that reads and processes events | The fraud detection service reads "orders" |
| **Lag** | How far behind a consumer is from the latest event | High lag = consumer can't keep up with producer |
| **Watermark** | How late an event can arrive and still be included | Accept events up to 2 minutes late |
| **Windowing** | Grouping events by time: tumbling, sliding, session | "Sum sales in the last 5 minutes" |
| **Exactly-once** | Each event is processed exactly once — not zero, not twice | Critical for financial data |
| **At-least-once** | Event is processed ≥ once — duplicates possible | Acceptable for log analytics |
| **At-most-once** | Event may be dropped but never duplicated | Acceptable for sampling |
| **Backpressure** | When a consumer is overwhelmed, signal the producer to slow down | Prevents memory overflow |

---

### Streaming tools — what each one is

**Apache Kafka** (covered in depth in Chapter 26) is the dominant event streaming platform. Think of it as a distributed, durable, high-throughput message queue. Producers write events; consumers read them. Events are retained for a configurable period (e.g. 7 days) so consumers can replay history.

**Apache Flink** is a distributed stream processing framework. Where Kafka stores and delivers events, Flink processes them — joins, aggregations, windowed computations. Flink can also do batch processing. Used at companies like Alibaba, Uber, Netflix.

**Spark Structured Streaming** extends Spark's DataFrame API to continuous streams. You write the same code you'd write for batch, but it runs incrementally as new data arrives. Good choice if your team already knows PySpark.

**AWS Kinesis / Google Pub/Sub / Azure Event Hubs** are managed cloud equivalents of Kafka — you don't manage brokers, partitions, or replication yourself. Trade-off: less control, but no operational overhead.

---

## 4. The Modern Data Stack

### What is the Modern Data Stack (MDS)?

The "Modern Data Stack" is a term for a collection of cloud-native, typically SaaS tools that work together to move, store, transform, and visualise data. It emerged as cloud object storage became cheap, compute became elastic, and SQL became the transformation language of choice (via dbt).

The defining characteristics:
- **Cloud-native** — no servers to manage, scales automatically
- **SQL-centric transformation** — dbt displaced Python for most warehouse-layer transforms
- **Separation of storage and compute** — you pay for data stored (cheap) and compute used (on-demand)
- **Modular** — each layer uses the best tool for the job, not a monolithic platform

---

### The MDS layers — each tool explained

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      MODERN DATA STACK                                   │
├─────────────────┬──────────────────┬──────────────────┬─────────────────┤
│   INGEST        │   STORE          │   TRANSFORM      │   SERVE         │
│                 │                  │                  │                 │
│ Fivetran        │ Snowflake         │ dbt              │ Looker          │
│ Airbyte         │ BigQuery         │ Python/Spark     │ Tableau         │
│ Stitch          │ Redshift         │ SQL              │ Power BI        │
│ Custom Python   │ Databricks       │                  │ Metabase        │
│                 │ S3 / GCS / ADLS  │                  │ Evidence        │
├─────────────────┴──────────────────┴──────────────────┴─────────────────┤
│                         ORCHESTRATION                                    │
│              Airflow · Prefect · Dagster · dbt Cloud                     │
├──────────────────────────────────────────────────────────────────────────┤
│                    DATA QUALITY & OBSERVABILITY                           │
│               Great Expectations · dbt tests · Monte Carlo               │
└──────────────────────────────────────────────────────────────────────────┘
```

**Ingestion layer — getting data in:**

| Tool | What it is | When to use |
|------|-----------|-------------|
| **Fivetran** | Managed SaaS connector service. 200+ pre-built connectors (Salesforce, Stripe, Google Analytics, etc.). Handles schema changes automatically. Expensive but zero maintenance. | When you're connecting to common SaaS tools and want to move fast |
| **Airbyte** | Open-source Fivetran alternative. Self-hosted. 300+ connectors. More control, more maintenance. | When you need custom connectors or have cost constraints |
| **Stitch** | Lighter-weight managed connector service. Fewer connectors than Fivetran but cheaper. | Small-to-medium teams connecting common sources |
| **Custom Python** | Write your own extraction scripts using `requests`, `SQLAlchemy`, `pandas`. | Internal databases, unusual APIs, sources no connector supports |

**Storage layer — where data lives:**

| Tool | What it is | When to use |
|------|-----------|-------------|
| **Snowflake** | Cloud data warehouse. Separates storage (S3 under the hood) from compute (virtual warehouses). SQL-native. Multi-cloud. | The most popular choice for mid-to-large analytics teams |
| **BigQuery** | Google's serverless data warehouse. No infrastructure to manage. Pay per query. Extremely fast for large-scale analytics. | GCP shops; when you want zero infrastructure management |
| **Amazon Redshift** | AWS's managed data warehouse. Tightly integrated with S3, Glue, and the AWS ecosystem. | AWS shops; especially when data is already in S3 |
| **Databricks** | Unified platform built on Apache Spark + Delta Lake. Bridges data engineering and ML. | When you have both analytics and ML workloads; when Spark skills are strong |
| **S3 / GCS / ADLS** | Cloud object storage — cheap, durable, scalable. The "floor" of the data lake. | Every cloud architecture uses one of these as the raw storage layer |

**Transformation layer — making data useful:**

| Tool | What it is | When to use |
|------|-----------|-------------|
| **dbt** | Transformation framework using SQL. You write SELECT statements; dbt handles CREATE TABLE, dependency resolution, testing, and documentation. | Standard choice for warehouse-layer transformations |
| **Python / pandas** | Traditional scripted transforms. Flexible, powerful, no SQL required. | Complex transforms SQL can't do; pre-warehouse cleaning |
| **PySpark** | Distributed Python transforms. Same pandas-like API but runs across a cluster. | Transforms on data > 50GB that won't fit on one machine |

**Orchestration layer — running pipelines on schedule:**

| Tool | What it is | When to use |
|------|-----------|-------------|
| **Apache Airflow** | Open-source workflow orchestration. DAGs defined in Python. Most widely used. | Most enterprises; large community; extensive operator library |
| **Prefect** | Python-native orchestration. Easier to write than Airflow; better UI; cloud option available. | Python-first teams; when Airflow feels too heavy |
| **Dagster** | Asset-oriented orchestration. Instead of "run this task", you define "produce this data asset". Best observability. | When data lineage and asset tracking matter; ML-heavy teams |
| **dbt Cloud** | Managed scheduling for dbt models. Built-in CI/CD for data transforms. | When your transforms are 100% dbt; simplest option for dbt teams |

---

### MDS by maturity stage

You don't need everything at once. Here's how to think about tooling as you grow:

| Stage | Team size | Typical stack | Monthly cost range |
|-------|-----------|--------------|-------------------|
| **Solo analyst** | 1 person | DuckDB + dbt + local Python + Metabase | $0–$50 |
| **Small team** | 2–5 people | PostgreSQL/Redshift + dbt + Airflow (self-hosted) + Metabase | $200–$1,000 |
| **Growing** | 5–20 people | Snowflake/BigQuery + dbt Cloud + Airflow (managed) + Looker | $2,000–$10,000 |
| **Scale** | 20+ people | Databricks or full managed cloud stack | $10,000+ |

> **Key insight for presales conversations:** Clients often want to jump straight to the "scale" stack because it sounds impressive. The right answer is almost always: start simpler and grow. The tools at the "growing" stage are the same tools used by most mid-market companies.

---
---

## 5. File Formats

### Why file format is one of the most important decisions in DE

Choosing the wrong file format is one of the most impactful performance and cost mistakes a data engineer can make — and one of the most common. A query that takes 4 minutes on CSV might take 3 seconds on Parquet. The same dataset might occupy 500MB as CSV and 40MB as Parquet with Zstd compression.

The decision is not "what's the best format?" — it's "what format fits the access pattern, the tooling, and the team?"

🎬 **Movies analogy:** The same film can be stored as raw 8K ProRes footage (huge, uncompressed, for editing), H.264 MP4 (compressed, universal, for distribution), or a 480p stream (tiny, lossy, for preview). Same content, wildly different characteristics — the right choice depends on what you're doing with it.

---

### CSV — the universal format

**What it is:** Plain text, rows separated by newlines, columns separated by a delimiter (usually comma). Human-readable. Understood by literally every tool that exists.

**How it stores data:**
```
order_id,customer_id,amount,status,order_date
O001,C01,150.50,delivered,2024-01-15
O002,C02,80.00,shipped,2024-01-15
O003,C01,250.00,delivered,2024-01-16
```

Every cell is stored as text. The database or tool reading it must infer or be told what type each column is. `"150.50"` is a string until you parse it as a float. `"2024-01-15"` is a string until you parse it as a date.

**When to use CSV:**
- Data exchange with external systems, partners, or humans — it's the lingua franca
- Small files (< 100k rows) where performance doesn't matter
- When the recipient system only accepts CSV
- Quick one-off scripts and analysis

**When NOT to use CSV:**
- Large files (> 1M rows) — it's slow to read and large on disk
- When schema matters — types are not stored, must be re-inferred every read
- When you need column pruning — to read one column you must scan the entire file
- Production pipelines — switch to Parquet

```python
import pandas as pd

# Reading CSV — all values start as strings
df = pd.read_csv("orders.csv")
print(df.dtypes)
# order_id      object   ← string (even though it should be int or str ID)
# customer_id   object   ← string
# amount        float64  ← pandas inferred this one correctly
# status        object   ← string, correct
# order_date    object   ← STRING — not a date! must parse manually

# Always specify types explicitly for production CSV reads
df = pd.read_csv(
    "orders.csv",
    dtype={
        "order_id":    str,      # preserve leading zeros, prevent int conversion
        "customer_id": str,
        "amount":      float,
    },
    parse_dates=["order_date"],  # parse dates during read, not after
    na_values=["", "N/A", "null", "NULL"],  # standardise nulls
)
```

---

### JSON and JSONL — for nested and semi-structured data

**What JSON is:** JavaScript Object Notation. A text format for representing structured data with key-value pairs, arrays, and nesting. Every REST API returns JSON. Every webhook sends JSON. You cannot avoid it.

**Two flavours:**

**JSON** — a single document (one big object or array):
```json
[
  {"order_id": "O001", "customer": {"id": "C01", "city": "São Paulo"}, "items": [{"sku": "A1", "qty": 2}]},
  {"order_id": "O002", "customer": {"id": "C02", "city": "Rio"},       "items": [{"sku": "B2", "qty": 1}]}
]
```

**JSONL (JSON Lines)** — one JSON object per line:
```
{"order_id": "O001", "amount": 150.50, "status": "delivered"}
{"order_id": "O002", "amount": 80.00,  "status": "shipped"}
{"order_id": "O003", "amount": 250.00, "status": "delivered"}
```

JSONL is better for data engineering because:
- You can append a new line without rewriting the whole file
- You can read line by line (streaming) without loading the whole file into memory
- It handles large files naturally
- Kafka messages are usually JSONL under the hood

```python
import json
import pandas as pd

# ── READING JSON ──────────────────────────────────────────────────────────
# Simple array of objects → DataFrame directly
df = pd.read_json("orders.json")

# JSONL (one object per line)
df = pd.read_json("orders.jsonl", lines=True)

# ── REAL-WORLD: flattening nested JSON ────────────────────────────────────
# API responses are almost always nested — this is one of the most
# common real-world DE tasks

raw_api_response = [
    {
        "order_id": "O001",
        "customer": {"id": "C01", "city": "São Paulo", "state": "SP"},
        "items": [
            {"sku": "A1", "qty": 2, "price": 50.0},
            {"sku": "B2", "qty": 1, "price": 80.0}
        ],
        "payment": {"method": "credit_card", "total": 180.0}
    },
    {
        "order_id": "O002",
        "customer": {"id": "C02", "city": "Rio", "state": "RJ"},
        "items": [{"sku": "C3", "qty": 3, "price": 30.0}],
        "payment": {"method": "boleto", "total": 90.0}
    }
]

# Flatten top-level nested dicts (customer, payment)
# sep="_" means "customer.id" becomes "customer_id"
df_orders = pd.json_normalize(raw_api_response, sep="_")
# Result: order_id | customer_id | customer_city | customer_state |
#         payment_method | payment_total
# BUT: "items" is still a list column

# Explode the items array (one row per item)
df_items = pd.json_normalize(
    raw_api_response,
    record_path="items",             # the array to explode
    meta=["order_id",                # fields to carry over from parent
          ["customer", "id"],
          ["payment", "total"]],
    sep="_"
)
# Result: one row per item
# order_id | customer_id | payment_total | sku | qty | price
```

---

### Parquet — the analytics standard

**What Parquet is:** A binary, columnar, compressed file format designed specifically for analytical workloads. Developed by Twitter and Cloudera, now Apache open-source. The default format for data lakes, Spark, BigQuery, Snowflake, Athena, and virtually every modern analytics tool.

**The columnar storage concept — this is why Parquet is fast:**

Analytical queries almost always read a few columns across many rows. "What is the total revenue by category?" reads `revenue` and `category` from 10 million rows — it doesn't touch `customer_name`, `address`, `phone`, etc.

**Row storage (CSV, relational DB row store):**
```
Row 1: │ order_id=O001 │ customer=C01 │ date=2024-01-15 │ amount=150 │ status=delivered │
Row 2: │ order_id=O002 │ customer=C02 │ date=2024-01-15 │ amount=80  │ status=shipped   │
Row 3: │ order_id=O003 │ customer=C01 │ date=2024-01-16 │ amount=250 │ status=delivered │

To answer "SUM(amount) WHERE status = 'delivered'":
→ Must read EVERY row's EVERY column to extract amount and status
→ I/O cost: all bytes on disk
```

**Columnar storage (Parquet):**
```
order_id col:  │ O001 │ O002 │ O003 │ ...
customer col:  │ C01  │ C02  │ C01  │ ...
date col:      │ 2024-01-15 │ 2024-01-15 │ 2024-01-16 │ ...
amount col:    │ 150  │ 80   │ 250  │ ...   ← READ ONLY THIS
status col:    │ delivered │ shipped │ delivered │ ...   ← AND THIS

To answer "SUM(amount) WHERE status = 'delivered'":
→ Read ONLY the amount and status columns
→ Skip order_id, customer, date entirely
→ I/O cost: 2 columns instead of 5 = 60% less reading
→ At scale (100 columns, 1 billion rows): 99% less reading
```

**Additional Parquet features:**
- **Row groups** — data is physically divided into groups of rows. Each group stores min/max statistics per column. A query like `WHERE date > '2024-06-01'` can skip entire row groups whose max date is earlier.
- **Dictionary encoding** — repeated values (like `status="delivered"`) are stored once and referenced by integer ID. Massive space saving for low-cardinality columns.
- **Schema embedded** — the file knows its own column names and types. No external schema file needed.
- **Splittable** — a Parquet file can be read by multiple workers in parallel (each reads a row group).

```python
import pandas as pd
import pyarrow as pa
import pyarrow.parquet as pq
from pathlib import Path
import time

# ── WRITING PARQUET ───────────────────────────────────────────────────────
df = pd.read_csv("orders.csv", parse_dates=["order_date"])

# Simple write (default snappy compression)
df.to_parquet("orders.parquet", index=False)

# Choose compression codec
df.to_parquet("orders_snappy.parquet", compression="snappy")    # fast, moderate compression
df.to_parquet("orders_gzip.parquet",   compression="gzip")      # smaller file, slower write
df.to_parquet("orders_zstd.parquet",   compression="zstd")      # best ratio + speed (modern default)
df.to_parquet("orders_none.parquet",   compression=None)        # no compression (debugging)

# ── READING — column pruning ───────────────────────────────────────────────
# Read only the columns you need — the rest never leave disk
df_full = pd.read_parquet("orders.parquet")                      # all columns
df_slim = pd.read_parquet("orders.parquet",
                           columns=["order_id", "amount", "status"])  # 3 columns only

# ── READING — row filtering (predicate pushdown) ───────────────────────────
# Filters are pushed down to the file layer — row groups are skipped
df_jan = pd.read_parquet(
    "orders.parquet",
    filters=[("order_date", ">=", "2024-01-01"),
             ("order_date", "<",  "2024-02-01")]
)

# ── PARTITIONED PARQUET ───────────────────────────────────────────────────
# The most important Parquet pattern for production pipelines
df["year"]  = df["order_date"].dt.year
df["month"] = df["order_date"].dt.month

# Write partitioned dataset (creates a directory tree)
pq.write_to_dataset(
    pa.Table.from_pandas(df),
    root_path="data/warehouse/orders/",
    partition_cols=["year", "month"],       # columns to partition on
    compression="snappy",
    existing_data_behavior="delete_matching",  # idempotent: overwrites partition on rerun
)

# Creates:
# data/warehouse/orders/year=2024/month=1/part-00000000.parquet
# data/warehouse/orders/year=2024/month=2/part-00000000.parquet
# data/warehouse/orders/year=2023/month=12/part-00000000.parquet

# Now this query reads ONLY the year=2024/month=1/ directory:
df_jan24 = pd.read_parquet(
    "data/warehouse/orders/",
    filters=[("year", "==", 2024), ("month", "==", 1)]
)

# ── QUERYING PARQUET WITH DUCKDB ──────────────────────────────────────────
import duckdb
con = duckdb.connect()

# DuckDB can query Parquet files directly with full SQL
result = con.execute("""
    SELECT
        year,
        month,
        COUNT(*) AS orders,
        SUM(amount) AS revenue
    FROM read_parquet('data/warehouse/orders/**/*.parquet')
    WHERE year = 2024
    GROUP BY year, month
    ORDER BY year, month
""").df()
```

---

### Avro — for streaming and schema evolution

**What Avro is:** A binary, row-based format with schema embedded in the file. Developed by the Apache Hadoop project, Avro is the dominant format for Kafka messages and event streaming systems.

**Why Avro exists (the streaming problem Parquet doesn't solve):**

In streaming, events are written one at a time, not in bulk. Parquet is optimised for bulk analytical writes (it buffers rows into "row groups" before writing). Writing to Parquet row-by-row is very inefficient.

Avro is designed for row-by-row streaming writes while also handling the schema evolution problem elegantly.

**Schema evolution — why it matters:**

When a Kafka producer adds a new field to its messages, existing consumers (reading old messages) must not break. Avro's schema registry model solves this:
- **Schema Registry** (e.g., Confluent Schema Registry): a central store for all schema versions
- Producers register their schema when publishing; consumers fetch the schema when reading
- Backward compatibility: new optional fields → old consumers still work (they ignore the new field)
- Forward compatibility: readers with a newer schema can read data written with an older schema

```python
# pip install fastavro
import fastavro
import io

# Define Avro schema
SCHEMA = {
    "type": "record",
    "name": "Order",
    "namespace": "com.olist.orders",
    "fields": [
        {"name": "order_id",   "type": "string"},
        {"name": "amount",     "type": "double"},
        {"name": "status",     "type": "string"},
        {"name": "timestamp",  "type": "long",   "logicalType": "timestamp-millis"},
        # Adding a new optional field (backward compatible — existing records still valid)
        {"name": "currency",   "type": ["null", "string"], "default": None},
    ]
}

# Write Avro
records = [
    {"order_id": "O001", "amount": 150.0, "status": "delivered",
     "timestamp": 1700000000000, "currency": "BRL"},
    {"order_id": "O002", "amount": 80.0,  "status": "shipped",
     "timestamp": 1700000100000, "currency": None},
]
with open("orders.avro", "wb") as f:
    fastavro.writer(f, fastavro.parse_schema(SCHEMA), records)

# Read Avro
with open("orders.avro", "rb") as f:
    reader = fastavro.reader(f)
    for record in reader:
        print(record)
```

**Avro vs Parquet — the decision:**
```
Use Avro when:                          Use Parquet when:
- Writing to Kafka topics               - Writing to data lakes
- Streaming row-by-row inserts          - Bulk analytical writes
- Schema evolution is critical          - Query performance is critical
- Message serialisation for APIs        - Reading subsets of columns
- Interoperability in streaming         - Columnar compression benefits needed
  pipelines (Kafka → Flink → Sink)
```

---

### ORC — Oracle Row Columnar

**What ORC is:** A columnar binary format, similar to Parquet, developed for the Apache Hive ecosystem. ORC was the dominant format before Parquet became the standard.

**ORC vs Parquet:**

| Feature | Parquet | ORC |
|---------|---------|-----|
| Ecosystem | Universal (Spark, BigQuery, pandas, DuckDB, Athena) | Hive, Spark, Presto — primarily |
| Compression | Snappy, Gzip, Zstd, LZ4 | Zlib, Snappy, LZ4 |
| Nested types | Supported | Supported |
| ACID support | Via Delta/Iceberg | Native ACID in Hive 3+ |
| Tool support | Wider | Narrower (Hadoop-centric) |
| Default choice | Yes (for modern stacks) | Only when Hive/Hadoop ecosystem requires it |

> **Practical rule:** Always default to Parquet unless you're in a Hive-native environment or a legacy Hadoop stack that requires ORC.

---

### Delta Lake and Apache Iceberg — table formats for data lakehouses

This is where file formats get interesting. Parquet, Avro, and ORC are file formats — they define how data is encoded on disk. **Delta Lake and Apache Iceberg are table formats** — they sit on top of Parquet files and add:

1. **ACID transactions** — multiple writers don't corrupt each other; reads never see partial writes
2. **Time travel** — query data as it was at any previous point in time
3. **Schema evolution** — change column names/types safely, with history
4. **Efficient upserts** — merge new data with existing data without rewriting everything
5. **Partition evolution** — change partitioning strategy without rewriting the whole dataset

**The problem they solve:**

```
Without Delta/Iceberg (plain Parquet):

Writer 1 starts writing to year=2024/month=01/part-00.parquet
Writer 2 starts reading year=2024/month=01/ at the same time
→ Reader sees PARTIAL data — Writer 1 hasn't finished yet
→ This is a race condition — data corruption in production

With Delta Lake:
→ Writer 1 writes to a TEMP location first
→ Only after write is complete, the transaction log is updated atomically
→ Reader 2 sees EITHER the old complete state OR the new complete state
→ Never partial writes. ACID guaranteed.
```

**Delta Lake transaction log:**
```
data/warehouse/orders/
  part-00000001.parquet   ← data files
  part-00000002.parquet
  part-00000003.parquet
  _delta_log/
    00000000000000000001.json   ← "Added part-00000001.parquet (100 rows)"
    00000000000000000002.json   ← "Added part-00000002.parquet (150 rows)"
    00000000000000000003.json   ← "Deleted part-00000001.parquet (re-written)"
```

Every write, delete, or update is recorded in the log. To "time travel" to version 1, you just read the files referenced by log entry 1.

```python
# Delta Lake with PySpark (pip install delta-spark)
from pyspark.sql import SparkSession

spark = (SparkSession.builder
         .config("spark.jars.packages", "io.delta:delta-core_2.12:2.4.0")
         .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension")
         .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")
         .getOrCreate())

# Write Delta table
df.write.format("delta").mode("overwrite").save("data/warehouse/orders_delta/")

# Time travel — read as of a specific version
df_v1 = spark.read.format("delta") \
             .option("versionAsOf", 1) \
             .load("data/warehouse/orders_delta/")

# Time travel — read as of a specific timestamp
df_yesterday = spark.read.format("delta") \
                    .option("timestampAsOf", "2024-01-14") \
                    .load("data/warehouse/orders_delta/")

# MERGE (upsert) — update existing rows, insert new ones
from delta.tables import DeltaTable

target = DeltaTable.forPath(spark, "data/warehouse/orders_delta/")
target.alias("target").merge(
    df_updates.alias("source"),
    "target.order_id = source.order_id"  # join condition
).whenMatchedUpdateAll() \
 .whenNotMatchedInsertAll() \
 .execute()
```

**Delta Lake vs Apache Iceberg:**

| Feature | Delta Lake | Apache Iceberg |
|---------|-----------|----------------|
| Origin | Databricks (2019) | Netflix (2018), now Apache |
| Primary engine | Databricks, Spark | Multiple (Spark, Flink, Trino, BigQuery) |
| Catalog support | Databricks Unity Catalog; Hive | Hive, Glue, REST (Nessie), Snowflake |
| Engine neutrality | Works best with Databricks | Truly engine-agnostic |
| Time travel | Yes | Yes |
| Schema evolution | Yes | Yes (more flexible) |
| Partition evolution | Via OPTIMIZE | Native, no rewrite |
| Adoption | Dominant in Databricks shops | Growing fast; multi-cloud favourite |
| Community | Large, Databricks-backed | Apache, multi-company |

> **Rule of thumb:** If you use Databricks, use Delta Lake. If you need to read the same data from Spark, Flink, Trino, and BigQuery without picking a winner, use Iceberg.

---

### Format comparison — the complete picture

| Format | Type | Schema stored? | Compression | Column pruning | Best for |
|--------|------|---------------|-------------|----------------|---------|
| **CSV** | Row, text | No | Manual (gzip) | No | Data exchange, small files |
| **JSON** | Row, text | No | Manual (gzip) | No | API responses, configs |
| **JSONL** | Row, text | No | Manual (gzip) | No | Log streams, events |
| **Parquet** | Columnar, binary | Yes | Snappy/Gzip/Zstd | Yes | Analytics, data lakes |
| **Avro** | Row, binary | Yes | Deflate/Snappy | No | Kafka, schema evolution |
| **ORC** | Columnar, binary | Yes | Zlib/Snappy | Yes | Hive/Hadoop ecosystems |
| **Delta Lake** | Columnar + log | Yes + history | Snappy/Zstd | Yes | ACID, time travel, upserts |
| **Iceberg** | Columnar + log | Yes + history | Snappy/Zstd | Yes | Multi-engine, large tables |

---

### Deciding which format to use — a decision flowchart

```
Is the data going to a human or external system?
  Yes → CSV or JSON

Is the data going to a Kafka topic?
  Yes → Avro (with Schema Registry) or JSON (simpler, less safe)

Is the data going to a data lake or warehouse for analytics?
  Yes → Parquet (default choice)
        + Delta Lake if you need ACID/time travel/upserts
        + Iceberg if you need multi-engine access

Is the data in a legacy Hadoop/Hive environment?
  Yes → ORC (if system requires it) otherwise Parquet
```

---

## 6. Compression, Partitioning and Storage Layout

### Compression codecs — when to pick each

Compression reduces storage cost and improves read performance (less data to read from disk). The trade-off is always CPU time vs I/O time.

| Codec | Compression ratio | Speed | CPU usage | Best for |
|-------|-----------------|-------|-----------|---------|
| **None** | 1.0× (no compression) | Instant | None | Debugging; when read speed is critical on fast SSD |
| **Snappy** | ~3–5× | Very fast | Low | Default for most analytics — balanced |
| **LZ4** | ~2–3× | Fastest | Very low | Streaming, real-time, when latency matters most |
| **Gzip** | ~5–8× | Medium | Medium | Cold storage, archival — smallest files, acceptable read speed |
| **Zstd** | ~6–10× | Fast | Low-medium | Modern default — better than Snappy in ratio, nearly as fast |
| **Brotli** | ~8–12× | Slow | High | Web/API data; rarely used in DE |

```python
# Comparing compression in practice
import pandas as pd
import time
from pathlib import Path

df = pd.read_csv("data/olist/olist_orders_dataset.csv")

results = []
for codec in [None, "snappy", "gzip", "zstd"]:
    path = f"data/test_{codec or 'none'}.parquet"
    t = time.perf_counter()
    df.to_parquet(path, compression=codec)
    write_ms = (time.perf_counter() - t) * 1000
    size_kb   = Path(path).stat().st_size / 1024
    t = time.perf_counter()
    pd.read_parquet(path)
    read_ms = (time.perf_counter() - t) * 1000
    results.append({"codec": codec or "none", "size_kb": round(size_kb),
                    "write_ms": round(write_ms), "read_ms": round(read_ms)})

pd.DataFrame(results).sort_values("size_kb")
# codec   size_kb   write_ms   read_ms
# zstd      1,234        180        45
# gzip      1,456        380        52
# snappy    1,890         60        40
# none      7,234         20        80
```

**The verdict for most production pipelines:** Use Snappy for hot data (queried frequently), Gzip or Zstd for cold/archival data (queried rarely, stored long-term).

---

### Partitioning strategy — the most important Parquet optimisation

Partitioning creates a directory tree based on column values. When a query filters on a partition column, the engine skips entire directories — never reading files that can't match the filter.

```
Without partitioning:
data/orders.parquet   ← 5GB file, covers 2 years
Query: WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31'
→ Reads all 5GB, returns January rows

With partitioning by year+month:
data/orders/year=2024/month=1/part-0.parquet   ← 20MB
data/orders/year=2024/month=2/part-0.parquet   ← 22MB
...etc
Query: WHERE year=2024 AND month=1
→ Reads ONLY the 20MB file for January 2024
→ 250× less I/O
```

**Choosing partition columns:**

The partition column should be a column analysts frequently filter on. The most common choice:
- **Date partitions** — `year`, `month`, `day` (or `date`) are right for nearly all analytics data
- **Geographic partitions** — `country`, `region` if most queries filter by geography
- **Category partitions** — `department`, `product_category` if queries are category-scoped

**Partition granularity — the small file problem:**

```
Too coarse: partition by year only
data/orders/year=2024/part-0.parquet   ← 2GB for the year → still large

Too fine: partition by date + customer_id
data/orders/date=2024-01-15/customer=C000001/part-0.parquet   ← 0.1KB
→ 1 million customers × 365 days = 365 million tiny files
→ Metadata overhead kills performance

Sweet spot: partition by year + month
→ Each file is ~50–200MB after compression
→ A typical daily read touches 1 file
→ Total files for 2 years: 24
```

---

## 7. Schema Management and Evolution

### What is a schema?

A **schema** defines the structure of a dataset: column names, data types, nullable flags, and constraints.

```python
# Schema inference (what pandas does automatically from CSV)
df = pd.read_csv("orders.csv")
df.dtypes
# order_id      object    ← pandas guessed string
# amount        float64   ← correct
# order_date    object    ← STRING — not a date! bad inference

# Schema enforcement (what you want in production)
import pyarrow as pa

ORDERS_SCHEMA = pa.schema([
    pa.field("order_id",   pa.string(),  nullable=False),   # required
    pa.field("customer_id",pa.string(),  nullable=False),
    pa.field("amount",     pa.float64(), nullable=False),
    pa.field("order_date", pa.date32(),  nullable=False),
    pa.field("status",     pa.string(),  nullable=True),    # optional
])

# Writing with schema validation — rejects non-conforming data
table = pa.Table.from_pandas(df, schema=ORDERS_SCHEMA)
# Raises if df["order_id"] has nulls (nullable=False)
# Raises if a column is missing
# Coerces types where possible
```

---

### Schema evolution — the challenges you'll face in production

Source schemas change. New columns appear. Columns get renamed. Types change. How your pipeline handles this determines whether evolution is painless or a production incident.

**Type 1: Adding a column — backward compatible, usually safe**
```
Old schema: order_id, customer_id, amount, status
New schema: order_id, customer_id, amount, status, discount_pct   ← new

Old files: discount_pct is NULL
New files: discount_pct has values
→ Reading old + new together: old rows have NULL for discount_pct — acceptable
→ Action: add the column to the schema as nullable, handle NULLs downstream
```

**Type 2: Removing a column — forward compatible, risky**
```
Old schema: order_id, customer_id, amount, status
New schema: order_id, customer_id, amount          ← status removed

→ Any downstream query using "status" breaks immediately
→ Action: never silently remove; deprecate first, alert consumers, then remove after grace period
```

**Type 3: Renaming a column — breaking change**
```
Old: amount → New: payment_value

→ All downstream SQL/code using "amount" breaks
→ Action: add the new name as an alias first, migrate consumers, then drop old name
```

**Type 4: Type change — potentially breaking**
```
Old: rating FLOAT (e.g., 8.8) → New: rating STRING (e.g., "8.8/10")

→ All numeric operations downstream break (SUM, AVG, BETWEEN)
→ Action: never do this silently; migrate consumers first
```

```python
# Schema drift detection — run on every extract
def detect_schema_drift(
    df: pd.DataFrame,
    expected_schema: dict,
    table_name: str,
) -> list:
    """
    Compare actual DataFrame schema to expected schema.
    Returns list of drift events (new columns, missing columns, type changes).
    """
    drift_events = []

    actual_types = df.dtypes.astype(str).to_dict()

    # New columns (not in expected)
    for col in df.columns:
        if col not in expected_schema:
            drift_events.append({
                "event":   "new_column",
                "column":  col,
                "type":    actual_types[col],
                "severity":"warning",
                "message": f"New column '{col}' in {table_name} — not in expected schema",
            })

    # Missing columns (expected but not in actual)
    for col, expected_type in expected_schema.items():
        if col not in df.columns:
            drift_events.append({
                "event":    "missing_column",
                "column":   col,
                "severity": "error",
                "message":  f"Expected column '{col}' missing from {table_name}",
            })

    # Type changes
    for col, expected_type in expected_schema.items():
        if col in actual_types and not actual_types[col].startswith(expected_type):
            drift_events.append({
                "event":         "type_change",
                "column":        col,
                "expected_type": expected_type,
                "actual_type":   actual_types[col],
                "severity":      "error",
                "message":       f"'{col}' type changed: expected {expected_type}, got {actual_types[col]}",
            })

    # Log and persist drift events
    for event in drift_events:
        level = "ERROR" if event["severity"] == "error" else "WARNING"
        print(f"[SCHEMA DRIFT] [{level}] {event['message']}")

    return drift_events


# Data contracts (YAML-defined, loaded into Python)
OLIST_ORDERS_CONTRACT = {
    "order_id":    "object",
    "customer_id": "object",
    "amount":      "float",
    "order_date":  "datetime",
    "status":      "object",
}

drift = detect_schema_drift(df, OLIST_ORDERS_CONTRACT, "olist_orders")
if any(e["severity"] == "error" for e in drift):
    raise ValueError("Schema validation failed — pipeline aborted")
```

---
---

## 8. Extract

### What extraction means in practice

**Extract** = read data from wherever it currently lives. This sounds simple but is where most pipeline bugs happen — sources are unreliable, schemas change, APIs rate-limit, and files are corrupt.

🎬 **Movie analogy:** Extraction is the camera crew — gathering raw footage from multiple locations (on-set cameras, drone footage, archival clips). Each source has different equipment, formats, and reliability. The crew must handle failures and ensure all footage is collected before the editor starts work.

---

### Extracting from CSV files

```python
import pandas as pd
from pathlib import Path
import logging

logger = logging.getLogger(__name__)

def extract_csv(
    path: str,
    date_columns: list = None,
    required_columns: list = None,
    encoding: str = "utf-8",
) -> pd.DataFrame:
    """
    Safely extract data from a CSV file with validation.

    Parameters
    ----------
    path             : path to CSV file
    date_columns     : columns to parse as datetime
    required_columns : columns that must exist (raises if missing)
    encoding         : file encoding (try 'latin-1' if utf-8 fails)
    """
    path = Path(path)

    if not path.exists():
        raise FileNotFoundError(f"Source file not found: {path}")

    if path.stat().st_size == 0:
        raise ValueError(f"Source file is empty: {path}")

    try:
        df = pd.read_csv(
            path,
            parse_dates=date_columns or [],
            encoding=encoding,
            low_memory=False,
        )
    except UnicodeDecodeError:
        logger.warning(f"UTF-8 failed for {path}, trying latin-1")
        df = pd.read_csv(path, encoding="latin-1")

    # Validate required columns
    if required_columns:
        missing = set(required_columns) - set(df.columns)
        if missing:
            raise ValueError(f"Missing required columns in {path.name}: {missing}")

    # Clean column names
    df.columns = df.columns.str.lower().str.strip().str.replace(" ", "_")

    logger.info(f"✓ Extracted {len(df):,} rows from {path.name}")
    return df


# Usage — Olist
orders = extract_csv(
    "data/olist/olist_orders_dataset.csv",
    date_columns=["order_purchase_timestamp", "order_delivered_customer_date"],
    required_columns=["order_id", "customer_id", "order_status"],
)
```

---

### Extracting from relational databases

```python
import pandas as pd
from sqlalchemy import create_engine, text
from typing import Optional
import logging

logger = logging.getLogger(__name__)

def extract_from_db(
    connection_string: str,
    query: str,
    params: dict = None,
    chunksize: Optional[int] = None,
) -> pd.DataFrame:
    """
    Extract data from a relational database using SQLAlchemy.

    Works with: PostgreSQL, MySQL, SQLite, SQL Server, BigQuery (via connector).
    Never put credentials in code — use environment variables.
    """
    engine = create_engine(connection_string)

    with engine.connect() as conn:
        if chunksize:
            # Read in chunks for large tables
            chunks = []
            for chunk in pd.read_sql(text(query), conn, params=params,
                                     chunksize=chunksize):
                chunks.append(chunk)
                logger.info(f"  Read chunk of {len(chunk):,} rows")
            df = pd.concat(chunks, ignore_index=True)
        else:
            df = pd.read_sql(text(query), conn, params=params)

    logger.info(f"✓ Extracted {len(df):,} rows from database")
    return df


# Incremental extraction — only new/changed data
def extract_incremental(
    connection_string: str,
    table: str,
    watermark_col: str,
    last_watermark: str,
) -> pd.DataFrame:
    """
    Extract only rows newer than the last run's watermark.
    This is critical for efficiency — never reload full history if you can avoid it.
    """
    query = text(f"""
        SELECT *
        FROM {table}
        WHERE {watermark_col} > :watermark
        ORDER BY {watermark_col}
    """)

    return extract_from_db(
        connection_string,
        str(query),
        params={"watermark": last_watermark}
    )


# Usage
import os

# ✅ Credentials from environment variables (never hardcode)
DB_URL = os.environ.get("DATABASE_URL", "sqlite:///data/olist.db")

# Full extract (first run only)
df_full = extract_from_db(
    DB_URL,
    "SELECT * FROM orders WHERE order_status = 'delivered'",
)

# Incremental extract (subsequent runs)
last_run = "2024-01-31 23:59:59"
df_new   = extract_incremental(
    DB_URL,
    table="orders",
    watermark_col="order_purchase_timestamp",
    last_watermark=last_run,
)
```

---

### Extracting from REST APIs

```python
import requests
import pandas as pd
import time
from typing import Optional, Dict, Any
import logging

logger = logging.getLogger(__name__)

def extract_from_api(
    url: str,
    params: Dict[str, Any] = None,
    headers: Dict[str, str] = None,
    rate_limit_sleep: float = 0.1,
    max_retries: int = 3,
    timeout: int = 30,
) -> dict:
    """
    Extract data from a REST API with retry logic and rate limiting.
    Returns the parsed JSON response.
    """
    for attempt in range(1, max_retries + 1):
        try:
            response = requests.get(
                url,
                params=params,
                headers=headers,
                timeout=timeout,
            )
            response.raise_for_status()   # raises for 4xx/5xx status codes
            time.sleep(rate_limit_sleep)  # be kind to the API
            return response.json()

        except requests.exceptions.HTTPError as e:
            if response.status_code == 429:   # Too Many Requests
                wait = int(response.headers.get("Retry-After", 60))
                logger.warning(f"Rate limited — waiting {wait}s")
                time.sleep(wait)
            elif response.status_code >= 500:  # Server error — retry
                logger.warning(f"Server error {response.status_code}, attempt {attempt}/{max_retries}")
                time.sleep(2 ** attempt)  # exponential backoff
            else:
                raise   # Client error (400, 401, 404) — don't retry

        except requests.exceptions.ConnectionError:
            logger.warning(f"Connection error, attempt {attempt}/{max_retries}")
            time.sleep(2 ** attempt)

    raise RuntimeError(f"Failed after {max_retries} retries: {url}")


def extract_paginated_api(
    base_url: str,
    page_param: str = "page",
    limit_param: str = "limit",
    page_size: int = 100,
    headers: Dict[str, str] = None,
) -> pd.DataFrame:
    """
    Extract all pages from a paginated REST API.
    Many APIs return data in pages of N records.
    """
    all_records = []
    page = 1

    while True:
        logger.info(f"Fetching page {page}...")
        response = extract_from_api(
            base_url,
            params={page_param: page, limit_param: page_size},
            headers=headers,
        )

        # Common response patterns:
        records = response.get("data", response.get("results", response if isinstance(response, list) else []))

        if not records:
            break   # No more pages

        all_records.extend(records)

        # Check if we've reached the last page
        total = response.get("total", response.get("count"))
        if total and len(all_records) >= total:
            break

        page += 1
        logger.info(f"  Fetched {len(all_records):,} records so far")

    df = pd.json_normalize(all_records)
    logger.info(f"✓ Total records: {len(df):,}")
    return df
```

---

### Extracting from multiple sources and combining

```python
def extract_olist_all_tables(data_dir: str) -> dict:
    """
    Extract all Olist CSV files into a dictionary of DataFrames.
    Pattern: extract everything, validate, then transform.
    """
    data_dir = Path(data_dir)
    tables = {
        "orders":     ("olist_orders_dataset.csv",         ["order_purchase_timestamp"]),
        "customers":  ("olist_customers_dataset.csv",       []),
        "payments":   ("olist_order_payments_dataset.csv",  []),
        "products":   ("olist_products_dataset.csv",        []),
        "sellers":    ("olist_sellers_dataset.csv",         []),
        "order_items":("olist_order_items_dataset.csv",     []),
        "reviews":    ("olist_order_reviews_dataset.csv",   []),
    }

    extracted = {}
    for name, (filename, date_cols) in tables.items():
        path = data_dir / filename
        if path.exists():
            extracted[name] = extract_csv(str(path), date_columns=date_cols)
        else:
            logger.warning(f"⚠️  File not found: {filename} — skipping")

    logger.info(f"✓ Extracted {len(extracted)}/{len(tables)} tables")
    return extracted


# Usage
raw = extract_olist_all_tables("data/olist/")
print({name: df.shape for name, df in raw.items()})
```

---

## 9. Transform

### Transform is where value is created

Transformation takes raw, messy, source-specific data and turns it into a clean, consistent, analytics-ready form. This is the "T" in ETL — and where most data engineering logic lives.

**Transformation categories:**
```
1. CLEANING          Remove/fix bad data (nulls, duplicates, outliers, encoding)
2. STANDARDISING     Consistent formats (dates, currency, case, codes)
3. ENRICHING         Add computed columns or join reference data
4. AGGREGATING       Roll up to higher granularity
5. BUSINESS LOGIC    Apply domain-specific rules (RFM, SCD, etc.)
6. SHAPING           Pivot, melt, join, filter to target schema
```

---

### The transformation layer — building a clean table

```python
import pandas as pd
import numpy as np
from pathlib import Path
import logging

logger = logging.getLogger(__name__)

def transform_orders(raw: dict) -> pd.DataFrame:
    """
    Transform raw Olist data into a clean, analytics-ready orders table.

    Target schema:
    order_id | customer_unique_id | order_month | status | total_payment |
    delivery_days | is_late | avg_review_score | customer_state
    """

    # ── 1. MERGE SOURCES ─────────────────────────────────────────────────
    orders    = raw["orders"].copy()
    customers = raw["customers"].copy()
    payments  = raw["payments"].copy()
    reviews   = raw["reviews"].copy()

    # Aggregate payments to one row per order
    payments_agg = (payments
                    .groupby("order_id")["payment_value"]
                    .sum()
                    .reset_index(name="total_payment"))

    # Aggregate reviews to one row per order (take mean if multiple)
    reviews_agg = (reviews
                   .groupby("order_id")["review_score"]
                   .mean()
                   .reset_index(name="avg_review_score"))

    # Join all sources
    df = (orders
          .merge(customers[["customer_id", "customer_unique_id", "customer_state"]],
                 on="customer_id", how="left")
          .merge(payments_agg, on="order_id", how="left")
          .merge(reviews_agg,  on="order_id", how="left"))

    row_before = len(df)
    logger.info(f"After merges: {len(df):,} rows")

    # ── 2. PARSE DATES ────────────────────────────────────────────────────
    date_cols = ["order_purchase_timestamp", "order_delivered_customer_date",
                 "order_estimated_delivery_date"]
    for col in date_cols:
        df[col] = pd.to_datetime(df[col], errors="coerce")

    # ── 3. DERIVED COLUMNS ────────────────────────────────────────────────
    df["order_month"] = df["order_purchase_timestamp"].dt.to_period("M").astype(str)
    df["order_year"]  = df["order_purchase_timestamp"].dt.year
    df["order_week"]  = df["order_purchase_timestamp"].dt.isocalendar().week

    # Delivery performance
    delivered = df["order_delivered_customer_date"].notna()
    df["delivery_days"] = np.where(
        delivered,
        (df["order_delivered_customer_date"] - df["order_purchase_timestamp"]).dt.days,
        np.nan
    )

    df["is_late"] = np.where(
        delivered,
        (df["order_delivered_customer_date"] > df["order_estimated_delivery_date"]).astype(int),
        np.nan
    )

    # ── 4. STANDARDISE ────────────────────────────────────────────────────
    df["order_status"] = df["order_status"].str.lower().str.strip()

    # Map status codes to clean labels
    status_map = {
        "delivered":   "delivered",
        "shipped":     "in_transit",
        "processing":  "processing",
        "invoiced":    "processing",
        "canceled":    "cancelled",
        "unavailable": "cancelled",
        "approved":    "processing",
        "created":     "processing",
    }
    df["status_clean"] = df["order_status"].map(status_map).fillna("unknown")

    # ── 5. FILL / IMPUTE ──────────────────────────────────────────────────
    df["total_payment"]     = df["total_payment"].fillna(0)
    df["avg_review_score"]  = df["avg_review_score"].round(2)

    # ── 6. COLUMN SELECTION AND RENAME ────────────────────────────────────
    output_cols = [
        "order_id", "customer_unique_id", "customer_state",
        "order_month", "order_year", "order_week",
        "status_clean",
        "total_payment",
        "delivery_days", "is_late",
        "avg_review_score",
        "order_purchase_timestamp",
    ]
    df = df[output_cols].rename(columns={"status_clean": "status"})

    # ── 7. FINAL VALIDATION ───────────────────────────────────────────────
    assert df["order_id"].duplicated().sum() == 0, "Duplicate order_ids after transform!"
    assert df["total_payment"].min() >= 0,         "Negative payment values found!"

    logger.info(f"✓ Transform complete: {len(df):,} rows "
                f"(dropped {row_before - len(df)} during merge)")
    return df
```

---

### Incremental transform — only process new data

```python
def transform_incremental(
    raw_new: pd.DataFrame,
    existing_transformed: pd.DataFrame,
    watermark_col: str,
) -> pd.DataFrame:
    """
    Transform only new records, then append to existing transformed data.

    This is the key efficiency pattern for daily pipelines:
    - Don't reprocess 2 years of history every day
    - Only transform yesterday's new data
    - Append to the existing clean table
    """
    # Apply transform to new records only
    transformed_new = transform_orders_single(raw_new)

    # Append
    combined = pd.concat([existing_transformed, transformed_new], ignore_index=True)

    # Deduplicate (in case of re-runs)
    combined = combined.drop_duplicates(subset=["order_id"], keep="last")

    logger.info(f"✓ Incremental transform: added {len(transformed_new):,} new rows. "
                f"Total: {len(combined):,}")
    return combined
```

---

### Business logic transforms — common patterns

```python
# ── RFM SCORES ────────────────────────────────────────────────────────────
def compute_rfm(orders_df: pd.DataFrame, reference_date=None) -> pd.DataFrame:
    """Compute RFM segments for all customers."""
    if reference_date is None:
        reference_date = orders_df["order_purchase_timestamp"].max()

    rfm = (orders_df
           .query("status == 'delivered'")
           .groupby("customer_unique_id")
           .agg(
               last_order = ("order_purchase_timestamp", "max"),
               frequency  = ("order_id",                 "count"),
               monetary   = ("total_payment",            "sum"),
           )
           .reset_index())

    rfm["recency_days"] = (reference_date - rfm["last_order"]).dt.days

    for col, labels in [
        ("recency_days", [5,4,3,2,1]),   # inverted: low days = high score
        ("frequency",    [1,2,3,4,5]),
        ("monetary",     [1,2,3,4,5]),
    ]:
        score_col = col.split("_")[0][0] + "_score"
        rfm[score_col] = pd.qcut(
            rfm[col].rank(method="first"),
            5, labels=labels, duplicates="drop"
        ).astype(int)

    conditions = [
        (rfm["r_score"] >= 4) & (rfm["f_score"] >= 4),
        (rfm["r_score"] >= 3) & (rfm["f_score"] >= 3),
        (rfm["r_score"] >= 4) & (rfm["f_score"] <= 2),
        (rfm["r_score"] <= 2) & (rfm["f_score"] >= 3),
    ]
    rfm["segment"] = np.select(conditions, ["Champions","Loyal","Promising","At Risk"],
                                default="Others")
    return rfm


# ── SELLER SCORECARD ──────────────────────────────────────────────────────
def compute_seller_scorecard(orders_df: pd.DataFrame, items_df: pd.DataFrame,
                              sellers_df: pd.DataFrame) -> pd.DataFrame:
    """Build a seller performance scorecard."""
    # Join items to orders to get seller
    merged = items_df.merge(
        orders_df[["order_id", "total_payment", "is_late", "avg_review_score",
                   "order_month", "status"]],
        on="order_id", how="inner"
    )

    scorecard = merged.groupby("seller_id").agg(
        total_orders     = ("order_id",           "nunique"),
        total_revenue    = ("price",               "sum"),
        avg_review       = ("avg_review_score",    "mean"),
        late_rate        = ("is_late",             "mean"),
        cancellation_rate= ("status",
                            lambda x: (x == "cancelled").mean()),
    ).reset_index().round(3)

    # Performance tier
    scorecard["tier"] = np.select(
        [
            (scorecard["avg_review"] >= 4.0) & (scorecard["late_rate"] < 0.10),
            (scorecard["avg_review"] >= 3.5) | (scorecard["late_rate"] < 0.20),
        ],
        ["Gold", "Silver"],
        default="Bronze"
    )

    return scorecard.merge(sellers_df[["seller_id", "seller_state"]], on="seller_id", how="left")
```

---

## 10. Load

### Load patterns — how and where you write data

**Load** = writing transformed data to the target destination. The three main patterns are:

| Pattern | What it does | When to use |
|---------|-------------|-------------|
| **Append** | Add new rows to existing table | Event data, logs — data is immutable |
| **Overwrite (Full Replace)** | Delete all existing data, write fresh | Small lookup/dimension tables |
| **Upsert (Merge)** | Insert new rows, update changed rows | Slowly changing data, fact tables with corrections |
| **Incremental Insert** | Insert only rows not already there | Idempotent append |

---

### Load to files (Parquet, CSV)

```python
def load_to_parquet(
    df: pd.DataFrame,
    output_path: str,
    partition_cols: list = None,
    mode: str = "overwrite",  # "overwrite" or "append"
) -> None:
    """Write DataFrame to Parquet, with optional partitioning."""
    import pyarrow as pa
    import pyarrow.parquet as pq

    output_path = Path(output_path)

    if mode == "overwrite" and output_path.exists():
        import shutil
        if output_path.is_dir():
            shutil.rmtree(output_path)
        else:
            output_path.unlink()

    if partition_cols:
        output_path.mkdir(parents=True, exist_ok=True)
        pq.write_to_dataset(
            pa.Table.from_pandas(df),
            root_path=str(output_path),
            partition_cols=partition_cols,
            compression="snappy",
            existing_data_behavior="delete_matching",  # idempotent
        )
    else:
        output_path.parent.mkdir(parents=True, exist_ok=True)
        df.to_parquet(str(output_path), index=False, compression="snappy")

    logger.info(f"✓ Loaded {len(df):,} rows → {output_path}")
```

---

### Load to a database (append, overwrite, upsert)

```python
from sqlalchemy import create_engine, text
import pandas as pd

def load_to_db(
    df: pd.DataFrame,
    table: str,
    connection_string: str,
    mode: str = "append",   # "append", "replace", "upsert"
    key_columns: list = None,
) -> None:
    """
    Load a DataFrame to a relational database.

    mode="append"  → INSERT INTO table — fastest, no duplicate check
    mode="replace" → DROP + CREATE + INSERT — for full reloads of small tables
    mode="upsert"  → INSERT OR UPDATE based on key_columns — for updates
    """
    engine = create_engine(connection_string)

    if mode in ("append", "replace"):
        df.to_sql(
            table,
            engine,
            if_exists="append" if mode == "append" else "replace",
            index=False,
            chunksize=10_000,
            method="multi",   # faster than default row-by-row
        )
        logger.info(f"✓ {mode}: {len(df):,} rows → {table}")

    elif mode == "upsert":
        # SQLite-compatible upsert using INSERT OR REPLACE
        if not key_columns:
            raise ValueError("key_columns required for upsert mode")

        with engine.begin() as conn:
            # Create temp table
            temp_table = f"_temp_{table}_{id(df)}"
            df.to_sql(temp_table, conn, if_exists="replace", index=False)

            # Upsert from temp to target
            cols         = ", ".join(df.columns)
            update_cols  = ", ".join([f"{c} = excluded.{c}"
                                      for c in df.columns if c not in key_columns])
            conflict_key = ", ".join(key_columns)

            conn.execute(text(f"""
                INSERT INTO {table} ({cols})
                SELECT {cols} FROM {temp_table}
                ON CONFLICT ({conflict_key})
                DO UPDATE SET {update_cols}
            """))

            conn.execute(text(f"DROP TABLE IF EXISTS {temp_table}"))

        logger.info(f"✓ Upserted {len(df):,} rows → {table} (key: {key_columns})")
```

---

### Idempotent loading — safe to re-run

The most important property of a production load step: running it twice must produce the same result as running it once.

```python
def load_daily_partition(
    df: pd.DataFrame,
    table: str,
    partition_date: str,
    connection_string: str,
) -> None:
    """
    Idempotent daily partition load.
    DELETE the partition for this date, then INSERT.
    Running twice = same result.
    """
    engine = create_engine(connection_string)

    with engine.begin() as conn:
        # Step 1: Delete existing data for this partition
        conn.execute(text(f"""
            DELETE FROM {table}
            WHERE DATE(order_date) = :partition_date
        """), {"partition_date": partition_date})

        # Step 2: Insert new data
        df.to_sql(table, conn, if_exists="append", index=False, method="multi")

    logger.info(f"✓ Idempotent load: {len(df):,} rows for {partition_date} → {table}")


# ⚠️ NON-idempotent (BAD) — running twice doubles the rows
def bad_load(df, table, conn):
    df.to_sql(table, conn, if_exists="append", index=False)   # ← append blindly

# ✅ IDEMPOTENT (GOOD) — delete partition first, then insert
def good_load(df, table, partition_date, conn):
    conn.execute(text(f"DELETE FROM {table} WHERE date = :d"), {"d": partition_date})
    df.to_sql(table, conn, if_exists="append", index=False)
```

---

## 11. ETL vs ELT

### The architecture shift

**ETL (Extract → Transform → Load):** transform data on the way in, load only clean data.
**ELT (Extract → Load → Transform):** load raw data first, then transform inside the warehouse.

```
ETL (traditional):                    ELT (modern):
────────────────────                  ──────────────────────────────
Source  →  Transform  →  Load        Source  →  Load  →  Transform
         (on your server)           (into warehouse)   (inside warehouse)
           Clean data only            Raw data first    dbt models run in SQL

When cloud storage was expensive:     When storage is cheap and
transform BEFORE loading to save      compute scales infinitely:
space.                                load everything raw, transform later.
```

---

### ETL — when to use it

```
✅ Use ETL when:
- Source data is very messy (PII to scrub, sensitive data to mask)
- Target storage is expensive (legacy on-premise DW)
- Transformation requires complex Python/ML logic
- You don't want raw data in the warehouse at all

Example: Healthcare pipeline
Raw EHR data (contains PII) → Python: remove PII, anonymise → Load clean only
```

```python
# ETL pattern: transform BEFORE loading
def etl_healthcare():
    # Extract
    raw = pd.read_csv("patients.csv")

    # Transform FIRST — remove PII before it touches the warehouse
    df = raw.copy()
    df["patient_id"] = df["ssn"].apply(hash_sha256)  # anonymise
    df = df.drop(columns=["ssn", "name", "address"])   # drop PII
    df["age_band"] = pd.cut(df["age"], bins=[0,25,40,60,100],
                             labels=["<25","25-40","40-60","60+"])
    df = df.drop(columns=["age"])  # don't store exact age

    # Load — only anonymised data reaches the DB
    df.to_sql("patients_anon", engine, if_exists="append", index=False)
```

---

### ELT — when to use it

```
✅ Use ELT when:
- Target is a cloud warehouse (BigQuery, Snowflake, Redshift) — cheap storage
- Transformation logic may change — easier to re-run SQL than re-ingest
- You want to keep raw data for auditing and reprocessing
- dbt handles your transformations

Example: Olist analytics pipeline
Olist CSVs → Load raw to Snowflake → dbt models clean and aggregate
```

```python
# ELT pattern: load raw first, transform inside warehouse with SQL/dbt
def elt_olist():
    # Extract
    raw = extract_olist_all_tables("data/olist/")

    # Load RAW — no transformation yet
    for table_name, df in raw.items():
        df.to_sql(
            f"raw_{table_name}",  # raw_ prefix signals "untransformed"
            engine,
            if_exists="replace",
            index=False
        )
        print(f"✓ Loaded raw_{table_name}: {len(df):,} rows")

    # Transformations happen as SQL queries (or dbt models) inside the DB:
    # raw_orders → stg_orders (clean) → dim_customers (dimensional) → fct_orders (fact)
```

---

### dbt in the ELT pattern

**dbt (data build tool)** is a CLI tool that lets you write SQL SELECT statements to define transformations. It handles compilation, execution, dependency resolution, and testing.

```sql
-- dbt model: models/staging/stg_orders.sql
-- This runs as: CREATE TABLE stg_orders AS (...)

WITH source AS (
    SELECT * FROM {{ source('olist', 'raw_orders') }}
),

cleaned AS (
    SELECT
        order_id,
        customer_id,
        order_status,
        CAST(order_purchase_timestamp AS TIMESTAMP)        AS order_ts,
        CAST(order_delivered_customer_date AS TIMESTAMP)   AS delivered_ts,
        CAST(order_estimated_delivery_date AS TIMESTAMP)   AS estimated_ts,

        -- Derived
        DATE_TRUNC('month', order_purchase_timestamp)      AS order_month,
        DATEDIFF('day',
            order_purchase_timestamp,
            order_delivered_customer_date)                 AS delivery_days,

        CASE
            WHEN order_delivered_customer_date > order_estimated_delivery_date
            THEN 1 ELSE 0
        END                                                AS is_late

    FROM source
    WHERE order_id IS NOT NULL
)

SELECT * FROM cleaned
```

```yaml
# dbt model config with tests — models/staging/schema.yml
version: 2
models:
  - name: stg_orders
    description: "Cleaned order-level data"
    columns:
      - name: order_id
        description: "Unique order identifier"
        tests:
          - unique
          - not_null
      - name: order_status
        tests:
          - accepted_values:
              values: ['delivered', 'shipped', 'processing', 'cancelled', 'unavailable']
      - name: delivery_days
        tests:
          - not_null:
              where: "order_status = 'delivered'"
```

---
---

## 12. Dimensional Modelling

### Why dimensional modelling?

Normalised 3NF schemas (from SQL 101) are optimal for transactional systems — they prevent anomalies and support fast writes. For analytics, they're painful: answering "revenue by product category by customer state by month" requires joining 6+ tables, and analysts must understand every join key.

**Dimensional modelling** deliberately denormalises for query simplicity. A few big, flat tables beat many small normalised ones when analysts need speed and accessibility.

🎬 **Movies analogy:** In production, every scene is stored separately (raw footage, 3NF). For the audience, you cut and arrange into a coherent sequence that's easy to watch (star schema). The editing trades storage redundancy for viewing simplicity.

---

### Star schema — the foundation of analytics

A **star schema** has two types of tables:
- **Fact table**: the centre of the star. Stores measurable events — each row is a transaction, order, or measurement. Contains foreign keys to dimension tables and numeric measures.
- **Dimension table**: the points of the star. Stores descriptive context — who, what, where, when. Denormalised on purpose.

```
                    dim_date
                   ┌──────────────────┐
                   │ date_id (PK)     │
                   │ full_date        │
                   │ year, quarter    │
                   │ month, month_name│
                   │ day_of_week      │
                   │ is_weekend       │
                   └────────┬─────────┘
                            │ FK
    dim_customer     fct_orders          dim_product
    ┌────────────┐  ┌──────────────────┐  ┌────────────────┐
    │customer_sk │◀─│customer_sk (FK)  │  │product_sk (PK) │
    │customer_id │  │product_sk  (FK)──┼─▶│product_id      │
    │name        │  │date_id     (FK)  │  │product_name    │
    │city, state │  │seller_sk   (FK)──┼─▶│category_en     │
    │region      │  │                  │  │avg_price       │
    └────────────┘  │ ── MEASURES ──   │  └────────────────┘
                    │ item_revenue     │
                    │ freight_value    │        dim_seller
                    │ total_payment    │       ┌────────────────┐
                    │ delivery_days    │◀──────│seller_sk (PK)  │
                    │ is_late          │       │seller_id       │
                    │ avg_review_score │       │seller_state    │
                    └──────────────────┘       │tier            │
                                               └────────────────┘

Grain of fct_orders: ONE ROW PER ORDER LINE ITEM
(One order with 3 products = 3 rows in the fact table)
```

**Why surrogate keys?** Every dimension uses an integer surrogate key (customer_sk, not customer_id) as the primary key. This is because:
1. Natural keys can change (customer changes email, which is their natural key)
2. Natural keys from multiple source systems may collide
3. Integer lookups are faster than string lookups
4. Allows SCD Type 2 history (same customer_id, multiple customer_sk values at different times)

---

### Building a date dimension in full

```python
import pandas as pd
import numpy as np
from datetime import date

def build_date_dim(start_date: str, end_date: str) -> pd.DataFrame:
    """
    Build a complete date dimension table.
    Every analytics warehouse needs this — never compute date attributes
    ad-hoc in queries. Pre-calculate everything here.
    """
    date_range = pd.date_range(start=start_date, end=end_date, freq="D")
    df = pd.DataFrame({"full_date": date_range})

    # Primary key — YYYYMMDD integer (compact, sortable, human-readable)
    df["date_id"]      = df["full_date"].dt.strftime("%Y%m%d").astype(int)

    # Year/Month/Day components
    df["year"]         = df["full_date"].dt.year
    df["quarter"]      = df["full_date"].dt.quarter
    df["quarter_label"]= "Q" + df["full_date"].dt.quarter.astype(str)
    df["year_quarter"] = df["full_date"].dt.year.astype(str) + "-" + df["quarter_label"]
    df["month"]        = df["full_date"].dt.month
    df["month_name"]   = df["full_date"].dt.month_name()
    df["month_abbr"]   = df["full_date"].dt.strftime("%b")
    df["year_month"]   = df["full_date"].dt.strftime("%Y-%m")
    df["week"]         = df["full_date"].dt.isocalendar().week.astype(int)
    df["day_of_month"] = df["full_date"].dt.day
    df["day_of_year"]  = df["full_date"].dt.dayofyear

    # Day of week
    df["day_of_week"]  = df["full_date"].dt.dayofweek      # 0=Monday, 6=Sunday
    df["day_name"]     = df["full_date"].dt.day_name()     # "Monday"
    df["day_abbr"]     = df["full_date"].dt.strftime("%a") # "Mon"
    df["is_weekend"]   = df["day_of_week"] >= 5

    # Business day indicator
    df["is_business_day"] = df["day_of_week"] < 5

    # Brazilian public holidays (expand as needed)
    brazilian_holidays = {
        # Fixed national holidays
        date(2023, 1, 1), date(2023, 4, 7),  date(2023, 4, 21),
        date(2023, 5, 1), date(2023, 9, 7),  date(2023, 10, 12),
        date(2023, 11, 2),date(2023, 11, 15),date(2023, 12, 25),
        # 2024
        date(2024, 1, 1), date(2024, 4, 21), date(2024, 5, 1),
        date(2024, 9, 7), date(2024, 10, 12),date(2024, 11, 2),
        date(2024, 11, 15),date(2024, 12, 25),
    }
    df["is_holiday"]        = df["full_date"].dt.date.isin(brazilian_holidays)
    df["is_business_day"]   = df["is_business_day"] & ~df["is_holiday"]

    # Fiscal year (assume July 1 start — common in Brazil)
    df["fiscal_year"]    = np.where(df["month"] >= 7, df["year"], df["year"] - 1)
    df["fiscal_quarter"] = ((df["month"] - 7) % 12 // 3 + 1)
    df["fiscal_year_label"] = "FY" + df["fiscal_year"].astype(str)

    # Relative flags (useful for "last 30 days" type filters)
    today = pd.Timestamp.today().normalize()
    df["days_ago"]        = (today - df["full_date"]).dt.days
    df["is_last_7_days"]  = df["days_ago"].between(0, 6)
    df["is_last_30_days"] = df["days_ago"].between(0, 29)
    df["is_last_90_days"] = df["days_ago"].between(0, 89)
    df["is_ytd"]          = (df["year"] == today.year) & (df["full_date"] <= today)

    return df.set_index("date_id").reset_index()


dim_date = build_date_dim("2016-01-01", "2026-12-31")
print(f"Date dim: {len(dim_date):,} rows × {len(dim_date.columns)} columns")
# Date dim: 4,018 rows × 26 columns
```

---

### Building dimension tables with surrogate keys

```python
def build_dim_customer(customers_df: pd.DataFrame) -> pd.DataFrame:
    """
    Build customer dimension from raw Olist customers table.
    One row per unique customer. Adds surrogate key and enriched geography.
    """
    dim = customers_df.copy()

    # Deduplicate — customers table has one row per order, not per customer
    dim = dim.drop_duplicates("customer_unique_id")

    # Standardise
    dim["customer_city"]  = dim["customer_city"].str.strip().str.title()
    dim["customer_state"] = dim["customer_state"].str.upper().str.strip()

    # Enrich with state full name
    state_names = {
        "SP":"São Paulo",         "RJ":"Rio de Janeiro",    "MG":"Minas Gerais",
        "RS":"Rio Grande do Sul", "PR":"Paraná",            "SC":"Santa Catarina",
        "BA":"Bahia",             "GO":"Goiás",             "ES":"Espírito Santo",
        "PE":"Pernambuco",        "CE":"Ceará",             "MA":"Maranhão",
        "AM":"Amazonas",          "PA":"Pará",              "MT":"Mato Grosso",
        "MS":"Mato Grosso do Sul","RN":"Rio Grande do Norte","PB":"Paraíba",
        "AL":"Alagoas",           "PI":"Piauí",             "TO":"Tocantins",
        "SE":"Sergipe",           "RO":"Rondônia",          "AP":"Amapá",
        "AC":"Acre",              "RR":"Roraima",           "DF":"Distrito Federal",
    }
    dim["state_name"] = dim["customer_state"].map(state_names).fillna(dim["customer_state"])

    # Brazil geographic regions
    regions = {
        "SP":"Southeast","RJ":"Southeast","MG":"Southeast","ES":"Southeast",
        "PR":"South",    "SC":"South",    "RS":"South",
        "BA":"Northeast","PE":"Northeast","CE":"Northeast","MA":"Northeast",
        "RN":"Northeast","PB":"Northeast","AL":"Northeast","PI":"Northeast",
        "SE":"Northeast",
        "AM":"North",    "PA":"North",    "TO":"North",    "RO":"North",
        "AP":"North",    "AC":"North",    "RR":"North",
        "GO":"Central-West","MT":"Central-West","MS":"Central-West","DF":"Central-West",
    }
    dim["region"] = dim["customer_state"].map(regions).fillna("Other")

    # Add surrogate key — sequential integer
    dim = dim.reset_index(drop=True)
    dim.index = dim.index + 1          # start from 1 (convention)
    dim.index.name = "customer_sk"
    dim = dim.reset_index()

    return dim[[
        "customer_sk",         # surrogate key
        "customer_unique_id",  # natural key (for joining to transactions)
        "customer_state",      # 2-letter code
        "state_name",          # full state name
        "region",              # geographic region
        "customer_city",       # city
    ]]


def build_dim_product(products_df: pd.DataFrame,
                      translations_df: pd.DataFrame) -> pd.DataFrame:
    """
    Build product dimension with English category names.
    Includes product physical attributes for logistics analysis.
    """
    dim = products_df.merge(
        translations_df[["product_category_name", "product_category_name_english"]],
        on="product_category_name",
        how="left"
    )
    # Clean English category name
    dim["category_en"] = (dim["product_category_name_english"]
                          .str.replace("_", " ", regex=False)
                          .str.title()
                          .fillna("Unknown"))

    # Create category groupings for higher-level analysis
    electronics = {"Computers", "Tablets Printing Image", "Electronics",
                   "Computers Accessories", "Pc Gamer", "Telephony"}
    fashion      = {"Fashion Bags Accessories", "Fashion Shoes",
                    "Fashion Male Clothing", "Fashion Female Clothing"}
    home         = {"Furniture Decor", "Housewares", "Home Appliances",
                    "Bed Bath Table", "Garden Tools"}

    dim["category_group"] = dim["category_en"].apply(
        lambda c: "Electronics" if c in electronics
                  else "Fashion" if c in fashion
                  else "Home & Garden" if c in home
                  else "Other"
    )

    # Surrogate key
    dim = dim.reset_index(drop=True)
    dim.index = dim.index + 1
    dim.index.name = "product_sk"
    dim = dim.reset_index()

    return dim[[
        "product_sk",
        "product_id",
        "category_en",
        "category_group",
        "product_name_length",
        "product_description_length",
        "product_photos_qty",
        "product_weight_g",
        "product_length_cm",
        "product_height_cm",
        "product_width_cm",
    ]]


def build_dim_seller(sellers_df: pd.DataFrame) -> pd.DataFrame:
    """Build seller dimension with geography."""
    dim = sellers_df.copy()
    dim["seller_state"] = dim["seller_state"].str.upper().str.strip()

    # Add region (reusing region mapping from dim_customer)
    regions = {
        "SP":"Southeast","RJ":"Southeast","MG":"Southeast","ES":"Southeast",
        "PR":"South",    "SC":"South",    "RS":"South",
        "BA":"Northeast","PE":"Northeast","CE":"Northeast",
        "AM":"North",    "PA":"North",
        "GO":"Central-West","MT":"Central-West","MS":"Central-West","DF":"Central-West",
    }
    dim["seller_region"] = dim["seller_state"].map(regions).fillna("Other")

    dim = dim.reset_index(drop=True)
    dim.index = dim.index + 1
    dim.index.name = "seller_sk"
    dim = dim.reset_index()

    return dim[["seller_sk", "seller_id", "seller_state", "seller_region",
                "seller_city"]]
```

---

### Building the fact table

The fact table is the most important table in the schema. Its **grain** (what one row represents) must be defined precisely before you write a single line of code.

```python
import pandas as pd
import numpy as np

def build_fct_orders(
    orders_df:   pd.DataFrame,
    items_df:    pd.DataFrame,
    payments_df: pd.DataFrame,
    reviews_df:  pd.DataFrame,
    dim_customer: pd.DataFrame,
    dim_product:  pd.DataFrame,
    dim_seller:   pd.DataFrame,
) -> pd.DataFrame:
    """
    Build the central fact table.

    GRAIN: One row per order line item (order_id + order_item_id).
    An order with 3 products generates 3 fact rows.
    This allows product-level analysis while keeping order-level context.

    FOREIGN KEYS: customer_sk, product_sk, seller_sk, date_id
    DEGENERATE DIMENSIONS: order_id, order_item_id (IDs with no dimension table)
    MEASURES: item_revenue, freight_value, total_payment, delivery_days, is_late, avg_review
    """

    # ── AGGREGATE PAYMENTS TO ORDER LEVEL ────────────────────────────────
    # payments is 1:N with orders (multiple payment methods per order)
    payments_agg = (payments_df
                    .groupby("order_id")["payment_value"]
                    .sum()
                    .reset_index(name="total_payment"))

    # ── AGGREGATE REVIEWS TO ORDER LEVEL ─────────────────────────────────
    # reviews can have multiple entries per order (edge case)
    reviews_agg = (reviews_df
                   .groupby("order_id")["review_score"]
                   .mean()
                   .round(2)
                   .reset_index(name="avg_review_score"))

    # ── BUILD BASE FROM ORDER ITEMS ───────────────────────────────────────
    fct = items_df.merge(
        orders_df[[
            "order_id", "customer_id",
            "order_purchase_timestamp",
            "order_delivered_customer_date",
            "order_estimated_delivery_date",
            "order_status"
        ]],
        on="order_id", how="inner"    # inner: only completed order-item pairs
    )

    # ── ADD PAYMENT AND REVIEW ────────────────────────────────────────────
    fct = fct.merge(payments_agg, on="order_id", how="left")
    fct = fct.merge(reviews_agg,  on="order_id", how="left")

    # ── RESOLVE SURROGATE KEYS ────────────────────────────────────────────
    # Map natural keys to surrogate keys from dimension tables

    # Customer: orders has customer_id, dimensions has customer_unique_id
    # Need to go through customers table to get customer_unique_id
    # Here we assume customer_sk was added during dim build
    cust_map = dict(zip(dim_customer["customer_unique_id"],
                        dim_customer["customer_sk"]))
    prod_map = dict(zip(dim_product["product_id"],   dim_product["product_sk"]))
    sell_map = dict(zip(dim_seller["seller_id"],     dim_seller["seller_sk"]))

    # Join to get customer_unique_id first
    # (orders table has customer_id, not customer_unique_id)
    from_customers = fct[["order_id", "customer_id"]].drop_duplicates()
    # We'd need the raw customers table here — simplified for clarity
    # In practice: join orders → customers to get customer_unique_id
    # Then map customer_unique_id → customer_sk

    fct["product_sk"] = fct["product_id"].map(prod_map)
    fct["seller_sk"]  = fct["seller_id"].map(sell_map)

    # ── DATE KEY ──────────────────────────────────────────────────────────
    fct["date_id"] = (fct["order_purchase_timestamp"]
                      .dt.strftime("%Y%m%d")
                      .astype("Int64"))    # nullable integer for NaT safety

    # ── COMPUTED MEASURES ─────────────────────────────────────────────────
    delivered_mask = fct["order_delivered_customer_date"].notna()

    fct["delivery_days"] = np.where(
        delivered_mask,
        (fct["order_delivered_customer_date"] -
         fct["order_purchase_timestamp"]).dt.days,
        np.nan
    )

    fct["is_late"] = np.where(
        delivered_mask,
        (fct["order_delivered_customer_date"] >
         fct["order_estimated_delivery_date"]).astype("Int8"),
        pd.NA
    )

    # ── VALIDATE BEFORE RETURNING ─────────────────────────────────────────
    n_before = len(fct)
    assert fct["order_id"].notnull().all(),       "Null order_ids in fact table"
    assert fct["item_revenue"].min() >= 0,         "Negative item revenue"
    assert (fct["total_payment"] >= 0).all(),      "Negative total payment"

    fct_clean = fct[[
        "order_id", "order_item_id",
        "product_sk", "seller_sk", "date_id",
        "price",                   # rename below
        "freight_value",
        "total_payment",
        "avg_review_score",
        "delivery_days",
        "is_late",
        "order_status",
    ]].rename(columns={"price": "item_revenue"})

    print(f"✓ fct_orders: {len(fct_clean):,} rows "
          f"(grain: order line item)")
    return fct_clean
```

---

## 13. Slowly Changing Dimensions

### Why SCD matters — the historical accuracy problem

Dimension data changes over time. A customer moves cities. A product is re-categorised. A seller's tier improves. Without SCD, you face a dilemma:

- If you **overwrite** the old value: your current data is correct, but historical reports are wrong. A 2022 sales report for "São Paulo customers" might include a customer who moved there in 2023.
- If you **never update**: your historical reports are correct, but current queries return stale values.

**SCD Types** are the solution — different strategies for different business requirements:

| Type | Strategy | History kept? | Complexity | Use when |
|------|---------|--------------|-----------|---------|
| **Type 0** | Never update — always keep original | No (frozen) | None | Attributes that should never change (birthdate, creation date) |
| **Type 1** | Overwrite — always current value | No | Low | Correcting data errors; history doesn't matter |
| **Type 2** | New row per change — full history | Yes (unlimited) | High | Anything you need to report historically accurately |
| **Type 3** | Add "previous value" column | One level | Medium | When you only care about current vs immediately prior |

---

### SCD Type 1 — Overwrite

```python
def scd_type1_upsert(
    new_df: pd.DataFrame,
    existing_df: pd.DataFrame,
    natural_key: str,
) -> pd.DataFrame:
    """
    SCD Type 1: overwrite existing records, no history.

    Use for:
    - Fixing typos in names
    - Updating phone numbers or email addresses
    - Any attribute where "what was the value last year?" is irrelevant

    Side effect: Historical fact table rows will now join to the UPDATED
    dimension value, not the value at the time of the fact. This is sometimes
    intentional (always use current name) and sometimes a bug.
    """
    # Keep all existing rows NOT in the new update
    unchanged = existing_df[~existing_df[natural_key].isin(new_df[natural_key])]
    # Add the new (updated) rows
    result = pd.concat([unchanged, new_df], ignore_index=True)
    print(f"SCD1: {len(existing_df)} → {len(result)} rows "
          f"({len(new_df)} updated)")
    return result
```

---

### SCD Type 2 — Full history

SCD Type 2 is the most powerful and most common approach for analytics. Every change creates a new row with `effective_from`, `effective_to`, and `is_current` columns.

```python
import pandas as pd
from datetime import date

def scd_type2_update(
    new_snapshot: pd.DataFrame,
    existing_dim: pd.DataFrame,
    natural_key:  str,
    tracked_cols: list,
    effective_date: str,
) -> pd.DataFrame:
    """
    SCD Type 2: add new row for each changed record.
    Old row is "expired" with effective_to = effective_date - 1 day.
    New row is "current" with effective_from = effective_date.

    Use for:
    - Customer address changes (for geographic revenue attribution)
    - Product category reclassification
    - Seller tier promotions/demotions
    - Employee department transfers

    The key analytical benefit: a fact table row joins to
    the dimension row effective AT THE TIME of the fact.

    Example query in DuckDB/SQL:
        SELECT f.order_id, s.tier
        FROM fct_orders f
        JOIN dim_seller s
          ON f.seller_id = s.seller_id
         AND f.order_date BETWEEN s.effective_from
                              AND COALESCE(s.effective_to, '9999-12-31')
    """
    # Get only the currently active rows
    current_rows = existing_dim[existing_dim["is_current"] == 1].copy()

    # Merge current state with new snapshot to find changes
    merged = new_snapshot.merge(
        current_rows[[natural_key] + tracked_cols].add_suffix("_old").rename(
            columns={natural_key + "_old": natural_key}
        ),
        on=natural_key,
        how="left"
    )

    # A record changed if ANY tracked column differs
    changed_mask = pd.Series(False, index=merged.index)
    for col in tracked_cols:
        col_new = col
        col_old = col + "_old"
        if col_old in merged.columns:
            changed_mask |= (merged[col_new] != merged[col_old]).fillna(True)

    changed_keys = merged.loc[changed_mask, natural_key].tolist()
    new_keys     = new_snapshot[
        ~new_snapshot[natural_key].isin(existing_dim[natural_key])
    ][natural_key].tolist()

    print(f"SCD2 update [{effective_date}]: "
          f"{len(changed_keys)} changed, "
          f"{len(new_keys)} new, "
          f"{len(new_snapshot) - len(changed_keys) - len(new_keys)} unchanged")

    # Step 1: Expire old rows for changed records
    existing_dim.loc[
        (existing_dim[natural_key].isin(changed_keys)) &
        (existing_dim["is_current"] == 1),
        ["is_current", "effective_to"]
    ] = [0, effective_date]

    # Step 2: Build new rows for changed + new records
    new_rows = new_snapshot[
        new_snapshot[natural_key].isin(changed_keys + new_keys)
    ].copy()

    new_rows["effective_from"] = effective_date
    new_rows["effective_to"]   = None    # NULL = currently active
    new_rows["is_current"]     = 1

    # Assign new surrogate keys (extend from max existing SK)
    max_sk = existing_dim.get("sk", pd.Series([0])).max()
    new_rows["sk"] = range(int(max_sk) + 1,
                           int(max_sk) + 1 + len(new_rows))

    return pd.concat([existing_dim, new_rows], ignore_index=True)


# ── EXAMPLE ───────────────────────────────────────────────────────────────
# Initial dimension
dim_sellers = pd.DataFrame({
    "sk":             [1,      2      ],
    "seller_id":      ["S001", "S002" ],
    "seller_state":   ["SP",   "RJ"   ],
    "tier":           ["Gold", "Silver"],
    "effective_from": ["2023-01-01", "2023-01-01"],
    "effective_to":   [None,  None   ],
    "is_current":     [1,     1      ],
})

# New month: S001 promoted, S003 added
new_data = pd.DataFrame({
    "seller_id":    ["S001",     "S002",   "S003"  ],
    "seller_state": ["SP",       "RJ",     "MG"    ],
    "tier":         ["Platinum", "Silver", "Bronze" ],  # S001 changed, S002 same, S003 new
})

updated = scd_type2_update(
    new_data, dim_sellers,
    natural_key="seller_id",
    tracked_cols=["tier", "seller_state"],
    effective_date="2024-02-01"
)

print(updated[["sk","seller_id","tier","effective_from","effective_to","is_current"]])
# sk  seller_id  tier      effective_from  effective_to  is_current
# 1   S001       Gold      2023-01-01      2024-02-01    0    ← expired
# 2   S002       Silver    2023-01-01      None          1    ← unchanged
# 3   S001       Platinum  2024-02-01      None          1    ← new current
# 4   S003       Bronze    2024-02-01      None          1    ← brand new
```

---

## 14. Data Vault — Architecture and Working Example

### What is Data Vault?

Data Vault is a modelling methodology designed for enterprise data warehouses that need to be:
- **Agile** — source systems change, the warehouse must adapt without rewrites
- **Auditable** — every data point must be traceable to its source with load timestamps
- **Scalable** — parallel ingestion across many source systems

It was invented by Dan Linstedt in the 1990s and formalised as Data Vault 2.0. It is widely used in large European banks, insurance companies, and government systems — organisations where audit trails are legally required.

**The three building blocks:**

```
HUBS                LINKS                   SATELLITES
──────────          ──────────────────       ──────────────────────────────────
Core business       Relationships            Descriptive attributes +
entities with       between hubs             history of changes
unique biz keys

Hub_Customer        Link_OrderCustomer       Sat_Customer_Profile
─────────────       ──────────────────       ──────────────────────────────────
customer_hk   (PK)  link_hk        (PK)      customer_hk  (FK→Hub_Customer)
customer_bk         customer_hk    (FK)      load_date
load_date           order_hk       (FK)      record_source
record_source       load_date                name
                    record_source            email
                                             city
Hub_Order                                    state
─────────────
order_hk      (PK)  Link_OrderProduct        Sat_Customer_Status
order_bk            ──────────────────       ──────────────────────────────────
load_date           link_hk                  customer_hk
record_source       order_hk                 load_date
                    product_hk               record_source
                    load_date                vip_status
                    record_source            tier
                                             load_end_date  (NULL = current)
```

**Why this structure?**
- **Hubs** never change once loaded — they are the stable identity backbone
- **Links** capture when relationships are established — never deleted
- **Satellites** capture changes over time with full history — immutable, append-only

```python
import pandas as pd
import hashlib
from datetime import datetime

def hash_key(value: str, prefix: str = "") -> str:
    """
    Generate a surrogate hash key for Data Vault.
    In production, use MD5 or SHA-1 for performance.
    Hash keys are deterministic: same input always produces same hash.
    This enables parallel loading from multiple sources.
    """
    clean = f"{prefix}|{str(value).strip().upper()}"
    return hashlib.md5(clean.encode()).hexdigest()


def build_hub(
    df: pd.DataFrame,
    natural_key_col: str,
    hub_name: str,
    source: str,
) -> pd.DataFrame:
    """
    Build a Hub table.
    Hubs store unique business keys — nothing else.
    They are the stable identifiers that never change.
    """
    hub = df[[natural_key_col]].drop_duplicates().copy()

    # Hash key — deterministic surrogate key
    hub[f"{hub_name}_hk"] = hub[natural_key_col].apply(
        lambda x: hash_key(x, prefix=hub_name)
    )
    hub[f"{hub_name}_bk"] = hub[natural_key_col]    # business key (original)
    hub["load_date"]       = datetime.now().isoformat()
    hub["record_source"]   = source

    return hub[[f"{hub_name}_hk", f"{hub_name}_bk",
                "load_date", "record_source"]]


def build_link(
    df: pd.DataFrame,
    hub_key_cols: list,    # e.g., ["order_hk", "customer_hk"]
    link_name: str,
    source: str,
) -> pd.DataFrame:
    """
    Build a Link table capturing a relationship between two or more hubs.
    Links are append-only — once a relationship is recorded, it is never deleted.
    This preserves a complete history of all relationships that ever existed.
    """
    link = df[hub_key_cols].drop_duplicates().copy()

    # Composite hash key from all hub keys
    link[f"{link_name}_hk"] = link.apply(
        lambda row: hash_key("|".join(str(row[c]) for c in hub_key_cols),
                             prefix=link_name),
        axis=1
    )
    link["load_date"]     = datetime.now().isoformat()
    link["record_source"] = source

    return link[[f"{link_name}_hk"] + hub_key_cols + ["load_date", "record_source"]]


def build_satellite(
    df: pd.DataFrame,
    hub_key_col: str,
    attribute_cols: list,
    sat_name: str,
    source: str,
) -> pd.DataFrame:
    """
    Build a Satellite table with descriptive attributes.
    Satellites are append-only and track all historical changes.
    The "current" record has load_end_date = NULL (high date convention).

    When an attribute changes:
    - Old row: load_end_date is updated to the new load_date
    - New row: inserted with new values and load_end_date = NULL

    In a pure Data Vault, satellites are NEVER updated — they are only appended.
    The load_end_date pattern is an optimisation for query performance.
    """
    sat = df[[hub_key_col] + attribute_cols].copy()
    sat["load_date"]     = datetime.now().isoformat()
    sat["load_end_date"] = None    # NULL = current record
    sat["record_source"] = source

    # Hash difference — detect actual changes (don't insert if nothing changed)
    sat["hash_diff"] = sat[attribute_cols].apply(
        lambda row: hash_key("|".join(str(v) for v in row.values)),
        axis=1
    )

    return sat[[hub_key_col, "load_date", "load_end_date",
                "record_source", "hash_diff"] + attribute_cols]


# ── FULL WORKING EXAMPLE — Olist in Data Vault ────────────────────────────

import pandas as pd
from pathlib import Path

def build_olist_data_vault(data_dir: str) -> dict:
    """Build a Data Vault from Olist raw data."""
    orders    = pd.read_csv(f"{data_dir}/olist_orders_dataset.csv")
    customers = pd.read_csv(f"{data_dir}/olist_customers_dataset.csv")
    items     = pd.read_csv(f"{data_dir}/olist_order_items_dataset.csv")
    products  = pd.read_csv(f"{data_dir}/olist_products_dataset.csv")

    SOURCE = "olist_prod"

    # ── HUBS (one per business entity) ────────────────────────────────────
    hub_order    = build_hub(orders,    "order_id",                "Hub_Order",    SOURCE)
    hub_customer = build_hub(customers, "customer_unique_id",      "Hub_Customer", SOURCE)
    hub_product  = build_hub(products,  "product_id",              "Hub_Product",  SOURCE)
    hub_seller   = build_hub(items,     "seller_id",               "Hub_Seller",   SOURCE)

    # ── LINKS (relationships between entities) ────────────────────────────
    # Add hash keys to source tables for linking
    orders_with_hk = orders.copy()
    orders_with_hk["order_hk"] = orders["order_id"].apply(
        lambda x: hash_key(x, "Hub_Order"))

    cust_with_hk = customers.copy()
    cust_with_hk["customer_hk"] = customers["customer_unique_id"].apply(
        lambda x: hash_key(x, "Hub_Customer"))

    # Join orders to customer unique IDs
    link_source = orders_with_hk.merge(
        cust_with_hk[["customer_id", "customer_hk"]],
        on="customer_id", how="inner"
    )
    link_order_customer = build_link(
        link_source, ["order_hk", "customer_hk"],
        "Link_OrderCustomer", SOURCE
    )

    items_with_hk = items.copy()
    items_with_hk["order_hk"]   = items["order_id"].apply(lambda x: hash_key(x, "Hub_Order"))
    items_with_hk["product_hk"] = items["product_id"].apply(lambda x: hash_key(x, "Hub_Product"))
    items_with_hk["seller_hk"]  = items["seller_id"].apply(lambda x: hash_key(x, "Hub_Seller"))

    link_order_product = build_link(
        items_with_hk, ["order_hk", "product_hk", "seller_hk"],
        "Link_OrderLineItem", SOURCE
    )

    # ── SATELLITES (descriptive attributes) ───────────────────────────────
    orders_with_hk["order_hk"] = orders["order_id"].apply(
        lambda x: hash_key(x, "Hub_Order"))
    sat_order_details = build_satellite(
        orders_with_hk, "order_hk",
        ["order_status", "order_purchase_timestamp",
         "order_delivered_customer_date", "order_estimated_delivery_date"],
        "Sat_OrderDetails", SOURCE
    )

    cust_with_hk = cust_with_hk.merge(
        customers.drop_duplicates("customer_unique_id"),
        on="customer_id", how="left", suffixes=("", "_dup")
    )
    sat_customer_profile = build_satellite(
        cust_with_hk, "customer_hk",
        ["customer_city", "customer_state"],
        "Sat_CustomerProfile", SOURCE
    )

    vault = {
        "Hub_Order":           hub_order,
        "Hub_Customer":        hub_customer,
        "Hub_Product":         hub_product,
        "Hub_Seller":          hub_seller,
        "Link_OrderCustomer":  link_order_customer,
        "Link_OrderLineItem":  link_order_product,
        "Sat_OrderDetails":    sat_order_details,
        "Sat_CustomerProfile": sat_customer_profile,
    }

    for name, df in vault.items():
        print(f"  {name:<25}: {len(df):,} rows × {len(df.columns)} cols")

    return vault


### Data Vault vs Star Schema — when to choose which

| Dimension | Data Vault | Star Schema |
|-----------|-----------|-------------|
| **Query simplicity** | Complex — requires joining hub+link+sat | Simple — join fact to dimension |
| **Agility** | High — add satellites without touching existing tables | Low — schema changes affect existing models |
| **Load parallelism** | High — hubs, links, satellites load independently | Low — must build in order |
| **Audit trail** | Complete — every row has load_date + record_source | Partial — depends on SCD type |
| **Historical accuracy** | Perfect — every change is recorded | Depends on SCD type |
| **BI tool compatibility** | Poor — BI tools need flat tables | Excellent — designed for BI |
| **Team skills required** | High — specialist knowledge | Moderate — SQL + dimensional modelling |
| **Best for** | Large enterprise, regulatory compliance, many source systems | Analytics teams, BI-first, direct analyst access |

**Common pattern:** Build Data Vault for the integration layer (raw vault), then create **Information Marts** (star schemas) on top for BI tools to query. Best of both worlds — audit trail in the vault, query simplicity in the mart.
---

## 15. DAGs and Workflow Concepts

### Why orchestration exists — the problem it solves

Without orchestration, pipelines are scripts you run manually or via cron. They work fine with one pipeline. With ten pipelines that have interdependencies, they become a maintenance nightmare.

**The problems orchestration solves:**
- **Dependencies** — Pipeline B must run after Pipeline A finishes. If A fails, don't start B.
- **Scheduling** — Run at 6am daily, only on weekdays, on the 1st of every month.
- **Retries** — If a step fails due to a transient error (network blip), retry automatically.
- **Monitoring** — Know which pipeline is running, which failed, and how long each step took.
- **Backfill** — Re-run the pipeline for January 1–31 because the source data was corrected.
- **Parallelism** — Run independent steps at the same time to reduce total runtime.

---

### What is a DAG?

A **DAG (Directed Acyclic Graph)** is the data structure used to represent a pipeline's execution structure.

- **Directed**: arrows show direction (A must run before B)
- **Acyclic**: no cycles allowed (A → B → C → A would be a cycle — impossible)
- **Graph**: nodes are tasks; edges are dependencies

🎬 **Movie analogy:** Film production is a DAG. Casting must complete before principal photography begins. Sets must be built before filming on them. The score can't be composed until the final edit is locked. Sound mixing can't happen before the score is recorded. All tasks flow in one direction — toward the released film. No step depends on a step that comes after it.

```
Example pipeline DAG:

extract_orders ──┐
                 ├──▶ validate ──▶ transform ──▶ load ──▶ notify
extract_payments─┘                     │
                                       ▼
                               compute_rfm ──▶ update_crm

Properties visible here:
- extract_orders and extract_payments have no dependency between them → run PARALLEL
- validate depends on BOTH extracts → waits for both to complete before starting
- compute_rfm and load both depend on transform → can run in PARALLEL after transform
- notify waits for BOTH load and update_crm to complete
```

---

## 16. Apache Airflow

### What Airflow is — and why it dominates

**Apache Airflow** is an open-source platform for authoring, scheduling, monitoring, and backfilling data pipelines. Pipelines are defined as Python code — which means they can be version controlled, tested, reviewed, and dynamically generated.

Airflow was built at Airbnb in 2014, open-sourced in 2015, and became an Apache top-level project in 2019. It is the most widely adopted orchestration tool across the industry.

**What makes Airflow distinctive:**
- **Pipelines are Python code** — not YAML, not GUI-configured
- **Massive operator library** — pre-built connectors for BigQuery, Snowflake, S3, Spark, Postgres, HTTP APIs, and hundreds more
- **Web UI** — visual DAG graph, task logs, run history, backfill controls
- **Backfill** — trivially re-run any historical date range
- **Execution date model** — every run has a logical date (which date's data am I processing?), decoupled from the actual clock time

---

### Airflow architecture — how it works internally

Understanding Airflow's architecture is important because it explains its behaviour, its limitations, and common production issues.

```
┌─────────────────────────────────────────────────────────────┐
│                    AIRFLOW ARCHITECTURE                      │
├──────────────┬──────────────┬──────────────┬────────────────┤
│  Web Server  │  Scheduler   │   Workers    │ Metadata DB    │
│              │              │              │                │
│ Flask app    │ Scans DAG    │ Execute the  │ PostgreSQL /   │
│ Shows UI:    │ files every  │ actual task  │ MySQL          │
│ - DAG graph  │ N seconds    │ code         │                │
│ - Run history│              │              │ Stores:        │
│ - Task logs  │ Creates task │ Can be:      │ - DAG runs     │
│ - Trigger    │ instances    │ - Same host  │ - Task states  │
│   manual     │              │   (Local)    │ - Variables    │
│   runs       │ Monitors     │ - Remote via │ - Connections  │
│              │ states       │   Celery/K8s │                │
└──────────────┴──────────────┴──────────────┴────────────────┘
```

**Key components:**

| Component | Role | What breaks if it fails |
|-----------|------|------------------------|
| **Web Server** | Flask app serving the Airflow UI | UI down, but pipelines keep running |
| **Scheduler** | Scans DAG files, creates DagRun and TaskInstance records, triggers tasks | Pipelines stop scheduling entirely |
| **Workers** | Actually run the Python functions your tasks define | Tasks queue up and never execute |
| **Metadata DB** | PostgreSQL/MySQL — stores all state (run history, task status, etc.) | Entire Airflow stops working |
| **DAG folder** | Directory where your .py DAG files live | Scheduler can't find new DAGs |

**Executor types:**

| Executor | How it works | When to use |
|----------|-------------|-------------|
| **LocalExecutor** | Tasks run as subprocesses on the same machine as the scheduler | Development, small teams, < 10 concurrent tasks |
| **CeleryExecutor** | Tasks distributed to worker nodes via a message queue (Redis/RabbitMQ) | Production, multi-machine, many concurrent tasks |
| **KubernetesExecutor** | Each task runs in its own Kubernetes pod | Kubernetes environments; perfect isolation per task |
| **SequentialExecutor** | Tasks run one at a time, same process | Development only — too slow for production |

---

### Writing Airflow DAGs — full reference

```python
# dag_olist_daily.py
# Place this in your Airflow DAGs folder (~/airflow/dags/ or configured folder)

from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python  import PythonOperator, BranchPythonOperator
from airflow.operators.email   import EmailOperator
from airflow.operators.bash    import BashOperator
from airflow.sensors.filesystem import FileSensor
import pandas as pd
import logging

logger = logging.getLogger(__name__)

# ── DEFAULT ARGUMENTS ─────────────────────────────────────────────────────
# Applied to every task in the DAG unless overridden at task level
default_args = {
    "owner":             "data-engineering",
    "depends_on_past":   False,          # each run is independent (don't block on prior failure)
    "email":             ["alerts@company.com"],
    "email_on_failure":  True,           # email if a task fails
    "email_on_retry":    False,          # don't email on retry (too noisy)
    "retries":           2,              # retry a failed task 2 times
    "retry_delay":       timedelta(minutes=5),     # wait 5 min between retries
    "retry_exponential_backoff": True,   # double wait each retry: 5min, 10min
    "max_retry_delay":   timedelta(hours=1),
    "execution_timeout": timedelta(hours=1),       # kill task if it runs > 1 hour
}

# ── DAG DEFINITION ────────────────────────────────────────────────────────
with DAG(
    dag_id="olist_daily_pipeline",
    default_args=default_args,
    description="Daily Olist E-Commerce ETL pipeline",
    schedule_interval="0 6 * * *",    # every day at 06:00 UTC
    start_date=datetime(2024, 1, 1),
    catchup=False,                     # don't backfill historical missed runs
    max_active_runs=1,                 # only one active run at a time (prevent overlap)
    tags=["olist", "daily", "etl"],
    doc_md="""
    ## Olist Daily ETL Pipeline
    Extracts Olist e-commerce data, transforms it, loads to warehouse.
    SLA: data ready by 07:00 UTC.
    Owner: data-engineering@company.com
    """,
) as dag:

    # ── TASK FUNCTIONS ────────────────────────────────────────────────────
    # The actual work lives in these functions.
    # **context contains Airflow metadata: execution date, task instance, etc.

    def extract(**context):
        """Extract Olist data for execution date."""
        ds = context["ds"]              # execution date as "YYYY-MM-DD" string
        ti = context["ti"]             # task instance — use for XCom

        logger.info(f"Extracting for {ds}")
        df = pd.read_csv(f"data/olist/orders_{ds}.csv")

        # Push the output path to XCom so the next task can find it
        output_path = f"/tmp/olist_raw_{ds}.parquet"
        df.to_parquet(output_path, index=False)
        ti.xcom_push(key="raw_path", value=output_path)

        logger.info(f"Extracted {len(df):,} rows → {output_path}")


    def validate(**context):
        """Run quality checks on extracted data."""
        ds      = context["ds"]
        ti      = context["ti"]
        raw_path = ti.xcom_pull(task_ids="extract", key="raw_path")

        df = pd.read_parquet(raw_path)

        # Critical checks — raise AirflowException to fail the task
        from airflow.exceptions import AirflowException
        if len(df) < 10:
            raise AirflowException(f"Only {len(df)} rows for {ds} — expected ≥ 10")
        if df["order_id"].duplicated().sum() > 0:
            raise AirflowException("Duplicate order_ids detected")
        if df["order_id"].isnull().sum() > 0:
            raise AirflowException("Null order_ids detected")

        ti.xcom_push(key="validation_passed", value=True)
        logger.info(f"Validation passed: {len(df):,} rows OK")


    def transform(**context):
        """Transform raw data into analytics-ready form."""
        ds       = context["ds"]
        ti       = context["ti"]
        raw_path = ti.xcom_pull(task_ids="extract", key="raw_path")

        df = pd.read_parquet(raw_path)
        df["order_date"] = pd.to_datetime(df["order_purchase_timestamp"])
        df["order_month"]= df["order_date"].dt.to_period("M").astype(str)
        # ... more transforms ...

        clean_path = f"/tmp/olist_clean_{ds}.parquet"
        df.to_parquet(clean_path, index=False)
        ti.xcom_push(key="clean_path", value=clean_path)
        ti.xcom_push(key="row_count",  value=len(df))


    def load(**context):
        """Load clean data to DuckDB warehouse."""
        import duckdb, os

        ds         = context["ds"]
        ti         = context["ti"]
        clean_path = ti.xcom_pull(task_ids="transform", key="clean_path")

        df  = pd.read_parquet(clean_path)
        con = duckdb.connect(os.environ["WAREHOUSE_PATH"])

        # Idempotent: delete partition then insert
        con.execute("DELETE FROM fct_orders WHERE order_month = ?", [ds[:7]])
        con.register("_df", df)
        con.execute("INSERT INTO fct_orders SELECT * FROM _df")
        con.close()

        rows_loaded = len(df)
        ti.xcom_push(key="rows_loaded", value=rows_loaded)
        logger.info(f"Loaded {rows_loaded:,} rows for {ds}")


    def post_load_check(**context):
        """Verify the data landed correctly in the warehouse."""
        import duckdb, os

        ds          = context["ds"]
        ti          = context["ti"]
        rows_loaded = ti.xcom_pull(task_ids="load", key="rows_loaded")

        con          = duckdb.connect(os.environ["WAREHOUSE_PATH"])
        rows_in_db   = con.execute(
            "SELECT COUNT(*) FROM fct_orders WHERE order_month = ?", [ds[:7]]
        ).fetchone()[0]
        con.close()

        if rows_in_db != rows_loaded:
            from airflow.exceptions import AirflowException
            raise AirflowException(
                f"Row count mismatch: loaded {rows_loaded}, found {rows_in_db} in DB"
            )
        logger.info(f"Post-load check passed: {rows_in_db:,} rows confirmed")


    # ── TASK INSTANCES ────────────────────────────────────────────────────
    t_extract   = PythonOperator(task_id="extract",         python_callable=extract)
    t_validate  = PythonOperator(task_id="validate",        python_callable=validate)
    t_transform = PythonOperator(task_id="transform",       python_callable=transform)
    t_load      = PythonOperator(task_id="load",            python_callable=load)
    t_dq        = PythonOperator(task_id="post_load_check", python_callable=post_load_check)
    t_notify    = EmailOperator(
        task_id="notify_success",
        to=["analytics@company.com"],
        subject="Olist pipeline {{ ds }} completed",
        html_content="""
            <h3>Olist Daily Pipeline — SUCCESS</h3>
            <p>Date: {{ ds }}</p>
            <p>Rows loaded: {{ ti.xcom_pull(task_ids='load', key='rows_loaded') }}</p>
        """,
    )

    # ── DEPENDENCY CHAIN ──────────────────────────────────────────────────
    # >> operator means "must run before"
    t_extract >> t_validate >> t_transform >> t_load >> t_dq >> t_notify
```

---

### XCom — passing data between tasks

**XCom (cross-communication)** is Airflow's mechanism for tasks to share small pieces of data. XCom values are stored in the metadata database and can be pushed by any task and pulled by any downstream task.

```python
# Pushing a value
context["ti"].xcom_push(key="row_count", value=10_000)
context["ti"].xcom_push(key="output_path", value="/tmp/data.parquet")

# Pulling a value (must know the producing task_id)
row_count   = context["ti"].xcom_pull(task_ids="transform", key="row_count")
output_path = context["ti"].xcom_pull(task_ids="extract",   key="raw_path")

# Implicit return value XCom (task return value is auto-stored)
def my_task():
    return {"rows": 100, "path": "/tmp/data.parquet"}   # stored as XCom automatically

result = context["ti"].xcom_pull(task_ids="my_task")    # pulls the returned dict
```

**XCom limitations — important to understand:**

| Limitation | Why it matters | Solution |
|-----------|---------------|---------|
| Stored in metadata DB | Max size ~48KB (MySQL MEDIUMBLOB) | Never push large DataFrames via XCom |
| Serialised as pickle or JSON | Complex objects may not serialise | Push file paths, not DataFrames |
| Not designed for large data | Passing a 10M row DataFrame via XCom will corrupt Airflow | Write to disk/S3, push the path |

> **Rule:** XCom should carry **metadata** (paths, counts, timestamps, flags), never actual data payloads. Tasks communicate via shared storage (disk, S3, DuckDB) — XCom just passes the location.

---

### Airflow common pitfalls

| Pitfall | What happens | How to avoid |
|---------|-------------|--------------|
| **Importing at module level** | Slow DAG parsing — imports run every time scheduler scans | Import inside task functions |
| **Top-level code that takes time** | Scheduler runs DAG file to discover structure; slow = slow UI | Keep top-level code instant |
| **Passing data via XCom** | XCom is limited to ~48KB; large data crashes metadata DB | Pass file paths, not data |
| **`catchup=True` on a new DAG with old start_date** | Creates hundreds of DagRuns immediately | Always set `catchup=False` unless you explicitly need backfill |
| **`depends_on_past=True` + first run fails** | All subsequent runs are permanently blocked | Use only when truly necessary; monitor carefully |
| **Not setting `max_active_runs`** | Two daily runs for the same pipeline overlap, causing race conditions | Always set `max_active_runs=1` for sequential pipelines |
| **Resource leaks in tasks** | File handles, DB connections not closed → worker memory leak | Use context managers (`with open()`, `with engine.connect()`) |

---

## 17. Prefect — Python-native Orchestration

### What Prefect is and why it exists

**Prefect** is a Python-native workflow orchestration tool that was built explicitly to address Airflow's pain points. Released in 2019 by Jeremiah Lowin (former Airflow contributor), Prefect 2.0 (2022) redesigned the entire model.

**What makes Prefect different from Airflow:**

| Feature | Airflow | Prefect |
|---------|---------|---------|
| Definition | DAG files in a special folder | Any Python script decorated with `@flow` |
| Execution model | Scheduler polls DAG files; workers run tasks | Any Python process can be a worker |
| Local development | Needs Airflow running to test | Run locally with `python my_flow.py` |
| Dynamic workflows | Hard (requires DAG generation patterns) | Native — flows can dynamically create tasks |
| Error handling | Caught at task boundary only | Full Python exception handling available |
| Deployment | Deploy to Airflow server | Deploy to Prefect Cloud or self-hosted |
| Learning curve | Steep (many Airflow-specific concepts) | Gentle (mostly Python you already know) |

```python
# pip install prefect

from prefect import flow, task
from prefect.tasks import task_input_hash
from datetime import timedelta
import pandas as pd

# @task decorator turns a function into a Prefect task
# retries, cache, timeout are all configured here

@task(
    retries=3,
    retry_delay_seconds=60,
    cache_key_fn=task_input_hash,        # cache: don't re-run if inputs unchanged
    cache_expiration=timedelta(hours=1),
    log_prints=True,
)
def extract_orders(run_date: str) -> str:
    """Extract and save raw orders for run_date."""
    df   = pd.read_csv(f"data/olist/orders_{run_date}.csv")
    path = f"/tmp/orders_raw_{run_date}.parquet"
    df.to_parquet(path, index=False)
    print(f"Extracted {len(df):,} rows")
    return path


@task(retries=2)
def transform_orders(raw_path: str, run_date: str) -> str:
    df = pd.read_parquet(raw_path)
    df["order_date"]  = pd.to_datetime(df["order_purchase_timestamp"])
    df["order_month"] = df["order_date"].dt.to_period("M").astype(str)
    clean_path = f"/tmp/orders_clean_{run_date}.parquet"
    df.to_parquet(clean_path, index=False)
    print(f"Transformed {len(df):,} rows")
    return clean_path


@task(retries=2)
def load_orders(clean_path: str, run_date: str) -> int:
    import duckdb
    df  = pd.read_parquet(clean_path)
    con = duckdb.connect("data/warehouse.ddb")
    con.execute("DELETE FROM fct_orders WHERE date_str = ?", [run_date])
    con.register("_df", df)
    con.execute("INSERT INTO fct_orders SELECT *, ? as date_str FROM _df", [run_date])
    rows = len(df)
    con.close()
    print(f"Loaded {rows:,} rows")
    return rows


# @flow is the pipeline — it orchestrates tasks
@flow(
    name="olist-daily",
    description="Daily Olist ETL pipeline",
    log_prints=True,
)
def olist_daily_pipeline(run_date: str = None):
    from datetime import date, timedelta
    run_date = run_date or str(date.today() - timedelta(1))

    # Tasks called like regular Python functions — Prefect handles orchestration
    raw_path   = extract_orders(run_date)
    clean_path = transform_orders(raw_path, run_date)
    rows       = load_orders(clean_path, run_date)

    print(f"Pipeline complete: {rows:,} rows for {run_date}")
    return rows


# Run locally — no server needed
if __name__ == "__main__":
    olist_daily_pipeline()

# Schedule via Prefect Cloud or server:
# from prefect.deployments import Deployment
# from prefect.server.schemas.schedules import CronSchedule
# deployment = Deployment.build_from_flow(
#     flow=olist_daily_pipeline,
#     name="olist-daily",
#     schedule=CronSchedule(cron="0 6 * * *"),
# )
# deployment.apply()
```

**When to choose Prefect over Airflow:**
- Your team is Python-first and finds Airflow's DAG model frustrating
- You need dynamic task generation (tasks created at runtime based on data)
- You want to develop and test locally without running an Airflow server
- Small-to-medium team; don't want to manage Airflow infrastructure

**When Airflow still wins:**
- Large enterprise with existing Airflow investment
- You need the extensive Airflow operator library
- Your team has strong Airflow experience
- You're running in a cloud environment with managed Airflow (Cloud Composer, MWAA)

---

## 18. Dagster — Asset-Oriented Orchestration

### What Dagster is and the asset model

**Dagster** is an open-source data orchestration platform built around the concept of **data assets** rather than tasks. Released by Elementl in 2018, Dagster 1.0 in 2022 introduced the "Software-Defined Asset" (SDA) model which fundamentally changed how people think about pipeline orchestration.

**The core insight:** most pipelines are really about producing data assets (tables, files, models). The orchestration is secondary. Dagster flips this: you define the assets you want to produce, and Dagster figures out how to run the code to produce them.

**Airflow/Prefect model:** "Run this task at 6am. If it succeeds, run this other task."

**Dagster model:** "I want to produce `fct_orders`. `fct_orders` depends on `stg_orders` and `dim_customer`. Tell me when any upstream asset changes, so I know when to recompute."

```python
# pip install dagster dagster-webserver

from dagster import asset, AssetIn, Definitions, ScheduleDefinition, define_asset_job
import pandas as pd
import duckdb

# @asset defines a data asset — a table, file, or object Dagster should produce
# The function name IS the asset name

@asset(
    description="Raw extracted orders from Olist CSV files",
    group_name="raw",
)
def raw_orders() -> pd.DataFrame:
    """Extract raw orders. Returns DataFrame."""
    df = pd.read_csv("data/olist/olist_orders_dataset.csv",
                     parse_dates=["order_purchase_timestamp"])
    return df   # Dagster handles storage (you configure IOManagers)


@asset(
    description="Raw payment data aggregated to order level",
    group_name="raw",
)
def raw_payments() -> pd.DataFrame:
    df = pd.read_csv("data/olist/olist_order_payments_dataset.csv")
    return df.groupby("order_id")["payment_value"].sum().reset_index(name="total_payment")


@asset(
    ins={
        "orders":   AssetIn("raw_orders"),    # depends on raw_orders
        "payments": AssetIn("raw_payments"),  # depends on raw_payments
    },
    description="Cleaned and enriched orders fact table",
    group_name="staging",
)
def stg_orders(orders: pd.DataFrame, payments: pd.DataFrame) -> pd.DataFrame:
    """
    Transform raw orders and join payments.
    Dagster automatically passes the upstream assets as arguments.
    """
    df = orders.merge(payments, on="order_id", how="left")
    df["order_month"] = df["order_purchase_timestamp"].dt.to_period("M").astype(str)
    df["delivery_days"] = (
        (df["order_delivered_customer_date"] - df["order_purchase_timestamp"]).dt.days
    )
    return df


@asset(
    ins={"stg": AssetIn("stg_orders")},
    description="Monthly revenue aggregation for BI dashboards",
    group_name="marts",
)
def monthly_revenue(stg: pd.DataFrame) -> pd.DataFrame:
    return stg.groupby("order_month").agg(
        orders  = ("order_id",     "count"),
        revenue = ("total_payment","sum"),
    ).reset_index()


# ── Dagster Definitions — register everything ─────────────────────────────
defs = Definitions(
    assets=[raw_orders, raw_payments, stg_orders, monthly_revenue],
    schedules=[
        ScheduleDefinition(
            job=define_asset_job("olist_daily", selection="*"),
            cron_schedule="0 6 * * *",
        )
    ],
)

# Run locally:
# dagster dev -f my_pipeline.py
# Opens UI at localhost:3000 showing your asset graph, lineage, run history
```

**What Dagster's UI shows that Airflow doesn't:**
- **Asset lineage graph** — visually shows which asset depends on which, end-to-end across all pipelines
- **Asset freshness** — which assets are stale (upstream changed but downstream not recomputed)?
- **Asset materialisation history** — when was each asset last produced? How long did it take? How many rows?
- **Partition support** — natively understands "I want to produce monthly_revenue for every month from 2020–2024"

**Orchestration tools — the decision framework:**

| Question | Points to... |
|----------|-------------|
| "We already have Airflow running in production" | Airflow |
| "We want to quickly build pipelines in pure Python" | Prefect |
| "We want to manage data assets, not just tasks" | Dagster |
| "We want the best lineage and observability" | Dagster |
| "We need 300+ pre-built connectors (operators)" | Airflow |
| "We're on GCP with Cloud Composer" | Airflow (Cloud Composer is managed Airflow) |
| "We're on AWS with MWAA" | Airflow (MWAA is managed Airflow) |
| "We're dbt-first, want simple scheduling" | dbt Cloud |

---

### Cron — the simplest orchestrator

Before any of the above tools existed, pipelines were scheduled with **cron**, a Unix utility that runs commands on a schedule. cron is still widely used for simple, single-machine pipelines.

```bash
# Edit your crontab: crontab -e

# Format: minute hour day_of_month month day_of_week command
# ┌───────────── minute (0–59)
# │  ┌────────── hour (0–23)
# │  │  ┌─────── day of month (1–31)
# │  │  │  ┌──── month (1–12)
# │  │  │  │  ┌─ day of week (0=Sunday, 6=Saturday)
# │  │  │  │  │
# *  *  *  *  *  command

# Run daily at 6am
0 6 * * * python /pipelines/olist_daily.py >> /logs/olist.log 2>&1

# Run every weekday at 8am
0 8 * * 1-5 python /pipelines/report.py

# Run on the 1st of every month at midnight
0 0 1 * * python /pipelines/monthly_close.py

# Run every 15 minutes
*/15 * * * * python /pipelines/sensor.py
```

**When cron is appropriate:**
- One-server environment
- Simple pipelines with no dependencies
- No retry needed (or retry logic is in the script itself)
- No web UI needed

**When cron is NOT appropriate:**
- Pipelines with dependencies between them
- You need retries on failure
- You need to monitor multiple pipelines
- Anything that runs on multiple machines

---

### Scheduling reference — cron expressions

```
Expression          Meaning
──────────────────────────────────────────────────────────
0 6 * * *           Every day at 06:00
0 */6 * * *         Every 6 hours (00:00, 06:00, 12:00, 18:00)
0 6 * * 1-5         Weekdays (Mon–Fri) at 06:00
0 6 * * 1           Every Monday at 06:00
0 6 * * 0,6         Weekends (Sat, Sun) at 06:00
0 0 1 * *           First day of every month at midnight
0 0 1 1 *           January 1st every year at midnight
*/15 * * * *        Every 15 minutes
0 6,12,18 * * *     Three times a day: 06:00, 12:00, 18:00
0 6 * * 1-5 [week#<52]  Every weekday, all year (advanced)

Airflow special values (not standard cron):
@daily     = 0 0 * * *      (midnight every day)
@hourly    = 0 * * * *      (top of every hour)
@weekly    = 0 0 * * 0      (midnight every Sunday)
@monthly   = 0 0 1 * *      (midnight on the 1st)
@yearly    = 0 0 1 1 *      (midnight on Jan 1st)
None       = manual trigger only (no schedule)
```

---
---

## 18. Data Quality

### Why data quality matters more than anything else

Bad data causes wrong decisions. A 2% error in revenue data means executives are making strategy decisions on incorrect numbers. Silently bad data is worse than no data — at least with no data you know you're guessing.

**The five dimensions of data quality — with real consequences:**

| Dimension | Definition | Bad example | Business consequence |
|-----------|-----------|-------------|---------------------|
| **Completeness** | All expected data is present | 10% of orders have no customer_id | Customer-level analysis is biased |
| **Accuracy** | Values are correct | Revenue shows $100 when it should be $1,000 (decimal error) | Strategy built on wrong numbers |
| **Consistency** | Same fact represented the same way across systems | Order shows "delivered" in DB but "processing" in API | Conflicting reports |
| **Timeliness** | Data available when needed | Yesterday's data arrives at noon instead of 6am | Morning dashboards are empty |
| **Uniqueness** | No duplicate records | Same order_id appears twice with different amounts | Revenue double-counted |

---

### The Great Expectations framework — architecture

**Great Expectations (GX)** is the most widely used Python library for data quality. Understanding its architecture matters because GX has specific terminology that appears everywhere in data quality conversations.

**GX core concepts:**

| Concept | What it is | Analogy |
|---------|-----------|---------|
| **Expectation** | A single testable assertion: "this column should not be null" | One test case |
| **Expectation Suite** | A named collection of expectations for one table | A test file |
| **Data Source** | A connection to data (DataFrame, database, file) | The subject under test |
| **Batch** | The actual data being validated at a point in time | One day's data file |
| **Validator** | Runs the suite against a batch | Test runner |
| **Checkpoint** | A saved configuration: "run this suite against this data source" | A test pipeline |
| **Data Docs** | Auto-generated HTML documentation of all expectations and their pass/fail history | Test report |

```
GX validation flow:

Data Source (DW, file, DF)
       ↓
    Batch (today's data)
       ↓
  Validator runs Expectation Suite
       ↓
   Validation Result
       ↓
  Checkpoint (save config + notify)
       ↓
  Data Docs (rendered HTML report)
```

```python
# pip install great_expectations
import great_expectations as gx
import pandas as pd

# ── CREATE CONTEXT ────────────────────────────────────────────────────────
# Context manages all GX configuration
context = gx.get_context()

# ── DEFINE DATA SOURCE ────────────────────────────────────────────────────
# Pandas in-memory DataFrame source (simplest for learning)
ds = context.sources.add_pandas("olist_orders_source")

# ── BUILD EXPECTATION SUITE ───────────────────────────────────────────────
suite = context.suites.add(gx.ExpectationSuite(name="olist_orders_suite"))

# Add expectations
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToNotBeNull(column="order_id")
)
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToBeUnique(column="order_id")
)
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToBeBetween(
        column="total_payment", min_value=0, max_value=100_000
    )
)
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToBeInSet(
        column="order_status",
        value_set={"delivered", "shipped", "processing", "cancelled",
                   "invoiced", "unavailable", "approved", "created"}
    )
)
suite.add_expectation(
    gx.expectations.ExpectTableRowCountToBeBetween(
        min_value=100, max_value=200_000
    )
)
suite.add_expectation(
    gx.expectations.ExpectColumnValuesToMatchRegex(
        column="order_id",
        regex=r"^[a-f0-9]{32}$"    # Olist order IDs are 32-char hex strings
    )
)

# ── RUN VALIDATION ────────────────────────────────────────────────────────
df = pd.read_csv("data/olist/olist_orders_dataset.csv")
batch_request = ds.add_dataframe_asset("orders").build_batch_request(dataframe=df)

validator = context.get_validator(
    batch_request=batch_request,
    expectation_suite_name="olist_orders_suite"
)
result = validator.validate()

# ── INSPECT RESULTS ───────────────────────────────────────────────────────
print(f"Validation success: {result.success}")
print(f"Statistics: {result['statistics']}")

for er in result.results:
    if not er.success:
        print(f"FAILED: {er.expectation_config.expectation_type}")
        print(f"  Column: {er.expectation_config.kwargs.get('column')}")
        print(f"  Details: {er.result}")
```

---

### DIY quality framework — for simpler use cases

```python
from dataclasses import dataclass, field
from typing import List, Callable
import pandas as pd

@dataclass
class QualityCheck:
    """A single data quality rule."""
    name:        str
    check_fn:    Callable[[pd.DataFrame], bool]
    severity:    str = "error"    # "error" = fail pipeline; "warning" = log and continue
    description: str = ""

@dataclass
class QualityReport:
    """Result of running a suite of quality checks."""
    table_name: str
    run_date:   str
    passed:     List[str] = field(default_factory=list)
    warnings:   List[str] = field(default_factory=list)
    errors:     List[str] = field(default_factory=list)

    @property
    def is_ok(self) -> bool:
        return len(self.errors) == 0

    def print_summary(self) -> None:
        total = len(self.passed) + len(self.warnings) + len(self.errors)
        status = "✓ PASSED" if self.is_ok else "✗ FAILED"
        print(f"\n{status}: {self.table_name} — {len(self.passed)}/{total} checks OK")
        for e in self.errors:
            print(f"  ERROR:   {e}")
        for w in self.warnings:
            print(f"  WARNING: {w}")


def run_quality_checks(
    df: pd.DataFrame,
    checks: List[QualityCheck],
    table_name: str,
    run_date:   str,
) -> QualityReport:
    report = QualityReport(table_name=table_name, run_date=run_date)

    for check in checks:
        try:
            passed = check.check_fn(df)
        except Exception as e:
            passed = False
            print(f"  Check '{check.name}' raised exception: {e}")

        if passed:
            report.passed.append(check.name)
        elif check.severity == "warning":
            report.warnings.append(f"{check.name}: {check.description}")
        else:
            report.errors.append(f"{check.name}: {check.description}")

    report.print_summary()
    return report


# Example checks for Olist orders
OLIST_CHECKS = [
    QualityCheck(
        name="order_id_not_null",
        check_fn=lambda df: df["order_id"].isnull().sum() == 0,
        description="order_id must never be null",
    ),
    QualityCheck(
        name="order_id_unique",
        check_fn=lambda df: df["order_id"].duplicated().sum() == 0,
        description=f"duplicate order_ids found",
    ),
    QualityCheck(
        name="payment_non_negative",
        check_fn=lambda df: (df["total_payment"] >= 0).all(),
        description="total_payment must be ≥ 0",
    ),
    QualityCheck(
        name="row_count_minimum",
        check_fn=lambda df: len(df) >= 100,
        description="must have at least 100 rows",
    ),
    QualityCheck(
        name="delivery_days_reasonable",
        check_fn=lambda df: (df["delivery_days"].dropna() <= 90).all(),
        description="delivery_days > 90 is suspicious — investigate",
        severity="warning",
    ),
    QualityCheck(
        name="payment_null_rate",
        check_fn=lambda df: df["total_payment"].isnull().mean() < 0.01,
        description="total_payment null rate must be < 1%",
    ),
]
```

---

## 21. Data Warehouses vs Data Lakes vs Lakehouses

### Three storage paradigms — explained with real trade-offs

```
DATA LAKE                    DATA WAREHOUSE             DATA LAKEHOUSE
──────────────────           ────────────────────       ─────────────────────────────
Raw files: CSV, JSON,        Structured SQL tables.     Files (like lake) +
Parquet — any format.        Schema strictly enforced.  transaction layer (like DW).
No fixed schema.             Schema-on-write.
Schema-on-read.
                             Optimised for analytics    ACID transactions.
Cheap storage.               queries. Often columnar    Time travel. Schema evolution.
~$0.02/GB/month (S3).        storage internally.        SQL queryable.
                                                         Same cheap file storage.
Not directly SQL-queryable   Directly SQL-queryable.
without a query engine.                                 SQL queryable through engine.
                             Compute is bundled
Separate compute engines     with storage — you pay     Compute and storage
(Athena, Spark) query it.   for the warehouse.         are separated.

Best for:                    Best for:                  Best for:
Raw storage, ML training,    BI, reporting, SQL         Both ML/DE AND BI from
archives, data that isn't    analytics, dashboards,     the same data. Modern
ready yet. "Data before it   structured business data.  standard for new builds.
has a purpose."

Examples:                    Examples:                  Examples:
Amazon S3                    Snowflake                  Databricks (Delta Lake)
Google Cloud Storage         Google BigQuery            Apache Iceberg on S3
Azure ADLS Gen2              Amazon Redshift            AWS Lake Formation
                             Azure Synapse              Delta Lake on Azure
```

---

## 22. DuckDB — In-Process OLAP Engine

### What DuckDB is — and why it matters

**DuckDB** is an in-process SQL OLAP database. "In-process" means it runs inside your Python program — there is no server, no connection string to configure, no port to open. It's like SQLite, but designed for analytical queries rather than transactional operations.

**What makes DuckDB remarkable:**
- Zero-installation: `pip install duckdb`, nothing else
- Queries Parquet files on disk without loading them into memory
- Columnar, vectorised execution engine — same architectural principles as BigQuery
- Full SQL including window functions, CTEs, UNNEST, JSON functions
- Reads from pandas DataFrames directly (zero copy in many cases)
- Can query multiple Parquet files simultaneously with SQL
- Significantly faster than pandas for GROUP BY, aggregations, and large JOINs

**How DuckDB works internally:**

DuckDB uses **vectorised columnar execution**. Instead of processing one row at a time (like most traditional databases), it processes a batch of values from one column at a time. This maps extremely well to modern CPU SIMD instructions (Single Instruction, Multiple Data) and cache behaviour.

```
Traditional row-at-a-time (interpreted) execution:
Row 1: fetch → evaluate predicate → maybe aggregate → move to row 2 → ...
→ One CPU operation per row

DuckDB vectorised execution:
Fetch 1024 values from "amount" column as a vector
Apply predicate to all 1024 at once using SIMD
Sum all matching values in one CPU instruction
→ 1 CPU operation per 1024 rows = ~1000× more efficient per cycle
```

```python
# pip install duckdb
import duckdb
import pandas as pd

# ── CONNECTION ────────────────────────────────────────────────────────────
con = duckdb.connect(":memory:")         # in-memory (no persistence)
con = duckdb.connect("analytics.ddb")   # file-backed (persists across sessions)

# ── QUERY CSV DIRECTLY — no loading needed ────────────────────────────────
result = con.execute("""
    SELECT
        order_status,
        COUNT(*) AS orders,
        ROUND(AVG(payment_value), 2) AS avg_payment
    FROM read_csv_auto('data/olist_orders.csv')
    GROUP BY order_status
    ORDER BY orders DESC
""").df()

# ── QUERY PARQUET — including partitioned datasets ────────────────────────
result = con.execute("""
    SELECT year, month, SUM(revenue) AS total_revenue
    FROM read_parquet('data/warehouse/orders/**/*.parquet')
    WHERE year = 2024
    GROUP BY year, month
    ORDER BY year, month
""").df()

# ── QUERY PANDAS DATAFRAMES DIRECTLY ────────────────────────────────────
df = pd.read_csv("data/olist_orders.csv")
con.register("orders", df)         # register DataFrame as a virtual table

result = con.execute("""
    SELECT
        DATE_TRUNC('month', CAST(order_purchase_timestamp AS TIMESTAMP)) AS month,
        COUNT(*) AS orders,
        SUM(payment_value) AS revenue
    FROM orders
    WHERE order_status = 'delivered'
    GROUP BY 1
    ORDER BY 1
""").df()

# ── JOIN MULTIPLE SOURCES ─────────────────────────────────────────────────
orders    = pd.read_csv("data/olist_orders.csv")
payments  = pd.read_csv("data/olist_order_payments.csv")
customers = pd.read_csv("data/olist_customers.csv")

con.register("orders",    orders)
con.register("payments",  payments)
con.register("customers", customers)

result = con.execute("""
    SELECT
        c.customer_state,
        COUNT(DISTINCT o.order_id)       AS orders,
        ROUND(SUM(p.payment_value), 2)   AS revenue,
        ROUND(AVG(p.payment_value), 2)   AS avg_order
    FROM orders o
    JOIN customers c  ON o.customer_id = c.customer_id
    JOIN payments  p  ON o.order_id    = p.order_id
    WHERE o.order_status = 'delivered'
    GROUP BY c.customer_state
    ORDER BY revenue DESC
    LIMIT 10
""").df()

# ── DuckDB vs pandas — when to use each ──────────────────────────────────
# pandas:    < 500MB, row operations, ML preprocessing, string manipulation
# DuckDB:    aggregations, GROUP BY, JOINs, anything SQL-shaped
# Both:      use both — DuckDB for SQL aggregation, pandas for the result
```

**DuckDB vs alternatives:**

| | DuckDB | pandas | Spark | BigQuery |
|--|--------|--------|-------|---------|
| Setup | `pip install` | `pip install` | Complex cluster setup | Cloud account |
| Scale | Single machine (up to ~50GB RAM) | Single machine | Distributed | Distributed |
| SQL support | Full SQL | Limited (via .query()) | Spark SQL (slightly different) | Standard SQL |
| Speed (aggregations) | 10-100× faster than pandas | Baseline | Fast at scale | Fastest at scale |
| Cost | Free | Free | Cloud compute | Pay per query |
| Best for | Local OLAP, dev, medium data | Row ops, ML prep | Big data at scale | Serverless analytics |

---

## 23. Snowflake — Cloud Data Warehouse Deep Dive

### What Snowflake is

**Snowflake** is a cloud-native data warehouse built as a multi-cloud SaaS platform (runs on AWS, GCP, and Azure). It was founded in 2012 and IPO'd in 2020 in the largest software IPO in history at the time.

**The fundamental architectural innovation:** Snowflake completely separates storage from compute. You pay for data stored (cheap, in S3/GCS/Azure Blob under the hood) and compute separately (virtual warehouses that you spin up and down). Before Snowflake, data warehouses bundled both — you paid for the hardware whether you used it or not.

---

### Snowflake architecture — in depth

```
SNOWFLAKE ARCHITECTURE

┌─────────────────────────────────────────────────────────────────┐
│                     CLOUD SERVICES LAYER                         │
│  (Authentication, Query Optimizer, Metadata, Security, TX Mgmt) │
└─────────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
   │  Virtual WH  │  │  Virtual WH  │  │  Virtual WH  │
   │  "COMPUTE_WH"│  │  "ANALYTICS" │  │  "LOADING"   │
   │  (XS: 1 node)│  │  (L: 8 nodes)│  │  (S: 2 nodes)│
   │              │  │              │  │              │
   │  Your BI     │  │  Data team   │  │  ETL jobs    │
   │  dashboard   │  │  queries     │  │  (Fivetran)  │
   └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
          │                 │                 │
          └─────────────────┼─────────────────┘
                            ▼
              ┌─────────────────────────────┐
              │     STORAGE LAYER           │
              │  (S3/GCS/Azure Blob — micro │
              │   partitioned, columnar,    │
              │   auto-compressed, shared   │
              │   across all warehouses)    │
              └─────────────────────────────┘
```

**Key concepts:**

**Virtual Warehouse** — compute cluster you spin up for queries. Sizes: XS (1 server), S (2), M (4), L (8), XL (16), 2XL (32), etc. You can have multiple virtual warehouses pointing at the same data — one for BI, one for ETL, one for data science — they don't compete.

**Auto-suspend / Auto-resume** — a virtual warehouse automatically suspends after N minutes of inactivity (configurable, default 10 min). Immediately resumes when a query arrives. You pay only when the warehouse is active. This is the key cost control mechanism.

**Credits** — Snowflake pricing unit. 1 credit per hour per XS node. An XS warehouse costs 1 credit/hour; an L warehouse (8 nodes) costs 8 credits/hour. Credits are pre-purchased at ~$2–4 each depending on cloud region and contract.

**Micro-partitioning** — Snowflake automatically partitions data into ~50–500MB encrypted columnar micro-partitions. Metadata stores min/max values per column per micro-partition, enabling automatic partition pruning without explicit `PARTITION BY` declarations. This is different from Parquet's manual partitioning — Snowflake handles it automatically.

**Time Travel** — query data as it was at any previous point within the retention period (default 1 day; up to 90 days on Enterprise). No setup required — works on all tables automatically.

```sql
-- Query data from 24 hours ago
SELECT * FROM fct_orders AT(OFFSET => -86400);  -- 86400 seconds = 24 hours

-- Query data at a specific timestamp
SELECT * FROM fct_orders AT(TIMESTAMP => '2024-01-14 06:00:00'::TIMESTAMP);

-- Query data before a specific DML operation
SELECT * FROM fct_orders BEFORE(STATEMENT => '<query_id>');

-- Recover accidentally deleted table
CREATE TABLE fct_orders_backup CLONE fct_orders AT(OFFSET => -3600);
```

**Zero-Copy Clone** — instantly create a copy of a table, schema, or database without copying data. The clone shares storage with the original until it diverges. Invaluable for dev/test environments.

```sql
-- Clone production table to dev — instant, no data copy
CREATE TABLE fct_orders_dev CLONE fct_orders;

-- Clone entire database for testing
CREATE DATABASE olist_staging CLONE olist_production;
-- Now the QA team can work on olist_staging without touching production
```

**Snowpipe** — continuous automated data loading. Snowflake monitors a cloud storage location (S3, GCS, ADLS) and automatically loads new files as they arrive — no ETL scheduling needed for ingestion.

```
Snowpipe flow:
New file arrives in S3 → S3 event notification → Snowpipe → COPY INTO Snowflake table
Latency: typically 1–5 minutes
Cost: serverless (pay per file loaded), not per virtual warehouse hour
```

**Snowflake editions:**

| Edition | Key features | Target |
|---------|-------------|--------|
| Standard | Basic warehouse, 1-day Time Travel | Small teams |
| Enterprise | 90-day Time Travel, multi-cluster warehouses, column masking | Most enterprises |
| Business Critical | HIPAA/PCI compliance, private link, TriSecure encryption | Healthcare, finance |
| Virtual Private | Completely isolated Snowflake instance | Government, regulated industries |

---

## 24. BigQuery — Google's Serverless Analytics Engine

### What BigQuery is

**BigQuery** is Google Cloud's fully managed, serverless data warehouse. "Serverless" means there are no virtual warehouses to configure, no clusters to size, no auto-suspend settings to tune. You write a SQL query; BigQuery figures out how many resources to use.

**History and architecture context:** BigQuery was built from Google's internal Dremel system (2010 paper). It was the first cloud data warehouse to demonstrate that columnar storage + massively parallel execution could process petabytes in seconds. Every other cloud warehouse (Redshift, Snowflake) was built in response to BigQuery's market impact.

---

### BigQuery architecture — how it achieves speed

```
BigQuery architecture:

┌──────────────────────────────────────────────────────────────┐
│                    DREMEL EXECUTION ENGINE                    │
│  (massively parallel SQL engine — thousands of nodes)        │
└──────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
   ┌──────────────────────┐     ┌──────────────────────────┐
   │   COLOSSUS           │     │   JUPITER NETWORK        │
   │   (Google's          │     │   (Google's petabit       │
   │   distributed        │     │   datacenter network)     │
   │   file system —      │     │                          │
   │   stores columnar    │     │   Allows storage and      │
   │   data in "Capacitor"│     │   compute to communicate  │
   │   format)            │     │   at near-memory speed    │
   └──────────────────────┘     └──────────────────────────┘
```

**Capacitor format** — BigQuery's proprietary columnar format (not Parquet, though BigQuery can read Parquet). Heavily optimised for Google's infrastructure. Values are dictionary-encoded, run-length encoded, and bit-packed. Metadata at the column block level enables predicate pushdown.

**Dremel execution** — when you run a SQL query, BigQuery's Dremel engine creates a distributed execution plan across thousands of servers. The parallelism scales automatically with the query complexity and data size. This is why BigQuery can scan a 100TB table in 30 seconds.

**Slots** — BigQuery's compute unit. 1 slot = 1 virtual CPU. On-demand pricing: you get up to 2,000 slots per project. Flat-rate pricing: purchase dedicated slots (100 at minimum). Slots are shared across all queries in your project.

---

### BigQuery pricing — understanding the cost model

BigQuery has two pricing models, and choosing the wrong one is expensive:

**On-demand pricing:**
- Pay $5 per TB of data **scanned** by a query (not stored, not returned — scanned)
- 1 TB free per month per project
- First 10 GB free per query

**Flat-rate pricing:**
- Buy committed slots (100 slot minimum, ~$2,000/month)
- Unlimited queries within your slot capacity
- Better for high-volume query workloads

**Critical cost implication:** `SELECT *` from a 10TB table costs $50 every time you run it, even if you're only looking at the result. `SELECT customer_id, revenue FROM ...` reads only 2 columns, costs 90% less.

```sql
-- EXPENSIVE: scans all columns, all rows
SELECT * FROM `project.dataset.fct_orders` LIMIT 100;
-- BigQuery still scans the ENTIRE table — LIMIT doesn't reduce scan cost

-- CHEAP: only scans 2 columns
SELECT customer_id, revenue FROM `project.dataset.fct_orders`;

-- CHEAPEST: partition pruning reduces rows scanned
SELECT customer_id, SUM(revenue) AS total
FROM `project.dataset.fct_orders`
WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31'   -- uses partition
GROUP BY customer_id;
-- With date partitioning, this scans 1/12th of the year's data
```

---

### BigQuery partitioning and clustering

**Partitioning** in BigQuery: unlike Parquet where you create physical directories, BigQuery partitions are metadata-driven. You declare a partition column in the table definition and BigQuery handles the physical organisation transparently.

```sql
-- Create a partitioned table (by date)
CREATE TABLE `project.dataset.fct_orders`
PARTITION BY DATE(order_date)
AS SELECT * FROM raw_orders;

-- Query cost: scans only partitions within the date range
SELECT * FROM `project.dataset.fct_orders`
WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31';
-- Cost: 1/365th of scanning the whole table (approximately)

-- Partition expiration: automatically delete old partitions
CREATE TABLE `project.dataset.fct_orders`
PARTITION BY DATE(order_date)
OPTIONS(partition_expiration_days=365);  -- auto-delete partitions > 1 year old
```

**Clustering**: after partitioning, BigQuery can further organise data within each partition by one or more columns. Queries filtering on clustered columns scan fewer bytes.

```sql
-- Partition by date, cluster by customer_state and product_category
CREATE TABLE `project.dataset.fct_orders`
PARTITION BY DATE(order_date)
CLUSTER BY customer_state, product_category
AS SELECT * FROM raw_orders;

-- This query benefits from both partition pruning AND clustering
SELECT SUM(revenue)
FROM `project.dataset.fct_orders`
WHERE order_date = '2024-01-15'         -- partition pruning
  AND customer_state = 'SP'             -- clustering filter
  AND product_category = 'Electronics'; -- clustering filter
```

**Partitioning vs clustering:**

| | Partitioning | Clustering |
|--|-------------|-----------|
| What it does | Divides table into physical segments by column value | Sorts data within partitions by column value |
| Cost benefit | Eliminates entire partitions from scan | Reduces bytes scanned within a partition |
| Limit | 1 partition column | Up to 4 clustering columns |
| Best for | Date/time range filters, high-cardinality date column | Frequently filtered categorical columns |
| When to use | Almost always on date-based tables | When partition alone doesn't narrow scans enough |

---

### BigQuery vs Snowflake — the comparison

| Feature | BigQuery | Snowflake |
|---------|---------|-----------|
| Pricing model | Per TB scanned (on-demand) or slots | Per credit per warehouse-hour |
| Serverless | Truly serverless | Virtual warehouses (manual or auto) |
| Multi-cloud | GCP only (native) | AWS, GCP, Azure (multi-cloud) |
| SQL dialect | Standard SQL (some Google extensions) | Standard SQL (close to ANSI) |
| Partitioning | Metadata-driven, automatic | Micro-partitioning (automatic) |
| Time travel | 7 days (default), 7 days max | 1–90 days |
| Clone | Yes (BigQuery Snapshots) | Zero-Copy Clone (better) |
| ML built-in | BigQuery ML (run models in SQL) | Snowpark ML, Cortex |
| Streaming ingestion | Streaming inserts (real-time) | Snowpipe (near-real-time) |
| External tables | Yes (query GCS directly) | Yes (S3/GCS/ADLS) |
| Ecosystem | GCP-native (Cloud Composer, Dataflow, Looker) | Multi-cloud; many BI integrations |
| **Typical use** | GCP shops; pay-per-query; serverless preference | Multi-cloud; per-warehouse billing; large enterprises |

---

## 25. dbt — How It Actually Works

### What dbt is — beyond the marketing

**dbt (data build tool)** is a command-line tool that enables analysts and engineers to transform data in their warehouse using SQL. The core insight: your warehouse is powerful enough to run transformations — you don't need to extract data, transform it in Python, and load it back. Instead, dbt compiles your SQL SELECT statements into CREATE TABLE/VIEW statements and runs them in your warehouse.

**What dbt does:**
1. Takes your `.sql` files (which are just SELECT statements)
2. Compiles them into runnable SQL (adds CREATE TABLE/VIEW wrappers, resolves `{{ ref() }}` dependencies)
3. Runs them in your warehouse in dependency order
4. Runs your tests (unique, not_null, accepted_values, relationships)
5. Generates documentation from your models and schema files

**What dbt does NOT do:**
- dbt doesn't move data from sources to the warehouse (that's Fivetran/Airbyte/custom ingestion)
- dbt doesn't schedule itself (use dbt Cloud, Airflow, or Prefect for that)
- dbt doesn't connect to source systems

---

### The ref() system — dbt's dependency management

The `{{ ref('model_name') }}` function is the heart of dbt. When you write `{{ ref('stg_orders') }}` in a model, dbt:
1. Knows that this model depends on `stg_orders`
2. Ensures `stg_orders` runs before this model
3. Resolves the correct database/schema/table name for the environment (dev vs prod)

This is what makes dbt's lineage possible — every dependency is explicit.

```sql
-- models/staging/stg_orders.sql
-- dbt compiles this to:
-- CREATE VIEW analytics.stg_orders AS (...)

WITH source AS (
    -- {{ source() }} references an external table (not a dbt model)
    SELECT * FROM {{ source('olist', 'raw_orders') }}
    -- resolves to: SELECT * FROM raw.olist_orders_dataset
),

renamed_and_typed AS (
    SELECT
        order_id,
        customer_id,
        order_status,
        CAST(order_purchase_timestamp AS TIMESTAMP)    AS ordered_at,
        CAST(order_delivered_customer_date AS TIMESTAMP) AS delivered_at,
        CAST(order_estimated_delivery_date AS TIMESTAMP) AS estimated_at,

        -- Derived columns
        DATE_TRUNC('month', order_purchase_timestamp)  AS order_month,

        CASE
            WHEN order_delivered_customer_date > order_estimated_delivery_date
            THEN 1 ELSE 0
        END                                            AS is_late

    FROM source
    WHERE order_id IS NOT NULL   -- staging models filter obviously bad rows
)

SELECT * FROM renamed_and_typed


-- models/marts/fct_orders.sql
-- References stg_orders (above) — dbt will run stg_orders first

WITH orders AS (
    SELECT * FROM {{ ref('stg_orders') }}  -- ref() = dependency declaration
),

payments AS (
    SELECT order_id, SUM(payment_value) AS total_payment
    FROM {{ ref('stg_payments') }}
    GROUP BY 1
),

final AS (
    SELECT
        o.order_id,
        o.customer_id,
        o.ordered_at,
        o.order_month,
        o.is_late,
        p.total_payment
    FROM orders o
    LEFT JOIN payments p ON o.order_id = p.order_id
)

SELECT * FROM final
```

---

### dbt materialisation types — what actually gets created in the warehouse

| Materialisation | What gets created | When to use |
|----------------|-------------------|-------------|
| **view** (default) | A SQL view — no data stored; query is re-run every time | Staging models that are cheap to recompute |
| **table** | A physical table — data stored, recomputed on each `dbt run` | Dimensions and aggregations; faster for downstream queries |
| **incremental** | Table that only processes new rows on each run | Fact tables with millions of rows; too slow to rebuild daily |
| **ephemeral** | Not materialised in DB; compiled as CTE in downstream model | Intermediate logic you don't want as a table or view |

```sql
-- Incremental model — only processes new data each run
-- models/marts/fct_orders.sql

{{
    config(
        materialized='incremental',
        unique_key='order_id',        -- deduplication key for upsert
        on_schema_change='sync_all_columns'  -- what to do if schema changes
    )
}}

SELECT
    order_id,
    customer_id,
    ordered_at,
    total_payment
FROM {{ ref('stg_orders') }}

{% if is_incremental() %}
    -- On incremental runs: only process rows newer than the latest in the table
    -- On full refresh: this block is ignored, all rows processed
    WHERE ordered_at > (SELECT MAX(ordered_at) FROM {{ this }})
{% endif %}
```

---

### dbt testing — your built-in data quality layer

```yaml
# models/schema.yml
version: 2

models:
  - name: fct_orders
    description: "Core orders fact table. Grain: one row per order_item."
    columns:
      - name: order_id
        description: "Unique order identifier"
        tests:
          - unique            # no duplicates
          - not_null          # no nulls

      - name: order_status
        tests:
          - accepted_values:
              values: ['delivered', 'shipped', 'processing', 'cancelled',
                       'invoiced', 'unavailable', 'approved', 'created']

      - name: total_payment
        tests:
          - not_null
          - dbt_utils.expression_is_true:
              expression: ">= 0"   # payment must be non-negative

      - name: customer_id
        tests:
          - relationships:      # referential integrity check
              to: ref('dim_customer')
              field: customer_id

  - name: dim_customer
    columns:
      - name: customer_unique_id
        tests:
          - unique
          - not_null
```

```bash
# dbt commands
dbt run                              # compile and run all models
dbt run --select stg_orders          # run one model
dbt run --select +fct_orders         # run fct_orders and all its dependencies (+)
dbt run --select fct_orders+         # run fct_orders and all its dependents (+)
dbt run --select tag:daily           # run models tagged 'daily'
dbt run --full-refresh               # rebuild all incremental models from scratch

dbt test                             # run all tests
dbt test --select fct_orders         # test one model

dbt build                            # run + test in one command (recommended for CI)

dbt docs generate                    # generate documentation HTML
dbt docs serve                       # open docs site in browser (localhost:8080)
                                     # shows lineage DAG, column docs, test results

dbt debug                            # test connection to warehouse
dbt compile                          # show compiled SQL without running
dbt snapshot                         # run SCD Type 2 snapshots
```

---
---

## 24. End-to-End Pipeline Walkthrough — Olist

### What we're building

A production-grade daily pipeline for the Olist e-commerce data that:
1. Extracts all 8 CSV files
2. Validates raw data against contracts
3. Transforms to a star schema
4. Loads to a local DuckDB data warehouse
5. Runs quality checks post-load
6. Outputs a daily KPI report

```python
# pipeline/olist_daily.py
"""
Olist Daily ETL Pipeline
Runs: daily at 06:00 UTC
Input:  data/olist/*.csv
Output: data/warehouse/olist.ddb (DuckDB)

Usage:
    python pipeline/olist_daily.py --date 2024-01-15
    python pipeline/olist_daily.py  (defaults to yesterday)
"""

import argparse
import logging
import sys
import time
from datetime import date, timedelta
from pathlib import Path
from typing import Dict

import duckdb
import numpy as np
import pandas as pd

# ── LOGGING SETUP ─────────────────────────────────────────────────────────
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    handlers=[
        logging.StreamHandler(sys.stdout),
        logging.FileHandler("logs/pipeline.log"),
    ]
)
logger = logging.getLogger("olist_pipeline")


# ─────────────────────────────────────────────────────────────────────────
# STEP 1: EXTRACT
# ─────────────────────────────────────────────────────────────────────────

def extract_all(data_dir: Path) -> Dict[str, pd.DataFrame]:
    """Extract all Olist CSV files. Log shape of each."""
    logger.info("=== STEP 1: EXTRACT ===")

    sources = {
        "orders":      ("olist_orders_dataset.csv",        ["order_purchase_timestamp",
                                                             "order_delivered_customer_date",
                                                             "order_estimated_delivery_date"]),
        "customers":   ("olist_customers_dataset.csv",     []),
        "payments":    ("olist_order_payments_dataset.csv",[]),
        "order_items": ("olist_order_items_dataset.csv",   []),
        "products":    ("olist_products_dataset.csv",      []),
        "sellers":     ("olist_sellers_dataset.csv",       []),
        "reviews":     ("olist_order_reviews_dataset.csv", []),
        "category_tr": ("product_category_name_translation.csv", []),
    }

    raw = {}
    for name, (filename, date_cols) in sources.items():
        path = data_dir / filename
        if not path.exists():
            logger.warning(f"  ⚠  {filename} not found — skipping")
            continue
        df = pd.read_csv(path, parse_dates=date_cols, low_memory=False)
        df.columns = df.columns.str.lower().str.strip().str.replace(" ", "_")
        logger.info(f"  ✓ {name}: {df.shape[0]:,} rows × {df.shape[1]} cols")
        raw[name] = df

    return raw


# ─────────────────────────────────────────────────────────────────────────
# STEP 2: VALIDATE
# ─────────────────────────────────────────────────────────────────────────

def validate_raw(raw: Dict[str, pd.DataFrame]) -> bool:
    """Run critical validation checks. Returns False if any critical check fails."""
    logger.info("=== STEP 2: VALIDATE ===")
    all_ok = True

    def check(name, condition, msg, critical=True):
        nonlocal all_ok
        if not condition:
            level = "ERROR" if critical else "WARNING"
            logger.log(logging.ERROR if critical else logging.WARNING,
                       f"  {'✗' if critical else '⚠'} [{level}] {name}: {msg}")
            if critical:
                all_ok = False
        else:
            logger.info(f"  ✓ {name}")

    if "orders" in raw:
        o = raw["orders"]
        check("orders_not_empty",     len(o) > 0,              f"0 rows — source may be empty")
        check("order_id_not_null",    o["order_id"].isnull().sum() == 0,
              f"{o['order_id'].isnull().sum()} null order_ids")
        check("order_id_unique",      o["order_id"].duplicated().sum() == 0,
              f"{o['order_id'].duplicated().sum()} duplicate order_ids")
        check("valid_statuses",
              o["order_status"].isin({"delivered","shipped","processing","cancelled",
                                      "invoiced","unavailable","approved","created"}).all(),
              f"Invalid statuses: {set(o['order_status'].unique()) - {'delivered','shipped','processing','cancelled','invoiced','unavailable','approved','created'}}",
              critical=False)

    if "payments" in raw:
        p = raw["payments"]
        check("payment_amount_positive",
              (p["payment_value"] >= 0).all(),
              f"{(p['payment_value'] < 0).sum()} negative payment values",
              critical=False)

    return all_ok


# ─────────────────────────────────────────────────────────────────────────
# STEP 3: TRANSFORM
# ─────────────────────────────────────────────────────────────────────────

def transform(raw: Dict[str, pd.DataFrame]) -> Dict[str, pd.DataFrame]:
    """Build the full star schema from raw sources."""
    logger.info("=== STEP 3: TRANSFORM ===")
    tables = {}

    # ── DIMENSION: date ───────────────────────────────────────────────────
    date_range = pd.date_range("2016-01-01", "2026-12-31", freq="D")
    dim_date = pd.DataFrame({"full_date": date_range})
    dim_date["date_id"]    = dim_date["full_date"].dt.strftime("%Y%m%d").astype(int)
    dim_date["year"]       = dim_date["full_date"].dt.year
    dim_date["month"]      = dim_date["full_date"].dt.month
    dim_date["month_name"] = dim_date["full_date"].dt.month_name()
    dim_date["quarter"]    = dim_date["full_date"].dt.quarter
    dim_date["is_weekend"] = dim_date["full_date"].dt.dayofweek >= 5
    tables["dim_date"] = dim_date
    logger.info(f"  ✓ dim_date:     {len(dim_date):,} rows")

    # ── DIMENSION: customer ───────────────────────────────────────────────
    if "customers" in raw:
        c = raw["customers"].copy()
        c = c.drop_duplicates("customer_unique_id")
        c["customer_state"] = c["customer_state"].str.upper().str.strip()
        state_region = {
            "SP":"Southeast","RJ":"Southeast","MG":"Southeast","ES":"Southeast",
            "PR":"South","SC":"South","RS":"South",
            "BA":"Northeast","PE":"Northeast","CE":"Northeast","MA":"Northeast",
            "AM":"North","PA":"North","TO":"North",
            "GO":"Central-West","MT":"Central-West","MS":"Central-West","DF":"Central-West",
        }
        c["region"] = c["customer_state"].map(state_region).fillna("Other")
        c.index = range(1, len(c) + 1)
        c.index.name = "customer_sk"
        tables["dim_customer"] = c.reset_index()[
            ["customer_sk","customer_unique_id","customer_state","region"]
        ]
        logger.info(f"  ✓ dim_customer: {len(tables['dim_customer']):,} rows")

    # ── DIMENSION: product ────────────────────────────────────────────────
    if "products" in raw:
        p  = raw["products"].copy()
        tr = raw.get("category_tr", pd.DataFrame())
        if len(tr):
            p  = p.merge(tr, on="product_category_name", how="left")
            p["category_en"] = (p["product_category_name_english"]
                                .str.replace("_", " ").str.title()
                                .fillna("Unknown"))
        else:
            p["category_en"] = p["product_category_name"].str.title().fillna("Unknown")
        p.index = range(1, len(p) + 1)
        p.index.name = "product_sk"
        tables["dim_product"] = p.reset_index()[["product_sk","product_id","category_en"]]
        logger.info(f"  ✓ dim_product:  {len(tables['dim_product']):,} rows")

    # ── DIMENSION: seller ─────────────────────────────────────────────────
    if "sellers" in raw:
        s = raw["sellers"].copy()
        s["seller_state"] = s["seller_state"].str.upper().str.strip()
        s.index = range(1, len(s) + 1)
        s.index.name = "seller_sk"
        tables["dim_seller"] = s.reset_index()[["seller_sk","seller_id","seller_state"]]
        logger.info(f"  ✓ dim_seller:   {len(tables['dim_seller']):,} rows")

    # ── FACT: orders ──────────────────────────────────────────────────────
    if "orders" in raw and "order_items" in raw:
        orders = raw["orders"].copy()
        items  = raw["order_items"].copy()
        pays   = raw.get("payments", pd.DataFrame())
        revs   = raw.get("reviews",  pd.DataFrame())

        # Aggregate payments
        if len(pays):
            pays_agg = pays.groupby("order_id")["payment_value"].sum().reset_index(name="total_payment")
        else:
            pays_agg = pd.DataFrame(columns=["order_id","total_payment"])

        # Aggregate reviews
        if len(revs):
            revs_agg = revs.groupby("order_id")["review_score"].mean().round(2).reset_index(name="avg_review")
        else:
            revs_agg = pd.DataFrame(columns=["order_id","avg_review"])

        # Build fact
        fct = items.merge(orders[["order_id","customer_id",
                                   "order_purchase_timestamp",
                                   "order_delivered_customer_date",
                                   "order_estimated_delivery_date",
                                   "order_status"]], on="order_id", how="inner")
        fct = fct.merge(pays_agg, on="order_id", how="left")
        fct = fct.merge(revs_agg, on="order_id", how="left")

        # Surrogate key lookups
        if "dim_customer" in tables:
            cust_map = dict(zip(
                raw["customers"]["customer_unique_id"],
                range(1, len(raw["customers"]) + 1)
            ))
            # Join through customer natural key
            fct = fct.merge(raw["customers"][["customer_id","customer_unique_id"]],
                           on="customer_id", how="left")
            fct["customer_sk"] = fct["customer_unique_id"].map(cust_map)

        if "dim_product" in tables:
            prod_map = dict(zip(tables["dim_product"]["product_id"],
                                tables["dim_product"]["product_sk"]))
            fct["product_sk"] = fct["product_id"].map(prod_map)

        if "dim_seller" in tables:
            sell_map = dict(zip(tables["dim_seller"]["seller_id"],
                                tables["dim_seller"]["seller_sk"]))
            fct["seller_sk"] = fct["seller_id"].map(sell_map)

        # Derived measures
        fct["date_id"] = (fct["order_purchase_timestamp"]
                          .dt.strftime("%Y%m%d").astype("Int64"))
        delivered = fct["order_delivered_customer_date"].notna()
        fct["delivery_days"] = np.where(
            delivered,
            (fct["order_delivered_customer_date"] - fct["order_purchase_timestamp"]).dt.days,
            np.nan
        )
        fct["is_late"] = np.where(
            delivered,
            (fct["order_delivered_customer_date"] > fct["order_estimated_delivery_date"]).astype("Int8"),
            pd.NA
        )

        tables["fct_orders"] = fct[[
            "order_id","order_item_id",
            "customer_sk","product_sk","seller_sk","date_id",
            "price","freight_value","total_payment","avg_review",
            "delivery_days","is_late","order_status",
        ]].rename(columns={"price": "item_revenue"})
        logger.info(f"  ✓ fct_orders:   {len(tables['fct_orders']):,} rows")

    return tables


# ─────────────────────────────────────────────────────────────────────────
# STEP 4: LOAD
# ─────────────────────────────────────────────────────────────────────────

def load_to_duckdb(tables: Dict[str, pd.DataFrame], db_path: str) -> None:
    """Load all tables to DuckDB warehouse."""
    logger.info("=== STEP 4: LOAD ===")
    con = duckdb.connect(db_path)

    for table_name, df in tables.items():
        # Idempotent: replace the table entirely
        con.execute(f"DROP TABLE IF EXISTS {table_name}")
        con.register(f"_{table_name}_temp", df)
        con.execute(f"CREATE TABLE {table_name} AS SELECT * FROM _{table_name}_temp")
        con.unregister(f"_{table_name}_temp")
        logger.info(f"  ✓ {table_name}: {len(df):,} rows loaded")

    con.close()


# ─────────────────────────────────────────────────────────────────────────
# STEP 5: POST-LOAD QUALITY CHECKS
# ─────────────────────────────────────────────────────────────────────────

def post_load_checks(db_path: str) -> bool:
    """Verify warehouse contents after loading."""
    logger.info("=== STEP 5: POST-LOAD CHECKS ===")
    con = duckdb.connect(db_path)
    all_ok = True

    checks = [
        ("fct_orders has rows",
         "SELECT COUNT(*) FROM fct_orders", lambda r: r > 0, r"Expected rows > 0"),
        ("fct_orders order_id not null",
         "SELECT COUNT(*) FROM fct_orders WHERE order_id IS NULL", lambda r: r == 0,
         "Null order_ids in fct_orders"),
        ("total_payment positive",
         "SELECT COUNT(*) FROM fct_orders WHERE total_payment < 0", lambda r: r == 0,
         "Negative total_payment values"),
        ("dim_customer has rows",
         "SELECT COUNT(*) FROM dim_customer", lambda r: r > 0, "Empty dim_customer"),
    ]

    for name, query, expectation, fail_msg in checks:
        result = con.execute(query).fetchone()[0]
        if expectation(result):
            logger.info(f"  ✓ {name} (value: {result:,})")
        else:
            logger.error(f"  ✗ {name}: {fail_msg} (value: {result:,})")
            all_ok = False

    con.close()
    return all_ok


# ─────────────────────────────────────────────────────────────────────────
# STEP 6: REPORTING
# ─────────────────────────────────────────────────────────────────────────

def generate_daily_report(db_path: str, run_date: str) -> pd.DataFrame:
    """Generate a daily KPI summary from the warehouse."""
    logger.info("=== STEP 6: DAILY REPORT ===")
    con = duckdb.connect(db_path)

    kpis = con.execute("""
        SELECT
            COUNT(DISTINCT order_id)                           AS total_orders,
            COUNT(DISTINCT customer_sk)                        AS unique_customers,
            ROUND(SUM(total_payment), 2)                       AS total_revenue,
            ROUND(AVG(total_payment), 2)                       AS avg_order_value,
            ROUND(AVG(CASE WHEN is_late = 1 THEN 1.0
                           WHEN is_late = 0 THEN 0.0
                           ELSE NULL END) * 100, 1)            AS late_delivery_pct,
            ROUND(AVG(avg_review), 2)                          AS avg_review_score,
            COUNT(CASE WHEN order_status = 'cancelled' THEN 1 END) * 100.0
                / COUNT(*)                                     AS cancellation_rate
        FROM fct_orders
        WHERE order_status != 'cancelled'
    """).df()

    logger.info(f"\n{'='*50}")
    logger.info(f"DAILY KPI REPORT — {run_date}")
    logger.info(f"{'='*50}")
    for col in kpis.columns:
        logger.info(f"  {col:<30}: {kpis[col].iloc[0]}")

    con.close()
    return kpis


# ─────────────────────────────────────────────────────────────────────────
# MAIN PIPELINE RUNNER
# ─────────────────────────────────────────────────────────────────────────

def run_pipeline(
    data_dir: str = "data/olist",
    db_path:  str = "data/warehouse/olist.ddb",
    run_date: str = None,
) -> bool:
    """Run the full Olist ETL pipeline. Returns True if successful."""
    run_date = run_date or str(date.today() - timedelta(days=1))
    start    = time.perf_counter()

    logger.info(f"\n{'='*60}")
    logger.info(f"OLIST DAILY PIPELINE — Run date: {run_date}")
    logger.info(f"{'='*60}")

    Path(db_path).parent.mkdir(parents=True, exist_ok=True)
    Path("logs").mkdir(exist_ok=True)

    try:
        # Step 1: Extract
        raw = extract_all(Path(data_dir))

        # Step 2: Validate
        if not validate_raw(raw):
            logger.error("Validation failed — aborting pipeline")
            return False

        # Step 3: Transform
        tables = transform(raw)

        # Step 4: Load
        load_to_duckdb(tables, db_path)

        # Step 5: Post-load checks
        if not post_load_checks(db_path):
            logger.error("Post-load checks failed")
            return False

        # Step 6: Report
        generate_daily_report(db_path, run_date)

        elapsed = time.perf_counter() - start
        logger.info(f"\n✓ Pipeline completed in {elapsed:.1f}s")
        return True

    except Exception as e:
        logger.exception(f"Pipeline failed: {e}")
        return False


if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Olist Daily ETL Pipeline")
    parser.add_argument("--date",     default=None,              help="Run date (YYYY-MM-DD)")
    parser.add_argument("--data-dir", default="data/olist",      help="Path to Olist CSV files")
    parser.add_argument("--db",       default="data/warehouse/olist.ddb", help="DuckDB output path")
    args = parser.parse_args()

    success = run_pipeline(
        data_dir=args.data_dir,
        db_path=args.db,
        run_date=args.date,
    )
    sys.exit(0 if success else 1)
```

---

---

## 25. Apache Spark — Architecture and Internals

### What Spark is and why it exists

**Apache Spark** is a distributed data processing framework. It was created at UC Berkeley's AMPLab in 2009 and became an Apache project in 2010. Spark was built to solve the limitations of Hadoop MapReduce — primarily that MapReduce wrote intermediate results to disk after every step, making multi-step computations extremely slow.

**Spark's core innovation:** Keep data in memory (RAM) across computation steps. Intermediate results don't touch disk unless necessary. A 10-step processing pipeline can run up to 100× faster than MapReduce.

🎬 **Movies analogy:** Hadoop MapReduce is like a film editor who, after every cut, prints the film to physical reels before making the next cut. Spark is an editor who keeps the entire film in RAM on a fast workstation — no printing between cuts. The final product is the same; the process is orders of magnitude faster.

---

### Spark architecture — how it works

```
SPARK CLUSTER ARCHITECTURE

┌─────────────────────────────────────────────────────────────────────────┐
│                            DRIVER NODE                                   │
│                                                                          │
│  Your PySpark Script                                                     │
│       │                                                                  │
│       ▼                                                                  │
│  SparkContext / SparkSession (entry point)                               │
│       │                                                                  │
│  DAG Scheduler: converts your transformations into a DAG of stages       │
│       │                                                                  │
│  Task Scheduler: splits stages into tasks; assigns tasks to executors    │
│                                                                          │
└──────────────────────────┬──────────────────────────────────────────────┘
                           │
           ┌───────────────┼───────────────┐
           ▼               ▼               ▼
  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐
  │  EXECUTOR 1    │  │  EXECUTOR 2    │  │  EXECUTOR 3    │
  │  (Worker node) │  │  (Worker node) │  │  (Worker node) │
  │                │  │                │  │                │
  │  Partition 1   │  │  Partition 4   │  │  Partition 7   │
  │  Partition 2   │  │  Partition 5   │  │  Partition 8   │
  │  Partition 3   │  │  Partition 6   │  │  Partition 9   │
  │                │  │                │  │                │
  │  JVM process   │  │  JVM process   │  │  JVM process   │
  │  with N cores  │  │  with N cores  │  │  with N cores  │
  └────────────────┘  └────────────────┘  └────────────────┘
           │               │               │
           └───────────────┼───────────────┘
                           ▼
                  Cluster Manager
            (YARN / Kubernetes / Mesos / Standalone)
```

**Key architectural components:**

| Component | Role | Analogy |
|-----------|------|---------|
| **Driver** | The JVM process running your PySpark code. Creates the execution plan, coordinates workers. | Film director — sees the whole picture, gives instructions |
| **SparkContext/Session** | Entry point in the Driver. `spark = SparkSession.builder.getOrCreate()` | Your workstation's connection to the studio |
| **Executor** | JVM process on a worker node that runs tasks and stores data partitions in memory | Individual editor workstations in the studio |
| **Partition** | A chunk of data (rows) processed by one task on one executor. More partitions = more parallelism. | One reel of film assigned to one editor |
| **Task** | One unit of work: apply a transformation to one partition | "Edit this scene" assigned to one editor |
| **Stage** | A group of tasks that can run in parallel without shuffling data | All editors working independently in parallel |
| **Shuffle** | Moving data between partitions to group by key (required for GROUP BY, JOIN) | All editors gathering in a room to synchronise |
| **Cluster Manager** | Manages the cluster resources (YARN in Hadoop; Kubernetes; Databricks; EMR) | Studio facilities management |

---

### Lazy evaluation — the most important Spark concept

Spark uses **lazy evaluation**: transformations don't execute immediately. They build up a computation plan (a DAG) and only execute when an **action** is called.

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

spark = SparkSession.builder.appName("LazyExample").getOrCreate()

# These are TRANSFORMATIONS — nothing runs yet
df = spark.read.parquet("data/orders/")           # nothing happens
filtered = df.filter(F.col("status") == "delivered")  # nothing happens
grouped  = filtered.groupBy("customer_id") \
                   .agg(F.sum("revenue").alias("total")) # nothing happens
sorted   = grouped.orderBy("total", ascending=False)    # nothing happens

# This is an ACTION — Spark now executes the entire plan
sorted.show(10)    # ← triggers execution of all steps above

# Other actions that trigger execution:
df.count()         # returns a Python int
df.collect()       # returns a Python list (careful with large datasets!)
df.write.parquet("output/")  # writes to storage
```

**Why lazy evaluation matters:**

1. **Optimisation**: Spark's Catalyst Optimizer inspects the full plan before executing and rewrites it. It can push filters down (reducing data earlier), reorder JOINs, eliminate unnecessary columns.
2. **Efficiency**: if you call `.count()` after `.filter()`, Spark doesn't materialise the filtered DataFrame in memory — it counts while filtering.
3. **Cost**: you're not paying for compute while building up the plan.

---

### RDD vs DataFrame vs Dataset — understanding the layers

Spark has three programming interfaces. In 2024, you should almost always use DataFrames.

| Interface | Language | Introduced | Use today? |
|-----------|---------|-----------|-----------|
| **RDD** (Resilient Distributed Dataset) | Scala, Java, Python | Spark 1.0 (2014) | Rarely — for very low-level operations |
| **DataFrame** | Scala, Java, Python, R | Spark 1.3 (2015) | Yes — primary interface |
| **Dataset** | Scala, Java only | Spark 1.6 (2016) | Yes in Scala/Java; not in Python |

**RDD** is the low-level building block — an unstructured distributed collection of objects. You operate on it with functional programming (map, filter, reduce). No schema, no query optimisation.

**DataFrame** is a distributed table with a schema (column names and types). Operations compile to the same optimised execution engine regardless of whether you use SQL or the DataFrame API. This is what you should use.

```python
# RDD — don't use unless you have to
rdd = spark.sparkContext.textFile("data/orders.txt")
result = rdd.map(lambda line: line.split(",")) \
            .filter(lambda row: row[3] == "delivered") \
            .map(lambda row: float(row[4])) \
            .sum()

# DataFrame — use this instead (same result, faster, more readable)
result = spark.read.csv("data/orders.csv", header=True, inferSchema=True) \
              .filter(F.col("status") == "delivered") \
              .agg(F.sum("revenue")).collect()[0][0]
```

---

### Shuffles — the performance bottleneck to understand

A **shuffle** happens when Spark needs to redistribute data across partitions — typically for GROUP BY, JOIN, and DISTINCT operations.

```
Without shuffle (each partition processes independently):

Partition 1: [customer A, customer B, customer C]  ──▶  filter ──▶ result
Partition 2: [customer D, customer A, customer E]  ──▶  filter ──▶ result
Partition 3: [customer F, customer A, customer G]  ──▶  filter ──▶ result

This is fast — no communication between nodes.

With shuffle (GROUP BY customer_id):

Before shuffle:                      After shuffle:
Partition 1: [A, B, C]               Partition 1: [all A rows]
Partition 2: [D, A, E]    ──────────▶ Partition 2: [all B, C rows]
Partition 3: [F, A, G]               Partition 3: [all D, E, F, G rows]

All customer A rows (spread across 3 partitions) must be gathered
to one partition before they can be grouped.

This requires network I/O — data moves across the cluster.
Shuffles are the most expensive operation in Spark.
```

**Minimising shuffles — practical patterns:**

```python
# EXPENSIVE: wide transformation (causes shuffle)
# GROUP BY forces all same-key rows to same partition
df.groupBy("customer_id").agg(F.sum("revenue"))

# LESS EXPENSIVE: reduce data before shuffling
df.filter(F.col("status") == "delivered") \
  .select("customer_id", "revenue") \       # drop columns before shuffle
  .groupBy("customer_id").agg(F.sum("revenue"))

# EXPENSIVE: JOIN on large tables causes shuffle of both sides
orders.join(customers, on="customer_id")

# CHEAP: broadcast join — small table is sent to every executor (no shuffle)
from pyspark.sql.functions import broadcast

orders.join(broadcast(customers), on="customer_id")
# Use broadcast when one table fits in executor memory (~10MB typical limit)
# customers table (100K rows) easily fits; orders table (10M rows) does not
```

---

### PySpark — complete practical reference

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.types import StructType, StructField, StringType, DoubleType, LongType, DateType
from pyspark.sql.window import Window

# ── SESSION ───────────────────────────────────────────────────────────────
spark = (SparkSession.builder
         .appName("OlistAnalysis")
         .config("spark.sql.adaptive.enabled", "true")        # AQE: auto-optimise at runtime
         .config("spark.sql.adaptive.coalescePartitions.enabled", "true")  # auto-merge small partitions
         .config("spark.sql.shuffle.partitions", "200")        # default shuffle partitions
         .getOrCreate())

spark.sparkContext.setLogLevel("WARN")   # reduce verbose logging

# ── READ ──────────────────────────────────────────────────────────────────
# CSV with explicit schema (avoids expensive schema inference pass)
schema = StructType([
    StructField("order_id",    StringType(), nullable=False),
    StructField("customer_id", StringType(), nullable=False),
    StructField("status",      StringType(), nullable=True),
    StructField("amount",      DoubleType(), nullable=True),
])
df = spark.read.csv("data/orders.csv", header=True, schema=schema)

# Parquet (recommended — faster, typed, smaller)
df = spark.read.parquet("data/warehouse/orders/")

# Partitioned Parquet (Spark auto-discovers partitions)
df = spark.read.parquet("data/warehouse/orders/")
# spark automatically adds year, month columns from path

# ── INSPECT ───────────────────────────────────────────────────────────────
df.printSchema()                    # column names and types
df.show(5, truncate=False)          # first 5 rows, don't truncate long strings
df.count()                          # row count (ACTION — triggers execution)
len(df.columns)                     # column count (no execution — just metadata)
df.describe("amount").show()        # summary stats (count, mean, stddev, min, max)

# ── SELECT ────────────────────────────────────────────────────────────────
df.select("order_id", "customer_id", "amount")           # specific columns
df.select("*")                                           # all columns (usually avoid)
df.select(F.col("amount") * 1.1)                         # expression
df.select(F.col("amount").alias("amount_with_tax"))       # rename

# ── FILTER ────────────────────────────────────────────────────────────────
df.filter(F.col("status") == "delivered")
df.filter("status = 'delivered'")                        # SQL string syntax
df.where(F.col("amount") > 100)                          # .where() == .filter()
df.filter((F.col("status") == "delivered") & (F.col("amount") > 100))
df.filter(F.col("status").isin(["delivered", "shipped"]))
df.filter(F.col("customer_id").isNotNull())

# ── ADD/MODIFY COLUMNS ────────────────────────────────────────────────────
df = df.withColumn("amount_usd",    F.col("amount") / 5.0)
df = df.withColumn("is_high_value", (F.col("amount") > 500).cast("integer"))
df = df.withColumn("order_month",   F.date_format(F.col("ordered_at"), "yyyy-MM"))
df = df.withColumn("status_upper",  F.upper(F.col("status")))
df = df.withColumn(
    "tier",
    F.when(F.col("amount") > 1000, "Premium")
     .when(F.col("amount") > 500,  "Standard")
     .otherwise("Basic")
)

# Rename column
df = df.withColumnRenamed("amount", "payment_value")

# Drop columns
df = df.drop("redundant_col", "another_col")

# ── AGGREGATION ───────────────────────────────────────────────────────────
df.groupBy("customer_id").agg(
    F.count("order_id").alias("orders"),
    F.sum("amount").alias("total_spent"),
    F.avg("amount").alias("avg_order"),
    F.max("ordered_at").alias("last_order"),
    F.countDistinct("product_id").alias("unique_products"),
).show()

# Multiple groupBy columns
df.groupBy("customer_state", "order_month").agg(
    F.count("*").alias("orders"),
    F.sum("amount").alias("revenue"),
).orderBy("customer_state", "order_month")

# ── JOINS ─────────────────────────────────────────────────────────────────
customers = spark.read.parquet("data/warehouse/dim_customer/")
orders    = spark.read.parquet("data/warehouse/fct_orders/")

# Standard join (causes shuffle on large tables)
result = orders.join(customers, on="customer_id", how="left")

# Broadcast join (no shuffle — customers is small)
result = orders.join(broadcast(customers), on="customer_id", how="left")

# Join on different column names
result = orders.join(
    customers,
    orders["cust_id"] == customers["customer_id"],
    how="inner"
)

# ── WINDOW FUNCTIONS ──────────────────────────────────────────────────────
window_by_customer = Window.partitionBy("customer_id").orderBy("ordered_at")

df = df.withColumn("order_number",   F.row_number().over(window_by_customer))
df = df.withColumn("prev_order_amt", F.lag("amount", 1).over(window_by_customer))
df = df.withColumn("running_total",  F.sum("amount").over(
    window_by_customer.rowsBetween(Window.unboundedPreceding, Window.currentRow)
))

# ── WRITE ─────────────────────────────────────────────────────────────────
# Parquet (recommended)
df.write.mode("overwrite").parquet("data/warehouse/output/")

# Partitioned Parquet
df.write \
  .mode("overwrite") \
  .partitionBy("year", "month") \
  .parquet("data/warehouse/output/")

# Single file (for small results)
df.coalesce(1).write.mode("overwrite").csv("output/result.csv", header=True)

# Delta Lake
df.write.format("delta").mode("overwrite").save("data/warehouse/delta_orders/")

# SQL table
df.write.mode("overwrite").saveAsTable("analytics.fct_orders")

# ── SQL INTERFACE ─────────────────────────────────────────────────────────
df.createOrReplaceTempView("orders")
customers.createOrReplaceTempView("customers")

result = spark.sql("""
    SELECT
        c.customer_state,
        DATE_FORMAT(o.ordered_at, 'yyyy-MM')  AS month,
        COUNT(DISTINCT o.order_id)            AS orders,
        ROUND(SUM(o.amount), 2)               AS revenue
    FROM orders o
    LEFT JOIN customers c ON o.customer_id = c.customer_id
    WHERE o.status = 'delivered'
    GROUP BY 1, 2
    ORDER BY 1, 2
""")
result.show()
```

---

### Spark Structured Streaming — the streaming API

Spark Structured Streaming extends the DataFrame API to handle streaming data. You write the same code as batch, but Spark incrementally processes new data as it arrives.

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

spark = SparkSession.builder \
    .appName("FraudStreamDetection") \
    .getOrCreate()

# Read from Kafka (requires spark-sql-kafka connector)
kafka_df = spark.readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "transactions") \
    .option("startingOffsets", "latest") \
    .load()

# Kafka messages arrive as binary key-value pairs
# Deserialise the value (JSON in this case)
from pyspark.sql.types import StructType, StructField, StringType, DoubleType

transaction_schema = StructType([
    StructField("transaction_id", StringType()),
    StructField("customer_id",    StringType()),
    StructField("amount",         DoubleType()),
    StructField("merchant",       StringType()),
    StructField("timestamp",      StringType()),
])

transactions = kafka_df.select(
    F.from_json(F.col("value").cast("string"), transaction_schema).alias("data")
).select("data.*")

# Apply streaming fraud detection
suspicious = transactions \
    .filter(F.col("amount") > 5000) \
    .withColumn("alert_type", F.lit("high_amount")) \
    .withColumn("processed_at", F.current_timestamp())

# Write to another Kafka topic
query = suspicious.writeStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("topic", "fraud_alerts") \
    .option("checkpointLocation", "/tmp/checkpoint/") \
    .outputMode("append") \
    .start()

# Or write to Parquet files
query2 = suspicious.writeStream \
    .format("parquet") \
    .option("path", "data/streaming_output/") \
    .option("checkpointLocation", "/tmp/checkpoint2/") \
    .trigger(processingTime="1 minute") \   # micro-batch every 1 minute
    .outputMode("append") \
    .start()

query.awaitTermination()   # keep the stream running
```

---

### Spark performance tuning — practical patterns

```python
# ── REPARTITION vs COALESCE ───────────────────────────────────────────────
# repartition: FULL SHUFFLE — redistributes data evenly
df.repartition(200)                # change partition count (causes shuffle)
df.repartition(200, "customer_id") # repartition by column (for join/groupby perf)

# coalesce: MERGE without shuffle — reduce partition count only
df.coalesce(10)   # merge down to 10 partitions (no shuffle, but uneven sizes)
# Use coalesce before writing to avoid many small files

# ── CACHING / PERSISTENCE ─────────────────────────────────────────────────
# If a DataFrame is used multiple times in your pipeline, cache it
# Otherwise Spark recomputes it from source each time it's needed

expensive_join = orders.join(customers, on="customer_id")
expensive_join.cache()             # cache in memory (deserialized)
expensive_join.persist()           # same as cache() with default storage level

# Remove from cache when no longer needed
expensive_join.unpersist()

# ── BROADCAST HINTS ───────────────────────────────────────────────────────
from pyspark.sql.functions import broadcast

# Force broadcast join even if Spark auto-broadcast threshold not met
small_lookup = spark.read.parquet("data/country_codes.parquet")  # 500KB
result = orders.join(broadcast(small_lookup), on="country_code")

# ── ADAPTIVE QUERY EXECUTION (Spark 3.0+) ────────────────────────────────
# Enable AQE — Spark adjusts the plan at runtime based on data statistics
spark = SparkSession.builder \
    .config("spark.sql.adaptive.enabled", "true") \
    .config("spark.sql.adaptive.coalescePartitions.enabled", "true") \
    .getOrCreate()
# AQE automatically: coalesces small post-shuffle partitions,
# converts sort-merge joins to broadcast joins when one side is small,
# handles data skew

# ── HANDLING SKEW ─────────────────────────────────────────────────────────
# Data skew: one partition has much more data than others (e.g., "Unknown" customer_id)
# Symptoms: one task takes 10× longer than others; "straggler" task
# Solution: salt the skewed key

import random

# Add a random salt to create sub-partitions for the skewed key
df_salted = df.withColumn("skew_key_salt",
    F.concat(F.col("skewed_key"), F.lit("_"),
             (F.rand() * 10).cast("int").cast("string"))
)
# Now join on skew_key_salt instead of skewed_key
```

---
---

## 27. Apache Kafka — Event Streaming Platform

### What is Kafka?

**Apache Kafka** (created at LinkedIn 2011, open-sourced 2012) is a distributed event streaming platform. It solves a specific problem: how do you move large volumes of data between many systems in real time, reliably, and with the ability to replay events?

Before Kafka, connecting systems looked like this:
```
System A ──▶ System B
System A ──▶ System C     ← custom integration per connection
System A ──▶ System D     ← brittle, N² complexity

After Kafka:
System A ──▶ Kafka ──▶ System B
System B ──▶         ──▶ System C    ← N integrations instead of N²
System C ──▶         ──▶ System D    ← decoupled: producers don't know consumers
```

---

### Kafka architecture — the internals

```
┌────────────────────────────────────────────────────────────────────────────┐
│                             KAFKA CLUSTER                                   │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  TOPIC: "orders" (3 partitions, replication factor 2)                │  │
│  │                                                                      │  │
│  │  Partition 0:  [msg0][msg3][msg6][msg9] ...  ← Broker 1 (leader)    │  │
│  │                                             ← Broker 2 (replica)    │  │
│  │                                                                      │  │
│  │  Partition 1:  [msg1][msg4][msg7][msg10] ... ← Broker 2 (leader)    │  │
│  │                                             ← Broker 3 (replica)    │  │
│  │                                                                      │  │
│  │  Partition 2:  [msg2][msg5][msg8][msg11] ... ← Broker 3 (leader)    │  │
│  │                                             ← Broker 1 (replica)    │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  BROKERS (Kafka servers): Broker 1, Broker 2, Broker 3               │  │
│  │  Each broker holds some partitions as leader, others as replica      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
│                                                                             │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │  ZooKeeper (or KRaft in Kafka 3.x)                                   │  │
│  │  Manages broker metadata, leader election, cluster coordination      │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────────┘
         ▲                                        │
         │ produce(topic, key, value)             │ consume(topic, partition, offset)
         │                                        ▼
  ┌─────────────┐                        ┌──────────────────────────────────┐
  │  PRODUCER   │                        │  CONSUMER GROUP: fraud-detectors │
  │             │                        │                                  │
  │ Order svc   │                        │  Consumer 1: reads Partition 0   │
  │ Payment svc │                        │  Consumer 2: reads Partition 1   │
  │ User svc    │                        │  Consumer 3: reads Partition 2   │
  └─────────────┘                        └──────────────────────────────────┘
```

---

### Core Kafka concepts explained

**Topics and Partitions:**
```
A TOPIC is a named, ordered log of messages.
Think of it like a database table: all "orders" events go to the "orders" topic.

A PARTITION is an ordered sub-sequence of a topic.
- Each partition is an immutable, append-only log
- Messages are assigned to partitions by hash of their KEY (or round-robin if no key)
- Using the same key → same partition → preserves ordering for that key
  (important: all orders from customer C001 are in the same partition → in-order processing)

Why partition?
- PARALLELISM: 3 partitions → 3 consumers can process simultaneously
- SCALE: add more partitions → handle more throughput
- ORDER guarantee: within one partition, messages are strictly ordered
```

**Offsets:**
```
Each message in a partition gets a sequential integer offset: 0, 1, 2, 3, ...

Partition 0: [msg at offset 0][msg at offset 1][msg at offset 2]...
                                                    ↑
                                        Consumer is at offset 2
                                        (has processed offsets 0 and 1)

Consumers "commit" their offset to Kafka — this is what allows:
- Resuming after failure (restart from last committed offset)
- Replaying events (reset offset to 0 → reprocess from the beginning)
- Multiple independent consumers (each has its own offset position)
```

**Consumer Groups:**
```
A CONSUMER GROUP is a set of consumers that share the work of consuming a topic.
Each partition is assigned to exactly ONE consumer in the group at a time.

Topic with 3 partitions, Consumer Group with 3 consumers:
  Partition 0 → Consumer A
  Partition 1 → Consumer B
  Partition 2 → Consumer C
  → Maximum parallelism: 3 consumers processing simultaneously

Consumer Group with 2 consumers:
  Partition 0 → Consumer A
  Partition 1 → Consumer A
  Partition 2 → Consumer B
  → Consumer A handles 2 partitions, Consumer B handles 1

Consumer Group with 4 consumers:
  Partition 0 → Consumer A
  Partition 1 → Consumer B
  Partition 2 → Consumer C
  Consumer D  → idle (no partition assigned)
  → Never have more consumers than partitions

MULTIPLE GROUPS can consume the same topic independently.
Example: both the fraud detection group AND the analytics group
  consume the "transactions" topic, each maintaining their own offsets.
  Adding a new consumer group doesn't affect existing ones.
```

**Retention and Replay:**
```
Kafka retains messages for a configurable period (default: 7 days).
This means:
- If your consumer falls behind, it can catch up
- You can replay historical events by resetting offsets
- New consumers can start from the beginning of the topic
- This is fundamentally different from traditional message queues
  (RabbitMQ, SQS) where messages are deleted after consumption
```

---

### Kafka delivery semantics

```
AT MOST ONCE: message may be lost, never processed twice
  Producer sends → doesn't wait for acknowledgment
  Consumer: process then commit offset
  Use for: high-volume metrics where occasional loss is OK

AT LEAST ONCE: message never lost, may be processed twice
  Producer: resends until acknowledged
  Consumer: commit offset only after successful processing
  Use for: most cases — consumers must be idempotent
  Risk: if consumer crashes between process and commit, re-processes on restart

EXACTLY ONCE: each message processed exactly once
  Requires: idempotent producers + transactional consumers
  Kafka transactions: read-process-write in a single atomic operation
  Use for: financial transactions, billing, any system where duplicates cause harm
  Cost: higher latency, more complexity
```

---

### Kafka Connect — moving data in and out

**Kafka Connect** is a framework for streaming data between Kafka and external systems without writing code. You configure connectors that run as plugins inside Kafka Connect workers.

```
Source Connectors (external system → Kafka):
  PostgreSQL CDC Connector  → streams all DB changes as events
  S3 Source Connector       → reads files from S3 into Kafka
  HTTP Source Connector     → polls a REST API and publishes to Kafka

Sink Connectors (Kafka → external system):
  Snowflake Sink Connector  → writes Kafka messages to Snowflake tables
  S3 Sink Connector         → writes batches to S3 as Parquet/JSON files
  Elasticsearch Connector   → indexes events for search
  JDBC Sink Connector       → inserts into any JDBC database
```

---

### Schema Registry — type safety for Kafka

Without schema enforcement, a producer can change the message format and break all consumers silently. **Schema Registry** (from Confluent) solves this:

```
Producer registers schema v1 → Schema Registry
Producer writes Avro message with schema_id=1 embedded in message header

Consumer reads message → fetches schema_id=1 from Registry → deserialises correctly

Producer tries to add breaking change → Schema Registry REJECTS IT
(unless compatibility mode = NONE)

Compatibility modes:
BACKWARD:  new schema can read data written with old schema (add optional fields)
FORWARD:   old schema can read data written with new schema (remove optional fields)
FULL:      both backward and forward compatible
NONE:      any change allowed (no safety — not recommended for production)
```

---

### Kafka vs alternatives

| Feature | Kafka | AWS Kinesis | GCP Pub/Sub | Azure Event Hubs |
|---------|-------|------------|-------------|-----------------|
| **Type** | Self-hosted or managed | AWS managed | GCP managed | Azure managed |
| **Kafka API compatible** | Native | No | No | Yes (Event Hubs for Kafka) |
| **Retention** | Configurable (days) | 7 days max | 7 days (Lite) | 7 days |
| **Replay** | ✅ Full | Limited | Limited | Limited |
| **Consumer groups** | ✅ | ✅ | ✅ | ✅ |
| **Schema Registry** | Confluent Schema Registry | AWS Glue Schema Registry | ✅ built-in | Limited |
| **Throughput** | Highest | High | High | High |
| **Operational overhead** | High (if self-hosted) | None | None | None |
| **Cost** | Infra cost | Pay per shard | Pay per message | Pay per throughput unit |
| **Best for** | Multi-cloud, large scale, full control | AWS-native | GCP-native, serverless | Azure-native |

---

## 28. Apache Spark — Architecture and Internals

### What is Spark?

**Apache Spark** (created at UC Berkeley AMPLab 2009, open-sourced 2010) is a distributed computing engine for large-scale data processing. Its key innovation over Hadoop MapReduce was in-memory processing — data is cached in memory between steps rather than written to disk after every operation, making it 10-100x faster.

---

### Spark architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                             SPARK APPLICATION                                │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  DRIVER (your Python script / Jupyter notebook)                     │    │
│  │                                                                     │    │
│  │  SparkContext / SparkSession                                         │    │
│  │    - Converts your transformations into a DAG of tasks              │    │
│  │    - Negotiates resources with the cluster manager                  │    │
│  │    - Schedules tasks on executors                                   │    │
│  │    - Collects results back (careful: collect() brings data here!)   │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                              │                                               │
│                              │ schedules tasks via cluster manager           │
│                              ▼                                               │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │  CLUSTER MANAGER (YARN / Kubernetes / Standalone / Mesos)           │    │
│  │    Allocates resources (cores, memory) across the cluster           │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                              │                                               │
│          ┌───────────────────┼────────────────────┐                         │
│          ▼                   ▼                     ▼                         │
│  ┌───────────────┐  ┌───────────────┐  ┌───────────────────────────────┐   │
│  │   EXECUTOR 1  │  │   EXECUTOR 2  │  │         EXECUTOR 3             │   │
│  │   (Worker Node│  │   (Worker Node│  │         (Worker Node)          │   │
│  │               │  │               │  │                                │   │
│  │  ┌──────────┐ │  │  ┌──────────┐ │  │  ┌──────────┐  ┌──────────┐  │   │
│  │  │ Task 1   │ │  │  │ Task 2   │ │  │  │ Task 3   │  │ Task 4   │  │   │
│  │  │(Partition│ │  │  │(Partition│ │  │  │(Partition│  │(Partition│  │   │
│  │  │   0)     │ │  │  │   1)     │ │  │  │   2)     │  │   3)     │  │   │
│  │  └──────────┘ │  │  └──────────┘ │  │  └──────────┘  └──────────┘  │   │
│  │               │  │               │  │                                │   │
│  │  JVM Process  │  │  JVM Process  │  │  JVM Process                  │   │
│  │  (Python via  │  │               │  │                                │   │
│  │   PySpark)    │  │               │  │                                │   │
│  └───────────────┘  └───────────────┘  └───────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

### Lazy evaluation — why Spark is fast

One of Spark's key performance features: **transformations are lazy**. When you call `.filter()` or `.select()`, Spark doesn't execute anything. It builds a plan. Only when you call an **action** (`.count()`, `.collect()`, `.write()`) does Spark execute the entire optimised plan.

```python
from pyspark.sql import SparkSession
from pyspark.sql import functions as F

spark = SparkSession.builder.appName("example").getOrCreate()

# These are TRANSFORMATIONS — nothing executes
df = spark.read.parquet("data/orders/")           # lazy: just reads metadata
df2 = df.filter(F.col("status") == "delivered")   # lazy: adds filter to plan
df3 = df2.select("order_id", "revenue", "month")  # lazy: adds projection to plan
df4 = df3.groupBy("month").agg(F.sum("revenue"))  # lazy: adds aggregation to plan

# THIS is an ACTION — Spark now executes the ENTIRE plan
result = df4.collect()  # ← triggers execution

# Why is lazy evaluation good?
# Spark's Catalyst Optimizer can:
# 1. Reorder filters to eliminate rows early
# 2. Push predicates down to the storage layer (read less data)
# 3. Combine multiple transformations into one stage
# 4. Eliminate columns not needed in the output
#
# The "show plan" for debugging:
df4.explain(mode="extended")  # shows the unoptimised and optimised execution plan
```

---

### RDD vs DataFrame vs Dataset

```
RDD (Resilient Distributed Dataset) — the low-level API
  rdd = sc.textFile("data/orders.csv")
  rdd.filter(lambda x: "delivered" in x).count()

  - Unstructured: just a collection of objects
  - No type information, no schema
  - Python objects = Java serialisation overhead
  - No Catalyst optimizer (can't be optimised)
  - Still useful for: custom serialisation, non-tabular data, legacy code

DataFrame — the high-level API (use this)
  df = spark.read.parquet("data/orders/")
  df.filter(df["status"] == "delivered").count()

  - Tabular with named columns and types
  - Catalyst optimizer can optimise your code
  - Same API in Python, Scala, Java, R
  - Internally uses Apache Arrow / Tungsten for efficient memory

Dataset — typed DataFrame (Scala/Java only)
  Not available in Python (type system limitation)

→ Default choice for all new code: DATAFRAME
```

---

### Shuffles — the expensive operation

A **shuffle** is when Spark needs to redistribute data across partitions — sending data between worker nodes over the network. It is the most expensive operation in Spark.

```
Shuffles happen when:
- groupBy()  → rows with same key must go to same partition
- join()     → matching rows must be co-located
- distinct() → deduplication needs all copies of a value together
- orderBy()  → total sort requires one globally-sorted output

Cost: network I/O + disk I/O (spill to disk if data > memory)

How to minimise shuffles:
1. Filter BEFORE groupBy/join — fewer rows to shuffle
2. Broadcast small tables:
   from pyspark.sql.functions import broadcast
   df.join(broadcast(small_lookup), on="key")
   → small_lookup is sent to all workers, no shuffle needed
3. Partition by join keys beforehand:
   df.repartition("customer_id").write.parquet(...)
   → Future joins on customer_id skip the shuffle
4. Use bucketing for repeated joins on the same key
```

---

### Spark vs pandas — the decision

```
Use PANDAS when:
  ✅ Data fits in RAM (< ~10GB on a typical machine)
  ✅ Interactive EDA and exploration
  ✅ Complex Python logic (iterrows, apply, custom classes)
  ✅ Integration with Python ML ecosystem (scikit-learn, etc.)
  ✅ Fast development — no cluster setup

Use SPARK when:
  ✅ Data exceeds one machine's RAM (> 50GB)
  ✅ Production pipelines on large datasets
  ✅ Parallel processing across a cluster
  ✅ Streaming data (Structured Streaming)
  ✅ SQL + DataFrame API needed at scale
  ✅ Existing Databricks/EMR/Dataproc infrastructure

GREY ZONE (10GB–50GB):
  ✅ DuckDB is often faster than both for analytics queries on this range
  ✅ Polars can handle this range on a single powerful machine
  Consider Spark if you already have the infrastructure
```

---

### Structured Streaming — real-time with DataFrame API

```python
# Structured Streaming: same DataFrame API, continuous execution
from pyspark.sql import SparkSession
from pyspark.sql import functions as F
from pyspark.sql.types import StructType, StringType, DoubleType, LongType

spark = SparkSession.builder.appName("fraud_streaming").getOrCreate()

# Define the schema of incoming Kafka messages
schema = StructType().add("transaction_id", StringType()) \
                     .add("customer_id",    StringType()) \
                     .add("amount",         DoubleType()) \
                     .add("merchant",       StringType()) \
                     .add("timestamp",      LongType())

# Read from Kafka as a streaming DataFrame
kafka_stream = (spark.readStream
                .format("kafka")
                .option("kafka.bootstrap.servers", "kafka:9092")
                .option("subscribe", "transactions")
                .option("startingOffsets", "latest")
                .load())

# Parse the JSON payload from Kafka
transactions = (kafka_stream
                .select(F.from_json(
                    F.col("value").cast("string"),
                    schema
                ).alias("data"))
                .select("data.*"))

# Apply business logic — same as batch DataFrame API!
flagged = (transactions
           .withWatermark("timestamp", "2 minutes")       # accept late data up to 2 min
           .groupBy(
               F.window("timestamp", "5 minutes"),        # tumbling 5-minute windows
               "customer_id"
           )
           .agg(
               F.count("transaction_id").alias("txn_count"),
               F.sum("amount").alias("total_amount"),
           )
           .filter(F.col("txn_count") >= 5))              # flag: ≥5 txns in 5 minutes

# Write alerts to Kafka (another topic)
query = (flagged.selectExpr("to_json(struct(*)) AS value")
         .writeStream
         .format("kafka")
         .option("kafka.bootstrap.servers", "kafka:9092")
         .option("topic", "fraud-alerts")
         .option("checkpointLocation", "/tmp/spark-checkpoints/")  # required for fault tolerance
         .outputMode("update")    # "append" | "update" | "complete"
         .trigger(processingTime="30 seconds")   # run every 30 seconds
         .start())

query.awaitTermination()  # block until manually stopped
```

---

## 29. Orchestration Deep Dive — Airflow, Prefect, Dagster

### Airflow internals — what happens when a DAG runs

```
1. DAG FILE PARSING (every 30 seconds by default)
   Scheduler scans ~/airflow/dags/ for .py files
   Imports each file and looks for DAG objects
   If DAG object found: stores DAG definition in metadata DB
   Warning: if your DAG file has import errors or takes > 30s to parse → scheduler slows

2. TASK SCHEDULING
   Scheduler checks: which scheduled DAG runs are due?
   Creates DagRun objects in metadata DB
   For each DagRun, creates TaskInstance objects (one per task)
   Sets TaskInstance state to SCHEDULED

3. TASK EXECUTION
   LocalExecutor:    runs tasks as subprocesses on the scheduler machine
   CeleryExecutor:   sends tasks to a Celery queue → distributed workers pick up tasks
   KubernetesExecutor: creates a Kubernetes pod per task (best for isolation)
   Task starts → state = RUNNING
   Task succeeds → state = SUCCESS
   Task fails → state = FAILED → retry if retries > 0

4. XCOM
   XCom (cross-communication) stores key-value pairs in the metadata DB
   Accessible via ti.xcom_push(key, value) and ti.xcom_pull(task_ids, key)
   WARNING: XCom is stored in the Airflow DB — keep values small (<1MB)
   For large data: write to S3/GCS, push only the path via XCom

5. CONNECTIONS & VARIABLES
   Connections: stored credentials for DBs, APIs, cloud services
   Variables: key-value configuration stored in Airflow DB
   Both are accessible in tasks: BaseHook.get_connection("my_db")
   Secret backends: can store in AWS Secrets Manager, HashiCorp Vault
```

---

### Airflow executor types

| Executor | How it works | Parallelism | When to use |
|----------|-------------|-------------|-------------|
| **SequentialExecutor** | Tasks run one at a time in the scheduler | None | Testing only |
| **LocalExecutor** | Tasks as subprocesses on one machine | Limited (CPU count) | Small teams, single server |
| **CeleryExecutor** | Tasks distributed to Celery workers via Redis/RabbitMQ | High (add workers) | Production, multiple machines |
| **KubernetesExecutor** | Each task in its own Kubernetes pod | Very high | Cloud-native, isolation required |
| **CeleryKubernetesExecutor** | Celery for standard tasks, K8s for special tasks | Hybrid | Complex mixed workloads |

---

## 30. Lakehouse Table Formats — Delta Lake and Apache Iceberg

### The problem: ACID on a data lake

Data lakes store data as files (Parquet). Files don't support:
- **Atomicity**: a write is either complete or not — no partial writes
- **Consistency**: concurrent readers don't see inconsistent state mid-write
- **Isolation**: multiple writers don't corrupt each other
- **Durability**: completed writes survive crashes

Table formats solve this by adding a **transaction log** on top of regular Parquet files.

---

### Delta Lake architecture

```
data/delta/orders/
├── _delta_log/                          ← The transaction log (JSON files)
│   ├── 00000000000000000000.json         ← Version 0: initial write
│   ├── 00000000000000000001.json         ← Version 1: append
│   ├── 00000000000000000002.json         ← Version 2: delete some rows
│   ├── 00000000000000000003.json         ← Version 3: schema change
│   └── 00000000000000000010.checkpoint.parquet  ← Checkpoint (every 10 versions)
│
├── part-0000-abc.parquet                 ← Data files (immutable)
├── part-0001-def.parquet
├── part-0002-ghi.parquet
└── part-0003-jkl.parquet

Each log entry records:
- ADD: "add this new data file to the table"
- REMOVE: "this data file is no longer part of the table" (tombstone)
- METADATA: "schema changed to X"
- PROTOCOL: "this table requires Delta reader/writer protocol version X"

Reading "current" table:
  1. Read the delta log from the beginning (or last checkpoint)
  2. Find all ADD entries not followed by REMOVE entries
  3. Read those Parquet files

Reading "version 2":
  1. Read delta log entries 0, 1, 2 only
  2. Find all ADD entries in those 3 entries
  3. Read those files (even if they were later removed)
```

```python
from deltalake import DeltaTable, write_deltalake
import pandas as pd

# Write — creates version 0
df = pd.read_csv("orders.csv")
write_deltalake("data/delta/orders", df, mode="overwrite")

# Append — creates version 1
df_new = pd.read_csv("orders_jan16.csv")
write_deltalake("data/delta/orders", df_new, mode="append")

# Update — Delta doesn't have UPDATE, but you can use merge
from deltalake.writer import write_deltalake

# Time travel — read any previous version
dt = DeltaTable("data/delta/orders")

# Current version
df_current = dt.to_pandas()

# Previous version
df_v0 = dt.load_as_version(0).to_pandas()

# As-of timestamp
df_old = dt.load_with_datetime("2024-01-15T23:59:59").to_pandas()

# Inspect history
print(dt.history())

# Vacuum — delete files no longer referenced by the log
dt.vacuum(retention_hours=168)  # delete files older than 7 days

# Optimize — compact small files
dt.optimize().compact()
# or co-locate related rows:
dt.optimize().z_order(["customer_id", "order_date"])
```

---

### Apache Iceberg — the challenger

Iceberg differs from Delta in several important ways:

```
KEY DIFFERENCES: ICEBERG vs DELTA LAKE

1. PARTITION EVOLUTION (Iceberg wins)
   Delta: partitioning is fixed at table creation. Want to change?
          Rewrite all data files.
   Iceberg: "hidden partitioning" — partitions are an implementation detail.
            Change partitioning strategy without rewriting any data.

2. ROW-LEVEL OPERATIONS (both support)
   Both support row-level updates/deletes/merges via copy-on-write or merge-on-read.

3. CATALOG REQUIREMENT
   Delta: no external catalog required (log is self-describing)
   Iceberg: requires a catalog (Hive, REST, AWS Glue, Project Nessie)
            catalog stores table metadata and enables multi-engine access

4. MULTI-ENGINE SUPPORT (Iceberg wins)
   Delta: primarily Databricks + open source spark
   Iceberg: Spark + Flink + Trino/Presto + BigQuery + Athena + Dremio

5. BRANCHING/TAGGING (Iceberg wins)
   Iceberg supports Git-like branches and tags on tables:
     create branch "dev" from "main"
     test changes on "dev"
     merge "dev" to "main"
   Delta Lake doesn't have this natively (Databricks has workarounds)

CHOOSING:
- On Databricks: Delta Lake (native, best performance)
- On AWS with Athena/EMR: Iceberg (best engine support)
- On multi-cloud or Trino/Flink: Iceberg
- Starting fresh: Iceberg has momentum; Delta has more operational tooling
```

---

## 31. dbt — How It Actually Works

### dbt's core insight

dbt turns the data warehouse into a first-class software environment. Instead of running SQL ad-hoc or in scripts, you write **SELECT statements** and dbt:
1. Wraps them in `CREATE TABLE AS` or `CREATE VIEW AS`
2. Resolves dependencies between models (`{{ ref('stg_orders') }}`)
3. Runs them in the correct order (topological sort)
4. Runs your tests after materialisation
5. Generates a data catalogue from your docstrings

---

### How dbt compiles and runs

```
YOUR JINJA+SQL FILE (models/marts/fct_orders.sql):
──────────────────────────────────────────────────
{{ config(materialized='table') }}

WITH orders AS (
    SELECT * FROM {{ ref('stg_orders') }}
),
payments AS (
    SELECT order_id, SUM(amount) AS total
    FROM {{ ref('stg_payments') }}
    GROUP BY 1
)
SELECT o.*, p.total FROM orders o LEFT JOIN payments p ON o.order_id = p.order_id


AFTER dbt COMPILATION (run: dbt compile):
──────────────────────────────────────────
CREATE TABLE analytics.fct_orders AS (
    WITH orders AS (
        SELECT * FROM analytics.stg_orders    ← ref() resolved to actual table name
    ),
    payments AS (
        SELECT order_id, SUM(amount) AS total
        FROM analytics.stg_payments
        GROUP BY 1
    )
    SELECT o.*, p.total FROM orders o LEFT JOIN payments p ON o.order_id = p.order_id
)


dbt BUILDS A DAG FROM ALL MODELS:
──────────────────────────────────
raw_orders → stg_orders → fct_orders → agg_monthly_revenue
raw_payments → stg_payments ↗

dbt resolves this DAG via the {{ ref() }} calls and runs models in topological order.
```

---

### Materialisation types

```python
# config at model level (in the .sql file):
{{ config(materialized='view') }}     # no data stored — query runs on demand
{{ config(materialized='table') }}    # full table, rebuilt from scratch each run
{{ config(materialized='incremental')}} # only process new rows (append/merge)
{{ config(materialized='ephemeral')}} # compiled as CTE, not materialised in DB
```

| Type | Storage | When rebuilt | Best for |
|------|---------|-------------|---------|
| **view** | None | On every query | Lightweight staging models, rarely queried |
| **table** | Full copy | Every dbt run | Marts that need fast query performance |
| **incremental** | Grows over time | Only new rows added | Large fact tables, event data |
| **ephemeral** | None (CTE) | N/A | Intermediate logic used by one downstream model |

```sql
-- Incremental model — the most important materialisation for DE
{{ config(
    materialized='incremental',
    unique_key='order_id',                    -- upsert key
    on_schema_change='sync_all_columns',      -- handle new columns safely
    incremental_strategy='merge',             -- insert new, update changed
) }}

SELECT
    order_id,
    customer_id,
    order_status,
    CAST(order_purchase_timestamp AS TIMESTAMP) AS ordered_at,
    SUM(payment_value) OVER (PARTITION BY order_id) AS total_payment
FROM {{ ref('stg_orders') }} o
LEFT JOIN {{ ref('stg_payments') }} p USING (order_id)

{% if is_incremental() %}
    -- Only when running in incremental mode (not on first run)
    WHERE o.order_purchase_timestamp > (
        SELECT MAX(ordered_at) FROM {{ this }}    -- {{ this }} = the existing table
    )
{% endif %}
```

---

### dbt tests

```yaml
# models/schema.yml
version: 2
models:
  - name: fct_orders
    columns:
      - name: order_id
        tests:
          - unique                              # built-in: no duplicate values
          - not_null                            # built-in: no null values

      - name: order_status
        tests:
          - accepted_values:
              values: ['delivered', 'shipped', 'processing', 'cancelled']

      - name: customer_sk
        tests:
          - relationships:
              to: ref('dim_customer')           # FK integrity check
              field: customer_sk

      - name: total_payment
        tests:
          - dbt_utils.expression_is_true:       # from dbt-utils package
              expression: ">= 0"
```

```bash
# Run specific tests:
dbt test --select fct_orders
dbt test --select source:olist             # test source freshness + quality
dbt build --select +fct_orders            # build fct_orders AND all its parents (+)
dbt build --select fct_orders+            # build fct_orders AND all its children
```

---

## 32. DuckDB — In-Process OLAP Engine

### What is DuckDB?

**DuckDB** (created at CWI Amsterdam 2018) is an in-process SQL OLAP database. "In-process" means it runs inside your Python program (or R, Java, etc.) — no server, no network, no configuration. Despite this simplicity, it is highly competitive with cloud warehouses for analytical queries on local or cloud-stored data.

Think of it as: **SQLite for analytics**, except 100x faster for analytical queries.

---

### How DuckDB achieves its speed

```
VECTORISED EXECUTION:
Traditional: process one row at a time
  row 1 → filter → project → aggregate
  row 2 → filter → project → aggregate
  ...1,000,000 rows...

DuckDB (and BigQuery, Snowflake): process in vectors of 1024 rows
  [row 1, row 2, ... row 1024] → filter → project → aggregate (SIMD operations)
  [row 1025 ... row 2048] → ...
  → Modern CPUs execute SIMD (Single Instruction Multiple Data) operations
  → 1 instruction operates on 16 float values simultaneously
  → Hardware-level parallelism

COLUMNAR STORAGE:
DuckDB reads Parquet natively and internally stores data by column.
When you SELECT SUM(revenue), it only touches the revenue column.

PARALLEL EXECUTION:
DuckDB uses all available CPU cores automatically.
A query on 10M rows on a 4-core laptop uses all 4 cores.
```

```python
# pip install duckdb
import duckdb
import pandas as pd

# In-memory database (fastest, no persistence)
con = duckdb.connect(":memory:")

# File-backed database (persists across sessions)
con = duckdb.connect("analytics.ddb")

# ── BASIC OPERATIONS ──────────────────────────────────────────────────────

# Register a pandas DataFrame as a virtual table
df = pd.read_csv("orders.csv")
con.register("orders", df)

# Query it with SQL (returns pandas DataFrame with .df())
result = con.execute("""
    SELECT
        DATE_TRUNC('month', order_date)  AS month,
        COUNT(DISTINCT order_id)          AS orders,
        SUM(payment_value)                AS revenue
    FROM orders
    WHERE order_status = 'delivered'
    GROUP BY 1
    ORDER BY 1
""").df()

# ── QUERY PARQUET FILES DIRECTLY (no loading!) ────────────────────────────
# DuckDB can query Parquet on disk, S3, or GCS without loading into memory
result = con.execute("""
    SELECT year, month, SUM(revenue) AS total_revenue
    FROM read_parquet('data/warehouse/orders/**/*.parquet')
    WHERE year = 2024
    GROUP BY year, month
    ORDER BY year, month
""").df()

# Query S3 directly (needs AWS credentials set as env vars or AWS config)
result = con.execute("""
    SELECT * FROM read_parquet('s3://my-bucket/data/orders/year=2024/**.parquet')
    LIMIT 100
""").df()

# ── WRITE TO PARQUET ──────────────────────────────────────────────────────
con.execute("""
    COPY (
        SELECT * FROM orders WHERE order_status = 'delivered'
    ) TO 'delivered_orders.parquet' (FORMAT 'parquet', COMPRESSION 'snappy')
""")

# ── PERFORMANCE BENCHMARK ─────────────────────────────────────────────────
import time

# Create 10M row test dataset
con.execute("""
    CREATE TABLE test AS
    SELECT
        range AS id,
        random() AS amount,
        CASE WHEN random() > 0.5 THEN 'delivered' ELSE 'shipped' END AS status
    FROM range(10_000_000)
""")

# DuckDB aggregation
t = time.perf_counter()
result = con.execute("SELECT status, SUM(amount), COUNT(*) FROM test GROUP BY 1").df()
print(f"DuckDB 10M rows: {time.perf_counter()-t:.3f}s")
# DuckDB 10M rows: ~0.15s

# pandas (for comparison)
df_test = con.execute("SELECT * FROM test").df()
t = time.perf_counter()
result2 = df_test.groupby("status")["amount"].agg(["sum", "count"])
print(f"pandas 10M rows: {time.perf_counter()-t:.3f}s")
# pandas 10M rows: ~3.5s
# DuckDB is ~20x faster here
```

---

### DuckDB vs alternatives

| | DuckDB | SQLite | pandas | Spark |
|--|--------|--------|--------|-------|
| **Type** | In-process OLAP | In-process OLTP | Python library | Distributed |
| **Query speed (analytics)** | ⚡⚡⚡⚡ | ⚡ | ⚡⚡ | ⚡⚡⚡ (at scale) |
| **SQL support** | Full (window fns, CTEs) | Full | Via pandasql | Full |
| **Data size** | Up to RAM | Up to disk | Up to RAM | Unlimited |
| **Setup** | pip install | Built into Python | pip install | Cluster |
| **Parallel execution** | ✅ automatic | ❌ | Limited | ✅ distributed |
| **Parquet native** | ✅ | ❌ | Via pyarrow | ✅ |
| **S3/GCS direct** | ✅ | ❌ | ❌ | ✅ |
| **Best for** | Local analytics, DE pipelines, replacing pandas for queries | Transactional apps | Data manipulation | Distributed processing |

---

## 33. Great Expectations — Data Quality Architecture

### GX architecture

```
EXPECTATIONS
  Individual rules: "column X must not be null"
  "column Y must be between 0 and 1000"
  "table must have at least 100 rows"

EXPECTATION SUITE
  A named collection of expectations for one dataset
  e.g., "olist_orders_suite" contains 12 expectations for the orders table

VALIDATOR
  Connects a suite to a real dataset
  Runs the expectations against actual data

CHECKPOINT
  A workflow that validates one or more data assets
  Typically called in your pipeline after extract or after load
  Can trigger actions on failure: send email, Slack alert, fail the pipeline

DATA DOCS
  Auto-generated HTML documentation
  Shows: what expectations exist, the last validation run, pass/fail per expectation
  Hosted locally or on S3/Azure Blob
```

```python
# pip install great_expectations
import great_expectations as gx
import pandas as pd

# ── SETUP ──────────────────────────────────────────────────────────────────
context = gx.get_context()   # initialises GX project in current directory

# ── DEFINE AN EXPECTATION SUITE ───────────────────────────────────────────
suite = context.suites.add(gx.ExpectationSuite(name="olist_orders_suite"))

from great_expectations.expectations import (
    ExpectColumnValuesToNotBeNull,
    ExpectColumnValuesToBeBetween,
    ExpectColumnValuesToBeInSet,
    ExpectColumnValuesToBeUnique,
    ExpectTableRowCountToBeBetween,
    ExpectColumnValuesToMatchRegex,
    ExpectColumnMeanToBeBetween,
)

# Completeness
suite.add_expectation(ExpectColumnValuesToNotBeNull(column="order_id"))
suite.add_expectation(ExpectColumnValuesToNotBeNull(column="customer_id"))

# Uniqueness
suite.add_expectation(ExpectColumnValuesToBeUnique(column="order_id"))

# Range
suite.add_expectation(ExpectColumnValuesToBeBetween(
    column="payment_value", min_value=0, max_value=100_000
))

# Allowed values
suite.add_expectation(ExpectColumnValuesToBeInSet(
    column="order_status",
    value_set={"delivered","shipped","processing","cancelled","invoiced","unavailable","approved","created"}
))

# Format
suite.add_expectation(ExpectColumnValuesToMatchRegex(
    column="order_id",
    regex=r"^[a-f0-9]{32}$"   # Olist order IDs are 32-char hex strings
))

# Table-level
suite.add_expectation(ExpectTableRowCountToBeBetween(min_value=100))

# Statistical
suite.add_expectation(ExpectColumnMeanToBeBetween(
    column="payment_value", min_value=50, max_value=500
))

# ── RUN VALIDATION ────────────────────────────────────────────────────────
df = pd.read_csv("data/orders.csv")
batch = context.sources.pandas_default.read_dataframe(df)
results = context.run_checkpoint(
    checkpoint_name="orders_checkpoint",
    batch_request=batch.build_batch_request(),
)

if not results.success:
    print("VALIDATION FAILED")
    for result in results.run_results.values():
        for er in result.results:
            if not er.success:
                print(f"  ✗ {er.expectation_config.type}: {er.result}")
```

---

### GX vs alternatives

| Tool | Approach | Strengths | When to use |
|------|---------|-----------|-------------|
| **Great Expectations** | Python library, expectation suites, data docs | Comprehensive, generates docs, many expectation types | Teams wanting full DQ platform |
| **dbt tests** | SQL-based, runs in dbt | Zero extra tooling if using dbt, native integration | dbt-first teams |
| **Soda** | SQL + YAML config, SaaS option | Simple YAML syntax, Soda Cloud dashboard | Teams wanting simplicity |
| **Pandera** | pandas-native schema validation | Tight pandas integration, DataFrames as schemas | Python-heavy teams |
| **Monte Carlo** | Managed observability SaaS | Auto-detect anomalies, no config needed | Enterprise, large data teams |
| **Custom assertions** | Python assertions/pytest | No dependencies, full control | Small teams, simple checks |

---

## 34. Snowflake — Cloud Warehouse Deep Dive

### Snowflake architecture

Snowflake's key innovation is **complete separation of storage, compute, and services**. This is why it can be true multi-cloud and why you only pay for what you use.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          SNOWFLAKE PLATFORM                                  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  CLOUD SERVICES LAYER                                                  │  │
│  │  (always on, no cost beyond storage + compute usage)                  │  │
│  │                                                                       │  │
│  │  - Authentication & access control                                    │  │
│  │  - Query optimiser (compiles SQL to execution plan)                   │  │
│  │  - Transaction manager (ACID guarantees)                              │  │
│  │  - Metadata service (table structure, statistics, file locations)     │  │
│  │  - Infrastructure manager (provisioning VWs)                          │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  COMPUTE LAYER (Virtual Warehouses — you pay for these)               │  │
│  │                                                                       │  │
│  │  ┌────────────────────────────────────────────────────────────────┐  │  │
│  │  │  VW: ANALYTICS_WH (XL, 4 nodes)  ← BI team queries            │  │  │
│  │  │  Running → you pay ~$7/hour                                    │  │  │
│  │  │  Suspended → you pay $0                                        │  │  │
│  │  └────────────────────────────────────────────────────────────────┘  │  │
│  │  ┌────────────────────────────────────────────────────────────────┐  │  │
│  │  │  VW: ETL_WH (L, 2 nodes)  ← dbt/pipeline runs                 │  │  │
│  │  │  Suspended → you pay $0                                        │  │  │
│  │  └────────────────────────────────────────────────────────────────┘  │  │
│  │                                                                       │  │
│  │  VWs have their own local cache (warm = faster, cold = from storage) │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │  STORAGE LAYER (micro-partitions on cloud object storage)             │  │
│  │                                                                       │  │
│  │  Data stored in Snowflake's proprietary columnar compressed format    │  │
│  │  on S3 (AWS), Azure Blob (Azure), or GCS (GCP)                       │  │
│  │                                                                       │  │
│  │  Price: ~$23/TB/month (compressed) — same cloud storage rates         │  │
│  │  All queries from all VWs read from the same storage layer            │  │
│  └───────────────────────────────────────────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

### Key Snowflake features

**Virtual Warehouses and Credits:**
```
Virtual Warehouse sizes:
XS = 1 credit/hour  ≈ $2-3/hour
S  = 2 credits/hour
M  = 4 credits/hour
L  = 8 credits/hour
XL = 16 credits/hour
2XL= 32 credits/hour

Auto-suspend: VW pauses after N minutes of inactivity (you stop paying)
Auto-resume:  VW wakes up automatically when a query arrives (~5-10s delay)

Multi-cluster VW: automatically adds clusters when query queue builds up
(handles concurrent user spikes without queuing)
```

**Time Travel:**
```sql
-- Query data as it was at a specific timestamp
SELECT * FROM fct_orders
  AT (TIMESTAMP => '2024-01-15 12:00:00'::timestamp);

-- Query data as it was 1 hour ago
SELECT * FROM fct_orders
  AT (OFFSET => -3600);

-- Restore a table to a previous state
CREATE TABLE fct_orders_restored
  CLONE fct_orders
  AT (TIMESTAMP => '2024-01-15 06:00:00'::timestamp);

-- Retention period: 0-90 days (default: 1 day on Standard, up to 90 on Enterprise)
```

**Zero-Copy Clone:**
```sql
-- Create an instant copy of a table, schema, or database
-- "Zero-copy" = doesn't physically copy data — shares the underlying micro-partitions
-- New data in either the original or clone creates new micro-partitions

-- Clone production data for development (instant, no storage cost until modified)
CREATE TABLE fct_orders_dev CLONE fct_orders;

-- Clone entire schema for testing
CREATE SCHEMA analytics_dev CLONE analytics;

-- Clone entire database for disaster recovery testing
CREATE DATABASE prod_backup CLONE production;
```

**Snowpipe — continuous ingestion:**
```
Traditional batch load: run COPY INTO table FROM s3://bucket/path every hour
Snowpipe: automatically loads files as soon as they land in S3/GCS/Azure

How it works:
1. File lands in S3
2. S3 event notification → SQS queue
3. Snowpipe detects the SQS message → triggers micro-batch load
4. File contents appear in the Snowflake table within minutes

Use for: near-real-time ingestion from streaming sources writing to S3
```

**Streams and Tasks — CDC within Snowflake:**
```sql
-- Stream: tracks changes (INSERT/UPDATE/DELETE) on a table
CREATE STREAM orders_changes ON TABLE raw_orders;

-- Query the stream to see what changed since last consumed
SELECT * FROM orders_changes;
-- Returns: METADATA$ACTION (INSERT/DELETE), METADATA$ISUPDATE

-- Task: scheduled SQL job inside Snowflake
CREATE TASK refresh_stg_orders
  WAREHOUSE = ETL_WH
  SCHEDULE = 'USING CRON 0 * * * * UTC'   -- every hour
WHEN
  SYSTEM$STREAM_HAS_DATA('orders_changes')  -- only run if stream has new data
AS
  INSERT INTO stg_orders
  SELECT * FROM orders_changes WHERE METADATA$ACTION = 'INSERT';
```

---

## 35. BigQuery — Google's Serverless Analytics Engine

### BigQuery architecture

BigQuery is built on three Google infrastructure layers:

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              BIGQUERY                                        │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  DREMEL QUERY ENGINE                                                  │   │
│  │  Massively parallel SQL query execution engine                       │   │
│  │  Developed at Google, 2006. Powers BigQuery.                         │   │
│  │                                                                      │   │
│  │  When you run a query:                                               │   │
│  │  1. Query is parsed and optimised                                    │   │
│  │  2. Distributed across thousands of workers (slots)                  │   │
│  │  3. Each worker reads its portion of the data                        │   │
│  │  4. Results aggregated and returned                                  │   │
│  │                                                                      │   │
│  │  This happens automatically — you don't manage any servers           │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  COLOSSUS STORAGE                                                     │   │
│  │  Google's distributed file system                                    │   │
│  │  Data stored in Capacitor (Google's columnar format, similar to ORC)│   │
│  │  Separated from compute — storage and query can scale independently  │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  JUPITER NETWORK                                                      │   │
│  │  Google's 1 petabit/second datacenter network                        │   │
│  │  Allows compute and storage to be on different machines              │   │
│  │  at near-memory bandwidth speeds                                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

### BigQuery pricing — understanding the cost model

```
ON-DEMAND PRICING:
  $5 per TB of data scanned (first 1TB/month free)
  You pay for how much data your query reads — not how long it takes

  SELECT COUNT(*) FROM orders        ← scans the full orders table
  If orders = 100GB compressed → you pay for 100GB scan
  If orders = 1TB uncompressed → compressed to ~200GB → you pay for 200GB

OPTIMISATION = reduce bytes scanned:
  1. Partition → query scans only matching partitions
  2. Cluster  → within a partition, related rows are co-located
  3. SELECT specific columns (not SELECT *) → only read columns you need
  4. Materialized views → query pre-computed results

FLAT-RATE PRICING:
  $2,000/month for 100 dedicated slots
  Unlimited queries — no per-TB charge
  Better when you have many users running many queries
```

---

### Partitioning and clustering in BigQuery

```sql
-- Partitioned table (by ingestion time or column)
CREATE TABLE analytics.fct_orders
PARTITION BY DATE(ordered_at)            -- partition by date
OPTIONS (
  partition_expiration_days = 365,       -- auto-delete partitions > 1 year
  require_partition_filter = TRUE        -- force queries to specify date filter
)
AS SELECT * FROM staging.stg_orders;

-- Clustered table (sort data within partitions by these columns)
CREATE TABLE analytics.fct_orders
PARTITION BY DATE(ordered_at)
CLUSTER BY customer_state, order_status  -- co-locate rows with same state+status
AS SELECT * FROM staging.stg_orders;

-- The benefit:
-- Without clustering: scan ALL rows in Jan 2024 partition to find SP+delivered
-- With clustering: skip to where SP+delivered rows are stored, read far less

-- Check estimated bytes scanned BEFORE running (EXPLAIN / preview)
-- In the BigQuery UI: bottom of editor shows "This query will process X.X GB"
-- Only run when you're confident about the cost
```

---

### BigQuery ML — ML inside the warehouse

```sql
-- Train a model without leaving BigQuery
CREATE MODEL analytics.revenue_forecast_model
OPTIONS (
  model_type = 'ARIMA_PLUS',           -- time series forecasting
  time_series_timestamp_col = 'date',
  time_series_data_col = 'revenue',
  holiday_region = 'BRAZIL'
) AS
SELECT date, SUM(revenue) AS revenue
FROM analytics.fct_orders
GROUP BY date;

-- Run inference
SELECT * FROM ML.FORECAST(
  MODEL analytics.revenue_forecast_model,
  STRUCT(30 AS horizon, 0.9 AS confidence_level)
);
```

---
---

# Part X — Cloud Platforms for Data Engineering

## 35. The Data Journey on Cloud Platforms

### How the cloud changed data engineering

Before cloud platforms (pre-2010), building a data warehouse meant purchasing physical servers, installing software, managing hardware failures, scaling by buying more machines, and employing a team to maintain all of it. This was accessible only to large organisations with significant capital.

Cloud platforms changed this by:
1. **Eliminating upfront capital** — pay monthly, not millions upfront
2. **Elastic scaling** — scale up in minutes, not months
3. **Managed services** — the cloud vendor manages patches, backups, availability
4. **Global reach** — deploy in any region instantly
5. **Integration** — services are designed to work together

Today, data engineering is almost entirely cloud-based. Understanding what services the three major clouds (AWS, GCP, Azure) plus Databricks and Snowflake provide is essential for practitioners and critical for presales conversations.

---

### The data engineering journey — where cloud services appear

```
┌─────────────────────────────────────────────────────────────────────────────────────┐
│                     DATA ENGINEERING JOURNEY ON CLOUD                               │
├──────────────┬────────────────┬──────────────┬────────────────┬─────────────────────┤
│   INGEST     │    STORE       │   PROCESS    │  ORCHESTRATE   │     SERVE           │
│              │                │              │                │                     │
│ AWS:         │ AWS:           │ AWS:         │ AWS:           │ AWS:                │
│ Kinesis      │ S3             │ EMR (Spark)  │ Step Functions │ Redshift (query)    │
│ DMS          │ Redshift       │ Glue (ETL)   │ MWAA (Airflow) │ QuickSight (BI)    │
│ AppFlow      │ RDS            │ Lambda       │ EventBridge    │ Athena (query)      │
│ Glue         │ DynamoDB       │ Athena       │                │                     │
│              │                │              │                │                     │
│ GCP:         │ GCP:           │ GCP:         │ GCP:           │ GCP:                │
│ Pub/Sub      │ GCS            │ Dataflow     │ Cloud Composer │ BigQuery (query)    │
│ Data Fusion  │ BigQuery       │ Dataproc     │ Cloud Scheduler│ Looker Studio (BI) │
│ DataStream   │ Bigtable       │ BigQuery     │ Workflows      │ Looker (BI)         │
│              │ Spanner        │              │                │                     │
│ Azure:       │ Azure:         │ Azure:       │ Azure:         │ Azure:              │
│ Event Hubs   │ ADLS Gen2      │ Databricks   │ Data Factory   │ Synapse (query)    │
│ Data Factory │ Synapse        │ HDInsight    │ Logic Apps     │ Power BI (BI)      │
│ IoT Hub      │ Cosmos DB      │ Azure Funct. │                │                     │
│              │                │              │                │                     │
│ Cross-cloud: │ Cross-cloud:   │ Cross-cloud: │ Cross-cloud:   │ Cross-cloud:        │
│ Fivetran     │ Snowflake      │ Databricks   │ Airflow        │ Tableau             │
│ Airbyte      │ Delta Lake     │ Spark        │ Prefect        │ Power BI            │
│ Kafka        │                │ dbt          │ Dagster        │ Metabase            │
└──────────────┴────────────────┴──────────────┴────────────────┴─────────────────────┘

GOVERNANCE & QUALITY (cuts across all stages):
AWS: Glue Data Catalog · Lake Formation · Macie
GCP: Dataplex · Data Catalog · DLP
Azure: Purview · Information Protection
Cross-cloud: Great Expectations · dbt tests · Monte Carlo
```

---

## 35. AWS — Amazon Web Services for Data Engineering

AWS is the largest cloud provider by market share (~33% in 2024). It was first to market (2006) and has the broadest service catalogue. Strong in enterprises, financial services, and any organisation that needs maximum configurability.

**AWS strengths for DE:**
- Largest service catalogue — something exists for every edge case
- Mature ecosystem — most problems have well-documented solutions
- Strong data lake story (S3 + Glue + Athena + Lake Formation)
- EMR for Spark at scale

**AWS weaknesses:**
- Complexity — many services with overlapping purposes (Glue vs EMR vs Lambda for processing?)
- Pricing opacity — easy to accidentally spend a lot
- Often requires more configuration than GCP/Azure equivalents

---

### AWS S3 — Simple Storage Service

**What it is:** Amazon's object storage service. The foundation of the AWS data lake. Every file you can imagine — CSV, Parquet, JSON, images, logs — lives in S3 buckets.

**How it works:** S3 is not a file system. It's a key-value store where keys are paths like `s3://my-bucket/data/orders/year=2024/month=01/orders.parquet` and values are the file bytes. There are no real directories — the slashes in keys are just naming conventions that tools treat as folders.

**Pricing:** ~$0.023 per GB per month (us-east-1). Data transfer within the same region is free. Data transfer out to the internet or to other regions costs extra — the "data transfer tax."

**Key concepts:**
- **Bucket**: top-level container. Globally unique name. One per "project" or "environment" is typical.
- **Prefix**: the path portion of the key. `data/orders/year=2024/` is a prefix.
- **Storage classes**: Standard (hot), Infrequent Access (cool), Glacier (cold/archive). Lifecycle policies automatically move data between classes.
- **Versioning**: keep multiple versions of a file (protects against accidental overwrites).
- **Event notifications**: S3 can trigger Lambda functions or Kinesis when files arrive — the foundation of event-driven pipelines.
- **S3 Select**: query specific columns/rows from a CSV or Parquet file without downloading the whole file.

**DE use cases:**
- Data lake raw zone: `s3://datalake/raw/orders/`
- Staging zone: `s3://datalake/staging/`
- Warehouse zone (queried by Athena/Redshift Spectrum): `s3://datalake/warehouse/`
- ML training data: `s3://ml-data/features/`
- Pipeline backups and audit logs

---

### AWS Glue — Serverless ETL and Data Catalog

**What it is:** A fully managed extract, transform, and load (ETL) service that also includes a Data Catalog. Two distinct capabilities bundled under one service name.

**Glue ETL:**
- Serverless Apache Spark environment
- Write Spark code in Python or Scala, Glue manages the cluster
- Glue crawlers: automatically scan S3 data and infer schemas
- Glue jobs: run Spark code on a schedule or trigger
- Glue DataBrew: visual, no-code data preparation tool

**Glue Data Catalog:**
- Metadata repository: stores table definitions (schema, partition info, location)
- Compatible with Hive Metastore — Athena, EMR, and Redshift Spectrum all read from it
- Glue crawlers populate it automatically by scanning S3 prefixes

**How Glue fits in a pipeline:**
```
S3 raw data
    │
    ▼
Glue Crawler ──▶ Glue Data Catalog (knows the schema)
                        │
                        ▼
              Athena queries the catalog
              to know where to find data
                        │
              Glue ETL job transforms data
              ──▶ writes clean data back to S3
```

**When to use Glue vs EMR:**
- Glue: serverless, pay-per-use, great for ETL jobs that run occasionally
- EMR: persistent cluster, better for long-running or frequent Spark jobs (cheaper at scale)

---

### AWS Kinesis — Managed Streaming

**What it is:** Amazon's managed event streaming service — conceptually equivalent to Kafka but fully managed by AWS. No brokers to manage, no partitions to configure manually.

**Kinesis components:**
- **Kinesis Data Streams**: core streaming store. Producers write; consumers read. Data retained 24hr (default) to 365 days. Priced per shard-hour.
- **Kinesis Data Firehose**: delivery service. Reads from Kinesis Streams and delivers to S3, Redshift, OpenSearch, or HTTP endpoints automatically. No consumers to write.
- **Kinesis Data Analytics**: run SQL or Apache Flink on streaming data in real-time.

**How Kinesis fits:**
```
Source systems
    │
    ▼
Kinesis Data Streams (event buffer)
    │
    ├──▶ Kinesis Data Analytics (real-time SQL/Flink processing)
    │          │
    │          ▼
    │      Real-time alerts, dashboards
    │
    └──▶ Kinesis Firehose ──▶ S3 (batched every 60 seconds)
                                   │
                                   ▼
                            Athena/Redshift query
```

**Kinesis vs Kafka:**
- Kinesis: no infrastructure to manage; AWS ecosystem integration; limited to 7 days retention (max); pricing by shard
- Kafka: full control; up to infinite retention (disk-limited); runs anywhere; requires operational expertise

**Shard:** Kinesis pricing unit and parallelism unit. 1 shard handles 1MB/s input and 2MB/s output. Scale out by adding shards. Minimum 1 shard, ~$10/month.

---

### AWS Redshift — Cloud Data Warehouse

**What it is:** Amazon's managed data warehouse service. Built on ParAccel (a columnar database), optimised for analytical queries on large datasets.

**Architecture:**
- **Leader node**: receives queries, creates execution plans, coordinates compute nodes
- **Compute nodes**: store and process data. The cluster scales by adding compute nodes.
- **Redshift Spectrum**: query data in S3 directly from Redshift without loading it — the "lakehouse" bridge

**Key features:**
- **COPY command**: fastest way to load data from S3 into Redshift — parallelises across all nodes
- **Materialized views**: stored pre-computed results, auto-refreshed when base data changes
- **Concurrency scaling**: automatically adds capacity during peak demand
- **RA3 nodes**: decouple storage (S3) from compute — pay for storage and compute separately (like Snowflake)
- **Redshift ML**: create ML models using SQL syntax (runs SageMaker under the hood)

**Redshift vs Snowflake vs BigQuery:**

| | Redshift | Snowflake | BigQuery |
|--|---------|-----------|---------|
| Cloud | AWS only | Multi-cloud | GCP only |
| Pricing | Per node-hour | Per credit | Per TB scanned |
| Scaling | Manual cluster resize | Virtual warehouse toggle | Automatic serverless |
| Multi-cluster | Concurrency scaling (auto) | Multiple virtual warehouses | Automatic |
| Data sharing | Yes | Very mature (Data Marketplace) | Analytics Hub |
| Best when | Deep AWS integration needed | Multi-cloud or cloud-agnostic | GCP-native; pay-per-query |

---

### AWS Athena — Serverless SQL on S3

**What it is:** Interactive query service. Write SQL, query data sitting in S3 as Parquet or CSV. Pay only for data scanned. No infrastructure to manage.

**How it works:**
1. Data sits in S3 as Parquet, CSV, JSON, or ORC
2. Table definitions in Glue Data Catalog tell Athena: where is the data, what are the columns, what is the partition scheme
3. You write a SQL SELECT; Athena executes it across the files in S3
4. Pay $5 per TB scanned

**DE use cases:**
- Ad-hoc exploration of raw data lake
- Quick validation of pipeline output
- Lightweight reporting without spinning up a warehouse
- One-time historical queries

**Athena cost optimisation:**
- Use Parquet (columnar) — scans 90% less data than CSV
- Partition your data — Athena skips partitions that don't match the WHERE clause
- Use compression (Snappy or Gzip) — less data to scan = cheaper

---

### AWS Lake Formation — Data Lake Governance

**What it is:** A managed service for building, securing, and managing data lakes on S3. It adds a permission layer on top of S3 + Glue.

**What it does:**
- Fine-grained access control: "allow analyst Alice to query column X in table Y but not column Z" — table, column, and row-level security
- Data lineage: tracks which pipelines produced which data
- Governed tables: ACID transactions on S3 data (Delta Lake-like capabilities)
- Blueprint-based data lake setup: point it at RDS or DynamoDB, it automatically ingests to S3

**When you need Lake Formation:**
- Multiple teams with different access rights to the same data lake
- Compliance requirements (HIPAA, GDPR) requiring column/row-level security
- Centralised governance for a large data organisation

---

### AWS Step Functions — Pipeline Orchestration

**What it is:** A serverless workflow orchestration service. Define workflows as state machines in JSON (Amazon States Language). Each state is a step — Lambda function, Glue job, ECS task, API call.

**When to use Step Functions instead of Airflow:**
- Pipeline steps are primarily AWS services (Lambda, Glue, ECS) — Step Functions integrates natively
- You don't want to manage an Airflow server
- Each pipeline run is billed by state transition (no always-on cost)
- Simpler workflows with < 20 steps

**When Airflow beats Step Functions:**
- Complex DAGs with many parallel branches and joins
- Non-AWS integrations (Snowflake, Databricks, HTTP APIs)
- You need a visual UI with run history and backfill controls
- Your team knows Python

---

### AWS MWAA — Managed Workflows for Apache Airflow

**What it is:** AWS's managed Airflow service. You provide DAG files; AWS manages the Airflow infrastructure (scheduler, webserver, workers, metadata database).

**Advantages over self-hosted Airflow:**
- No infrastructure management — AWS handles version upgrades, scaling, HA
- Native AWS integration — IAM roles for secure access to S3, Redshift, Glue
- CloudWatch integration for logs and metrics

**Considerations:**
- More expensive than self-hosted Airflow on EC2
- Less control over Airflow version and configuration
- Startup time when a worker scales from zero

---

### AWS EMR — Elastic MapReduce

**What it is:** AWS's managed Spark/Hadoop cluster service. Provision a cluster, run Spark/Hive/Presto, terminate the cluster. You pay only for the time the cluster runs.

**EMR vs Glue:**
- EMR: full cluster control, custom Spark configuration, cheaper for long-running or frequent jobs
- Glue: serverless, no cluster management, simpler for ETL jobs that run occasionally

**EMR on EKS:** run Spark jobs on a Kubernetes cluster rather than dedicated EC2 instances — better resource utilisation, faster startup.

**EMR Serverless:** run Spark or Hive without managing a cluster — similar to Glue but with the full EMR ecosystem.

---

## 36. GCP — Google Cloud Platform for Data Engineering

GCP is the home of BigQuery — arguably the most influential product in modern data engineering. Google invented key technologies that the industry is built on: Bigtable (influenced DynamoDB/Cassandra), MapReduce (inspired Hadoop), Colossus (distributed file system), Dremel (BigQuery's engine), Pub/Sub, and Dataflow. If you're working on BigQuery or analytics-heavy workloads, GCP is native territory.

---

### GCS — Google Cloud Storage

**What it is:** GCP's object storage equivalent to S3. Same model: buckets, objects, globally unique names. Used as the data lake raw storage layer in GCP architectures.

**Key differences from S3:**
- Strong consistency (every read immediately sees the latest write — S3 achieved this in 2020)
- Nearline/Coldline/Archive storage classes for cost tiering
- BigQuery can query GCS files directly (external tables) without loading

**GCS storage classes:**

| Class | Access frequency | Cost/GB/month | Retrieval cost |
|-------|-----------------|--------------|----------------|
| Standard | Frequent | ~$0.020 | None |
| Nearline | Once per month | ~$0.010 | $0.01/GB |
| Coldline | Once per quarter | ~$0.004 | $0.02/GB |
| Archive | Once per year | ~$0.0012 | $0.05/GB |

Lifecycle rules automatically move objects between classes based on age.

---

### GCP Pub/Sub — Messaging and Streaming

**What it is:** GCP's managed message queue and event streaming service. Combines elements of Kafka (event streaming) and traditional message queues (at-least-once delivery).

**How it differs from Kafka:**
- No retention of messages by offset — once a subscriber acknowledges a message, it's gone
- Can replay up to 7 days (with message retention feature enabled)
- Global (data routes across GCP's network automatically)
- Strictly at-least-once delivery (no exactly-once natively — use Dataflow for that)
- No concept of partitions — messages are distributed automatically

**DE use cases:**
- Ingest clickstream/event data
- Decouple microservices
- Fan-out: one published message → multiple subscribers process it
- Trigger Dataflow pipelines when new data arrives

**Pub/Sub message flow:**
```
Publisher ──▶ Topic ──▶ Subscription 1 ──▶ Subscriber A (real-time processing)
                   ──▶ Subscription 2 ──▶ Subscriber B (writes to GCS)
                   ──▶ Subscription 3 ──▶ Subscriber C (BigQuery streaming insert)
```

---

### GCP Dataflow — Managed Apache Beam

**What it is:** A fully managed service for executing Apache Beam pipelines — both batch and streaming. Dataflow handles cluster management, auto-scaling, and fault tolerance.

**Apache Beam:** an open-source unified programming model for batch and streaming data processing. You write one pipeline; it runs on Dataflow (GCP), Flink, Spark, or locally. The separation between "writing the pipeline" and "running it" is the key idea.

**Dataflow use cases:**
- ETL from Pub/Sub → BigQuery (streaming ingestion)
- Large-scale batch transformation (replacing Spark for GCP-native workloads)
- Real-time aggregations over event windows

**Dataflow vs Dataproc:**
- Dataflow: serverless, auto-scales, unified batch+stream, simpler operations
- Dataproc: Spark/Hadoop cluster, full control, cheaper for long-running workloads, familiar if your team knows Spark

```python
# Apache Beam pipeline example (runs on Dataflow or locally)
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions

options = PipelineOptions([
    "--runner=DataflowRunner",         # run on GCP Dataflow
    "--project=my-gcp-project",
    "--region=us-central1",
    "--temp_location=gs://my-bucket/temp/",
])

with beam.Pipeline(options=options) as pipeline:
    (
        pipeline
        | "Read from GCS"     >> beam.io.ReadFromText("gs://my-bucket/orders.csv")
        | "Parse CSV"         >> beam.Map(lambda line: line.split(","))
        | "Filter delivered"  >> beam.Filter(lambda row: row[3] == "delivered")
        | "Extract amount"    >> beam.Map(lambda row: float(row[4]))
        | "Sum amounts"       >> beam.CombineGlobally(sum)
        | "Write result"      >> beam.io.WriteToText("gs://my-bucket/result.txt")
    )
```

---

### GCP Dataproc — Managed Spark and Hadoop

**What it is:** GCP's managed Hadoop/Spark cluster service. Equivalent to AWS EMR. Provision clusters with pre-installed Spark, Hive, Presto. Pay per cluster-hour.

**Key features:**
- Clusters start in ~90 seconds (much faster than EMR's 10+ minutes)
- Dataproc Serverless: run Spark workloads without provisioning a cluster
- Native BigQuery connector: Spark jobs can read/write BigQuery directly
- Cloud Storage connector: HDFS operations work seamlessly on GCS

**Dataproc use cases:**
- Migrate on-premises Spark workloads to cloud
- Large-scale data processing where Dataflow (Beam) isn't sufficient
- Hive queries on data lake

---

### GCP Cloud Composer — Managed Airflow

**What it is:** GCP's managed Apache Airflow service. Equivalent to AWS MWAA. Built on GKE (Google Kubernetes Engine).

**Advantages:**
- Native GCP integration — IAM, GCS, BigQuery, Dataflow, Dataproc operators built-in
- Supports Airflow 2.x
- DAG files stored in GCS (shared across all workers automatically)

**Considerations:**
- Expensive — ~$300/month minimum even for small environments
- Slow startup for first task of the day
- Heavy resource consumption (entire Kubernetes cluster)

---

### GCP Data Fusion — Visual ETL

**What it is:** A code-free data integration service based on CDAP (open source). Build ETL pipelines with a drag-and-drop interface. Runs on Dataproc under the hood.

**Use cases:**
- Non-technical data engineers who prefer GUI over code
- Organisations migrating from on-premises ETL tools (Informatica, SSIS, Talend)
- Rapid prototyping of data pipelines

---

### GCP DataStream — CDC and Replication

**What it is:** A serverless CDC (Change Data Capture) service that replicates database changes to GCS or BigQuery in near real-time.

**Supported sources:** MySQL, PostgreSQL, Oracle, SQL Server, AlloyDB
**Supported destinations:** GCS, BigQuery

**How it works:**
1. Connect to source database (read replication slot/binlog)
2. Changes stream to DataStream
3. DataStream delivers to GCS or BigQuery
4. Latency: typically 1–5 minutes

**Use case:** Replicate production MySQL/Postgres databases to BigQuery for analytics without writing custom ETL code.

---

### Looker and Looker Studio — GCP BI Tools

**Looker Studio (formerly Data Studio):** Free, browser-based BI and dashboarding tool. Connects to BigQuery, GCS, Google Sheets, and many third-party sources. Good for simple dashboards. No semantic layer.

**Looker:** Enterprise BI platform, acquired by Google in 2019 for $2.6B. The key differentiator: **LookML** — a data modelling language that defines a "semantic layer" (business logic, metric definitions, access controls) separate from the visualisation layer. "Revenue" is defined once in LookML; every chart using it uses the same definition. Expensive.

---

## 37. Azure — Microsoft Azure for Data Engineering

Azure is the dominant choice in enterprise environments, especially those running Microsoft workloads (Office 365, Dynamics, SQL Server). If a client's IT team is Microsoft-heavy, Azure is often the default cloud choice.

**Azure strengths for DE:**
- Deep enterprise integration — Azure Active Directory, Microsoft licensing synergies
- Strong hybrid cloud story (Azure Arc, ExpressRoute)
- Power BI is the dominant enterprise BI tool and integrates natively
- Azure Databricks is a co-managed service with Databricks

---

### ADLS Gen2 — Azure Data Lake Storage Generation 2

**What it is:** Azure's data lake storage service. A superset of Azure Blob Storage with a Hadoop-compatible hierarchical file system.

**Why "Gen2" matters:** Gen1 was a separate product with high per-operation costs. Gen2 is built on top of Azure Blob Storage (cheaper) but adds:
- **Hierarchical namespace**: true directories (not just path prefixes like S3/GCS)
- **POSIX-compatible permissions**: file and directory level ACLs
- **Lower cost**: blob storage pricing + access tier tiering

**ADLS Gen2 access tiers:**

| Tier | Use case | Cost/GB/month | Access cost |
|------|---------|--------------|-------------|
| Hot | Frequently accessed | ~$0.018 | Low |
| Cool | Infrequent (~30 days) | ~$0.01 | Medium |
| Cold | Rarely accessed (~90 days) | ~$0.0045 | Higher |
| Archive | Long-term retention | ~$0.0018 | Highest (hrs to rehydrate) |

---

### Azure Synapse Analytics — Unified Analytics Platform

**What it is:** Microsoft's "everything for analytics" platform. It combines:
- **Synapse SQL Pool** (dedicated): a data warehouse, similar to Redshift
- **Synapse Serverless SQL**: query ADLS files with SQL on-demand (like Athena)
- **Synapse Spark**: managed Spark clusters for big data processing
- **Synapse Pipelines**: data integration (similar to Azure Data Factory, now integrated)
- **Synapse Link**: near-real-time sync from CosmosDB/Dataverse to Synapse

**The "unified" promise:** One service for warehouse + lake + Spark + orchestration. In practice, teams often use dedicated services (Databricks for Spark, Data Factory for pipelines) rather than everything through Synapse.

**Synapse vs Databricks on Azure:**
- Synapse: Microsoft-native, tight Power BI integration, SQL Server-like experience
- Databricks on Azure: better Spark performance, Delta Lake, better ML integration, more flexibility

---

### Azure Data Factory — Orchestration and ETL

**What it is:** Azure's managed data integration and orchestration service. Two main capabilities:
- **Data Factory pipelines**: visual ETL workflows (drag-and-drop, like Informatica)
- **Mapping Data Flows**: code-free data transformation (Spark-based under the hood)

**ADF vs Airflow:**
- ADF: visual, Azure-native, strong Microsoft connector library (Dynamics, SharePoint, SQL Server)
- Airflow: code-based, more flexible, any cloud/tool

**ADF components:**
- **Linked Services**: connections to data sources and sinks
- **Datasets**: representation of data in linked services
- **Activities**: tasks (Copy Data, Execute Databricks Notebook, Run Stored Procedure, etc.)
- **Pipeline**: sequence of activities
- **Trigger**: schedule or event that starts a pipeline
- **Integration Runtime**: the compute used to run activities (Azure-hosted, self-hosted, or Azure-SSIS)

**Integration Runtime types:**
- **Azure IR**: runs in the Azure cloud, for cloud-to-cloud data movement
- **Self-hosted IR**: runs on your on-premises or private network — for connecting to on-premises databases (SQL Server, Oracle) behind a firewall
- **Azure-SSIS IR**: runs SSIS packages in the cloud

---

### Azure Event Hubs — Managed Event Streaming

**What it is:** Azure's managed event streaming service — functionally equivalent to Kafka and AWS Kinesis. Event Hubs is actually **Kafka-compatible**: you can use Kafka client libraries to connect to it without any code changes.

**Key concepts:**
- **Namespace**: a container for multiple event hubs
- **Event Hub**: equivalent to a Kafka topic
- **Partition**: events are distributed across partitions
- **Consumer Group**: equivalent to Kafka consumer group
- **Capture**: automatically persist events to ADLS Gen2 or Azure Blob Storage — no consumer code needed

**Event Hubs Kafka compatibility:**
```python
# Standard Kafka client — works with Event Hubs
from confluent_kafka import Producer

producer = Producer({
    "bootstrap.servers": "my-namespace.servicebus.windows.net:9093",
    "sasl.mechanisms":   "PLAIN",
    "security.protocol": "SASL_SSL",
    "sasl.username":     "$ConnectionString",
    "sasl.password":     "Endpoint=sb://...",   # connection string
})
# Rest of Kafka code unchanged
```

---

### Azure Databricks — Managed Databricks on Azure

**What it is:** Databricks (the company) and Microsoft have a co-managed partnership. Azure Databricks is Databricks running on Azure infrastructure, with native Azure integrations:
- Azure Active Directory for authentication
- ADLS Gen2 for storage
- Azure Key Vault for secrets
- Power BI for serving (direct connector)

It is identical to Databricks on AWS/GCP in terms of features, but billing goes through Azure and it integrates with Azure security and governance tools.

---

### Azure Purview — Data Governance Catalog

**What it is:** Microsoft's enterprise data governance service — a metadata management and data lineage tool across all Azure and non-Azure data sources.

**What it does:**
- **Data Map**: automatically scans and catalogues data sources (Azure SQL, ADLS, Synapse, Power BI, on-premises, even non-Azure sources)
- **Data Catalog**: searchable inventory of all data assets with business glossary, ownership, classifications
- **Data Lineage**: tracks how data flows from source to report (end-to-end lineage)
- **Sensitivity Labels**: classify and track sensitive data (PII, financial, health)
- **Access policies**: unified access management across data sources

---

### Azure HDInsight — Managed Hadoop/Spark

**What it is:** Azure's managed Hadoop/Spark/Kafka/HBase cluster service. Equivalent to AWS EMR.

**Honest assessment:** HDInsight is less commonly chosen in new architectures. Azure Databricks is usually preferred for Spark workloads because of its better UX and managed Delta Lake support. HDInsight remains relevant for organisations with existing Hadoop investments.

---

## 38. Databricks — Unified Analytics Platform

### What Databricks is

**Databricks** was founded in 2013 by the original creators of Apache Spark at UC Berkeley. The company's mission is to democratise data and AI. It runs on all three major clouds (AWS, GCP, Azure) and is the primary commercial sponsor of Apache Spark and Delta Lake.

**What makes Databricks distinct from the cloud providers:**
- Multi-cloud (works the same on AWS, GCP, Azure)
- Built around a "Lakehouse" architecture — combines best of data lake and warehouse
- Delta Lake is native and first-class
- Excellent ML/AI integration via MLflow
- Best-in-class collaborative notebooks (like Jupyter but more powerful)
- Strong community — massive Spark documentation and support

---

### Databricks architecture

```
DATABRICKS PLATFORM

┌────────────────────────────────────────────────────────────────────────┐
│                       DATABRICKS WORKSPACE                              │
│                                                                         │
│  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────────┐  │
│  │ Notebooks  │  │    Jobs    │  │  SQL Editor│  │ Delta Live     │  │
│  │ (collab    │  │ (scheduled │  │  (SQL      │  │ Tables         │  │
│  │  coding)   │  │  pipelines)│  │  analysts) │  │ (streaming ETL)│  │
│  └─────┬──────┘  └─────┬──────┘  └─────┬──────┘  └───────┬────────┘  │
│        │               │               │                  │           │
│        └───────────────┼───────────────┼──────────────────┘           │
│                        ▼               ▼                               │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │              CLUSTER MANAGER                                     │  │
│  │  (Databricks Runtime = Spark + Delta Lake + libraries)          │  │
│  │  Clusters: All-purpose (dev) | Job clusters (prod)              │  │
│  └──────────────────────────────────────────────────────────────────┘  │
│                              │                                          │
│                              ▼                                          │
│  ┌──────────────────────────────────────────────────────────────────┐  │
│  │              UNITY CATALOG (Governance)                          │  │
│  │  Unified metastore across all workspaces and clouds             │  │
│  │  Column-level security · Row filters · Data lineage             │  │
│  └──────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
              Cloud object storage (S3/GCS/ADLS)
              storing data as Delta Lake files
```

---

### Databricks key features — explained

**Delta Lake (on Databricks):**
Delta Lake is open source (Apache License), but Databricks adds performance optimisations on top:
- **OPTIMIZE**: compacts small files into larger ones — improves query performance
- **ZORDER**: data skipping within files (like a multi-column index)
- **AUTO OPTIMIZE**: automatically runs OPTIMIZE and ZORDER in the background
- **Deletion vectors**: marks rows as deleted without rewriting files (faster DELETEs)
- **Liquid Clustering** (new): smarter, adaptive partitioning

**Delta Live Tables (DLT):**
A declarative framework for building reliable data pipelines with Delta Lake. You define what the data should look like, and Databricks handles how to compute it.

```python
# Delta Live Tables — declarative pipeline definition
import dlt
from pyspark.sql import functions as F

@dlt.table(
    comment="Raw orders from Olist ingestion",
    table_properties={"quality": "bronze"}
)
def raw_orders():
    return spark.read.csv("/mnt/raw/olist_orders.csv", header=True)

@dlt.table(
    comment="Cleaned orders with derived columns",
    table_properties={"quality": "silver"}
)
@dlt.expect("order_id_not_null", "order_id IS NOT NULL")   # built-in data quality
@dlt.expect_or_drop("valid_status", "order_status IN ('delivered','shipped','processing')")
def clean_orders():
    return (
        dlt.read("raw_orders")
        .withColumn("ordered_at", F.to_timestamp("order_purchase_timestamp"))
        .withColumn("delivery_days",
            F.datediff("order_delivered_customer_date", "order_purchase_timestamp"))
    )

@dlt.table(
    comment="Monthly revenue aggregation for BI",
    table_properties={"quality": "gold"}
)
def monthly_revenue():
    return (
        dlt.read("clean_orders")
        .filter(F.col("order_status") == "delivered")
        .groupBy(F.date_format("ordered_at", "yyyy-MM").alias("month"))
        .agg(F.sum("payment_value").alias("revenue"))
    )
```

**The Medallion Architecture** (Bronze/Silver/Gold) is most commonly associated with Databricks:
```
Bronze (raw):    Data as-is from source, minimal transformation, stored forever
Silver (cleaned): Validated, deduplicated, typed, enriched
Gold (serving):  Business-level aggregations ready for BI and ML
```

**Unity Catalog:**
Databricks' unified data governance layer. Before Unity Catalog (2022), each Databricks workspace had its own Hive Metastore — you couldn't share tables across workspaces. Unity Catalog adds:
- Unified namespace across all workspaces and clouds
- Column-level and row-level security
- Data lineage across all Databricks operations
- Fine-grained access control
- Data sharing across organisations

**MLflow:**
Open-source ML lifecycle management platform (created at Databricks):
- **Tracking**: log experiments, parameters, metrics, models
- **Models**: version and store ML models
- **Registry**: production model versioning and promotion workflow
- **Projects**: reproducible ML code packaging

---

### Databricks on the three clouds

| Feature | AWS | GCP | Azure |
|---------|-----|-----|-------|
| Storage | S3 | GCS | ADLS Gen2 |
| Auth | IAM | GCP IAM | Azure AD |
| Secrets | AWS Secrets Manager | GCP Secret Manager | Azure Key Vault |
| BI integration | QuickSight (limited) | Looker Studio | Power BI (excellent) |
| Billing | AWS bill + Databricks DBUs | GCP bill + DBUs | Azure bill + DBUs |
| Managed offering name | AWS Databricks | Google Databricks | Azure Databricks |

**DBU (Databricks Unit):** The Databricks pricing unit. 1 DBU per hour on a standard cluster node. Pricing: ~$0.07–$0.55 per DBU depending on workload type (jobs, SQL, ML) and tier.

---

## 39. Cross-Cloud Service Equivalence

### Complete service equivalence tables

| Category | AWS | GCP | Azure | Databricks | Snowflake |
|----------|-----|-----|-------|------------|-----------|
| **Object Storage** | S3 | Google Cloud Storage | Azure Blob Storage / ADLS Gen2 | — | — |
| **Data Warehouse** | Redshift | BigQuery | Synapse Analytics | Databricks SQL | Snowflake |
| **Data Lake** | S3 + Glue | GCS + BigQuery | ADLS Gen2 + Synapse | Delta Lake on cloud storage | Snowflake external tables |
| **Lakehouse** | S3 + Iceberg/Delta | GCS + BigQuery + BQ Omni | ADLS Gen2 + Delta Lake | Delta Lake (native) | Icehouse (future) |
| **Batch Processing** | EMR (Spark), Glue | Dataproc (Spark), Dataflow | Azure Databricks, HDInsight | Spark (native) | Snowpark |
| **Streaming** | Kinesis | Pub/Sub | Event Hubs | Spark Structured Streaming | Snowpipe (near-RT) |
| **Stream Processing** | Kinesis Analytics (Flink) | Dataflow (Beam) | Azure Stream Analytics | Structured Streaming | — |
| **ETL / Integration** | Glue, AppFlow | Data Fusion, Dataflow | Data Factory | DLT, notebooks | — |
| **Orchestration** | MWAA (Airflow), Step Functions | Cloud Composer (Airflow) | Data Factory | Databricks Jobs, DLT | Snowflake Tasks |
| **Metadata Catalog** | Glue Data Catalog, Lake Formation | Data Catalog, Dataplex | Azure Purview | Unity Catalog | Snowflake Data Catalog |
| **Governance** | Lake Formation, Macie | Dataplex, DLP | Purview | Unity Catalog | Snowflake Horizon |
| **Data Sharing** | AWS Clean Rooms | Analytics Hub | Azure Data Share | Delta Sharing | Data Marketplace |
| **Managed Airflow** | MWAA | Cloud Composer | — | — | — |
| **Notebook / IDE** | SageMaker Studio | Vertex AI Workbench | Azure Machine Learning | Databricks Notebooks | Snowflake Notebooks |
| **BI Tool** | QuickSight | Looker / Looker Studio | Power BI | — | — |
| **ML Platform** | SageMaker | Vertex AI | Azure ML | MLflow (native) | Snowflake ML |
| **CDC / Replication** | DMS, AppFlow | DataStream | Data Factory, Synapse Link | DLT | Snowflake Dynamic Tables |
| **Vector / GenAI** | Bedrock, OpenSearch | Vertex AI, AlloyDB | Azure OpenAI, AI Search | Mosaic AI | Cortex AI |

---

### Data journey mapped to cloud services

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                  COMPLETE DATA JOURNEY — ALL CLOUD SERVICES                 │
├──────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  1. SOURCES           2. INGEST           3. STORE (RAW)                    │
│  ─────────────        ──────────────       ─────────────────                 │
│  Operational DB    →  AWS:  Kinesis,       AWS: S3                          │
│  SaaS APIs            Glue, DMS            GCP: GCS                         │
│  IoT / Events      →  GCP:  Pub/Sub,       Azure: ADLS Gen2                 │
│  Files / CSVs          DataStream                                            │
│  CDC / Logs        →  Azure: Event Hubs,                                    │
│                        Data Factory                                          │
│                   →  Cross: Fivetran,                                       │
│                        Airbyte, Kafka                                        │
│                                                                              │
│  4. PROCESS           5. STORE (CLEAN)    6. SERVE                          │
│  ─────────────        ───────────────────  ──────────────────                │
│  AWS: EMR, Glue  →    AWS: Redshift,       AWS: Athena, QuickSight          │
│  GCP: Dataproc,        S3 + Delta          GCP: BigQuery, Looker             │
│    Dataflow      →    GCP: BigQuery,       Azure: Synapse, Power BI          │
│  Azure: Databricks,    GCS + Delta         DB: Databricks SQL               │
│    HDInsight     →    Azure: Synapse,      Snowflake: Snowflake              │
│  DB: Spark (native)     ADLS + Delta                                        │
│  Snowflake: Snowpark →  Snowflake                                            │
│  Cross: dbt, Spark      DB: Delta Lake                                      │
│                                                                              │
│  7. ORCHESTRATE       8. GOVERN           9. OBSERVE                        │
│  ─────────────────    ──────────────────   ─────────────────                 │
│  AWS: MWAA,          AWS: Lake Formation,  AWS: CloudWatch,                 │
│    Step Functions     Glue Catalog          CloudTrail                       │
│  GCP: Cloud           GCP: Dataplex,       GCP: Cloud Logging               │
│    Composer            Data Catalog         Azure: Monitor                   │
│  Azure: Data           Azure: Purview       DB: Databricks                  │
│    Factory             DB: Unity Catalog      observability                  │
│  Cross: Airflow,       Snowflake: Horizon   Cross: Monte Carlo,              │
│    Prefect,                                  Great Expectations              │
│    Dagster, dbt                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 40. Choosing a Cloud for Data Engineering

### Decision framework

There is no universally "best" cloud for data engineering. The right choice depends on your organisation's existing investments, team expertise, data sources, and specific workload requirements.

**Follow existing investments first:**

| Your organisation already uses... | Start with... |
|-----------------------------------|---------------|
| AWS for other workloads | AWS data services (EMR, Redshift, Glue) |
| Microsoft 365, Dynamics, SQL Server | Azure (Power BI, Synapse, Data Factory) |
| GCP (Gmail/Drive business tier) | GCP (BigQuery is excellent) |
| Mixed / no preference | Start with BigQuery (lowest operational overhead) or Snowflake (multi-cloud flexibility) |

**Workload-specific guidance:**

| Workload | Best choice | Why |
|----------|------------|-----|
| Pure analytics / BI reporting | BigQuery or Snowflake | Serverless/managed warehouse; SQL-first; easiest to start |
| ML + analytics on same data | Databricks | Unified Spark + MLflow + Delta Lake |
| Enterprise Microsoft shop | Azure Synapse + Power BI | Native AAD, Power BI connector, Microsoft support |
| Real-time streaming | AWS Kinesis or Kafka (any cloud) | Mature ecosystems |
| Multi-cloud data sharing | Snowflake or Delta Sharing | Cloud-agnostic |
| Maximum cloud vendor lock-in avoidance | Databricks + Delta Lake + Iceberg | Open standards, runs anywhere |
| Fast time to insight, minimal ops | BigQuery | Serverless, no infrastructure, pay-per-query |
| Cost at petabyte scale | Databricks on Spot/Preemptible | Spark on cheap spot instances |

---

## 41. Data Engineering in Presales Conversations

### Why presales matters for a data engineer

As you move toward a Product/Service Owner role, you'll frequently be in conversations where clients describe their data challenges and you need to map those challenges to solutions. This section covers the patterns you'll see repeatedly.

---

### Client maturity stages — what you'll encounter

Most clients you'll work with at Hexaware fall into one of four maturity stages. Recognising the stage quickly lets you have the right conversation.

**Stage 1 — No data infrastructure ("We have data but can't use it")**

Typical profile: data lives in spreadsheets, operational databases, and departmental silos. No data warehouse. Analysts export CSVs and do analysis locally.

What they say:
- "Our reports take days to produce"
- "Different teams have different numbers for the same metric"
- "We can't combine data from System A and System B"
- "Our analysts spend 80% of their time preparing data"

What they need: a foundational data platform — ingestion, a simple warehouse, basic reporting.

Your pitch: start with a cloud data warehouse (Snowflake or BigQuery), 2–3 Fivetran connectors for their key sources, a dbt project for basic transformations, and a BI tool (Power BI or Looker Studio). This can be operational in weeks.

**Stage 2 — Basic warehouse, struggling to scale ("Our data platform is a mess")**

Typical profile: has a data warehouse (often on-premises — Teradata, SQL Server DW, or early cloud — Redshift). Analysts can query it. But pipelines are fragile, undocumented, and slow.

What they say:
- "Our pipelines break constantly and we don't know why"
- "We have data but nobody trusts it"
- "Adding a new data source takes months"
- "Our warehouse queries are slow and expensive"
- "We have technical debt from 5 years of quick fixes"

What they need: modernisation — migrate to a modern cloud warehouse, implement data quality, implement proper orchestration, add documentation.

Your pitch: Airflow or Prefect for orchestration, dbt for documented and tested transformations, Great Expectations for data quality, migrate to Snowflake/BigQuery. The story is "reliability and trust."

**Stage 3 — Modern stack, scaling ("We need real-time data and ML")**

Typical profile: modern cloud warehouse, working pipelines, analysts are productive. Now hitting limitations: data is too slow (daily refreshes not enough), want ML models in production.

What they say:
- "Our business needs real-time data, not yesterday's"
- "We want to put ML models into our products"
- "We need to handle 10× more data volume"
- "Our analysts want self-service access to more data sources"

What they need: streaming ingestion (Kafka/Kinesis), feature stores, Databricks for ML, data mesh or distributed ownership.

Your pitch: streaming architecture with Kafka + Flink/Spark Structured Streaming, Databricks for ML, Delta Lake for the lakehouse pattern. The story is "from insights to real-time action."

**Stage 4 — Advanced, operating at scale ("We need to optimise and govern")**

Typical profile: large data platform, many teams using it, significant cloud spend, compliance requirements.

What they say:
- "Our cloud data costs are out of control"
- "We have data governance and compliance requirements (GDPR, HIPAA)"
- "Different teams are building the same pipelines independently"
- "We need data lineage for regulatory audits"
- "Our data platform team is a bottleneck for every project"

What they need: data platform team, data mesh, governance (Purview/Unity Catalog), cost optimisation, data products.

Your pitch: platform engineering, Unity Catalog or Purview for governance, data mesh operating model, cost optimisation review. The story is "platform as a product."

---

### Common client objections — and how to address them

**"Why can't we just use Excel/existing BI tool?"**

Root cause: they don't understand the volume, velocity, or variety problem they'll hit.

Response: "Excel works well for < 100k rows and 1–2 analysts. When you have millions of rows, dozens of analysts, data from 10 source systems, and need consistent numbers across reports — Excel becomes the bottleneck. The cost of a data platform is less than the cost of incorrect decisions made from inconsistent data."

---

**"Build vs Buy — why not just buy a packaged solution?"**

Root cause: they want to reduce risk and implementation effort.

Response: "Packaged solutions like Domo, Mode, or Tableau Prep handle specific use cases well. The limitation is that every business has unique data, unique business logic, and unique integration needs. Packaged tools are designed for the common case. dbt + Snowflake is already mostly 'buy' — you're paying for managed services; you're only building the business logic that's unique to you."

---

**"We're worried about cloud vendor lock-in"**

Root cause: past experience with technology dependencies, or enterprise risk management culture.

Response: "This is a legitimate concern. The strategy is to minimise lock-in at the most expensive layers. Object storage (S3/GCS/ADLS) is commodity — data can move. Parquet files on S3 are readable by every tool. The warehouse layer (Snowflake, BigQuery) has more lock-in, but migration tools exist. Databricks + Delta Lake + open formats is the most portable architecture. The real question is: is the cost of avoiding lock-in greater than the cost of using the best tool for the job?"

---

**"How do we justify the cost?"**

Root cause: limited data on ROI; stakeholders see costs but not benefits.

Response: quantify the cost of the problem, not just the cost of the solution. Concrete framings:
- "If a data analyst earns ₹15L/year and spends 40% of their time on data preparation, that's ₹6L/year of preparation cost per analyst. A data platform that reduces this to 10% pays for itself with 3–4 analysts."
- "A wrong business decision based on incorrect data — a missed market opportunity, a product launched in the wrong region — easily costs 10–100× the cost of a data platform."
- "Cloud data costs at your scale (~5TB/month) run $2–5k/month. That's less than one analyst's monthly salary."

---

**"Our data is too sensitive to put in the cloud"**

Root cause: security and compliance concerns; sometimes misunderstanding of cloud security.

Response: "Cloud providers invest billions annually in security — more than most organisations could spend on-premises. The major clouds have SOC2, ISO 27001, HIPAA, PCI-DSS certifications. The real question is: what is your specific compliance requirement? GDPR requires data residency — you can choose an EU data region. HIPAA allows cloud storage with a Business Associate Agreement. Financial data — Snowflake Business Critical or BigQuery have specific configurations. Let's map your specific requirements to the right configuration, rather than avoiding cloud altogether."

---

**"Migration will break our existing reports"**

Root cause: fear of disruption to current business operations.

Response: "Migration risk is real and manageable. The approach is: run old and new in parallel, validate that numbers match, then cut over report by report. We can start with new reports that don't exist yet — no risk. Then migrate non-critical reports. Then mission-critical ones last, with the longest parallel-run period. The goal is zero surprises on cutover day."

---

**"Our IT team doesn't have cloud skills"**

Root cause: skills gap; also sometimes political — IT wants to maintain control.

Response: "This is the most honest objection and the most common. The answer depends on your timeline. If you want to build internal capability: we structure the engagement as joint delivery + knowledge transfer, not just delivery. Your team learns by doing. If you need results fast with skill-building secondary: we deliver the platform, document it thoroughly, and provide hypercare support. Most clients end up wanting a hybrid — we build the foundation while your team comes up to speed."

---

### Common presales questions about specific technologies

**"Should we use Snowflake or BigQuery?"**

Honest answer: for most organisations, both work. The differentiators:
- Already on GCP → BigQuery (native, cheaper, no multi-cloud tax)
- Multi-cloud or cloud-agnostic → Snowflake (runs on any cloud, same experience)
- Very high query volume, complex workloads → depends on workload profiling
- Budget-conscious, pay-per-query → BigQuery (Snowflake credits burn even on idle virtual warehouses)

**"Do we need Kafka if we're not Netflix?"**

Honest answer: probably not at first. Kafka is engineering-intensive to operate. For most organisations, you need streaming when:
- Your data latency requirement is < 5 minutes
- You have > 100k events per second
- Multiple teams need to independently consume the same event stream

Otherwise: hourly batch (Airflow → Glue/dbt) handles 90% of use cases at 10% of the complexity.

**"Is Databricks worth the premium over just using Spark on EMR?"**

Honest answer: the DBU premium is ~30–50% over raw Spark on EMR. You get: managed infrastructure, Delta Lake optimisations, collaborative notebooks, Unity Catalog, MLflow, automatic cluster scaling. For teams doing both DE and ML on the same data, the productivity gains typically outweigh the cost premium. For pure ETL with no ML, EMR or Glue may be more cost-effective.

**"What is a data lakehouse and do we need it?"**

Honest answer: a lakehouse is the combination of (1) cheap object storage like S3 for data, (2) a table format (Delta Lake or Iceberg) for ACID transactions and schema, and (3) a query engine that can query it as if it were a warehouse. You likely want it if:
- You have both ML workloads (need raw data, Python, notebooks) and BI workloads (need fast SQL)
- You want to avoid having two copies of data (one in the lake, one in the warehouse)
- You want time travel and ACID on lake data

Otherwise, a data lake + separate warehouse (copy data from lake to warehouse for BI) is perfectly fine and simpler.

---
---

## Appendix A: DE Tool Comparison Matrix

```
╔══════════════════════════════════════════════════════════════════════════════════════╗
║                     DATA ENGINEERING TOOL LANDSCAPE — COMPLETE                      ║
╠═══════════════╦════════════════════════════╦══════════════════════════════════════════╣
║ CATEGORY      ║ TOOLS                      ║ WHEN TO USE                              ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ FILE FORMATS  ║ CSV                        ║ Data exchange with external parties      ║
║               ║ JSON / JSONL               ║ API responses, event logs                ║
║               ║ Parquet (default)          ║ ALL analytics pipelines, data lakes      ║
║               ║ Avro                       ║ Kafka messages, schema evolution         ║
║               ║ ORC                        ║ Legacy Hive/Hadoop ecosystems            ║
║               ║ Delta Lake                 ║ ACID on lake, time travel, Databricks    ║
║               ║ Apache Iceberg             ║ Multi-engine ACID, engine-agnostic       ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ COMPUTE       ║ pandas                     ║ < 500MB, row ops, ML prep, EDA           ║
║               ║ DuckDB                     ║ < 50GB SQL analytics, local dev          ║
║               ║ Polars                     ║ Fast single-machine pandas alternative   ║
║               ║ PySpark (Apache Spark)     ║ > 50GB distributed, Databricks/EMR       ║
║               ║ Dask                       ║ pandas API at medium scale               ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ STREAMING     ║ Apache Kafka               ║ Max control, multi-cloud, long retention ║
║               ║ AWS Kinesis                ║ AWS-native, serverless streaming         ║
║               ║ GCP Pub/Sub                ║ GCP-native, global messaging             ║
║               ║ Azure Event Hubs           ║ Azure-native, Kafka-compatible           ║
║               ║ Apache Flink               ║ Complex stateful stream processing       ║
║               ║ Spark Structured Streaming ║ Spark teams want streaming too           ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ INGESTION     ║ Custom Python              ║ Unusual APIs, internal systems           ║
║               ║ Fivetran                   ║ SaaS connectors, zero maintenance        ║
║               ║ Airbyte (open source)      ║ Custom connectors, cost-conscious        ║
║               ║ AWS Glue                   ║ AWS-native ETL, Glue Catalog             ║
║               ║ Azure Data Factory         ║ Azure-native, Microsoft sources          ║
║               ║ GCP Data Fusion            ║ GCP-native, visual ETL                   ║
║               ║ Debezium                   ║ CDC from relational databases to Kafka   ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ STORAGE LAKE  ║ Amazon S3                  ║ AWS data lake raw storage                ║
║               ║ Google Cloud Storage       ║ GCP data lake raw storage                ║
║               ║ Azure ADLS Gen2            ║ Azure data lake (hierarchical FS)        ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ WAREHOUSE     ║ Snowflake                  ║ Multi-cloud, virtual WH separation       ║
║               ║ Google BigQuery            ║ Serverless GCP, pay-per-query            ║
║               ║ Amazon Redshift            ║ AWS-native, RA3 separation               ║
║               ║ Azure Synapse              ║ Microsoft stack, Power BI native         ║
║               ║ Databricks SQL             ║ Lakehouse SQL, Delta Lake native         ║
║               ║ DuckDB                     ║ Local dev, embedded analytics, free      ║
║               ║ PostgreSQL                 ║ Small-medium scale, open source          ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ TRANSFORM     ║ dbt                        ║ SQL transforms in warehouse, standard    ║
║               ║ Python/pandas              ║ Complex logic SQL can't do               ║
║               ║ PySpark                    ║ Large-scale distributed transforms       ║
║               ║ Snowpark                   ║ Python/Java/Scala inside Snowflake       ║
║               ║ Delta Live Tables          ║ Declarative Databricks pipelines         ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ ORCHESTRATE   ║ Apache Airflow             ║ Most enterprises, widest operator lib    ║
║               ║ Prefect                    ║ Python-native, local dev friendly        ║
║               ║ Dagster                    ║ Asset-oriented, best observability       ║
║               ║ AWS MWAA                   ║ Managed Airflow on AWS                   ║
║               ║ GCP Cloud Composer         ║ Managed Airflow on GCP                   ║
║               ║ AWS Step Functions         ║ AWS-native serverless workflows          ║
║               ║ Azure Data Factory         ║ Azure ETL + orchestration                ║
║               ║ dbt Cloud                  ║ dbt-only scheduling, simple setup        ║
║               ║ cron                       ║ Single-server simple scripts             ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ DATA QUALITY  ║ Great Expectations         ║ Production DQ, data docs, expectation lib║
║               ║ dbt tests                  ║ DQ inside dbt (unique, not_null, etc.)   ║
║               ║ Pandera                    ║ pandas-native schema validation          ║
║               ║ Monte Carlo               ║ Managed observability, anomaly detection  ║
║               ║ Custom pytest              ║ Unit tests for transform functions        ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ CATALOG /     ║ AWS Glue Data Catalog      ║ AWS Hive-compatible metastore            ║
║ GOVERNANCE    ║ AWS Lake Formation         ║ AWS fine-grained access control          ║
║               ║ GCP Dataplex               ║ GCP data mesh governance                 ║
║               ║ Azure Purview              ║ Microsoft enterprise governance          ║
║               ║ Databricks Unity Catalog   ║ Multi-cloud unified metastore            ║
║               ║ Apache Atlas               ║ Open source catalog (Hadoop ecosystem)   ║
╠═══════════════╬════════════════════════════╬══════════════════════════════════════════╣
║ SERVING / BI  ║ Tableau                    ║ Enterprise visual analytics              ║
║               ║ Microsoft Power BI         ║ Microsoft ecosystem, Azure-native        ║
║               ║ Google Looker              ║ Semantic layer (LookML), enterprise      ║
║               ║ Looker Studio              ║ Free GCP dashboards                      ║
║               ║ Metabase (open source)     ║ Self-hosted BI, small teams              ║
║               ║ Apache Superset            ║ Open source BI, SQL-first                ║
║               ║ Evidence                   ║ Code-first BI (SQL + Markdown)           ║
╚═══════════════╩════════════════════════════╩══════════════════════════════════════════╝
```

---

## Appendix B: pip Install Guide for DE

```bash
# ── CORE (already from Python 101) ──────────────────────────────────────────
pip install pandas numpy matplotlib seaborn sqlalchemy openpyxl

# ── FILE FORMATS ─────────────────────────────────────────────────────────────
pip install pyarrow            # Parquet read/write — essential
pip install fastparquet        # Alternative Parquet library
pip install fastavro           # Avro read/write

# ── LOCAL ANALYTICS ──────────────────────────────────────────────────────────
pip install duckdb             # In-process OLAP — local data warehouse

# ── DATA QUALITY ─────────────────────────────────────────────────────────────
pip install great_expectations  # Production data quality
pip install pandera             # pandas schema validation
pip install cerberus            # Dict/JSON validation

# ── ORCHESTRATION ────────────────────────────────────────────────────────────
# Apache Airflow (full install)
pip install apache-airflow
pip install "apache-airflow[postgres,google,amazon,slack,http,dbt]"

# Prefect
pip install prefect prefect-aws prefect-gcp prefect-dbt

# Dagster
pip install dagster dagster-webserver dagster-pandas

# ── DISTRIBUTED PROCESSING ───────────────────────────────────────────────────
pip install pyspark            # Apache Spark Python API
pip install polars             # Fast single-machine alternative to pandas
pip install dask[complete]     # Parallel pandas

# ── STREAMING ────────────────────────────────────────────────────────────────
pip install confluent-kafka    # Kafka producer/consumer (recommended)
pip install kafka-python       # Alternative Kafka client

# ── TRANSFORMATION ───────────────────────────────────────────────────────────
# dbt (choose your database connector)
pip install dbt-core
pip install dbt-duckdb         # for DuckDB
pip install dbt-postgres       # for PostgreSQL
pip install dbt-bigquery       # for BigQuery
pip install dbt-snowflake      # for Snowflake

# ── CLOUD CLIENTS ────────────────────────────────────────────────────────────
# AWS
pip install boto3              # AWS SDK — S3, Redshift, Glue, etc.
pip install awswrangler        # pandas for AWS (S3, Redshift, Athena)

# GCP
pip install google-cloud-storage      # GCS
pip install google-cloud-bigquery     # BigQuery
pip install google-cloud-pubsub       # Pub/Sub
pip install pandas-gbq                # pandas ↔ BigQuery

# Azure
pip install azure-storage-blob        # Azure Blob / ADLS
pip install azure-identity            # Azure authentication

# Snowflake
pip install snowflake-connector-python
pip install snowflake-sqlalchemy       # SQLAlchemy dialect

# Databricks
pip install databricks-sdk            # Databricks REST API
pip install databricks-connect        # Connect local code to Databricks cluster

# ── LAKEHOUSE ────────────────────────────────────────────────────────────────
pip install delta-spark         # Delta Lake with PySpark
pip install pyiceberg           # Apache Iceberg Python library

# ── TESTING ──────────────────────────────────────────────────────────────────
pip install pytest pytest-mock  # Unit testing
pip install hypothesis          # Property-based testing for data

# ── LOGGING / MONITORING ─────────────────────────────────────────────────────
pip install structlog            # Structured JSON logging
pip install prometheus-client    # Metrics for Prometheus

# ── ALL DE TOOLS AT ONCE (development environment) ───────────────────────────
pip install pandas numpy pyarrow duckdb great_expectations \
            apache-airflow polars fastavro dbt-core dbt-duckdb \
            pytest pandera structlog boto3 confluent-kafka
```

---

## Appendix C: Glossary — 100+ Terms

| Term | Definition |
|------|-----------|
| **ACID** | Atomicity, Consistency, Isolation, Durability — properties guaranteeing database transaction reliability |
| **Airflow** | Open-source workflow orchestration platform; pipelines defined as Python DAGs |
| **Athena** | AWS serverless SQL query service that reads data from S3 |
| **Avro** | Binary row-based format with embedded schema; used with Kafka |
| **Backfill** | Re-running a pipeline for historical dates it missed or needs to reprocess |
| **Batch processing** | Processing data in bulk at fixed intervals (hourly, daily, weekly) |
| **BigQuery** | Google Cloud's serverless columnar data warehouse |
| **Broker** | A Kafka server that stores partitions and serves producers/consumers |
| **Bronze layer** | Raw ingested data in a Medallion architecture (no transformation) |
| **Catchup** | Airflow feature to run DAGs for all missed scheduled intervals |
| **CDC** | Change Data Capture — capture row-level inserts/updates/deletes from a database |
| **Checkpoint** | Saved stream processing state; allows recovery without losing progress |
| **Clustering** | Organising data within partitions by column value to improve query performance |
| **Columnar format** | File format storing data by column (Parquet, ORC) — fast for analytical queries |
| **Compaction** | Merging many small files into fewer large files to improve read performance |
| **Consumer group** | A set of Kafka consumers that collectively process a topic, each partition to one consumer |
| **DAG** | Directed Acyclic Graph — a pipeline's execution structure with tasks and dependencies |
| **Dagster** | Asset-oriented data orchestration platform with strong observability |
| **Data catalog** | Metadata repository listing all data assets — what exists, where, who owns it |
| **Data contract** | Formal agreement between data producer and consumer on schema and quality |
| **Data lake** | Raw file storage — all formats, cheap, schema-on-read (S3, GCS, ADLS) |
| **Data lakehouse** | Hybrid: lake storage (cheap) + warehouse semantics (ACID, SQL) |
| **Data lineage** | Record of where data came from and every transformation it went through |
| **Data mart** | A subject-specific subset of a data warehouse (e.g., finance data mart) |
| **Data mesh** | Decentralised data ownership model — domain teams own their data products |
| **Data quality** | How well data meets expectations: complete, accurate, consistent, timely, unique |
| **Data vault** | Modelling methodology using Hubs (keys), Links (relationships), Satellites (attributes) |
| **Data warehouse** | Structured SQL-queryable storage optimised for analytics queries |
| **Databricks** | Unified analytics platform built on Spark + Delta Lake; multi-cloud |
| **DBU** | Databricks Unit — the Databricks pricing unit (1 DBU = 1 virtual CPU-hour) |
| **dbt** | Data build tool — SQL-based ELT transformation framework with testing and docs |
| **Delta Lake** | Open table format adding ACID, time travel, and schema evolution to Parquet files |
| **Dimension table** | Warehouse table containing descriptive attributes (who, what, where, when) |
| **Driver** | The Spark process that creates the execution plan and coordinates workers |
| **DuckDB** | In-process analytical SQL database — no server needed, queries Parquet directly |
| **ELT** | Extract → Load → Transform: load raw data first, transform inside the warehouse |
| **EMR** | Amazon Elastic MapReduce — AWS managed Spark/Hadoop cluster service |
| **ETL** | Extract → Transform → Load: transform before loading |
| **Event** | A single timestamped record in a streaming system |
| **Executor** | A Spark worker process that runs tasks and holds partition data in memory |
| **Exactly-once** | Processing guarantee: each event processed exactly once, no duplicates, no losses |
| **Fact table** | Warehouse table containing measurable events with foreign keys to dimensions |
| **Fivetran** | Managed SaaS data connector service (200+ pre-built source connectors) |
| **Flink** | Apache Flink — distributed stateful stream processing framework |
| **Glue** | AWS managed ETL service + Data Catalog (Hive-compatible metadata store) |
| **Gold layer** | Business-ready aggregated data in a Medallion architecture |
| **Grain** | What one row represents in a table — the most fundamental modelling question |
| **Great Expectations** | Python library for data quality validation with expectation suites |
| **HDInsight** | Azure managed Hadoop/Spark cluster service |
| **Hub** | Data Vault component: table of unique business keys |
| **Iceberg** | Apache Iceberg — open table format for large-scale tables, engine-agnostic |
| **Idempotency** | Running a pipeline twice produces the same result as running it once |
| **Incremental load** | Process and load only new/changed data since the last pipeline run |
| **Integration Runtime** | Azure Data Factory compute executing activities; Azure, self-hosted, or SSIS |
| **IOManager** | Dagster component that handles asset materialisation and serialisation |
| **Job cluster** | Databricks cluster that starts for a specific job and terminates when done |
| **Kafka** | Apache Kafka — distributed event streaming platform |
| **Kinesis** | AWS managed event streaming service (equivalent to Kafka) |
| **Lake Formation** | AWS service for securing, governing, and managing data lakes |
| **Lag** | In Kafka: how many messages a consumer is behind the latest produced message |
| **Lakehouse** | Architecture combining data lake storage with warehouse query capabilities |
| **Lazy evaluation** | Spark defers computation until an action is called — builds a plan first |
| **Lineage** | Tracking data from source through all transformations to final destination |
| **Link** | Data Vault component: records relationships between Hub entities |
| **Materialisation** | dbt concept: how a model is stored — view, table, incremental, or ephemeral |
| **Medallion architecture** | Bronze/Silver/Gold layered data lake structure (Databricks pattern) |
| **Metadata DB** | Airflow's PostgreSQL/MySQL database storing run history, task states |
| **Micro-batch** | Small batches processed every few minutes — bridge between batch and streaming |
| **Micro-partition** | Snowflake's automatic columnar data organisation unit (~50–500MB) |
| **MLflow** | Open-source ML lifecycle management: tracking, models, registry (Databricks) |
| **Offset** | A message's sequential position within a Kafka partition |
| **OLAP** | Online Analytical Processing — analytical queries; few large reads |
| **OLTP** | Online Transaction Processing — operational systems; many small reads/writes |
| **Operator** | Airflow task type (PythonOperator, BashOperator, BigQueryOperator, etc.) |
| **Orchestration** | Scheduling and coordinating when and how pipeline steps run |
| **ORC** | Optimised Row Columnar — columnar format for Hive/Hadoop ecosystems |
| **Parquet** | Binary columnar format — standard for analytics and data lakes |
| **Partition** | Data divided by column value into separate files/directories; also Kafka segment |
| **Pipeline** | Automated sequence of data processing steps: extract → transform → load |
| **Polars** | Rust-based high-performance DataFrame library (pandas alternative) |
| **Prefect** | Python-native workflow orchestration with @flow and @task decorators |
| **Producer** | A client that writes events to a Kafka topic |
| **Pub/Sub** | Google Cloud managed messaging service (equivalent to Kafka) |
| **Purview** | Azure enterprise data governance and catalog service |
| **PySpark** | Python API for Apache Spark |
| **RDD** | Resilient Distributed Dataset — Spark's low-level unstructured data abstraction |
| **Redshift** | Amazon's managed columnar data warehouse |
| **ref()** | dbt function to reference another model — declares dependencies |
| **Replication factor** | Number of Kafka partition copies across brokers for durability |
| **Retention** | How long Kafka keeps events before deleting them |
| **S3** | Amazon Simple Storage Service — the foundational AWS data lake storage |
| **Satellite** | Data Vault component: descriptive attributes with history |
| **SCD** | Slowly Changing Dimension — how to handle dimension attribute changes over time |
| **Schema** | The structure of data: column names, types, constraints |
| **Schema evolution** | Changes to a dataset's schema over time (adding columns, type changes) |
| **Schema Registry** | Central store for Avro/JSON schema versions (e.g., Confluent Schema Registry) |
| **Sensor** | Airflow task that waits for an external condition before proceeding |
| **Silver layer** | Cleaned and validated data in a Medallion architecture |
| **SLA** | Service Level Agreement — data freshness/availability guarantee |
| **Slot** | BigQuery compute unit (1 slot = 1 virtual CPU) |
| **Snowflake** | Cloud data warehouse with separated storage and virtual warehouse compute |
| **Snowpipe** | Snowflake automated continuous loading from cloud storage |
| **Spark** | Apache Spark — distributed data processing framework |
| **Star schema** | Dimensional model with one central fact table surrounded by dimension tables |
| **Step Functions** | AWS serverless workflow orchestration (state machine-based) |
| **Streaming** | Processing data events continuously as they arrive |
| **Surrogate key** | Artificial integer primary key generated by the warehouse |
| **Synapse** | Azure Synapse Analytics — Microsoft's unified analytics platform |
| **Task** | A single unit of work in an orchestration DAG |
| **Time Travel** | Querying data as it existed at a previous point in time (Delta, Iceberg, Snowflake) |
| **Topic** | A named Kafka channel for a category of events |
| **Transformation** | Cleaning, joining, aggregating, and applying business logic to data |
| **Upsert** | Insert new rows and update existing rows matching a key — insert-or-update |
| **Unity Catalog** | Databricks unified data governance layer across workspaces and clouds |
| **Vertex AI** | Google Cloud's unified ML platform |
| **Virtual Warehouse** | Snowflake compute cluster — independently sized and billed from storage |
| **Watermark** | Maximum allowed event lateness in streaming; also pipeline state cursor |
| **XCom** | Airflow cross-communication mechanism — passes small values between tasks |
| **ZORDER** | Databricks Delta Lake multi-dimensional data skipping index |
| **Zero-Copy Clone** | Snowflake instant table clone sharing storage with original (no data copy) |

---

*End of Data Engineering 101*
*Chat #3 · June 2026 (v2 — Expanded Edition)*
