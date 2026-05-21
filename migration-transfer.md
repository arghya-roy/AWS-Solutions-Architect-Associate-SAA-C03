# AWS Exam Study Guide: Migration & Transfer Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Application Discovery Service · Application Migration Service · DMS · DataSync · Migration Hub · Snow Family · Transfer Family

---

## Migration Quick-Reference Matrix

| Service | Phase | What It Does | Exam Trigger Words |
|---|---|---|---|
| **Application Discovery Service** | Plan | Discover on-prem servers, dependencies, utilization | "assess", "inventory", "dependency mapping", "before migration" |
| **Migration Hub** | Plan + Track | Central dashboard to track all migrations | "track progress", "single pane", "migration status" |
| **Application Migration Service (MGN)** | Migrate | Lift-and-shift servers to AWS (rehost) | "rehost", "lift-and-shift", "replicate servers", "minimal downtime" |
| **Database Migration Service (DMS)** | Migrate | Migrate databases homogeneous or heterogeneous | "migrate database", "schema conversion", "CDC", "zero downtime DB" |
| **DataSync** | Migrate + Ongoing | Online data transfer (NFS/SMB/S3/EFS/FSx) | "sync files", "NFS to S3", "EFS to EFS", "scheduled transfer" |
| **Snow Family** | Migrate | Offline bulk data transfer via physical devices | "petabytes", "no internet", "edge computing", "physical transfer" |
| **Transfer Family** | Ongoing | Managed SFTP/FTPS/FTP/AS2 to/from S3 or EFS | "SFTP", "FTP", "partner file exchange", "legacy file transfer" |

---

## The 7 Rs of Migration (Exam Framework)

> The 7 Rs are the foundation of AWS migration strategy questions on both SAA and SAP exams.

| Strategy | Description | AWS Service |
|---|---|---|
| **Retire** | Decommission — no longer needed | N/A |
| **Retain** | Keep on-prem for now (not ready) | N/A |
| **Rehost** | Lift-and-shift; move as-is to EC2 | AWS MGN |
| **Replatform** | Lift-tinker-and-shift; minor optimizations | AWS MGN + RDS, Elastic Beanstalk |
| **Repurchase** | Move to SaaS (e.g., Salesforce, ServiceNow) | N/A |
| **Refactor / Re-architect** | Redesign for cloud-native | Lambda, ECS, DynamoDB |
| **Relocate** | Move VMware VMs to VMware Cloud on AWS | VMware Cloud on AWS |

> **Exam Tip:** "Minimal changes, move to EC2" = **Rehost (MGN)**. "Move DB to RDS with engine change" = **Replatform (DMS + SCT)**. "Redesign as microservices" = **Refactor**.

---

---

## 1. AWS Application Discovery Service

### What Problem Does It Solve?
Before migrating, you need to know **what you have**. Application Discovery Service automatically inventories on-premises servers, collects performance data, maps application dependencies, and helps you understand what needs to move and in what order.

---

### Discovery Modes

| Mode | How It Works | Best For |
|---|---|---|
| **Agentless Discovery** | Deploys OVA appliance in VMware vCenter; collects VM metadata without touching each server | VMware environments; quick inventory |
| **Agent-based Discovery** | Installs Discovery Agent on each server (Windows/Linux); collects OS, process, network connections | Physical servers, non-VMware, dependency mapping |

---

### What It Collects

| Data Type | Agentless | Agent-based |
|---|---|---|
| Server inventory (hostname, IP, OS) | ✅ | ✅ |
| CPU, RAM, disk utilization | ✅ | ✅ |
| Running processes | ❌ | ✅ |
| Network connections / dependencies | ❌ | ✅ |
| Application dependency maps | ❌ | ✅ |

---

### Integration with Migration Hub

- All discovery data automatically feeds into **AWS Migration Hub**.
- Discovery data used to group servers into **migration waves**.
- Exported to **Amazon Athena** for analysis via S3.

---

### Key Exam Scenarios

- **"Assess on-premises environment before migration"**: Application Discovery Service.
- **"Map application dependencies to plan migration order"**: Agent-based discovery.
- **"VMware environment, quick inventory without agents"**: Agentless discovery (OVA).
- **"Analyze collected discovery data with SQL queries"**: Export to S3 → query with Athena.

---

---

## 2. AWS Migration Hub

### What Problem Does It Solve?
Migration Hub provides a **single central dashboard** to track the progress of all application migrations across multiple AWS and partner migration tools. It answers: *"Where are we in the migration?"*

---

### Key Features

| Feature | Description |
|---|---|
| **Migration tracking** | Real-time status of server and database migrations |
| **Application grouping** | Group servers into logical applications for coordinated migration |
| **Tool integration** | Works with MGN, DMS, CloudEndure, and partner tools |
| **Home Region** | You must choose a Migration Hub home region — all tracking data stored there |
| **Migration Hub Refactor Spaces** | Incremental refactoring of monoliths to microservices |

---

### Migration Hub vs AWS Control Tower vs AWS Organizations

| Service | Purpose |
|---|---|
| **Migration Hub** | Track migration progress across tools |
| **Control Tower** | Governance and guardrails for multi-account setup |
| **Organizations** | Account management and SCPs |

---

### Key Exam Scenarios

- **"Single dashboard to track all migrations across AWS accounts"**: Migration Hub.
- **"Track progress of server migrations and database migrations together"**: Migration Hub.
- **"Gradually modernize monolith without big-bang rewrite"**: Migration Hub Refactor Spaces.

---

---

## 3. AWS Application Migration Service (AWS MGN)

### What Problem Does It Solve?
MGN (formerly CloudEndure Migration) automates **lift-and-shift (rehost) migration** of physical, virtual, or cloud servers to AWS with **minimal downtime** — without needing to re-architect or rewrite applications.

---

### How MGN Works

```
On-Premises Server
      ↓ (AWS Replication Agent installed)
Continuous Block-Level Replication (over TLS)
      ↓
Staging Area (low-cost EC2 + EBS in your AWS account)
      ↓ (Cutover triggered)
Launch Test/Cutover Instance (full-size EC2)
      ↓
Production EC2 Instance (minutes of downtime)
```

---

### MGN Key Concepts

| Concept | Description |
|---|---|
| **Replication Agent** | Installed on source server; continuously replicates block-level data to AWS |
| **Staging Area** | Low-cost replication servers in your VPC; always up-to-date copy |
| **Launch Template** | Defines target EC2 instance type, VPC, subnet, SG for launch |
| **Test Launch** | Launch a test instance from the replica without disrupting source |
| **Cutover** | Final migration; source system is stopped; cutover instance launched |
| **RPO** | Sub-second (continuous replication) |
| **RTO** | Minutes (instance launch time) |

---

### MGN vs Elastic Disaster Recovery (AWS DRS)

| Feature | AWS MGN | AWS Elastic DR (DRS) |
|---|---|---|
| **Primary use** | One-time migration to AWS | Ongoing DR replication |
| **Source** | On-prem / other cloud | On-prem / other cloud / AWS |
| **Replication** | Continuous until cutover | Continuous ongoing |
| **Failback** | ❌ Not designed for failback | ✅ Failback supported |
| **Use case** | Migrate servers to AWS | DR for servers (cross-region/cross-account) |

> **Exam Tip:** MGN = migrate once. DRS = ongoing disaster recovery with failback capability.

---

### MGN Supported Sources

- Physical servers (Windows, Linux)
- VMware vSphere VMs
- Microsoft Hyper-V VMs
- Other cloud providers (GCP, Azure)
- Other AWS regions/accounts

---

### Key Exam Scenarios

- **"Lift-and-shift 200 servers to AWS with minimal downtime"**: AWS MGN.
- **"Replicate on-prem server to AWS, test before cutting over"**: MGN test launch.
- **"Migrate from GCP to AWS with minimal disruption"**: AWS MGN.
- **"Ongoing DR replication with failback capability"**: AWS Elastic Disaster Recovery (not MGN).
- **"Minimize downtime during cutover"**: MGN sub-second RPO means near-zero data loss at cutover.

---

---

## 4. AWS Database Migration Service (AWS DMS)

### What Problem Does It Solve?
DMS migrates databases to AWS **quickly, securely, and with minimal downtime** — keeping the source database operational during migration. Supports both **homogeneous** (Oracle → Oracle) and **heterogeneous** (Oracle → Aurora PostgreSQL) migrations.

---

### DMS Architecture

```
Source Database ──→ DMS Replication Instance ──→ Target Database
(on-prem / cloud)   (EC2-based, in your VPC)    (RDS / Aurora / S3 / etc.)
```

---

### Migration Types

| Type | Description | Example |
|---|---|---|
| **Full Load** | One-time migration of all existing data | Small DB, acceptable downtime |
| **Full Load + CDC** | Initial full load + ongoing Change Data Capture | Most common; near-zero downtime |
| **CDC Only** | Only replicate ongoing changes (source already loaded) | Pre-loaded target, keep in sync |

#### Change Data Capture (CDC)
- Reads transaction logs from source.
- Continuously replicates inserts, updates, deletes to target.
- Source database remains fully operational during migration.
- Enables **near-zero downtime** migration.

---

### Homogeneous vs Heterogeneous Migration

| Type | Same Engine? | Schema Conversion | Tools |
|---|---|---|---|
| **Homogeneous** | ✅ (MySQL → MySQL, Oracle → Oracle) | Not required | DMS only |
| **Heterogeneous** | ❌ (Oracle → Aurora, SQL Server → PostgreSQL) | **Required** | **SCT first, then DMS** |

---

### AWS Schema Conversion Tool (SCT)

- **Standalone downloadable tool** (not a service).
- Converts source database schema (tables, views, stored procedures) to target format.
- Generates a migration assessment report showing conversion complexity.
- **Must be run BEFORE DMS** for heterogeneous migrations.
- Does NOT migrate data — only schema.

> **Exam Rule:** Heterogeneous migration = **SCT (schema) → DMS (data)**. Homogeneous = **DMS only**.

---

### DMS Supported Sources and Targets

#### Sources
Oracle, SQL Server, MySQL, MariaDB, PostgreSQL, SAP ASE, MongoDB, DB2, Azure SQL, RDS, S3, DocumentDB

#### Targets
Oracle, SQL Server, MySQL, MariaDB, PostgreSQL, Aurora, Redshift, S3, DynamoDB, OpenSearch, Kinesis, DocumentDB, Kafka, Neptune

> **Exam Tip:** DMS can use **Amazon S3 as a source** (for bulk loading) and **Amazon S3 as a target** (for data lake ingestion). This makes DMS useful for ETL-style workflows too.

---

### DMS Replication Instance Types

| Class | Use Case |
|---|---|
| **t3.micro / t3.small** | Dev/test, small databases |
| **c5 / r5 series** | Production; compute or memory optimized |
| **Multi-AZ** | HA replication instance; standby in another AZ |

---

### DMS Multi-AZ

- Creates a **standby replication instance in another AZ**.
- Automatic failover if primary replication instance fails.
- Recommended for production migrations.

---

### DMS Continuous Replication (Ongoing)

- After migration, DMS can continue running as a **continuous replication** solution.
- Keep on-prem and AWS databases in sync.
- Use for: phased cutovers, read replicas in AWS, hybrid read patterns.

---

### DMS vs Native DB Tools

| Feature | AWS DMS | Native DB tools (mysqldump, pg_dump) |
|---|---|---|
| **Downtime** | Near-zero (CDC) | Requires downtime |
| **Heterogeneous** | ✅ (with SCT) | ❌ |
| **Continuous sync** | ✅ | ❌ |
| **Managed service** | ✅ | Manual |
| **Cost** | Per replication instance-hour | Free (tool) |

---

### Key Exam Scenarios

- **"Migrate Oracle on-prem to Aurora PostgreSQL with minimal downtime"**: SCT (schema conversion) + DMS with Full Load + CDC.
- **"Keep on-prem MySQL in sync with RDS MySQL during phased migration"**: DMS CDC-only replication.
- **"Migrate MySQL to MySQL RDS"**: DMS only (homogeneous, no SCT).
- **"Migrate to Amazon Redshift from on-prem SQL Server"**: SCT + DMS (target: Redshift).
- **"DMS replication instance fails, migration interrupted"**: Use DMS Multi-AZ for HA.
- **"Load data from S3 CSV files into DynamoDB"**: DMS with S3 source and DynamoDB target.

---

---

## 5. AWS DataSync

### What Problem Does It Solve?
DataSync automates and accelerates **online data transfer** between on-premises storage (NFS, SMB, HDFS, object storage) and AWS storage services (S3, EFS, FSx). It handles scheduling, bandwidth throttling, data integrity verification, encryption, and monitoring — replacing manual scripts and tools like rsync.

---

### DataSync Architecture

```
On-Premises Storage         DataSync Agent (VM/hardware)
(NFS / SMB / HDFS)    ──→   (installed on-prem)         ──→  AWS Storage
Object Storage (S3 compat.)                                   (S3 / EFS / FSx)

AWS Storage ──→ DataSync (no agent needed for AWS-to-AWS) ──→ AWS Storage
```

---

### DataSync Locations (Source/Destination)

| Location Type | Notes |
|---|---|
| **NFS server** | On-prem NAS, Linux NFS exports |
| **SMB server** | Windows file shares |
| **HDFS** | Hadoop Distributed File System |
| **On-prem object storage** | S3-compatible APIs |
| **Amazon S3** (all storage classes) | Including Glacier tiers |
| **Amazon EFS** | Cross-region EFS replication |
| **Amazon FSx for Windows** | |
| **Amazon FSx for Lustre** | |
| **Amazon FSx for NetApp ONTAP** | |
| **Amazon FSx for OpenZFS** | |

---

### DataSync Key Features

| Feature | Description |
|---|---|
| **Speed** | Up to 10x faster than open-source tools; multi-threaded |
| **Integrity checks** | Verifies data at source and destination (checksums) |
| **Scheduling** | Hourly, daily, weekly, or custom cron |
| **Bandwidth throttling** | Limit bandwidth during business hours |
| **Filtering** | Include/exclude files by pattern |
| **Encryption** | In-transit TLS; at-rest per destination service |
| **Incremental transfers** | Only changed files are transferred after first full sync |

---

### DataSync Agent

- Required for **on-premises to AWS** transfers.
- Deployed as a VM (VMware ESXi, Microsoft Hyper-V, KVM) or hardware appliance.
- **Not required** for AWS-to-AWS transfers (e.g., EFS to EFS, S3 to S3).
- Communicates with AWS over Direct Connect, VPN, or internet.

---

### DataSync vs AWS Transfer Family vs Storage Gateway

| Feature | DataSync | Transfer Family | Storage Gateway |
|---|---|---|---|
| **Primary use** | Bulk/scheduled data migration & sync | Managed SFTP/FTP/AS2 file transfer | Hybrid on-prem to cloud storage |
| **Protocol** | NFS, SMB, HDFS, S3 API | SFTP, FTPS, FTP, AS2 | NFS, SMB, iSCSI |
| **Users** | Internal IT / migration teams | External partners / end users | On-prem applications |
| **Direction** | Bi-directional | In/out via S3 or EFS | On-prem app writes to gateway |
| **Scheduling** | ✅ Built-in | ❌ | ❌ |
| **Ongoing use** | Yes (scheduled sync) | Yes (always-on endpoint) | Yes (always-on) |

---

### DataSync vs S3 Sync (CLI)

| Feature | DataSync | aws s3 sync |
|---|---|---|
| **Speed** | Up to 10 Gbps | Limited |
| **Integrity verification** | ✅ End-to-end | Partial |
| **Scheduling** | ✅ Built-in | Manual cron |
| **Monitoring** | CloudWatch built-in | Manual |
| **Cost** | Per GB transferred | Free (EC2 data transfer costs) |
| **Use case** | Large scale, production | Ad-hoc, small data sets |

---

### Key Exam Scenarios

- **"Move 100TB from on-prem NAS to S3 over Direct Connect"**: DataSync with agent over Direct Connect.
- **"Sync on-prem NFS to EFS nightly"**: DataSync scheduled task.
- **"Replicate EFS to another region for DR"**: DataSync (AWS-to-AWS, no agent).
- **"Replace rsync scripts with managed, monitored transfers"**: DataSync.
- **"Migrate on-prem HDFS data to S3 data lake"**: DataSync HDFS source → S3 target.
- **"Partner uploads files via SFTP"**: Transfer Family (not DataSync).
- **"On-prem app writes to NFS share backed by S3"**: Storage Gateway File Gateway (not DataSync).

---

---

## 6. AWS Snow Family

### What Problem Does It Solve?
Snow Family provides **physical devices** for migrating large amounts of data to/from AWS when the internet or network connection is insufficient, too slow, or too costly. Also supports **edge computing** in environments with limited/no connectivity.

---

### Snow Family Devices

#### AWS Snowcone

| Property | Value |
|---|---|
| **Storage** | 8 TB HDD or 14 TB SSD |
| **Compute** | 2 vCPUs, 4 GB RAM |
| **Weight** | 4.5 lbs (smallest, rugged) |
| **Network** | 1 GbE, 10 GbE, WiFi |
| **Data transfer** | Ship device OR use DataSync over internet |
| **Use case** | Edge locations, IoT, tactical environments, small migrations |

#### AWS Snowball Edge

Two variants:

| | Storage Optimized | Compute Optimized |
|---|---|---|
| **Storage** | 80 TB usable HDD + 1 TB SSD | 28 TB NVMe SSD + 1 TB HDD |
| **Compute** | 40 vCPUs, 80 GB RAM | **104 vCPUs, 416 GB RAM, optional GPU** |
| **EC2 instances** | Up to 40 EC2 | Up to 104 EC2 |
| **Use case** | Large data transfer | Edge ML, video analysis, analytics |

#### AWS Snowmobile

| Property | Value |
|---|---|
| **Storage** | Up to **100 PB** per Snowmobile |
| **Form factor** | 45-foot shipping container (semi-truck) |
| **Use case** | Exabyte-scale data center migration |
| **Security** | GPS tracking, 24/7 security personnel, alarm monitoring |
| **Transfer rate** | Up to 1 Tbps |

---

### Snow Family Comparison

| Feature | Snowcone | Snowball Edge (Storage) | Snowball Edge (Compute) | Snowmobile |
|---|---|---|---|---|
| **Capacity** | 8-14 TB | 80 TB | 28 TB NVMe | 100 PB |
| **Edge compute** | Basic | Moderate | ✅ High (GPU) | ❌ |
| **Cluster** | ❌ | Up to 15 nodes | Up to 15 nodes | N/A |
| **Best for** | Small edge/IoT | Bulk migration | Edge ML/compute | Datacenter evacuation |

---

### When to Use Snow vs DataSync vs Direct Connect

| Scenario | Recommended |
|---|---|
| Data < 1 TB, good internet | DataSync or S3 CLI |
| Data 1TB–10TB, decent internet | DataSync over internet |
| Data > 10TB, slow/limited internet | Snowball Edge |
| Data > 1PB | Snowmobile |
| Edge computing, no internet | Snowball Edge Compute or Snowcone |
| Ongoing daily transfers | DataSync or Direct Connect |

> **Rule of Thumb:** If the transfer would take **more than a week** over your available bandwidth → use Snow device.
>
> Formula: `Time (days) = Data (GB) ÷ (Bandwidth Mbps × 0.1125 × 86400 seconds)`
> Example: 100TB at 100 Mbps = ~100,000 GB ÷ (100 × 0.1125 × 86400) = ~102 days → Use Snowball.

---

### Snow Family Edge Computing

Snowball Edge and Snowcone can run:
- **EC2 AMIs** (pre-installed, runs locally)
- **AWS Lambda functions** (using Greengrass)
- **AWS IoT Greengrass**
- **Amazon EKS Anywhere**
- **Amazon SageMaker Edge** (Compute Optimized with GPU)

Use cases: Military/tactical, oil rigs, ships, remote factories, disaster response zones.

---

### OpsHub

- GUI management tool for Snow devices.
- Available for Windows and Mac.
- Manage devices, transfer data, run compute workloads — without CLI.

---

### Snow Family Data Transfer Process

1. **Order device** in AWS Console.
2. AWS ships the device to you.
3. **Connect device** to your network.
4. **Transfer data** using Snowball client or OpsHub.
5. **Ship device back** to AWS.
6. AWS **imports data to S3** and **erases the device** (NIST 800-88 standards).
7. You receive confirmation in the console.

---

### Key Exam Scenarios

- **"Migrate 50TB from on-prem to S3, internet is 100Mbps"**: AWS Snowball Edge (Storage Optimized).
- **"Migrate entire 2PB data center to S3"**: AWS Snowmobile.
- **"Process data at oil rig with no internet"**: Snowball Edge Compute Optimized.
- **"Small rugged device for military edge deployment"**: AWS Snowcone.
- **"Run ML inference models at remote factory"**: Snowball Edge Compute Optimized with GPU.
- **"Cluster Snowball devices for higher capacity"**: Snowball Edge cluster (up to 15 nodes = up to 1.2 PB).

---

---

## 7. AWS Transfer Family

### What Problem Does It Solve?
Transfer Family provides **fully managed file transfer endpoints** supporting SFTP, FTPS, FTP, and AS2 protocols — storing transferred files directly in **Amazon S3** or **Amazon EFS**, without managing any file transfer infrastructure.

---

### Supported Protocols

| Protocol | Description | Security | Use Case |
|---|---|---|---|
| **SFTP** | SSH File Transfer Protocol | SSH key or password | Most common; secure file exchange |
| **FTPS** | FTP over SSL/TLS | TLS certificate | Legacy FTP clients requiring TLS |
| **FTP** | Plain File Transfer Protocol | None (plaintext) | Internal networks only; legacy |
| **AS2** | Applicability Statement 2 | Signing + encryption | EDI (Electronic Data Interchange), B2B |

---

### Transfer Family Architecture

```
External Partners / Users
    │
    ├── SFTP/FTPS/FTP ──→ Transfer Family Endpoint
    │                              │
    └── AS2 ─────────────→        ├──→ Amazon S3 (buckets/prefixes)
                                   └──→ Amazon EFS (file system)
```

---

### Transfer Family Key Features

| Feature | Description |
|---|---|
| **Custom hostname** | Use your own domain (e.g., sftp.company.com) via Route 53 |
| **Authentication** | Service-managed (username/password/SSH key), Lambda custom auth, Active Directory (via Directory Service) |
| **VPC endpoint** | Deploy in VPC for private access; assign Elastic IPs |
| **IAM role per user** | Scope each user to specific S3 bucket/prefix |
| **Logical directories** | Map virtual paths to actual S3 paths |
| **Multi-AZ** | Built-in HA across AZs |
| **Managed workflows** | Post-transfer processing via Lambda (virus scan, file routing) |

---

### Authentication Options

| Method | Description |
|---|---|
| **Service-managed** | Built-in username/SSH key or password |
| **Custom Lambda** | Lambda function validates credentials against any identity store |
| **AWS Directory Service** | Integrate with Microsoft AD for user auth |
| **IAM** | For API-based access |

---

### Transfer Family vs DataSync vs Storage Gateway

| Feature | Transfer Family | DataSync | Storage Gateway |
|---|---|---|---|
| **Protocol** | SFTP, FTPS, FTP, AS2 | NFS, SMB, HDFS | NFS, SMB, iSCSI |
| **Users** | External partners/clients | Internal IT | On-prem apps |
| **Backend** | S3 or EFS | S3, EFS, FSx | S3, EBS (snapshots) |
| **Always-on** | ✅ Persistent endpoint | Task-based | ✅ Persistent |
| **File workflows** | ✅ Managed workflows | ❌ | ❌ |
| **Use case** | B2B file exchange, SFTP | Migration, sync | Hybrid app storage |

---

### Transfer Family for AS2 (EDI)

- AS2 is used for **EDI (Electronic Data Interchange)** — standard for B2B transactions (invoices, purchase orders, shipping notices).
- Transfer Family AS2 integrates with trading partners directly.
- Replaces on-premises EDI translators and MFT (Managed File Transfer) solutions.

---

### Key Exam Scenarios

- **"Trading partners send EDI files via SFTP to S3"**: Transfer Family SFTP.
- **"Replace on-prem SFTP server with managed AWS service"**: Transfer Family.
- **"Partners upload files via SFTP; trigger processing Lambda"**: Transfer Family + Managed Workflows.
- **"Authenticate SFTP users against Active Directory"**: Transfer Family with Directory Service auth.
- **"B2B EDI transactions using AS2 protocol"**: Transfer Family AS2.
- **"SFTP endpoint accessible only from corporate network"**: Transfer Family VPC endpoint with Security Groups.
- **"Each user can only access their own S3 prefix"**: Transfer Family IAM role per user with S3 prefix policy.

---

---

## Master Comparison: All Migration Services

### By Migration Phase

```
PHASE 1 — DISCOVER & PLAN
├── Application Discovery Service → What servers exist? What are dependencies?
└── Migration Hub → Track everything in one place

PHASE 2 — MIGRATE
├── Servers → AWS MGN (lift-and-shift, minimal downtime)
├── Databases → DMS + SCT (homogeneous or heterogeneous)
├── Files/Data (online) → DataSync (NFS/SMB/HDFS → S3/EFS/FSx)
└── Files/Data (offline) → Snow Family (physical devices for large/no-internet)

PHASE 3 — ONGOING OPERATIONS
├── File transfer endpoints → Transfer Family (SFTP/FTP/AS2)
├── Continuous DB replication → DMS ongoing replication
└── Scheduled file sync → DataSync recurring tasks
```

---

### Data Transfer Decision Tree

```
Need to move data to AWS?
    │
    ├─→ Database?
    │       ├─→ Same engine → DMS only
    │       └─→ Different engine → SCT + DMS
    │
    ├─→ Files/Objects?
    │       ├─→ Internet available + < 1 week transfer time?
    │       │       ├─→ NFS/SMB/HDFS source → DataSync
    │       │       └─→ SFTP/FTP partners → Transfer Family
    │       │
    │       └─→ No internet OR > 1 week to transfer?
    │               ├─→ < 80 TB → Snowball Edge
    │               ├─→ > 80TB, < 100PB → Multiple Snowball Edge (cluster)
    │               └─→ > 1 PB → Snowmobile
    │
    └─→ Servers?
            └─→ Lift-and-shift → AWS MGN
```

---

### Key Service Differences Summary

| Comparison | Key Differentiator |
|---|---|
| **DataSync vs Storage Gateway** | DataSync = migration & sync tasks; Storage Gateway = persistent on-prem app integration |
| **DataSync vs Transfer Family** | DataSync = internal IT data movement; Transfer Family = external partner SFTP/FTP/AS2 endpoints |
| **MGN vs DRS** | MGN = one-time migration; DRS = ongoing disaster recovery with failback |
| **DMS vs SCT** | DMS = migrates DATA; SCT = converts SCHEMA (heterogeneous only) |
| **Snowball vs Snowmobile** | Snowball = up to 80TB per device; Snowmobile = up to 100PB (truck) |
| **Snowcone vs Snowball** | Snowcone = smallest/lightest for edge/IoT; Snowball = bulk data transfer |
| **Transfer Family vs S3 presigned URLs** | Transfer Family = persistent SFTP endpoint for partners; presigned URL = single-use HTTP(S) download/upload |

---

## Architecture Patterns to Know

### Pattern 1: Large-Scale Server Migration
```
On-Premises DC
    │
    ├── Application Discovery Service (inventory + dependency map)
    │            ↓
    ├── Migration Hub (group into waves, track progress)
    │            ↓
    └── AWS MGN (continuous block replication)
              ↓
         Test Launch → Verify → Cutover
              ↓
         Production EC2 in AWS VPC
```

### Pattern 2: Oracle to Aurora PostgreSQL Migration (Zero Downtime)
```
On-Prem Oracle DB (production, stays live)
    │
    ├── AWS SCT → Convert schema → Aurora PostgreSQL (schema loaded)
    │
    └── AWS DMS (Full Load + CDC)
              ↓
         Aurora PostgreSQL (data synced, CDC keeps in sync)
              ↓
         Application cutover (update connection string)
              ↓
         Stop DMS task → Decommission Oracle
```

### Pattern 3: Hybrid File Transfer Architecture
```
External Trading Partners → Transfer Family (SFTP/AS2) → S3 bucket
                                                               ↓
                                                    S3 Event → Lambda (process)
                                                               ↓
                                                         Results back to partner

Internal IT Teams → DataSync (NFS/SMB) → S3 / EFS / FSx
                                               ↓
                                    Athena / Glue / EMR (analytics)
```

### Pattern 4: Offline Migration + Ongoing Sync
```
On-Premises (100TB data, 10 Mbps internet)
    │
    ├── Phase 1: Snowball Edge → Ship to AWS → S3 (bulk data)
    │
    └── Phase 2: DataSync (agent on-prem) → Sync deltas/changes ongoing
                      │
                  Direct Connect (once provisioned for production)
```

### Pattern 5: Database Migration with Validation
```
Source: SQL Server on-prem
    │
    ├── 1. SCT: Convert SQL Server schema → Aurora MySQL schema
    │
    ├── 2. DMS Full Load: Migrate all existing data
    │
    ├── 3. DMS CDC: Capture ongoing changes (source stays live)
    │
    ├── 4. Validation: AWS DMS data validation + application testing
    │
    └── 5. Cutover: Stop writes to SQL Server → final flush → switch app
```

---

## Exam Tips Summary

### Application Discovery Service
- ✅ Agentless = VMware OVA, collects VM metadata only
- ✅ Agent-based = individual server install, captures process + network connections
- ✅ Agent-based needed for application dependency mapping
- ✅ Data feeds automatically into Migration Hub
- ✅ Export data to S3 → query with Amazon Athena

### Migration Hub
- ✅ Central tracking dashboard for all migration tools
- ✅ Must set a home region before use
- ✅ Refactor Spaces = incremental monolith to microservice migration

### AWS MGN
- ✅ Lift-and-shift (Rehost) strategy
- ✅ Continuous block-level replication; sub-second RPO
- ✅ Test launch before cutover — non-disruptive
- ✅ Supports physical, VMware, Hyper-V, GCP, Azure sources
- ✅ MGN ≠ DRS: MGN is one-way migration; DRS is ongoing DR with failback

### AWS DMS
- ✅ Homogeneous (same engine) = DMS only
- ✅ Heterogeneous (different engine) = SCT first, then DMS
- ✅ SCT converts schema only; DMS moves data
- ✅ Full Load + CDC = near-zero downtime migration (most common)
- ✅ DMS replication instance should be Multi-AZ for production
- ✅ CDC reads source transaction logs; source stays fully operational
- ✅ Can use S3 as both source AND target

### DataSync
- ✅ Agent required for on-prem → AWS; no agent for AWS → AWS
- ✅ Supports NFS, SMB, HDFS, S3-compatible, S3, EFS, all FSx types
- ✅ Built-in scheduling, throttling, integrity verification
- ✅ Up to 10x faster than open-source tools
- ✅ DataSync ≠ Storage Gateway: DataSync is migration/sync; SGW is persistent app integration

### Snow Family
- ✅ Snowcone = smallest (8-14TB), rugged, edge/IoT
- ✅ Snowball Edge Storage = 80TB, bulk migration
- ✅ Snowball Edge Compute = 28TB NVMe + 104 vCPUs + GPU, edge computing
- ✅ Snowmobile = 100PB, exabyte-scale, truck
- ✅ Rule of thumb: > 1 week to transfer via internet → use Snow
- ✅ Cluster up to 15 Snowball Edge nodes (up to ~1.2PB)
- ✅ Data erased per NIST 800-88 after import
- ✅ OpsHub = GUI management for Snow devices

### Transfer Family
- ✅ Managed SFTP/FTPS/FTP/AS2 endpoints into S3 or EFS
- ✅ External partner file exchange (B2B)
- ✅ AS2 = EDI/B2B transactions
- ✅ Custom Lambda authorizer for any identity provider
- ✅ AD integration for Windows user authentication
- ✅ IAM role per user → scope to specific S3 prefix
- ✅ VPC endpoint + Elastic IP for private/whitelisted access
- ✅ Managed Workflows = post-transfer Lambda processing (virus scan, routing)
