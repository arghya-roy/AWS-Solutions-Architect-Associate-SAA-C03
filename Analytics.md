# AWS Exam Study Guide: Analytics Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Athena · Data Exchange · Data Pipeline · EMR · Glue · Kinesis · Lake Formation · MSK · OpenSearch · QuickSight · Redshift

---

## 📋 Table of Contents

1. [Analytics Quick-Reference Matrix](#quick-reference)
2. [Analytics Architecture Overview](#overview)
3. [Amazon Athena](#athena)
4. [AWS Glue](#glue)
5. [Amazon EMR](#emr)
6. [Amazon Kinesis](#kinesis)
7. [Amazon MSK (Managed Streaming for Apache Kafka)](#msk)
8. [AWS Lake Formation](#lake-formation)
9. [Amazon OpenSearch Service](#opensearch)
10. [Amazon QuickSight](#quicksight)
11. [Amazon Redshift](#redshift)
12. [AWS Data Pipeline](#data-pipeline)
13. [AWS Data Exchange](#data-exchange)
14. [Master Comparisons](#comparisons)
15. [Architecture Patterns](#patterns)
16. [Exam Tips Summary](#exam-tips)

---

<a name="quick-reference"></a>
## Analytics Quick-Reference Matrix

| Service | Category | What It Does | Exam Trigger Words |
|---|---|---|---|
| **Athena** | Query | SQL on S3 data; serverless | "query S3", "no ETL", "pay per scan", "Presto SQL" |
| **Glue** | ETL + Catalog | Serverless ETL + metadata catalog | "ETL", "data catalog", "crawl schema", "PySpark ETL" |
| **EMR** | Big Data Processing | Managed Hadoop/Spark/Hive clusters | "Hadoop", "Spark", "HBase", "big data cluster", "HPC" |
| **Kinesis Data Streams** | Streaming | Real-time stream ingestion + processing | "real-time", "milliseconds", "shards", "custom consumers" |
| **Kinesis Firehose** | Streaming | Load streaming data to S3/Redshift/OpenSearch | "deliver to S3", "near-real-time", "no consumer code" |
| **Kinesis Data Analytics** | Streaming | SQL/Flink on streaming data | "real-time SQL", "Apache Flink", "stream analytics" |
| **Kinesis Video Streams** | Video Streaming | Ingest/store/process live video | "IoT video", "live camera", "Rekognition Video" |
| **MSK** | Streaming | Managed Apache Kafka | "Kafka", "migrate Kafka", "SASL/TLS", "Kafka Connect" |
| **Lake Formation** | Governance | S3 data lake + fine-grained access control | "data lake", "row/column security", "LF-Tags" |
| **OpenSearch** | Search + Analytics | Search, log analytics, dashboards | "full-text search", "Elasticsearch", "log analysis", "Kibana" |
| **QuickSight** | BI/Visualization | BI dashboards + ML insights | "dashboard", "BI tool", "SPICE", "embedded analytics" |
| **Redshift** | Data Warehouse | Columnar OLAP at petabyte scale | "data warehouse", "OLAP", "petabyte", "BI analytics" |
| **Data Pipeline** | Orchestration | Move/transform data between AWS services | "schedule data movement", "legacy ETL", "on-prem to AWS" |
| **Data Exchange** | Data Marketplace | Subscribe to third-party datasets | "third-party data", "data subscription", "data marketplace" |

---

<a name="overview"></a>
## Analytics Architecture Overview

### The Modern Data Analytics Stack

```
INGESTION LAYER:
  Real-time:  Kinesis Data Streams / Kinesis Firehose / MSK (Kafka)
  Batch:      S3 Direct Upload / Data Pipeline / AppFlow / DMS
  External:   AWS Data Exchange (third-party datasets)
        ↓
STORAGE LAYER:
  Raw (Data Lake):    Amazon S3
  Structured:         Amazon Redshift
  Search:             Amazon OpenSearch
  Governance:         AWS Lake Formation
        ↓
PROCESSING LAYER:
  Serverless ETL:    AWS Glue
  Big Data:          Amazon EMR (Spark/Hadoop)
  Streaming:         Kinesis Data Analytics (Flink)
  Interactive SQL:   Amazon Athena
        ↓
SERVING / VISUALIZATION LAYER:
  BI Dashboards:     Amazon QuickSight
  Search UI:         OpenSearch Dashboards
  APIs:              API Gateway + Lambda + Athena
```

---

<a name="athena"></a>
## 1. Amazon Athena

### What Problem Does It Solve?
Data in S3 is cheap to store but traditionally required loading into a database before querying — a time-consuming process. Athena is a **serverless, interactive query service** that runs standard SQL directly on data in S3 — no loading, no servers, no cluster management. You only pay for the data scanned.

**Real-World Example:**
> A company stores 10 TB of CloudTrail logs in S3. Instead of loading them into a database, a security engineer runs:
> ```sql
> SELECT eventName, userIdentity.arn, COUNT(*) as count
> FROM cloudtrail_logs
> WHERE eventTime > '2026-01-01'
>   AND errorCode IS NOT NULL
> GROUP BY eventName, userIdentity.arn
> ORDER BY count DESC
> LIMIT 20;
> ```
> Results in 30 seconds. Cost: 10 TB × $5/TB = $50 for the scan.

---

### Athena Technical Details

| Property | Details |
|---|---|
| **Engine** | Presto (now Trino) + Apache Spark (Athena Spark) |
| **Pricing** | $5 per TB scanned (no charge for DDL, failed queries) |
| **Formats** | CSV, JSON, Parquet, ORC, Avro, TSV, JSON, Ion |
| **Compression** | GZIP, Snappy, ZSTD, LZO |
| **Data location** | Amazon S3 only |
| **Schema** | AWS Glue Data Catalog (or Athena's own) |
| **Concurrency** | Default 25 concurrent DDL + 25 DML queries per account |

---

### Athena Performance and Cost Optimization

**Use columnar formats:**
- **Parquet** and **ORC** are columnar (Athena reads only needed columns).
- CSV/JSON: full file scan. Parquet/ORC: reads 10–100x less data.
- **Cost reduction: up to 87%** when switching from CSV to Parquet.

**Use partitioning:**
```sql
-- Partitioned by year/month/day → Athena only scans relevant partitions
CREATE EXTERNAL TABLE cloudtrail_logs (
  eventName STRING, eventTime STRING
)
PARTITIONED BY (year STRING, month STRING, day STRING)
STORED AS PARQUET
LOCATION 's3://my-bucket/logs/';

-- Query scans only Jan 2026 partition
SELECT * FROM cloudtrail_logs WHERE year='2026' AND month='01';
```

**Partition Projection:**
- Automatically compute partitions from values (no need to `MSCK REPAIR TABLE`).
- Configured in table properties; eliminates partition metadata overhead.

**Athena Result Caching:**
- Reuse results of identical queries within 7 days (no extra charge for cached queries).

---

### Athena Federated Query

- Query data across **multiple sources** in one SQL statement:
  - S3, RDS, DynamoDB, Redshift, DocumentDB, ElastiCache, on-prem JDBC sources.
- Uses **Lambda Data Source Connectors** to connect to non-S3 sources.
- Results combined and returned via Athena.

```sql
-- Query DynamoDB + S3 in one join
SELECT s.product_name, d.inventory_count
FROM s3_catalog.products s
JOIN dynamodb_catalog.inventory d ON s.product_id = d.product_id
WHERE s.category = 'Electronics';
```

---

### Athena for Apache Spark

- Run interactive Spark workloads through Athena (no cluster management).
- Jupyter-compatible notebooks.
- Supports: Python, Scala.
- Auto-scales compute resources.
- Use case: Data exploration and ML feature engineering on S3 data.

---

### Athena Workgroups

- Isolate queries by team/project; set per-workgroup cost limits.
- Control: data access, query result location, encryption, cost per query.
- **Cost Control**: set query scan limit (e.g., max $5 per query per user).

---

### Athena vs Redshift vs EMR

| Feature | Athena | Redshift | EMR |
|---|---|---|---|
| **Setup** | Zero (serverless) | Cluster provisioning | Cluster provisioning |
| **Data location** | S3 only | Loaded into Redshift + S3 (Spectrum) | S3 / HDFS |
| **Pricing** | Per TB scanned | Per node-hour (or serverless) | Per EC2 instance-hour |
| **Complexity** | Minimal | Moderate | High (Spark/Hadoop expertise) |
| **Ideal query** | Ad-hoc SQL on S3 | Complex BI, multi-join analytics | Batch ML, complex transformations |
| **Performance** | Good (columnar) | Excellent (MPP, columnar) | Excellent (distributed Spark) |

> **Exam Rule:** "Query data already in S3 without loading it" → **Athena**. "Petabyte BI warehouse with complex joins" → **Redshift**. "Spark/Hadoop big data processing" → **EMR**.

---

### Key Athena Exam Scenarios

| Scenario | Answer |
|---|---|
| Query 10TB of S3 logs with SQL instantly | Amazon Athena |
| Reduce Athena query cost by 87% | Convert data to Parquet + partition by date |
| Query across S3 + DynamoDB in one SQL | Athena Federated Query |
| Analyze CloudTrail/VPC Flow Logs/ALB access logs | Athena (native AWS log integration) |
| Analyze CUR (Cost and Usage Report) billing data | Athena + Glue Catalog |
| Serverless Spark on S3 data | Athena for Apache Spark |

---

<a name="glue"></a>
## 2. AWS Glue

### What Problem Does It Solve?
Data comes from many sources in different formats, and needs to be cleaned, transformed, and loaded into analytical stores. Building ETL pipelines requires managing Spark clusters, writing transformation code, scheduling jobs, and maintaining schema metadata. Glue is a **fully serverless ETL service** with a metadata catalog that eliminates all infrastructure management.

**Real-World Example:**
> A retailer receives daily CSV transaction files from 50 stores in S3. Glue: (1) Crawler scans the S3 bucket → automatically creates table schemas in the Data Catalog. (2) ETL job reads CSV, cleans data (remove nulls, standardize dates), joins with product catalog, converts to Parquet. (3) Triggers on schedule to load into Redshift. All without managing any Spark cluster.

---

### AWS Glue Core Components

#### Glue Data Catalog
- **Central metadata repository** for all data assets.
- Stores: database names, table names, schemas, data locations, partitions.
- Integrated with: Athena, Redshift Spectrum, EMR, Lake Formation, Spark.
- One Data Catalog per AWS account per region.
- Replaces Apache Hive Metastore for AWS-native workloads.

#### Glue Crawlers
- Automatically scan data sources → **infer schema → update Data Catalog**.
- Supported sources: S3, JDBC (RDS, Redshift, DynamoDB), DeltaLake, Iceberg.
- Run on a schedule or on-demand.
- Creates/updates table definitions without writing DDL.

```
Glue Crawler runs on S3 bucket:
  Reads sample files → infers schema (field names, data types)
  Creates table in Glue Data Catalog:
    Table: "sales_data"
    Schema: { date: string, product_id: string, amount: double, quantity: int }
    Location: s3://my-bucket/sales/
    Format: Parquet
    Partitions: year=2026/month=01/day=15
```

#### Glue ETL Jobs
- Run **PySpark or Scala Spark** code in a managed serverless environment.
- Or use **Glue Visual ETL** (no-code drag-and-drop interface).
- Automatically generates transformation code from visual designer.
- **DynamicFrame**: Glue's enhanced version of Spark DataFrame with schema flexibility.

#### Glue Job Types

| Type | Description | Use Case |
|---|---|---|
| **Spark** | Standard PySpark/Scala; serverless Spark cluster | Large-scale ETL |
| **Spark Streaming** | Continuous micro-batch processing from Kinesis/Kafka | Near-real-time ETL |
| **Python Shell** | Python scripts (no Spark); up to 16 DPU | Lightweight data tasks |
| **Ray** | Distributed Python with Ray framework | ML workloads |

#### Glue DPU (Data Processing Units)
- 1 DPU = 4 vCPUs + 16 GB RAM.
- Standard job: minimum 2 DPUs; can scale to 100 DPUs.
- Pricing: per DPU-hour.
- **Auto Scaling**: Glue automatically adjusts DPUs based on workload.

---

### Glue Workflow and Triggers

- **Workflow**: chain multiple Glue jobs and crawlers into a pipeline.
- **Triggers**:
  - Scheduled (cron)
  - Job completion (cascade jobs)
  - On-demand (manual)
  - EventBridge (event-driven)

---

### Glue DataBrew

- **Visual data preparation** tool for analysts (no code required).
- 350+ built-in transformations.
- Profile data (detect PII, data quality, statistics).
- Use case: clean and normalize data for analytics without writing Spark code.

---

### Glue Elastic Views

- Create **materialized views** across multiple data stores (S3, DynamoDB, Aurora).
- Views updated automatically as source data changes.
- Query using SQL without moving data.

---

### Glue vs EMR vs Athena for ETL

| Feature | Glue | EMR | Athena |
|---|---|---|---|
| **Type** | Serverless ETL | Managed Spark/Hadoop cluster | Serverless SQL query |
| **Code** | PySpark / visual | Any Spark/Hadoop tool | SQL only |
| **Setup** | Zero | Cluster config | Zero |
| **Use case** | ETL pipelines, Data Catalog | Complex big data processing | Ad-hoc queries on S3 |
| **Cost** | Per DPU-hour | Per EC2-hour | Per TB scanned |
| **Streaming** | ✅ (Glue Streaming) | ✅ (Spark Streaming) | ❌ |

---

### Key Glue Exam Scenarios

| Scenario | Answer |
|---|---|
| Automatically discover schema of S3 data | Glue Crawler |
| Convert CSV files in S3 to Parquet for Athena | Glue ETL job |
| Central schema repository for Athena + EMR + Redshift | Glue Data Catalog |
| ETL without writing Spark code | Glue Visual ETL (drag-and-drop) |
| Clean and prepare data for non-developers | AWS Glue DataBrew |
| Schedule ETL job to run after crawler completes | Glue Workflow with trigger |
| Near-real-time ETL from Kinesis to S3 | Glue Streaming job |

---

<a name="emr"></a>
## 3. Amazon EMR (Elastic MapReduce)

### What Problem Does It Solve?
Processing petabytes of data with frameworks like Apache Spark, Hadoop, Hive, HBase, and Presto requires managed distributed compute clusters. Setting up, configuring, and managing these clusters manually on EC2 is complex. EMR provides **managed big data clusters** with automatic scaling and support for all major open-source big data frameworks.

**Real-World Example:**
> A genomics company processes 1 PB of DNA sequencing data daily. They submit a Spark job to EMR: EMR launches 200 `r5.4xlarge` Spot instances, runs the analysis in parallel, writes results to S3, then terminates the cluster — paying only for the 4-hour processing window at Spot prices, saving 90% vs On-Demand 24/7 infrastructure.

---

### EMR Supported Frameworks

| Framework | Use Case |
|---|---|
| **Apache Spark** | In-memory distributed processing; ML; fast batch |
| **Hadoop MapReduce** | Original batch processing framework |
| **Apache Hive** | SQL-like queries on HDFS/S3 |
| **Apache HBase** | NoSQL database on HDFS (random access) |
| **Apache Flink** | Real-time stream processing |
| **Presto/Trino** | Interactive SQL (Athena uses this) |
| **Apache Kafka** | (Use MSK instead) |
| **TensorFlow/MXNet** | Distributed ML training |

---

### EMR Cluster Architecture

```
EMR Cluster:
  ├── Primary Node (Master): coordinates cluster, tracks job status, hosts HDFS NameNode
  ├── Core Nodes: run tasks AND store HDFS data; always running
  └── Task Nodes: run tasks ONLY (no HDFS); can use Spot Instances safely
```

**Multi-Master:** 3 primary nodes for high availability (active/standby).

---

### EMR Storage Options

| Option | Description | Use Case |
|---|---|---|
| **HDFS** | Distributed storage on core node disks | Intermediate data, high I/O |
| **EMRFS (S3)** | S3 as persistent EMR file system | Input/output data; persistent after cluster termination |
| **Local file system** | Each node's local disk | Temporary scratch space |
| **EBS** | Additional block storage per node | When more HDFS capacity needed |

> **Exam Tip:** Use **S3 (EMRFS) as primary storage** so data persists after cluster terminates. This enables **transient clusters** — terminate the cluster when done; launch a new one for the next job.

---

### EMR Deployment Options

| Option | Description | Use Case |
|---|---|---|
| **EMR on EC2** | Traditional EMR on EC2 instances | Most control; full framework support |
| **EMR on EKS** | Run EMR Spark jobs on existing EKS cluster | Share Kubernetes infrastructure |
| **EMR Serverless** | No cluster management; AWS scales automatically | Simplest; no instance selection needed |
| **EMR on Outposts** | EMR on AWS Outposts hardware | On-premises big data processing |

---

### EMR Instance Groups vs Instance Fleets

| Type | Description |
|---|---|
| **Instance Groups** | Uniform instance type per group; simple scaling |
| **Instance Fleets** | Mix of instance types + Spot + On-Demand; optimal for cost |

---

### EMR Auto Scaling

- **Managed Scaling**: AWS automatically scales core/task nodes based on YARN memory utilization.
- **Custom Auto Scaling**: CloudWatch alarms trigger scale-out/in rules.
- Task nodes (no HDFS) are ideal for Spot — interruption doesn't lose data.

---

### EMR Security

| Feature | Details |
|---|---|
| **Encryption at rest** | HDFS (Hadoop transparent encryption), S3 (SSE-S3/SSE-KMS) |
| **Encryption in transit** | TLS between nodes |
| **Kerberos** | Authentication for Hadoop ecosystem |
| **IAM roles** | EC2 instance profile for S3/other AWS access |
| **Lake Formation** | Fine-grained table/column access control |
| **VPC** | Launch cluster in private subnet |

---

### Key EMR Exam Scenarios

| Scenario | Answer |
|---|---|
| Run Apache Spark jobs on managed cluster | Amazon EMR |
| Process Hive queries on petabyte S3 dataset | Amazon EMR (Hive on EMRFS) |
| HBase NoSQL database for random-access genomics | Amazon EMR (HBase) |
| Reduce EMR cost by 90% for batch jobs | Spot Instances for task nodes + EMRFS |
| No cluster management for Spark jobs | EMR Serverless |
| Share existing EKS infrastructure for Spark | EMR on EKS |
| Run Spark jobs on-premises | EMR on Outposts |

---

<a name="kinesis"></a>
## 4. Amazon Kinesis

### What Problem Does It Solve?
Real-time data (clickstreams, IoT sensor readings, financial transactions, application logs) must be ingested and processed in milliseconds — before being stored. Kinesis is a family of services for **real-time data streaming**: ingesting, processing, analyzing, and delivering streaming data.

---

### Kinesis Family Overview

| Service | Purpose | Consumer | Latency |
|---|---|---|---|
| **Kinesis Data Streams (KDS)** | Ingest + custom processing | Custom consumers (Lambda, KCL, Flink) | Milliseconds |
| **Kinesis Data Firehose** | Load streams to destinations | No consumer code needed | 60 sec – 15 min |
| **Kinesis Data Analytics** | SQL/Flink on streams | Serverless stream processing | Milliseconds |
| **Kinesis Video Streams** | Ingest/store live video | Rekognition Video, custom | Seconds |

---

### Kinesis Data Streams (KDS)

#### Architecture
```
Producers → Shards → Consumer Applications
(IoT, app logs)  (ordered per shard)  (Lambda, KCL, Flink)
```

#### Shards
- **Shard** = unit of capacity; 1 shard = 1 MB/sec write + 2 MB/sec read.
- Shard count determines total throughput.
- **Shard splitting**: increase capacity (1 shard → 2 shards).
- **Shard merging**: reduce capacity (2 shards → 1 shard).
- Data within a shard is **ordered** by sequence number.

#### Capacity Modes
| Mode | Description | Use Case |
|---|---|---|
| **Provisioned** | You specify shard count; predictable cost | Known, steady throughput |
| **On-Demand** | Auto-scales shards; pay per GB | Variable, unknown throughput |

#### Record Retention
- Default: **24 hours**; maximum: **365 days** (extended retention).
- Records available for replay/reprocessing within the retention window.

#### Consumers
| Consumer Type | Description | Throughput |
|---|---|---|
| **Shared (pull)** | All consumers share 2 MB/sec per shard | 2 MB/sec shared |
| **Enhanced Fan-Out** | Each consumer gets own 2 MB/sec per shard | 2 MB/sec per consumer |

> **Exam Tip:** Enhanced Fan-Out = each registered consumer gets **dedicated** 2 MB/sec per shard throughput. Use when multiple consumers read the same stream simultaneously.

#### Partition Key
- Determines which shard a record goes to (hash of partition key).
- Same partition key → same shard → ordered records.
- Hot shard problem: if one key has disproportionate traffic → add randomness to partition key.

---

### Kinesis Data Firehose

**Kinesis Firehose = streaming delivery service** (not a storage service).

```
Data Sources → Firehose → Transformation (Lambda) → Destination
```

#### Supported Sources
- Kinesis Data Streams, Direct PUT (SDK), MSK, CloudWatch Logs, WAF, EventBridge, IoT.

#### Supported Destinations
| Destination | Notes |
|---|---|
| **Amazon S3** | Primary destination; buffer by size (up to 128 MB) or time (up to 15 min) |
| **Amazon Redshift** | COPY via S3 intermediate |
| **Amazon OpenSearch** | Direct index |
| **Splunk** | Third-party SIEM |
| **Datadog / Dynatrace / NewRelic** | Observability platforms |
| **HTTP Endpoint** | Any custom destination |
| **Snowflake** | Cloud data warehouse |

#### Key Properties
- **Fully managed** — no shards, no consumer code.
- **Automatic scaling** — handles any throughput.
- **Buffer**: accumulates data before delivery (size or time trigger).
- **Compression**: GZIP, Snappy, ZSTD before delivery to S3.
- **Data transformation**: invoke Lambda to transform records inline.
- **Backup**: send raw (pre-transformation) data to S3 simultaneously.

> **Exam Tip:** Firehose has **NO** real-time delivery — minimum 60-second buffer. For real-time processing → KDS. For reliable near-real-time delivery to S3/Redshift/OpenSearch → Firehose.

---

### Kinesis Data Analytics

| Option | Description | Language |
|---|---|---|
| **Apache Flink** | Full Apache Flink on managed infrastructure | Java, Python, Scala, SQL |
| **SQL Applications** | Real-time SQL on Kinesis streams (legacy, being deprecated) | SQL |

**Kinesis Data Analytics for Apache Flink:**
- Runs Apache Flink jobs without managing Flink clusters.
- Auto-scales Flink application parallelism.
- Durably stores Flink state and checkpoints in S3.
- Sources: KDS, MSK; Sinks: S3, KDS, MSK, Lambda.

**Common Flink Use Cases:**
- Real-time fraud detection on transaction streams.
- Anomaly detection on IoT sensor data.
- Real-time leaderboard computation.
- Stream enrichment (join stream with reference data from S3).

---

### KDS vs Firehose vs MSK

| Feature | Kinesis Data Streams | Kinesis Firehose | Amazon MSK |
|---|---|---|---|
| **Latency** | Milliseconds | 60 sec–15 min | Milliseconds |
| **Consumer code** | Required | Not needed | Required (Kafka consumer) |
| **Replay** | ✅ (up to 365 days) | ❌ | ✅ (configurable retention) |
| **Protocol** | AWS SDK | AWS SDK / HTTP | Kafka API |
| **Migration from Kafka** | Requires rewrite | Requires rewrite | ✅ Drop-in |
| **Destinations** | Custom (Lambda, KCL) | S3, Redshift, OpenSearch | Custom consumers |

---

### Key Kinesis Exam Scenarios

| Scenario | Answer |
|---|---|
| Real-time fraud detection on transaction stream | KDS + Lambda/Flink |
| Deliver streaming logs to S3 with 5-min buffer | Kinesis Firehose |
| Deliver logs to OpenSearch in near-real-time | Kinesis Firehose → OpenSearch |
| Multiple consumers read same stream simultaneously | KDS Enhanced Fan-Out |
| Real-time Flink stream processing | Kinesis Data Analytics (Flink) |
| Replay stream data from 3 days ago | KDS with 72-hour retention |
| Ingest IoT sensor data at scale | KDS (custom processing) or Firehose (S3 delivery) |
| Hot shard due to single partition key | Randomize partition key (add UUID suffix) |

---

<a name="msk"></a>
## 5. Amazon MSK (Managed Streaming for Apache Kafka)

### What Problem Does It Solve?
Apache Kafka is the industry standard for high-throughput event streaming — but running Kafka yourself requires managing ZooKeeper (or KRaft), broker nodes, replication, scaling, patching, and monitoring. MSK provides **fully managed Apache Kafka** — keeping Kafka open and compatible while AWS manages all infrastructure.

**Real-World Example:**
> A financial services company has built their entire event streaming architecture on Kafka. They're running self-managed Kafka on EC2. Moving to MSK: zero code changes (Kafka clients work unchanged), AWS manages broker patching and availability, and they gain CloudWatch integration for metrics.

---

### MSK Key Features

| Feature | Details |
|---|---|
| **Kafka compatibility** | Full Apache Kafka API; existing Kafka clients work unchanged |
| **Broker management** | AWS manages: patching, broker replacement, replication |
| **Storage** | Provisioned (EBS) or Tiered Storage (S3 for long-term retention) |
| **Security** | TLS encryption, SASL/SCRAM or IAM for auth, VPC isolation |
| **Scaling** | Increase broker storage (online); add broker nodes (with rebalancing) |
| **Multi-AZ** | Brokers distributed across 3 AZs |
| **MSK Connect** | Run Kafka Connect connectors (source/sink) on managed infrastructure |
| **MSK Serverless** | No broker management; auto-scales; pay per throughput |

---

### MSK Cluster Types

| Type | Description | Use Case |
|---|---|---|
| **Provisioned** | You choose broker instance type + EBS storage | Predictable throughput, cost control |
| **Serverless** | Auto-scales; no broker provisioning | Variable workloads, unknown scale |

---

### MSK vs Kinesis Data Streams

| Feature | MSK | Kinesis Data Streams |
|---|---|---|
| **Protocol** | Apache Kafka API | AWS Kinesis API |
| **Migration** | ✅ Drop-in for existing Kafka apps | Requires code rewrite |
| **Consumer groups** | ✅ Kafka consumer groups | ✅ (different concept) |
| **Retention** | Configurable (hours to years) | Up to 365 days |
| **Throughput unit** | Broker (partitions per broker) | Shards (1 MB/sec each) |
| **Ecosystem** | Full Kafka Connect/Streams ecosystem | AWS-native ecosystem |
| **Open source** | ✅ (fully Kafka compatible) | ❌ (AWS proprietary) |

> **Exam Rule:** "Migrating from Apache Kafka" or "need Kafka ecosystem (Connect, Streams)" → **MSK**. "New AWS-native streaming application" → **Kinesis Data Streams**.

---

### MSK Connect

- Run **Kafka Connect** connectors as a managed service (no Connect cluster management).
- Sources: databases (CDC via Debezium), S3, and more.
- Sinks: S3, OpenSearch, Redshift, and more.
- Use case: CDC (Change Data Capture) from RDS → MSK topic → OpenSearch.

---

### Key MSK Exam Scenarios

| Scenario | Answer |
|---|---|
| Migrate on-premises Kafka cluster to AWS | Amazon MSK |
| Kafka consumers must work without code changes | Amazon MSK |
| Run Kafka Connect on managed infrastructure | MSK Connect |
| Serverless Kafka — no broker management | MSK Serverless |
| Large-scale event streaming for microservices | MSK |

---

<a name="lake-formation"></a>
## 6. AWS Lake Formation

### What Problem Does It Solve?
Building a secure data lake on S3 is complex — setting up bucket policies, Glue crawlers, Athena access, and managing fine-grained permissions (e.g., "analyst Alice can query the Sales table but only rows where Region='US' and cannot see the SSN column") requires dozens of manual steps. Lake Formation provides a **centralized service** to set up, secure, and manage a data lake with column/row/cell-level security in days instead of months.

**Real-World Example:**
> A healthcare company stores patient records in S3. With Lake Formation:
> - Medical researchers see all rows but NOT SSN or DOB columns.
> - Claims analysts see only rows where State='CA' and can see Cost data but not Diagnosis.
> - Admins see everything.
> All enforced by Lake Formation — no S3 bucket policy complexity, works across Athena, EMR, Redshift Spectrum, Glue.

---

### Lake Formation Key Capabilities

| Capability | Description |
|---|---|
| **Data lake setup** | Registers S3 locations; manages access via Service-Linked Role |
| **Glue Data Catalog** | Uses Glue catalog as the central metadata store |
| **Data permissions** | Grants/revokes access at database/table/column/row level |
| **Column-level security** | Prevent specific columns from being seen by certain users |
| **Row-level security (RLS)** | Filter rows based on user identity (Data Filters) |
| **Cell-level security** | Combine column + row filters for cell-specific control |
| **LF-Tags (TBAC)** | Tag-Based Access Control — assign tags to resources; grant permissions by tag |
| **Data sharing** | Share data lake resources across AWS accounts (cross-account) |
| **Governed Tables** | ACID transactions + automatic data compaction on S3 |
| **Blueprints** | Automated workflows to ingest data from JDBC/S3/CloudTrail |

---

### Lake Formation Permissions Model

```
Traditional S3 + Glue: multiple overlapping policies
  S3 bucket policy + IAM policy + Glue resource policy + Athena workgroup = complex

Lake Formation: single unified permission model
  Grant(Alice, SELECT, table=sales, columns=['region','amount'], row_filter="region='US'")
  → automatically enforced by Athena, EMR, Redshift Spectrum, Glue
```

---

### LF-Tags (Tag-Based Access Control)

- Assign metadata tags to databases/tables/columns.
- Grant IAM principals permissions on tags (not individual resources).
- **Scalable**: when a new table is created with existing tags, permissions auto-apply.

```
Tag: classification=sensitive
  → Columns: SSN, DOB, CreditCard

Policy: developers cannot access classification=sensitive
  → Automatically prevents access to all SSN, DOB, CreditCard columns
     across all current and future tagged resources
```

---

### Lake Formation Governed Tables

- ACID transactions on S3 data (powered by Apache Iceberg under the hood).
- Concurrent read/write without corruption.
- Automatic data compaction (small file merging for Parquet performance).
- Time travel queries (query data as of a specific point in time).

---

### Key Lake Formation Exam Scenarios

| Scenario | Answer |
|---|---|
| Fine-grained column-level security on S3 data lake | AWS Lake Formation |
| Row-level security (users only see their region's data) | Lake Formation Row-Level Security |
| Cross-account data sharing with column security | Lake Formation cross-account permissions |
| Build secure data lake in days instead of months | AWS Lake Formation |
| ACID transactions on S3 data | Lake Formation Governed Tables |
| Scale permissions as new tables are added | LF-Tags (Tag-Based Access Control) |

---

<a name="opensearch"></a>
## 7. Amazon OpenSearch Service

### What Problem Does It Solve?
Storing logs and application data in S3 or DynamoDB makes it hard to search — "find all error logs mentioning 'timeout' in the last 2 hours from service X" can't be done with SQL. OpenSearch provides a **managed search and analytics engine** optimized for full-text search, log analysis, and real-time dashboards.

**Real-World Example:**
> A company collects 100 GB of application logs daily. Logs are streamed to OpenSearch via Kinesis Firehose. Operations teams use OpenSearch Dashboards (Kibana) to: search all logs for "NullPointerException", filter by microservice + time range, view error rate charts, set up alerts when error rate exceeds 1%. Full-text search results in milliseconds.

---

### OpenSearch Key Concepts

| Concept | Description |
|---|---|
| **Index** | Collection of documents (like a table in SQL) |
| **Document** | JSON record in an index (like a row) |
| **Shard** | Partition of an index (distributed across nodes) |
| **Replica** | Copy of a shard for HA and read scaling |
| **Mapping** | Schema definition for an index (field types) |
| **Query DSL** | JSON-based query language for OpenSearch |
| **OpenSearch Dashboards** | Built-in visualization UI (fork of Kibana) |

---

### OpenSearch Deployment Options

| Option | Description | Use Case |
|---|---|---|
| **Standard (managed)** | You choose instance types + AZs | Production; full control |
| **Serverless** | Auto-scales; pay per OCU (compute) + storage | Variable workloads; simple operations |

---

### OpenSearch Features

| Feature | Details |
|---|---|
| **Multi-AZ** | Deploy across 2–3 AZs with replicas |
| **UltraWarm** | Low-cost storage tier using S3 for old indices (read-only) |
| **Cold Storage** | Even cheaper S3-based tier; detach/attach as needed |
| **OpenSearch Dashboards** | Kibana-compatible visualization |
| **Encryption** | At rest (KMS) + in transit (TLS) |
| **Fine-grained access control** | Field-level and document-level security |
| **Alerting** | Define monitors + triggers + notifications |
| **Anomaly Detection** | ML-based anomaly detection on time-series data |
| **SQL support** | Query OpenSearch using SQL syntax |
| **Integration** | Kinesis Firehose, Lambda, CloudWatch, S3, DynamoDB Streams |

---

### OpenSearch Data Ingestion Patterns

```
1. Real-time logs:
   Application → CloudWatch Logs → Subscription Filter → Lambda → OpenSearch

2. Streaming data:
   Kinesis Data Streams → Kinesis Firehose → OpenSearch

3. Database changes:
   DynamoDB Streams → Lambda → OpenSearch (sync for search)
   RDS/Aurora → MSK (Debezium CDC) → Lambda → OpenSearch

4. Batch ingestion:
   S3 → Lambda → OpenSearch (bulk API)
```

---

### OpenSearch vs Athena vs Redshift (Search vs Analytics)

| Need | Service |
|---|---|
| Full-text search on logs | OpenSearch |
| Log dashboards + real-time alerting | OpenSearch |
| Ad-hoc SQL queries on S3 data | Athena |
| Petabyte BI warehouse analytics | Redshift |
| Search within documents (fuzzy, autocomplete) | OpenSearch |

---

### Key OpenSearch Exam Scenarios

| Scenario | Answer |
|---|---|
| Full-text search across application logs | Amazon OpenSearch Service |
| Real-time log monitoring with Kibana-like dashboards | OpenSearch + OpenSearch Dashboards |
| Stream logs from CloudWatch to OpenSearch | CloudWatch Logs Subscription → Lambda → OpenSearch |
| Search product catalog with fuzzy matching | Amazon OpenSearch Service |
| Log retention tiers: hot/warm/cold | OpenSearch UltraWarm (warm) + Cold Storage |
| DynamoDB search by non-key attributes | DynamoDB Streams → Lambda → OpenSearch |

---

<a name="quicksight"></a>
## 8. Amazon QuickSight

### What Problem Does It Solve?
Business users need to understand their data through visual dashboards and reports — but traditional BI tools are expensive, require IT setup, and don't scale to thousands of users. QuickSight is AWS's **cloud-native, serverless BI service** that enables any user to create interactive dashboards from AWS data sources.

**Real-World Example:**
> A retail company's CEO wants daily dashboards showing sales by region, inventory levels, and customer acquisition trends. QuickSight connects to Redshift, S3, and RDS, builds interactive dashboards in the browser, and embeds them in the company's internal portal — 500 analysts access them simultaneously with no server management.

---

### QuickSight Data Sources

| Source | Notes |
|---|---|
| **Amazon S3** | CSV, JSON, Parquet, ORC |
| **Amazon Athena** | Query S3 via Athena |
| **Amazon Redshift** | Data warehouse |
| **Amazon RDS / Aurora** | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server |
| **Amazon OpenSearch** | Search/analytics data |
| **Amazon DynamoDB** | Via Athena Federated Query |
| **Snowflake / Databricks** | Third-party data warehouses |
| **SaaS (Salesforce, Jira)** | Direct connectors |
| **Upload file** | CSV, Excel, JSON |

---

### SPICE (Super-fast, Parallel, In-memory Calculation Engine)

- QuickSight's **in-memory data store**.
- Import data into SPICE for sub-second query performance.
- 250M rows / 500 GB per dataset (SPICE storage allocated per user).
- Data refreshed on schedule or on-demand from source.

> **Exam Tip:** SPICE = fast, in-memory cache in QuickSight. For real-time data (no caching), use **Direct Query** mode (queries go directly to the source each time).

---

### QuickSight Q (ML-Powered Natural Language Queries)

- Ask questions in plain English: *"What were our top 10 products by revenue in Q1?"*
- QuickSight Q parses the question using ML and generates the chart automatically.

---

### QuickSight ML Insights

- **Anomaly Detection**: automatically flags unusual data points.
- **Forecasting**: ML-based future trend prediction.
- **Auto-Narratives**: generates plain English descriptions of charts.

---

### QuickSight Editions

| Edition | Target | Key Feature |
|---|---|---|
| **Standard** | Individual users | Basic dashboards, SPICE |
| **Enterprise** | Organizations | Row-level security, AD integration, column-level, VPC |

---

### QuickSight Pricing

| Model | Description |
|---|---|
| **Author** | Creates/modifies dashboards (~$18/month) |
| **Reader** | Views dashboards only; pay per session (~$0.30/session, max $5/month) |
| **Reader Pro** | Reader with advanced features (~$20/month) |

> **Exam Tip:** QuickSight Reader pricing = **pay-per-session** model. Great for large numbers of occasional dashboard viewers without per-seat licensing costs.

---

### Embedded Analytics

- Embed QuickSight dashboards into your own applications.
- Use `RegisteredUser` or `AnonymousUser` embedding.
- Use case: SaaS products showing analytics to their own customers.

---

### Key QuickSight Exam Scenarios

| Scenario | Answer |
|---|---|
| Executive dashboards from Redshift data | Amazon QuickSight |
| 1,000 users occasionally view dashboards; minimize cost | QuickSight Reader (pay-per-session) |
| Natural language data questions ("top products last month") | QuickSight Q |
| Embed analytics in your SaaS application | QuickSight embedded analytics |
| Sub-second dashboard performance on large datasets | QuickSight SPICE |
| Row-level security (each user sees their region's data) | QuickSight Enterprise + row-level security |

---

<a name="redshift"></a>
## 9. Amazon Redshift

*(Full Redshift coverage is in the Database guide. Key analytics-specific points below.)*

### Redshift Analytics-Specific Features

#### Redshift Spectrum
- Query **S3 data directly** from Redshift without loading it.
- Scales query processing across thousands of nodes.
- Use case: Join recent Redshift data with historical S3 data lake in one query.

```sql
-- Join Redshift table (hot data) with S3 Parquet (cold archive data)
SELECT r.customer_id, r.recent_orders, s.historical_orders
FROM redshift_table r
JOIN spectrum_schema.historical_orders_s3 s ON r.customer_id = s.customer_id;
```

#### Redshift Data Sharing
- Share live data between Redshift clusters **without copying it**.
- Producer cluster makes data available; consumer clusters query it in real-time.
- Cross-account data sharing supported.

#### Redshift ML
- `CREATE MODEL` SQL command → trains SageMaker model automatically.
- Make predictions with SQL: `SELECT product_id, ML_PREDICT(model_name, features) FROM table`.

#### Redshift Streaming Ingestion
- Direct, low-latency ingestion from **Kinesis Data Streams** or **MSK** into Redshift.
- No Firehose intermediary needed (sub-second latency to Redshift).

---

<a name="data-pipeline"></a>
## 10. AWS Data Pipeline

### What Problem Does It Solve?
Before Glue existed, AWS Data Pipeline was the primary service for scheduling and orchestrating data movement between AWS services and on-premises data sources. It's a **managed ETL/data movement orchestration service** that schedules, runs, and monitors data workflows.

> **Exam Note:** AWS Data Pipeline is a **legacy service**. AWS recommends AWS Glue or Step Functions for new workloads. It still appears on exam questions, especially for migrations from legacy architectures.

---

### Data Pipeline Key Concepts

| Concept | Description |
|---|---|
| **Pipeline** | Workflow definition containing activities and schedules |
| **Activity** | A unit of work (CopyActivity, HiveActivity, EmrActivity, ShellCommandActivity) |
| **Data Node** | Data source/destination (S3DataNode, SqlDataNode, DynamoDBDataNode) |
| **Schedule** | When to run (one-time, hourly, daily, etc.) |
| **Precondition** | Check conditions before running (S3KeyExists, DynamoDBTableExists) |
| **Resource** | Compute resource (EC2 instance or EMR cluster) |

---

### Data Pipeline vs AWS Glue vs Step Functions

| Feature | Data Pipeline | AWS Glue | Step Functions |
|---|---|---|---|
| **Status** | Legacy | Current, recommended | Current, recommended |
| **ETL** | ✅ (limited) | ✅ (full Spark ETL) | ❌ (orchestration only) |
| **Scheduling** | ✅ | ✅ | ✅ (via EventBridge) |
| **On-premises** | ✅ | Limited | ❌ |
| **Serverless** | Partial | ✅ | ✅ |
| **Use case** | Legacy migration | New ETL workloads | Complex multi-step workflows |

---

### Key Data Pipeline Exam Scenarios

| Scenario | Answer |
|---|---|
| Legacy on-premises to S3 ETL job migration | Modernize to AWS Glue |
| Existing Data Pipeline; what to migrate to | AWS Glue (for ETL) or Step Functions (for orchestration) |
| Schedule periodic EMR cluster for Hive queries | Data Pipeline EmrActivity (legacy) → use Glue or Step Functions |

---

<a name="data-exchange"></a>
## 11. AWS Data Exchange

### What Problem Does It Solve?
Many analytics use cases require **third-party data** — weather data for logistics modeling, financial market data for trading, demographic data for marketing. Finding, licensing, accessing, and keeping this data updated requires complex negotiations, custom API integrations, and data refresh pipelines. Data Exchange is a **data marketplace** where you subscribe to third-party datasets and have them delivered directly to your S3 bucket.

**Real-World Example:**
> A retail forecasting model needs weather data to predict sales. Instead of signing a contract with a weather data provider and building an API integration, they subscribe to a weather data provider on Data Exchange. The data is automatically delivered to their S3 bucket daily — directly queryable by Athena and loadable into Redshift.

---

### Data Exchange Key Features

| Feature | Details |
|---|---|
| **Marketplace** | Browse 3,500+ datasets from 300+ providers |
| **Data types** | Financial data, geospatial, weather, healthcare, demographics, IoT, alternative data |
| **Delivery** | Automatically delivered to S3 |
| **Format** | Varies by provider: CSV, Parquet, JSON, APIs, Amazon Redshift Tables |
| **Pricing** | Subscription-based (free or paid per dataset) |
| **Updates** | Provider pushes new data; you receive automatically |

---

### Data Exchange for APIs

- Some providers offer **API-based** data exchange (not just file delivery).
- Providers publish OpenAPI specifications; subscribers call via Data Exchange.

---

### Key Data Exchange Exam Scenarios

| Scenario | Answer |
|---|---|
| Add third-party weather data to your ML training dataset | AWS Data Exchange |
| Subscribe to financial market data feed delivered to S3 | AWS Data Exchange |
| Monetize proprietary dataset by publishing to others | AWS Data Exchange (as a provider) |
| Healthcare data provider delivers claims data to analytics platform | AWS Data Exchange |

---

<a name="comparisons"></a>
## Master Comparisons

### Streaming Services Comparison

| Feature | KDS | Firehose | MSK | Data Pipeline |
|---|---|---|---|---|
| **Latency** | ms | 60s–15min | ms | Minutes–hours |
| **Consumer code** | Required | None | Required | Configured |
| **Replay** | ✅ (365 days) | ❌ | ✅ (configurable) | ❌ |
| **Kafka compatible** | ❌ | ❌ | ✅ | ❌ |
| **Destinations** | Custom | S3/Redshift/OpenSearch | Custom | AWS services |
| **Best for** | Real-time custom | Near-RT delivery | Kafka migration | Legacy batch |

### ETL and Query Comparison

| Need | Service |
|---|---|
| Ad-hoc SQL on S3, no setup | Athena |
| ETL pipeline, serverless Spark | Glue |
| Complex big data processing, full Spark/Hadoop | EMR |
| Data lake governance + fine-grained security | Lake Formation |
| BI dashboards from AWS data | QuickSight |
| Petabyte OLAP data warehouse | Redshift |
| Full-text search + log analytics | OpenSearch |
| Migrate from Kafka | MSK |

### Analytics Stack Decision Tree

```
Where is the data?
  ├─→ S3 (batch) → Query with Athena → Visualize with QuickSight
  ├─→ S3 (needs ETL) → Transform with Glue → Load to Redshift → QuickSight
  ├─→ Real-time streams → Kinesis (KDS or Firehose) → OpenSearch/S3/Redshift
  ├─→ Kafka streams → MSK → Flink/Lambda → S3/Redshift
  ├─→ Big data Spark → EMR → S3/Redshift
  ├─→ Third-party data needed → Data Exchange → S3 → Athena
  └─→ Data lake security needed → Lake Formation over existing S3/Glue

Visualize everything → QuickSight
Search logs → OpenSearch
```

---

<a name="patterns"></a>
## Architecture Patterns to Know

### Pattern 1: Real-Time Analytics Pipeline (Lambda Architecture)
```
Data Sources (clickstreams, IoT, transactions)
        ├── Hot path (real-time):
        │   Kinesis Data Streams → Lambda/Flink → DynamoDB / OpenSearch
        │   (results in milliseconds)
        │
        └── Cold path (batch):
            Kinesis Firehose → S3 (Parquet, partitioned)
            → Glue ETL → Redshift / Athena
            (results in minutes/hours; complete history)
```

### Pattern 2: Data Lake Architecture
```
Data Sources → AWS Glue (ETL) → S3 Data Lake (raw/processed/curated zones)
                                      │
                AWS Lake Formation (security/governance)
                                      │
               ┌──────────────────────┼─────────────────────┐
               ↓                      ↓                     ↓
          Amazon Athena         Amazon Redshift         Amazon EMR
          (ad-hoc SQL)          (Spectrum queries)    (Spark processing)
               │                      │
               └──────────────────────┘
                            ↓
                    Amazon QuickSight (dashboards)
```

### Pattern 3: Log Analytics Pipeline
```
EC2 / Lambda / ECS applications
        ↓ (CloudWatch Agent / Firehose integration)
Kinesis Firehose
  ├── Transform: Lambda (parse + enrich log format)
  └── Backup: Raw logs → S3 (Parquet partitioned by date)
        ↓
Amazon OpenSearch
  └── OpenSearch Dashboards: real-time error monitoring, alert rules
        ↓ (when S3 analysis needed)
Amazon Athena → Query raw logs from S3
```

### Pattern 4: End-to-End Data Warehouse Pipeline
```
Operational Sources:
  RDS → DMS (CDC) → S3 (raw)
  Salesforce → AppFlow → S3 (raw)
  Clickstream → Kinesis Firehose → S3 (raw)
        ↓
AWS Glue (crawl schema → ETL transform → convert to Parquet)
        ↓
Amazon Redshift (load via COPY command)
  └── Redshift Spectrum (query historical S3 cold data)
        ↓
Amazon QuickSight (executive dashboards, ML insights)
```

### Pattern 5: Kafka Migration Path
```
On-Premises Kafka Cluster
  Producers (microservices) → Topics → Consumer Groups
        ↓ (MSK Replicator or MirrorMaker)
Amazon MSK
  Same Producers (unchanged) → Same Topics → Same Consumer Groups (unchanged)
        ↓ (MSK Connect)
  Kafka Connect Sink → S3 (historical data)
  Kafka Connect Sink → OpenSearch (search/monitoring)
        ↓ (Kinesis Data Analytics with Flink)
  Real-time stream processing → DynamoDB / Redshift
```

---

<a name="exam-tips"></a>
## Exam Tips Summary

### Amazon Athena
- ✅ Serverless SQL on S3; pay $5/TB scanned; zero setup
- ✅ Parquet/ORC + partitioning = up to 87% cost reduction
- ✅ Uses Glue Data Catalog for table metadata
- ✅ Federated Query = join S3 + DynamoDB + RDS in one SQL (via Lambda connectors)
- ✅ Athena Spark = serverless Spark on S3 (notebook interface)
- ✅ Workgroups = isolate teams; set query cost limits
- ✅ Result caching = reuse identical query results (7 days)

### AWS Glue
- ✅ Crawler → auto-discover schema → update Data Catalog
- ✅ Data Catalog = central metadata repo (works with Athena, EMR, Redshift Spectrum, Lake Formation)
- ✅ ETL jobs: Spark, Python Shell, Streaming (micro-batch), Ray
- ✅ DPU = 4 vCPU + 16 GB RAM; Auto Scaling available
- ✅ Glue DataBrew = no-code data preparation (350+ transforms)
- ✅ Glue Streaming = near-real-time ETL from Kinesis/Kafka

### Amazon EMR
- ✅ Managed Hadoop/Spark/Hive/HBase/Flink/Presto
- ✅ Primary/Core/Task node architecture; use Spot for task nodes safely
- ✅ EMRFS = use S3 as persistent storage (transient clusters)
- ✅ EMR Serverless = no cluster management; auto-scales for Spark/Hive
- ✅ EMR on EKS = run Spark on existing EKS cluster
- ✅ Use S3 for input/output; only HDFS for intermediate data

### Amazon Kinesis
- ✅ KDS: shard = 1 MB/sec in + 2 MB/sec out; Enhanced Fan-Out = 2 MB/sec per consumer
- ✅ KDS: Provisioned (fixed shards) vs On-Demand (auto-scale)
- ✅ KDS: retention 24 hrs default → up to 365 days (extended retention)
- ✅ Firehose: no real-time (60s–15min buffer); loads to S3/Redshift/OpenSearch; no replay
- ✅ Data Analytics: Apache Flink managed (serverless); millisecond stream processing
- ✅ Hot shard = randomize partition key

### Amazon MSK
- ✅ Fully managed Kafka; existing Kafka clients work unchanged
- ✅ Provisioned or Serverless cluster types
- ✅ MSK Connect = managed Kafka Connect (Debezium CDC, S3 sink)
- ✅ "Migrate Kafka" → MSK; "new streaming app" → KDS or Firehose

### AWS Lake Formation
- ✅ Centralized data lake governance on top of S3 + Glue Catalog
- ✅ Column-level, row-level, cell-level security across Athena/EMR/Redshift Spectrum
- ✅ LF-Tags (TBAC) = scalable permission management by tag
- ✅ Governed Tables = ACID transactions + time travel on S3
- ✅ Cross-account data sharing with fine-grained permissions

### Amazon OpenSearch
- ✅ Managed Elasticsearch/OpenSearch; full-text search + log analytics
- ✅ UltraWarm = S3-backed warm tier for older indices (lower cost)
- ✅ OpenSearch Dashboards = Kibana-compatible visualization
- ✅ Ingest via: Kinesis Firehose, Lambda, CloudWatch Logs subscription
- ✅ DynamoDB + OpenSearch = Streams → Lambda → OpenSearch for search on DynamoDB data
- ✅ Serverless OpenSearch = auto-scales; no capacity planning

### Amazon QuickSight
- ✅ Serverless BI; Author (creates) vs Reader (views) pricing
- ✅ SPICE = in-memory cache; Direct Query = real-time (no cache)
- ✅ Reader pricing = pay-per-session (~$0.30/session); great for many occasional users
- ✅ QuickSight Q = natural language → automatic chart generation
- ✅ ML Insights = anomaly detection, forecasting, auto-narratives
- ✅ Embedded analytics = integrate dashboards into your own app

### Amazon Redshift (Analytics Focus)
- ✅ Spectrum = query S3 from Redshift (no data loading needed)
- ✅ Streaming Ingestion = direct from KDS/MSK (sub-second to Redshift)
- ✅ Data Sharing = live cross-cluster data access without copying
- ✅ Redshift ML = `CREATE MODEL` SQL → SageMaker under the hood

### AWS Data Pipeline (Legacy)
- ✅ Legacy service; AWS recommends Glue (ETL) or Step Functions (orchestration)
- ✅ Supports on-premises data sources (differentiator from Glue)
- ✅ Activities: CopyActivity, HiveActivity, EmrActivity, ShellCommandActivity

### AWS Data Exchange
- ✅ Data marketplace; 3,500+ third-party datasets
- ✅ Data delivered automatically to your S3 bucket
- ✅ "Third-party data", "external dataset subscription" → Data Exchange
- ✅ Providers can monetize data; subscribers pay for access
