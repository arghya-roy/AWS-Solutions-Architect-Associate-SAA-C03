# AWS Exam Study Guide: Database Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Aurora · Aurora Serverless · DocumentDB · DynamoDB · ElastiCache · Keyspaces · Neptune · QLDB · RDS · Redshift

---

## 📋 Table of Contents

1. [Database Services Quick-Reference Matrix](#quick-reference)
2. [Database Selection Guide](#selection-guide)
3. [Amazon RDS](#rds)
4. [Amazon Aurora](#aurora)
5. [Amazon Aurora Serverless](#aurora-serverless)
6. [Amazon DynamoDB](#dynamodb)
7. [Amazon ElastiCache](#elasticache)
8. [Amazon DocumentDB](#documentdb)
9. [Amazon Keyspaces](#keyspaces)
10. [Amazon Neptune](#neptune)
11. [Amazon QLDB](#qldb)
12. [Amazon Redshift](#redshift)
13. [Master Comparisons](#comparisons)
14. [Architecture Patterns](#patterns)
15. [Exam Tips Summary](#exam-tips)

---

<a name="quick-reference"></a>
## Database Services Quick-Reference Matrix

| Service | Type | Engine | Use Case | Exam Trigger Words |
|---|---|---|---|---|
| **RDS** | Relational | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server | Traditional relational OLTP | "managed SQL", "relational DB", "multi-AZ failover" |
| **Aurora** | Relational | MySQL/PostgreSQL compatible | High-performance relational | "5x MySQL", "global database", "Aurora cluster" |
| **Aurora Serverless** | Relational (serverless) | MySQL/PostgreSQL | Variable/unpredictable workloads | "auto-pause", "scale to zero", "intermittent workload" |
| **DynamoDB** | NoSQL Key-Value/Document | Proprietary | High-scale, low-latency NoSQL | "millisecond latency", "serverless NoSQL", "key-value", "DAX" |
| **ElastiCache** | In-memory cache | Redis / Memcached | Sub-millisecond caching layer | "cache", "session store", "Redis", "microsecond latency" |
| **DocumentDB** | NoSQL Document | MongoDB-compatible | JSON documents, MongoDB migration | "MongoDB", "document database", "JSON" |
| **Keyspaces** | NoSQL Wide-column | Cassandra-compatible | Cassandra migration, time-series | "Cassandra", "wide-column", "CQL" |
| **Neptune** | Graph | Gremlin / SPARQL / openCypher | Relationships, graphs, social networks | "graph database", "relationships", "fraud graph", "knowledge graph" |
| **QLDB** | Ledger | SQL-like (PartiQL) | Immutable audit trail | "immutable ledger", "cryptographic audit", "financial transactions" |
| **Redshift** | Analytical (OLAP) | PostgreSQL-based columnar | Data warehouse, analytics | "data warehouse", "OLAP", "petabyte analytics", "BI" |

---

<a name="selection-guide"></a>
## Database Selection Guide

### By Workload Type

```
Transactional (OLTP)?
    ├─→ Relational, predictable load → RDS
    ├─→ Relational, high performance → Aurora
    ├─→ Relational, variable/intermittent → Aurora Serverless
    └─→ NoSQL, massive scale, ms latency → DynamoDB

Analytical (OLAP)?
    └─→ Data warehouse → Redshift

Caching?
    ├─→ Complex data structures, persistence, pub/sub → ElastiCache (Redis)
    └─→ Simple key-value, horizontal scale → ElastiCache (Memcached)

Special Purpose?
    ├─→ JSON/document (MongoDB compatible) → DocumentDB
    ├─→ Wide-column (Cassandra compatible) → Keyspaces
    ├─→ Graph relationships → Neptune
    └─→ Immutable audit ledger → QLDB
```

### By Migration Source

| On-Premises DB | AWS Target |
|---|---|
| MySQL | RDS MySQL or Aurora MySQL |
| PostgreSQL | RDS PostgreSQL or Aurora PostgreSQL |
| Oracle | RDS Oracle or Aurora PostgreSQL (re-platform) |
| SQL Server | RDS SQL Server |
| MongoDB | Amazon DocumentDB |
| Cassandra | Amazon Keyspaces |
| Redis | ElastiCache for Redis |
| Neo4j / graph DB | Amazon Neptune |
| Immutable ledger | Amazon QLDB |
| Teradata / Redshift / data warehouse | Amazon Redshift |

---

<a name="rds"></a>
## 1. Amazon RDS (Relational Database Service)

### What Problem Does It Solve?
Running a relational database on EC2 requires: OS patching, DB software installation and patching, backups, point-in-time recovery, HA setup, monitoring, and scaling — all manual. RDS is a **fully managed relational database service** that automates all of this, letting you focus on your schema and queries.

**Real-World Example:**
> An e-commerce company runs MySQL for their order management system. Instead of managing 3 EC2 instances (primary + standby + read replica), patching MySQL manually, and writing backup scripts, they use RDS MySQL Multi-AZ — AWS handles automatic failover, nightly backups with 35-day retention, automated patching, and read replicas for reporting — reducing DBA overhead by 80%.

---

### Supported Engines

| Engine | Notes |
|---|---|
| **MySQL** | Most popular; versions 5.7, 8.0 |
| **PostgreSQL** | Feature-rich; versions 12–16 |
| **MariaDB** | MySQL fork; community edition |
| **Oracle** | Bring Your Own License (BYOL) or License Included |
| **SQL Server** | Express, Web, Standard, Enterprise editions |
| **Db2** | IBM Db2 (added 2023) |

---

### RDS Key Features

#### Multi-AZ Deployment
- AWS automatically maintains a **synchronous standby replica** in another AZ.
- Automatic failover on primary failure (DNS endpoint updated, no connection string change needed).
- Failover time: **60–120 seconds**.
- Standby is **NOT accessible** for read traffic — only for failover.
- Backups taken from standby (no performance impact on primary).

```
Primary DB (us-east-1a) ──sync replication──→ Standby DB (us-east-1b)
         ↓                                              ↓
   Read/Write traffic                         Failover only (not readable)
```

> **Exam Tip:** Multi-AZ = **HA (High Availability)** not performance. Read Replicas = **performance** (offload reads). Don't confuse them.

#### Multi-AZ Cluster (New — 2022)
- One writer + **two readable standby instances** in different AZs.
- Standbys are readable (unlike classic Multi-AZ).
- Failover time: **35 seconds** (faster than classic Multi-AZ).
- Available for: MySQL 8.0.28+, PostgreSQL 13.4+.

#### Read Replicas
- **Asynchronous** replication of primary.
- Up to **15 read replicas** per RDS instance (MySQL/PostgreSQL/MariaDB).
- Use for: offloading read-heavy queries, analytics, reporting.
- Can be promoted to standalone DB (breaks replication).
- Can be in **different regions** (cross-region read replicas).
- Aurora: up to 15 replicas with <10ms replica lag.

```
Primary DB ──async──→ Read Replica 1 (same region)
           ──async──→ Read Replica 2 (different region)
           ──async──→ Read Replica 3 (different AZ)
```

---

### RDS Storage

| Storage Type | Use Case | Max Size |
|---|---|---|
| **gp2** (General Purpose SSD) | Legacy default; burstable | 64 TB |
| **gp3** (General Purpose SSD v3) | Current default; IOPS configurable | 64 TB |
| **io1 / io2** (Provisioned IOPS) | I/O-intensive databases | 64 TB |
| **Magnetic** | Legacy; not recommended | 4 TB |

**Storage Auto Scaling:**
- Automatically increases storage when free space is low.
- Configure **Maximum Storage Threshold**.
- Scales in 10 GB increments; no downtime.

---

### RDS Backup and Recovery

| Feature | Details |
|---|---|
| **Automated backups** | Daily full backup + transaction logs every 5 min → point-in-time recovery |
| **Retention period** | 0–35 days (0 = disabled) |
| **Manual snapshots** | On-demand; retained until you delete |
| **Cross-region copy** | Copy snapshots to another region for DR |
| **Restore** | Always creates a **new DB instance** (not in-place restore) |
| **Transaction log backup** | Every 5 minutes → restore to any second within retention period |

---

### RDS Security

| Feature | Details |
|---|---|
| **Encryption at rest** | KMS (must enable at creation; cannot enable later on existing DB) |
| **Encryption in transit** | SSL/TLS connections |
| **IAM database authentication** | MySQL and PostgreSQL; use IAM token instead of password |
| **Secrets Manager integration** | Auto-rotate DB passwords |
| **VPC** | RDS always deployed in VPC; private subnets recommended |
| **Security Groups** | Control which resources can connect on DB port |
| **RDS Proxy** | Connection pooling; reduces connection overhead (critical for Lambda) |

> **Exam Tip:** You **cannot enable encryption on an existing unencrypted RDS DB**. To encrypt: create snapshot → copy snapshot with encryption enabled → restore from encrypted snapshot.

---

### RDS Proxy

- Fully managed database proxy that pools and shares DB connections.
- Reduces connection overhead when using **Lambda** (Lambda creates new connections per invocation).
- Supports: MySQL, PostgreSQL, MariaDB, SQL Server, Aurora.
- Stores credentials in **Secrets Manager** (never exposes them to app).
- Enforces **IAM authentication**.
- Increases availability: failover handled by proxy (50% faster failover).

```
Lambda (hundreds of concurrent executions)
    ↓ (many short-lived connections)
RDS Proxy (maintains small warm connection pool)
    ↓ (few stable long-lived connections)
RDS / Aurora
```

---

### RDS Custom

- Deploy **Oracle or SQL Server** with full OS and DB engine access.
- For: custom patches, legacy apps requiring specific configurations.
- Between RDS (managed) and EC2 (self-managed).

---

### Key RDS Exam Scenarios

| Scenario | Answer |
|---|---|
| Automatic failover for MySQL with no app change | RDS MySQL Multi-AZ |
| Offload read queries from main RDS instance | RDS Read Replica |
| Lambda connects to RDS but exhausts connections | RDS Proxy |
| Encrypt existing unencrypted RDS | Snapshot → copy with encryption → restore |
| Move RDS snapshot to another region for DR | Copy snapshot cross-region |
| Application needs DB access without username/password | IAM database authentication |
| RDS needs access from on-prem without internet | VPC + Direct Connect/VPN |
| Minimize patching overhead for Oracle | RDS Oracle (automated patching by AWS) |

---

<a name="aurora"></a>
## 2. Amazon Aurora

### What Problem Does It Solve?
Traditional relational databases are constrained by their storage architecture — adding read replicas or scaling storage requires significant effort. Aurora is AWS's **cloud-native relational database** built from the ground up for the cloud, offering up to **5x the throughput of MySQL** and **3x PostgreSQL** with superior availability and automatic scaling.

**Real-World Example:**
> A social media platform's MySQL database struggles with 500,000 concurrent users. Aurora MySQL provides the same MySQL compatibility (zero code changes) with 5x the performance — automatically expanding storage from 10 GB to 128 TB as data grows, maintaining 6 copies of data across 3 AZs for 99.99% availability, and failing over in <30 seconds — all on a fully managed platform.

---

### Aurora Architecture (Unique — Exam Critical)

```
Writer Instance (Primary)
        │
        ├── Shared Cluster Volume (6 copies across 3 AZs, auto-grows)
        │   ├── AZ-1: Copy 1 + Copy 2
        │   ├── AZ-2: Copy 3 + Copy 4
        │   └── AZ-3: Copy 5 + Copy 6
        │
        └── Up to 15 Aurora Replicas (all share same storage volume)
            (sub-10ms replication lag; any can become primary)
```

**Key Architectural Differences from RDS:**
- Storage is **separate from compute** — all instances share one cluster volume.
- Storage **auto-grows** from 10 GB to **128 TB** in 10 GB increments automatically.
- **6 copies** of data across 3 AZs by default — can lose up to 2 copies without write loss.
- Replicas share the same storage — no replication lag for reads.
- Failover is **automatic in <30 seconds** (just points to a replica).

---

### Aurora Endpoints

| Endpoint Type | Points To | Use Case |
|---|---|---|
| **Cluster (Writer) Endpoint** | Current primary writer | All write operations |
| **Reader Endpoint** | Load-balanced across all replicas | All read operations |
| **Custom Endpoint** | Subset of instances you define | Route analytics queries to large instances |
| **Instance Endpoint** | Specific instance | Debugging, maintenance |

> **Exam Tip:** Applications should use **Cluster Endpoint** for writes and **Reader Endpoint** for reads. Never hardcode Instance Endpoints (they change on failover).

---

### Aurora High Availability

| Scenario | Aurora Behavior |
|---|---|
| Replica fails | Automatic removal from reader endpoint |
| Primary writer fails | Promotes highest-priority replica in <30 sec |
| All replicas fail | Aurora recreates a new replica automatically |
| AZ failure | Still operational (data on 2 remaining AZs) |
| Storage failure | Self-healing (repairs from other 5 copies) |

**Aurora Replica Failover Priority:**
- Each replica has a **failover priority tier (0–15)** — lower = preferred for promotion.
- Replicas of same tier → Aurora promotes the largest one.

---

### Aurora Global Database

- **One primary region** (read/write) + up to **5 secondary regions** (read-only).
- Replication lag: **< 1 second** (uses dedicated replication infrastructure).
- **Disaster recovery**: promote secondary region to primary in **< 1 minute** (RPO ~1 sec, RTO < 1 min).
- Use case: global apps needing local low-latency reads + DR.

```
Primary Region (us-east-1): Writer + 15 replicas
        ↓ (< 1 sec replication)
Secondary Region (eu-west-1): Read-only replicas → promote if primary fails
Secondary Region (ap-southeast-1): Read-only replicas
```

> **Exam Tip:** Aurora Global Database = cross-region replication + **< 1 minute RPO/RTO DR**. This is the key differentiator from RDS cross-region read replicas (which have higher replication lag and longer failover).

---

### Aurora Multi-Master

- ALL instances are **read/write** simultaneously.
- Write conflicts resolved by Aurora (first writer wins).
- Use case: continuous write availability (no downtime on writer failure).
- Available for MySQL only.

---

### Aurora vs RDS (Side-by-Side)

| Feature | Aurora | RDS |
|---|---|---|
| **Performance** | 5x MySQL, 3x PostgreSQL | Standard |
| **Storage scaling** | Auto 10 GB → 128 TB | Manual (up to 64 TB, auto-scaling available) |
| **Replicas** | Up to 15 (shared storage) | Up to 15 (async replication) |
| **Failover time** | < 30 seconds | 60–120 seconds |
| **Storage copies** | 6 copies / 3 AZs | 2 copies (1 in Multi-AZ) |
| **Global DB** | ✅ < 1 sec cross-region | Cross-region read replicas only |
| **Serverless option** | ✅ Aurora Serverless v2 | ❌ |
| **Cost** | Higher | Lower |
| **MySQL/PostgreSQL compat.** | ✅ | ✅ (+ Oracle, SQL Server, MariaDB) |

---

### Key Aurora Exam Scenarios

| Scenario | Answer |
|---|---|
| MySQL app needs 5x performance improvement | Aurora MySQL |
| Global app needs reads in EU and US-East with < 1 sec lag | Aurora Global Database |
| Failover in < 30 seconds with no data loss | Aurora (vs RDS's 60–120 sec) |
| Auto-scaling storage from 10 GB to 128 TB | Aurora (automatic) |
| Promote secondary region in < 1 minute for DR | Aurora Global Database |
| Zero-downtime writes even if primary fails | Aurora Multi-Master |

---

<a name="aurora-serverless"></a>
## 3. Amazon Aurora Serverless

### What Problem Does It Solve?
Traditional Aurora requires you to choose an instance size — too small means performance problems, too large means wasted cost. Aurora Serverless **automatically adjusts database capacity** based on actual usage, scaling to zero when idle (no charges) and scaling up instantly on demand.

**Real-World Example:**
> A startup's SaaS product gets zero traffic on weekends but peaks during business hours on weekdays. Aurora Serverless v2 scales from 0.5 ACUs (almost nothing) to 128 ACUs (maximum) automatically based on load — they pay only for what they use, with no idle capacity charges on weekends.

---

### Aurora Serverless Versions

| Version | Notes |
|---|---|
| **v1 (Legacy)** | Coarser scaling; cold start (seconds to wake); not recommended for new workloads |
| **v2 (Current)** | Fine-grained scaling (0.5 ACU increments); near-instant scaling; supports all Aurora features |

---

### Key Concepts

| Concept | Description |
|---|---|
| **ACU (Aurora Capacity Unit)** | ~2 GB memory + proportional CPU/network; the unit of scaling |
| **Minimum ACU** | Lowest capacity (v2: can be 0 = pause when idle) |
| **Maximum ACU** | Upper capacity limit (prevent cost overruns) |
| **Auto-pause** | Scale to zero after configurable idle period (cost savings; v1 feature) |
| **Scaling point** | Aurora waits for a transaction commit/rollback before scaling |

---

### Aurora Serverless v2 Key Properties

- Scales in **increments of 0.5 ACU** (very fine-grained).
- Scaling happens in **milliseconds** (no cold start like v1).
- Supports **Multi-AZ**, **read replicas**, **Global Database**, **Aurora Proxy**, and all Aurora features.
- Can be **mixed** with provisioned instances in the same cluster.
- Minimum: 0.5 ACU; Maximum: 128 ACU per instance.

---

### Aurora Serverless vs Provisioned Aurora

| Feature | Aurora Serverless v2 | Aurora Provisioned |
|---|---|---|
| **Scaling** | Automatic (0.5–128 ACU) | Manual instance type change |
| **Cost model** | Per ACU-second | Per instance-hour |
| **Cold start** | None (v2) | None |
| **Idle cost** | Near-zero (0.5 ACU min) | Full instance cost |
| **Read replicas** | ✅ (can be serverless or provisioned) | ✅ |
| **Global Database** | ✅ | ✅ |
| **Best for** | Variable/unpredictable load | Steady, predictable load |

---

### Key Aurora Serverless Exam Scenarios

| Scenario | Answer |
|---|---|
| Database that gets no traffic on weekends | Aurora Serverless v2 (scale to near-zero) |
| Unpredictable traffic spikes throughout the day | Aurora Serverless v2 |
| Dev/test database used a few hours per day | Aurora Serverless v2 (near-zero when idle) |
| Need to pay only for actual database usage | Aurora Serverless v2 |
| Variable load but need Global Database + replicas | Aurora Serverless v2 (supports all features) |

---

<a name="dynamodb"></a>
## 4. Amazon DynamoDB

### What Problem Does It Solve?
Relational databases struggle with extremely high throughput, unpredictable traffic, and global scale. DynamoDB is AWS's **fully managed NoSQL key-value and document database** that delivers single-digit millisecond performance at any scale — from zero to millions of requests per second — with zero infrastructure management.

**Real-World Example:**
> Amazon.com's shopping cart must handle millions of simultaneous shoppers during Prime Day. DynamoDB serves the cart data — each cart is a partition key (user ID), containing a list of items. It scales to millions of reads/writes per second automatically, maintains <10ms latency throughout, and replicates across multiple AZs with zero downtime — all without a single DBA.

---

### DynamoDB Core Concepts

| Concept | Description |
|---|---|
| **Table** | Collection of items (like a table in SQL, but schema-flexible) |
| **Item** | A single data record (like a row); up to **400 KB** |
| **Attribute** | A key-value field within an item (like a column, but flexible) |
| **Primary Key** | Uniquely identifies each item |
| **Partition Key** | Single attribute hash key (simple primary key) |
| **Sort Key** | Optional second key for range queries (composite primary key) |

---

### Primary Key Types

| Type | Structure | Use Case |
|---|---|---|
| **Simple (Partition Key only)** | `PK` alone must be unique | User profiles, product catalog |
| **Composite (PK + Sort Key)** | `PK + SK` combination must be unique | Orders per user, messages in thread |

**Example Composite Key Design:**
```
Table: Orders
PK: UserId = "user_123"
SK: OrderDate = "2026-01-15T10:30:00Z"

→ Get all orders for user_123: Query on PK = "user_123"
→ Get orders in January 2026: Query PK + SK BETWEEN "2026-01-01" and "2026-01-31"
```

---

### DynamoDB Capacity Modes

#### Provisioned Capacity Mode
- You specify **Read Capacity Units (RCU)** and **Write Capacity Units (WCU)**.
- 1 RCU = 1 strongly consistent read/sec OR 2 eventually consistent reads/sec for items ≤ 4 KB.
- 1 WCU = 1 write/sec for items ≤ 1 KB.
- Exceeding capacity → **ProvisionedThroughputExceededException** (throttled).
- **Auto Scaling**: automatically adjusts RCU/WCU based on actual usage.
- Best for: predictable, steady-state traffic.

#### On-Demand Capacity Mode
- No capacity planning — DynamoDB scales instantly.
- Pay per request: per million read/write request units.
- No throttling (within limits).
- Best for: unknown or highly variable traffic, new tables.
- More expensive per request than provisioned (for steady high traffic).

---

### DynamoDB Secondary Indexes

| Index Type | Key Range | Stored Data | Query Flexibility |
|---|---|---|---|
| **LSI (Local Secondary Index)** | Same PK, different SK | Same partition (in main table) | Extra sort keys on same partition |
| **GSI (Global Secondary Index)** | Different PK and/or SK | Separate partition | Query on any attribute |

**LSI:**
- Must be defined at **table creation** (cannot add later).
- Up to **5 LSIs** per table.
- Strong or eventual consistency.
- Shares RCU/WCU with main table.

**GSI:**
- Can be added **at any time** (even after table creation).
- Up to **20 GSIs** per table.
- Always **eventually consistent** reads.
- Has its own separate RCU/WCU capacity.
- Most flexible — any attribute can be PK/SK of a GSI.

---

### DynamoDB Consistency Models

| Model | Description | RCU Cost | Use Case |
|---|---|---|---|
| **Eventually Consistent Read** | May return stale data (replication lag) | 0.5 RCU | Most reads; highest throughput |
| **Strongly Consistent Read** | Returns most recent data | 1 RCU | When stale reads are unacceptable |
| **Transactional Read** | ACID transaction across multiple items | 2 RCU | Financial, inventory operations |

---

### DynamoDB Streams

- **Time-ordered sequence** of item-level changes (INSERT, MODIFY, REMOVE) in a table.
- Records retained for **24 hours**.
- Use with **Lambda** for event-driven processing.

**Example Use Case:**
```
DynamoDB item created/updated
        ↓
DynamoDB Stream event
        ↓
Lambda trigger
        ↓
├── Update Elasticsearch index
├── Send notification via SNS
└── Audit log to S3
```

> **Exam Tip:** DynamoDB Streams + Lambda = the standard pattern for change data capture (CDC) with DynamoDB.

---

### DynamoDB Global Tables

- **Multi-region, multi-master** replication.
- Write to any region → automatically replicated to all other regions.
- Replication lag: typically **< 1 second**.
- Enables: local reads AND local writes in multiple regions.
- Requires: DynamoDB Streams enabled.

```
us-east-1 (Read/Write) ←──── < 1 sec ────→ eu-west-1 (Read/Write)
          ↕                                           ↕
    ap-southeast-1 (Read/Write) ←──────────────────────
```

> **Exam Tip:** DynamoDB Global Tables = multi-master (all regions can write). Aurora Global Database = one primary writer (other regions read-only).

---

### DAX (DynamoDB Accelerator)

- **Fully managed in-memory cache** specifically for DynamoDB.
- Reduces read latency from **milliseconds → microseconds**.
- API-compatible with DynamoDB — minimal code change (just point to DAX endpoint).
- Works for: **Eventually consistent reads** only (not strongly consistent).
- Solves: read-heavy workloads, hot partition issues.

```
Application → DAX Cluster (microsecond cache) → DynamoDB (ms if cache miss)
```

| Feature | DAX | ElastiCache for DynamoDB |
|---|---|---|
| **Integration** | Native DynamoDB API | Custom caching logic in app |
| **Code change** | Minimal (change endpoint) | Significant (handle cache manually) |
| **Data type** | DynamoDB items only | Any data |
| **Consistency** | Eventually consistent | Configurable |

---

### DynamoDB TTL (Time to Live)

- Automatically **delete items** after a specified timestamp.
- No additional cost for deletions.
- Deleted items also appear in DynamoDB Streams (if enabled).
- Use case: session data, temporary tokens, log data with expiry.

---

### DynamoDB Transactions

- **ACID transactions** across multiple items/tables.
- `TransactGetItems`: atomically read up to 25 items.
- `TransactWriteItems`: atomically write up to 25 items.
- Use case: transfer money between accounts, inventory decrement + order create.

---

### DynamoDB Backup

| Method | Description |
|---|---|
| **On-demand backup** | Manual point-in-time snapshot (retained until deleted) |
| **PITR (Point-In-Time Recovery)** | Enable continuous backup; restore to any second in last 35 days |
| **Export to S3** | Export table data to S3 in DynamoDB JSON or ION format; no impact on PITR |

---

### Key DynamoDB Exam Scenarios

| Scenario | Answer |
|---|---|
| Sub-millisecond read latency for DynamoDB | DAX (DynamoDB Accelerator) |
| Global app needing multi-region writes | DynamoDB Global Tables |
| Trigger Lambda when DynamoDB item changes | DynamoDB Streams + Lambda |
| Auto-delete session data after 24 hours | DynamoDB TTL |
| Atomic transfer of credits between users | DynamoDB Transactions (TransactWriteItems) |
| Query DynamoDB by attributes other than PK | GSI (Global Secondary Index) |
| Unknown traffic, variable load | DynamoDB On-Demand capacity |
| Store/retrieve user shopping cart | DynamoDB (PK = userId) |
| Restore DynamoDB table to 3 days ago | Enable PITR → restore to point in time |

---

<a name="elasticache"></a>
## 5. Amazon ElastiCache

### What Problem Does It Solve?
Database queries take milliseconds; in-memory operations take microseconds. For high-traffic applications, repeatedly querying the database for the same data wastes resources and increases latency. ElastiCache provides a **fully managed in-memory caching layer** (Redis or Memcached) that serves frequently accessed data in **sub-millisecond latency**.

**Real-World Example:**
> An e-commerce site queries the same top 100 products hundreds of times per second. Instead of hitting the database each time (10ms), ElastiCache Redis serves them in <1ms — reducing database load by 95% and response times 10x. When a product updates, the cache is invalidated and re-populated from the DB.

---

### ElastiCache Engines

#### Redis
| Feature | Description |
|---|---|
| **Data structures** | Strings, Hashes, Lists, Sets, Sorted Sets, Streams, Bitmaps, HyperLogLog, Geospatial |
| **Persistence** | ✅ RDB (snapshots) + AOF (append-only file) |
| **Replication** | ✅ Primary + read replicas |
| **Pub/Sub** | ✅ Message broker capability |
| **Multi-AZ** | ✅ Automatic failover |
| **Cluster mode** | ✅ Sharding across 500 nodes |
| **Encryption** | ✅ At-rest + in-transit |
| **Lua scripting** | ✅ |
| **Transactions** | ✅ |
| **Authentication** | ✅ AUTH token + IAM |

#### Memcached
| Feature | Description |
|---|---|
| **Data structures** | Simple key-value only |
| **Persistence** | ❌ (pure in-memory, data lost on restart) |
| **Replication** | ❌ |
| **Multi-AZ** | ❌ |
| **Cluster mode** | ✅ Multi-threaded horizontal sharding |
| **Use case** | Simplest, highest-throughput key-value cache |

---

### Redis vs Memcached (Most Tested Comparison)

| Feature | Redis | Memcached |
|---|---|---|
| **Data types** | Rich (strings, lists, sets, sorted sets, hashes) | Simple strings only |
| **Persistence** | ✅ RDB + AOF | ❌ |
| **Replication + HA** | ✅ Multi-AZ failover | ❌ |
| **Pub/Sub** | ✅ | ❌ |
| **Sorted sets** | ✅ (leaderboards) | ❌ |
| **Geospatial** | ✅ | ❌ |
| **Multi-threading** | ❌ (single-threaded) | ✅ |
| **Horizontal scaling** | ✅ (cluster mode) | ✅ |
| **Session storage** | ✅ (with persistence) | ✅ (simple) |
| **Best for** | Complex caching, gaming, leaderboards, pub/sub, session | Simple, massive-scale key-value |

> **Exam Rule:** "Need HA, persistence, sorted sets, pub/sub, complex data" → **Redis**. "Simple cache, maximum performance, multi-threaded, don't need persistence" → **Memcached**.

---

### ElastiCache Caching Strategies

| Strategy | Description | Pros | Cons |
|---|---|---|---|
| **Lazy Loading (Cache-Aside)** | App checks cache first; if miss, fetch from DB + populate cache | Only caches accessed data | Cache miss = 3 trips; stale data possible |
| **Write-Through** | Write to cache AND DB simultaneously on every write | Always fresh cache | Write penalty; caching unused data |
| **Write-Behind (Write-Back)** | Write to cache immediately; async write to DB | Fastest writes | Risk of data loss if cache fails |
| **TTL-based expiration** | Set expiry on cache items | Prevents stale data indefinitely | Brief stale period |

---

### Redis Cluster Mode

| Mode | Architecture | Max Nodes | Max Memory |
|---|---|---|---|
| **Cluster Mode Disabled** | 1 primary + up to 5 replicas | 6 | Limited to node size |
| **Cluster Mode Enabled** | Up to 500 shards; each shard has replicas | 500 | Hundreds of TB |

---

### Common ElastiCache Use Cases

| Use Case | Engine | How |
|---|---|---|
| **DB query result caching** | Redis or Memcached | Cache SELECT results; invalidate on data change |
| **Session management** | Redis | Store user sessions; TTL auto-expires stale sessions |
| **Leaderboards** | Redis | Sorted Sets (ZADD, ZRANK, ZRANGE) |
| **Rate limiting** | Redis | INCR + EXPIRE per user per time window |
| **Pub/Sub messaging** | Redis | PUBLISH/SUBSCRIBE channels |
| **Real-time analytics** | Redis | HyperLogLog for unique counts; Bitmaps |
| **Geospatial indexing** | Redis | GEOADD, GEORADIUS for location queries |

---

### Key ElastiCache Exam Scenarios

| Scenario | Answer |
|---|---|
| Reduce database load from repetitive queries | ElastiCache in front of RDS/Aurora |
| Store user session data that survives node restart | ElastiCache Redis (with persistence) |
| Gaming leaderboard with real-time rankings | ElastiCache Redis (Sorted Sets) |
| Simple key-value cache, maximum throughput | ElastiCache Memcached |
| Pub/sub notification system | ElastiCache Redis |
| Cache needs HA with automatic failover | ElastiCache Redis Multi-AZ |
| Sub-millisecond DynamoDB read performance | DAX (not ElastiCache) |

---

<a name="documentdb"></a>
## 6. Amazon DocumentDB (with MongoDB compatibility)

### What Problem Does It Solve?
Applications using MongoDB need a managed cloud alternative that eliminates the operational overhead of running MongoDB clusters while maintaining **full MongoDB API compatibility** — so existing application code, drivers, and tools work without modification.

**Real-World Example:**
> An e-commerce company stores product catalogs as MongoDB documents (flexible JSON schema — different products have different attributes). They migrate from self-managed MongoDB on EC2 to DocumentDB — zero application code change (MongoDB drivers work unchanged), AWS handles backups, HA across 3 AZs, storage auto-scaling, and patching.

---

### DocumentDB Architecture

- Follows **Aurora's storage model**: 6 copies across 3 AZs, auto-grows.
- Cluster: 1 primary + up to **15 read replicas**.
- Storage auto-scales from 10 GB to **64 TB**.
- Compatible with **MongoDB 3.6, 4.0, 5.0** APIs (not 100% API compatible — some features differ).

---

### DocumentDB Key Features

| Feature | Details |
|---|---|
| **MongoDB compatibility** | MongoDB 3.6/4.0/5.0 drivers, tools (mongosh, Compass), Atlas connection |
| **Storage auto-scaling** | 10 GB → 64 TB automatically |
| **Multi-AZ** | 6 copies across 3 AZs |
| **Replication** | Up to 15 read replicas |
| **Backup** | Automated continuous backup + PITR (35 days) |
| **Encryption** | KMS at rest; TLS in transit |
| **Change streams** | MongoDB-compatible CDC (similar to DynamoDB Streams) |
| **Elastic Clusters** | Horizontally shard across multiple nodes for >1M reads/sec |

---

### DocumentDB vs MongoDB on EC2

| Feature | DocumentDB | Self-Managed MongoDB on EC2 |
|---|---|---|
| **Management** | Fully managed | Manual setup, patching, backup |
| **HA** | Built-in (6 copies, 3 AZs) | Manual replication setup |
| **Scaling** | Auto (storage); manual (instances) | Manual |
| **Cost** | Higher (managed) | Lower infra cost + higher ops cost |
| **API compatibility** | MongoDB 3.6/4.0/5.0 (not 100%) | Full native MongoDB |

---

### DocumentDB vs DynamoDB

| Feature | DocumentDB | DynamoDB |
|---|---|---|
| **Data model** | Document (MongoDB JSON) | Key-Value / Document |
| **Query language** | MongoDB Query Language (MQL) | DynamoDB API / PartiQL |
| **Schema** | Flexible (per MongoDB) | Schema-less |
| **Scale** | Vertical + replicas | Infinite horizontal |
| **MongoDB compatibility** | ✅ | ❌ |
| **Use case** | MongoDB migration | Massive scale NoSQL |

---

### Key DocumentDB Exam Scenarios

| Scenario | Answer |
|---|---|
| Migrate self-managed MongoDB to fully managed AWS | Amazon DocumentDB |
| Store flexible JSON documents (product catalog, user profiles) | Amazon DocumentDB |
| MongoDB Atlas connection to managed AWS DB | DocumentDB with Atlas compatibility |
| Need MongoDB change streams for CDC | DocumentDB change streams |

---

<a name="keyspaces"></a>
## 7. Amazon Keyspaces (for Apache Cassandra)

### What Problem Does It Solve?
Apache Cassandra is popular for high-scale, time-series, and IoT workloads — but managing Cassandra clusters (ring management, compaction, repair, upgrades) is operationally complex. Amazon Keyspaces provides **fully managed, serverless Cassandra** — using existing Cassandra Query Language (CQL) code with no infrastructure to manage.

**Real-World Example:**
> An IoT platform records sensor readings from 10 million devices (100 billion writes per month). Their Cassandra cluster on-premises requires a dedicated DBA team for ring management. Moving to Keyspaces: existing CQL code works unchanged, storage scales automatically to petabytes, and it handles 100 billion writes without provisioning any nodes.

---

### Keyspaces Key Features

| Feature | Details |
|---|---|
| **Cassandra compatibility** | CQL v3.x; Apache Cassandra 3.11 compatible drivers |
| **Serverless** | No cluster management; auto-scales |
| **Capacity modes** | On-demand (default) or provisioned |
| **Storage** | Automatic; no limits |
| **Encryption** | KMS at rest; TLS in transit |
| **PITR** | Point-in-time recovery up to 35 days |
| **Multi-AZ** | Data replicated across 3 AZs automatically |
| **IAM** | IAM-based access control (not Cassandra-native auth by default) |

---

### Keyspaces vs DynamoDB

| Feature | Keyspaces | DynamoDB |
|---|---|---|
| **Query language** | CQL (Cassandra Query Language) | DynamoDB API / PartiQL |
| **Migration** | From Cassandra (drop-in) | From any NoSQL |
| **Data model** | Wide-column | Key-value / Document |
| **Flexibility** | Cassandra-compatible | AWS-native NoSQL |
| **Use case** | Cassandra migrations; time-series | General NoSQL; gaming; e-commerce |

---

### Key Keyspaces Exam Scenarios

| Scenario | Answer |
|---|---|
| Migrate Cassandra cluster to fully managed AWS | Amazon Keyspaces |
| IoT time-series data at massive scale | Amazon Keyspaces |
| Use CQL queries without managing Cassandra | Amazon Keyspaces |
| High-write workload (billions of records) | Keyspaces or DynamoDB |

---

<a name="neptune"></a>
## 8. Amazon Neptune

### What Problem Does It Solve?
Relational databases are poor at **relationship-heavy queries** — finding "friends of friends", detecting fraud patterns across transaction networks, or traversing knowledge graphs requires complex recursive SQL joins that perform badly at scale. Neptune is a **fully managed graph database** optimized for storing and querying highly connected data.

**Real-World Example:**
> A bank needs to detect fraud rings where accounts are connected through shared phone numbers, addresses, and IP addresses. Neptune stores entities (accounts, phones, addresses) as vertices and their relationships as edges. A single graph traversal query finds all accounts connected within 3 hops to a flagged account — in milliseconds — instead of complex SQL JOINs that take minutes.

---

### Graph Models Supported

| Model | Query Language | Use Case |
|---|---|---|
| **Property Graph** | Gremlin, openCypher | Social networks, fraud detection, recommendations |
| **RDF (Resource Description Framework)** | SPARQL | Knowledge graphs, semantic web, linked data |

---

### Neptune Key Features

| Feature | Details |
|---|---|
| **Engines** | Gremlin (TinkerPop), SPARQL (RDF), openCypher |
| **Storage** | 6 copies across 3 AZs; auto-grows 10 GB to 64 TB |
| **Replicas** | Up to 15 read replicas |
| **HA** | Automatic failover < 30 seconds |
| **Encryption** | KMS at rest; TLS in transit |
| **Bulk load** | Load data from S3 using Neptune Loader |
| **Streams** | Change feed for graph updates |
| **Serverless** | ✅ Neptune Serverless (auto-scales) |
| **Full-text search** | Integration with OpenSearch |

---

### Neptune Use Cases (Exam-Tested)

| Use Case | Example |
|---|---|
| **Social networks** | Friends, followers, connections — "People you may know" |
| **Fraud detection** | Account rings connected through shared identifiers |
| **Recommendation engines** | "Customers who bought X also bought Y" |
| **Knowledge graphs** | Entity relationships for AI/search |
| **Network topology** | IT infrastructure dependency mapping |
| **Life sciences** | Drug-protein-disease relationship graphs |

---

### Key Neptune Exam Scenarios

| Scenario | Answer |
|---|---|
| Find all users connected to a fraudulent account within 3 hops | Amazon Neptune |
| Build a social network friend recommendation engine | Amazon Neptune |
| Model a knowledge graph with semantic relationships | Neptune with RDF/SPARQL |
| Store highly connected relationship data | Amazon Neptune |
| IT network topology and impact analysis | Amazon Neptune |

---

<a name="qldb"></a>
## 9. Amazon QLDB (Quantum Ledger Database)

### What Problem Does It Solve?
Many industries require an **immutable, verifiable record** of every transaction — financial systems (who transferred what), supply chains (product journey), HR systems (employment history). Traditional databases allow records to be modified or deleted, and provide no cryptographic proof of history. QLDB provides a **fully managed ledger database** with an immutable, cryptographically verifiable transaction log.

**Real-World Example:**
> A bank must prove that every fund transfer happened exactly as recorded, with no possibility of data tampering. QLDB records every transaction with a **SHA-256 cryptographic hash** chained to the previous transaction — creating a hash chain that makes any alteration mathematically detectable. Auditors can verify the complete history with a `GetDigest` API call — no third-party auditor needed.

---

### QLDB Key Concepts

| Concept | Description |
|---|---|
| **Journal** | Append-only, immutable sequence of all transactions (the ledger) |
| **Document** | Data stored as QLDB document (Amazon Ion format — superset of JSON) |
| **Table** | Collection of documents |
| **PartiQL** | SQL-compatible query language used with QLDB |
| **Digest** | Cryptographic hash of all history at a point in time |
| **Proof** | Cryptographic proof that a document existed at a specific revision |
| **Revision** | Each change to a document creates a new revision (all revisions kept) |

---

### QLDB Cryptographic Verification

```
Transaction 1: { "amount": 500, "from": "A", "to": "B" }
        ↓ Hash: SHA-256
        H1 = sha256(Transaction_1)

Transaction 2: { "amount": 300, "from": "B", "to": "C" }
        ↓ Hash: SHA-256 of (Transaction_2 + H1)
        H2 = sha256(Transaction_2 + H1)

Transaction 3: ...
        H3 = sha256(Transaction_3 + H2)

GetDigest API → returns H3
→ If ANY transaction was altered, H3 would be different
→ Mathematically proves integrity of entire history
```

---

### QLDB vs Traditional DB vs Blockchain

| Feature | QLDB | Traditional DB | Blockchain |
|---|---|---|---|
| **Immutability** | ✅ Append-only journal | ❌ Data can be modified | ✅ |
| **Cryptographic proof** | ✅ SHA-256 hash chain | ❌ | ✅ |
| **Decentralized** | ❌ (AWS central) | ❌ | ✅ |
| **Trusted authority** | ✅ (you trust AWS) | N/A | Not needed |
| **Performance** | High (centralized) | High | Low |
| **Use case** | Single-owner immutable ledger | General purpose | Multi-party decentralized trust |

> **Exam Rule:** "Need immutable audit log with cryptographic proof — single owner (internal use)" → **QLDB**. "Multi-party decentralized trust (multiple organizations)" → **Amazon Managed Blockchain**.

---

### Key QLDB Exam Scenarios

| Scenario | Answer |
|---|---|
| Track complete history of financial transactions with tamper-proof records | Amazon QLDB |
| Prove a supply chain record was not modified after the fact | Amazon QLDB |
| Immutable audit trail for healthcare record changes | Amazon QLDB |
| HR system where employment records can never be deleted | Amazon QLDB |
| Multi-company supply chain with no single trusted authority | Amazon Managed Blockchain (not QLDB) |

---

<a name="redshift"></a>
## 10. Amazon Redshift

### What Problem Does It Solve?
OLTP databases (RDS, Aurora, DynamoDB) are optimized for transactional workloads — fast individual record lookups, inserts, updates. But analytics queries (aggregations, JOINs across billions of rows, complex GROUP BYs) are extremely slow on OLTP databases. Redshift is a **fully managed, petabyte-scale data warehouse** optimized for OLAP (Online Analytical Processing) workloads.

**Real-World Example:**
> A retailer needs to analyze 10 years of sales data (5 TB) to find regional trends, seasonal patterns, and product affinity. Running this on RDS would take hours and kill production performance. Redshift executes the same query in seconds using **columnar storage** and **MPP (Massively Parallel Processing)** — querying only the relevant columns across distributed nodes in parallel.

---

### Redshift Architecture

```
Leader Node (query planning, result aggregation)
        │
        ├── Compute Node 1 (stores slice of data; executes query portion)
        ├── Compute Node 2 (stores slice of data; executes query portion)
        ├── Compute Node 3 (stores slice of data; executes query portion)
        └── Compute Node N (up to 128 nodes)
```

**Key design choices:**
- **Columnar storage**: data stored by column (not row) → only reads relevant columns for analytics.
- **Compression**: columnar data compresses 3–10x better than row storage.
- **MPP**: query distributed across all nodes in parallel.
- **Sort keys**: organize data for faster range scans.
- **Distribution keys**: control how data is distributed across nodes.

---

### Redshift Node Types

| Type | Description | Best For |
|---|---|---|
| **RA3** | Managed storage (scales separately from compute); up to 8 PB | Most workloads; recommended |
| **DC2** | Dense compute; local SSD storage | Compute-intensive, sub-terabyte |

---

### Redshift Key Features

#### Redshift Spectrum
- Query data **directly in S3** without loading it into Redshift.
- Extends Redshift queries to S3-based data lake.
- No data movement required; pay per TB scanned.
- Use case: Historical data in S3 (Parquet, ORC, JSON) + hot data in Redshift.

```
Redshift Cluster → Spectrum → S3 Data Lake (Parquet/ORC files)
                 ↑ Same SQL query, unified view
```

#### Redshift Serverless
- No cluster management; auto-scales based on workload.
- Pay per compute capacity used (RPU-hours).
- Ideal for: variable analytics workloads, dev/test, infrequent reporting.

#### Concurrency Scaling
- Automatically adds temporary capacity during peak query loads.
- Users always get fast query response regardless of concurrent users.
- First 1 hour per day free; billed per second after.

#### Redshift ML
- Create, train, and deploy ML models using **SQL** with SageMaker under the hood.
- `CREATE MODEL` SQL command trains a SageMaker model.
- Use case: customer churn prediction, product recommendation — without leaving SQL.

#### AQUA (Advanced Query Accelerator)
- Hardware-accelerated cache layer for RA3 nodes.
- Pushes computation closer to storage for up to **10x faster** query performance.

---

### Redshift Loading Data

| Method | Description | Best For |
|---|---|---|
| **COPY command** | Parallel load from S3, DynamoDB, EMR, SSH | Bulk loading (most efficient) |
| **INSERT** | Row-by-row insert via JDBC/ODBC | Small inserts only |
| **Kinesis Data Firehose** | Streaming ingestion to Redshift | Real-time data streams |
| **Glue ETL** | Transform and load from data lake | Complex ETL pipelines |
| **AWS DMS** | Migrate from OLTP databases | DB migration to Redshift |

> **Exam Tip:** For bulk loading, always use `COPY` from S3 — it's parallelized across all nodes. Never use INSERT for bulk operations.

---

### Redshift Security

| Feature | Details |
|---|---|
| **VPC** | Launch in VPC; private subnet recommended |
| **Encryption** | KMS at rest; SSL in transit |
| **Column-level security** | Grant SELECT on specific columns only |
| **Row-level security** | Filter rows based on user identity |
| **Federated query** | Query RDS/Aurora PostgreSQL/MySQL from Redshift |
| **Lake Formation** | Fine-grained access control on S3/Glue catalog data |

---

### Redshift vs RDS vs Athena (Analytics Comparison)

| Feature | Redshift | RDS/Aurora | Amazon Athena |
|---|---|---|---|
| **Type** | Data warehouse (OLAP) | Relational DB (OLTP) | Serverless query engine |
| **Storage** | Cluster (managed) | Instance + EBS/Cluster | S3 (your existing data) |
| **Performance** | MPP columnar (petabytes) | Row-based (fast single rows) | Serverless (slower large scans) |
| **Cost model** | Per node-hour (or serverless) | Per instance-hour | Per TB scanned |
| **Best for** | Large-scale analytics, BI | Transactional apps | Ad-hoc queries on S3 |
| **Setup** | Cluster provisioning | DB provisioning | Zero setup |
| **Data loading** | Required (COPY) | Direct writes | Not needed |

> **Exam Rule:**
> - "Query data already in S3 without loading" → **Athena**
> - "Petabyte analytics, BI dashboards, complex aggregations" → **Redshift**
> - "Transactional app with occasional analytics" → **RDS + read replica for analytics**

---

### Key Redshift Exam Scenarios

| Scenario | Answer |
|---|---|
| Petabyte-scale data warehouse for BI dashboards | Amazon Redshift |
| Query historical data in S3 without loading to Redshift | Redshift Spectrum |
| Load 1 TB of data from S3 into Redshift efficiently | COPY command from S3 |
| Analytics on streaming clickstream data | Kinesis Firehose → Redshift |
| Variable analytics workload (no cluster management) | Redshift Serverless |
| ML predictions using SQL only | Redshift ML |
| Query across RDS and S3 data in one SQL | Redshift Federated Query |

---

<a name="comparisons"></a>
## Master Comparisons

### SQL vs NoSQL Decision Matrix

| Need | Service |
|---|---|
| Structured relational data, SQL, ACID | RDS or Aurora |
| High-performance relational, auto-scaling storage | Aurora |
| Variable relational load, cost optimization | Aurora Serverless |
| Massive scale NoSQL, ms latency, serverless | DynamoDB |
| MongoDB-compatible documents | DocumentDB |
| Cassandra-compatible wide-column | Keyspaces |
| Relationship-heavy graph data | Neptune |
| Immutable, tamper-proof ledger | QLDB |
| Petabyte analytics, data warehouse | Redshift |
| Sub-millisecond caching | ElastiCache |

### Caching Decision

```
Need caching for your database?
    ├─→ DynamoDB backend → DAX (native, microsecond latency)
    └─→ Any other DB (RDS, Aurora, DocumentDB)
            ├─→ Complex data types, HA, persistence → Redis
            └─→ Simple key-value, max throughput, no HA needed → Memcached
```

### Multi-Region Database Decision

| Need | Service |
|---|---|
| Multi-region reads + cross-region failover < 1 min | Aurora Global Database |
| Multi-region reads + writes (multi-master) | DynamoDB Global Tables |
| Cross-region read scale | RDS cross-region read replicas |

### Analytics Stack Decision

```
Operational analytics (recent data): RDS read replica
Real-time streaming analytics: Kinesis → Redshift / OpenSearch
Historical analytics (petabytes): Redshift
Ad-hoc S3 queries: Athena
ML-powered analytics: Redshift ML or SageMaker
```

---

<a name="patterns"></a>
## Architecture Patterns to Know

### Pattern 1: OLTP + OLAP Separation
```
Application (writes) → Aurora (OLTP, primary)
                              ↓ (async replication)
                       Aurora Read Replica
                              ↓ (ETL via Glue)
                       Amazon Redshift (OLAP)
                              ↓
                       Amazon QuickSight (BI dashboard)
```

### Pattern 2: Multi-Layer Caching
```
User Request
        ↓
CloudFront (HTTP caching at edge)
        ↓ (cache miss)
Application
        ↓ (check cache)
ElastiCache Redis (query result cache, session store)
        ↓ (cache miss)
RDS/Aurora (primary database)
```

### Pattern 3: DynamoDB Event-Driven Pipeline
```
User action → DynamoDB write (On-Demand, ms latency)
                     ↓
              DynamoDB Streams (24h retention)
                     ↓
              Lambda (process change event)
                     ├─→ Update ElastiCache (invalidate cache)
                     ├─→ Send SNS notification
                     └─→ Write to Redshift (analytics)
```

### Pattern 4: Global Multi-Region Application
```
us-east-1:
  ├── Aurora Global Database (writer)
  ├── DynamoDB Global Table (writer)
  └── ElastiCache Redis (session)

eu-west-1:
  ├── Aurora Global Database (reader; promote in < 1 min for DR)
  ├── DynamoDB Global Table (writer — multi-master)
  └── ElastiCache Redis (session)

Route 53 Latency routing → nearest region
```

### Pattern 5: Data Lake + Warehouse Architecture
```
Data Sources (RDS, DynamoDB, S3, Kinesis, APIs)
        ↓
AWS Glue (ETL + Data Catalog)
        ↓
Amazon S3 (Data Lake — Parquet/ORC format)
        ├── Athena (ad-hoc SQL queries on S3)
        └── Redshift Spectrum (join with Redshift data)
                ↓
         Amazon Redshift (data warehouse — hot data)
                ↓
         Amazon QuickSight (BI visualizations)
```

---

<a name="exam-tips"></a>
## Exam Tips Summary

### Amazon RDS
- ✅ Multi-AZ = **HA** (standby not readable, failover 60–120 sec); Read Replica = **performance**
- ✅ Multi-AZ Cluster (new) = 1 writer + 2 readable standbys; failover in 35 sec
- ✅ RDS Proxy = connection pooling for Lambda (prevents DB connection exhaustion)
- ✅ Cannot encrypt existing unencrypted DB: snapshot → copy with encryption → restore
- ✅ Automated backup = PITR to any second in retention (0–35 days)
- ✅ Read replicas can be cross-region; can promote to standalone
- ✅ IAM DB authentication: MySQL + PostgreSQL only

### Amazon Aurora
- ✅ 6 copies across 3 AZs; storage auto-scales 10 GB → 128 TB; failover < 30 sec
- ✅ Up to 15 replicas sharing same storage (no replication lag for reads)
- ✅ Cluster Endpoint (write), Reader Endpoint (read-balanced), Custom Endpoint (subset)
- ✅ Global Database: < 1 sec cross-region replication; promote in < 1 min (DR)
- ✅ Multi-Master: all instances write simultaneously (MySQL only)
- ✅ Aurora ≠ RDS: no separate storage per instance; storage is shared cluster volume

### Aurora Serverless
- ✅ v2 = fine-grained (0.5 ACU), near-instant scaling, all Aurora features
- ✅ ACU = Aurora Capacity Unit (~2 GB RAM + proportional CPU)
- ✅ Best for: variable, intermittent, dev/test workloads
- ✅ v2 supports Global Database, Multi-AZ, read replicas — same as provisioned

### DynamoDB
- ✅ Partition Key + Sort Key = composite key (most flexible design)
- ✅ GSI = any attribute as key, eventually consistent, own capacity, add anytime
- ✅ LSI = same PK different SK, must define at creation, strong or eventual consistency
- ✅ DAX = microsecond latency; eventually consistent only; minimal code change
- ✅ Global Tables = multi-master multi-region (<1 sec); needs Streams enabled
- ✅ Streams + Lambda = CDC pattern (24h retention)
- ✅ TTL = auto-expire items; no extra cost
- ✅ On-Demand = no capacity planning, no throttling, pay per request

### ElastiCache
- ✅ Redis = rich data types, persistence, HA, pub/sub, sorted sets (leaderboards)
- ✅ Memcached = simple strings, multi-threaded, no persistence, no HA
- ✅ Redis Cluster Mode Enabled = shard up to 500 nodes (horizontal scale)
- ✅ Session management → Redis (persistence), simple cache → either
- ✅ Lazy loading = cache miss hits DB; write-through = always up-to-date cache

### DocumentDB
- ✅ MongoDB-compatible (3.6/4.0/5.0 drivers work)
- ✅ Same Aurora-style storage (6 copies, 3 AZs, auto-scales to 64 TB)
- ✅ Not 100% MongoDB API compatible — some features differ
- ✅ "Migrate MongoDB to AWS managed service" → DocumentDB

### Keyspaces
- ✅ Cassandra CQL-compatible; serverless; no cluster management
- ✅ "Migrate Cassandra to AWS" → Keyspaces
- ✅ On-demand or provisioned capacity; PITR 35 days; KMS encryption

### Neptune
- ✅ Graph database: Gremlin/openCypher (property graph), SPARQL (RDF/knowledge graph)
- ✅ "Highly connected data", "relationship traversal", "fraud rings", "social network" → Neptune
- ✅ Same storage model: 6 copies, 3 AZs, 15 replicas, failover < 30 sec

### QLDB
- ✅ Immutable append-only journal; cryptographic SHA-256 hash chain
- ✅ PartiQL query language; Amazon Ion data format
- ✅ "Tamper-proof", "immutable audit trail", "cryptographic verification" → QLDB
- ✅ QLDB ≠ Blockchain: QLDB = single-owner centralized; Blockchain = multi-party decentralized

### Redshift
- ✅ OLAP columnar data warehouse; MPP; petabyte scale
- ✅ COPY from S3 = most efficient bulk load method
- ✅ Spectrum = query S3 directly from Redshift (no data loading needed)
- ✅ Serverless = auto-scales, no cluster management, pay per RPU-hour
- ✅ Concurrency Scaling = handles spiky analytics demand
- ✅ Redshift ≠ Athena: Redshift = loaded data, fast complex analytics; Athena = query existing S3 data
- ✅ Federated Query = query RDS/Aurora PostgreSQL/MySQL without moving data
