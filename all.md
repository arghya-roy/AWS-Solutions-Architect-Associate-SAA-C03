# AWS Solutions Architect Exam — Complete Study Guide
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Total Services Covered:** 130+ AWS Services across 17 Categories
> **Last Updated:** May 2026

---

## 📋 Master Table of Contents

### Part 1 — Compute
- [1.1 AWS Lambda, AppSync & Fargate](#lambda-appsync-fargate)
- [1.2 Amazon EC2, EC2 Auto Scaling, Elastic Beanstalk, AWS Batch, Outposts, VMware Cloud, Wavelength, SAR](#compute)

### Part 2 — Containers
- [2.1 Amazon ECS, EKS, ECR, ECS Anywhere, EKS Anywhere, EKS Distro](#containers)

### Part 3 — Storage
- [3.1 EBS, EFS, FSx, S3, S3 Glacier, Storage Gateway, AWS Backup](#storage)

### Part 4 — Databases
- [4.1 Aurora, Aurora Serverless, DocumentDB, DynamoDB, ElastiCache, Keyspaces, Neptune, QLDB, RDS, Redshift](#databases)

### Part 5 — Networking & Content Delivery
- [5.1 VPC, ELB, Route 53, CloudFront, Global Accelerator, Direct Connect, VPNs, Transit Gateway, PrivateLink](#networking)

### Part 6 — Security, Identity & Compliance
- [6.1 IAM, IAM Identity Center, Cognito, Directory Service, KMS, CloudHSM, ACM, Secrets Manager, WAF, Shield, Network Firewall, Firewall Manager, GuardDuty, Inspector, Macie, Detective, Security Hub, Audit Manager, Artifact, RAM](#security)

### Part 7 — Application Integration
- [7.1 SQS, SNS, EventBridge, Step Functions, Amazon MQ, AppSync, AppFlow](#app-integration)

### Part 8 — Analytics
- [8.1 Athena, Glue, EMR, Kinesis, MSK, Lake Formation, OpenSearch, QuickSight, Redshift, Data Pipeline, Data Exchange](#analytics)

### Part 9 — Machine Learning
- [9.1 Comprehend, Forecast, Fraud Detector, Kendra, Lex, Polly, Rekognition, SageMaker, Textract, Transcribe, Translate](#ml)

### Part 10 — Management & Governance
- [10.1 Auto Scaling, CloudFormation, CloudTrail, CloudWatch, CLI, Compute Optimizer, Config, Control Tower, Health Dashboard, License Manager, Managed Grafana, Managed Prometheus, Organizations, Proton, Service Catalog, Systems Manager, Trusted Advisor, Well-Architected Tool](#mgmt)

### Part 11 — Migration & Transfer
- [11.1 Application Discovery, MGN, DMS, DataSync, Migration Hub, Snow Family, Transfer Family](#migration)

### Part 12 — Front-End Web & Mobile
- [12.1 Amplify, API Gateway, Device Farm, Pinpoint](#frontend)

### Part 13 — Media Services
- [13.1 Elastic Transcoder, Kinesis Video Streams](#media)

### Part 14 — Developer Tools
- [14.1 AWS X-Ray](#xray)

### Part 15 — Cost Management
- [15.1 Budgets, Cost and Usage Report, Cost Explorer, Savings Plans](#cost)

### Part 16 — AWS Transform
- [16.1 AWS Transform (Assessments, Windows, Mainframe, VMware, Custom Code)](#transform)

---


---
---

<a name="lambda-appsync-fargate"></a>
# PART 1.1 — Compute: Lambda, AppSync & Fargate

# AWS Exam Study Guide: Lambda, AppSync & Fargate
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)

---

## 1. AWS Lambda

### What Problem Does It Solve?
Lambda eliminates the need to provision and manage servers. You upload code, define a trigger, and AWS runs it — automatically scaling from 0 to thousands of concurrent executions. It solves the operational overhead of traditional server management for event-driven, short-lived workloads.

---

### Key Characteristics

| Property | Value |
|---|---|
| **Type** | Serverless, event-driven compute |
| **Execution Timeout** | Max **15 minutes** |
| **Memory** | 128 MB – 10,240 MB (scales CPU proportionally) |
| **Ephemeral Storage** | `/tmp` up to **10 GB** |
| **Deployment Package** | Up to 50 MB (zipped), 250 MB (unzipped), or use container images up to **10 GB** |
| **Concurrency Default** | 1,000 per region (soft limit, can be increased) |
| **Languages** | Node.js, Python, Java, Go, Ruby, .NET, custom runtimes |

---

### Invocation Types

| Type | Description | Use Case |
|---|---|---|
| **Synchronous** | Caller waits for response | API Gateway, ALB, Cognito |
| **Asynchronous** | Lambda queues the event, caller doesn't wait | S3 events, SNS, EventBridge |
| **Event Source Mapping** | Lambda polls the source | SQS, Kinesis, DynamoDB Streams, Kafka |

> **Exam Tip:** With **asynchronous** invocations, Lambda retries **twice** on failure. With **event source mapping** (SQS, Kinesis), Lambda retries until the event expires or succeeds.

---

### Lambda Triggers (Common Exam Scenarios)

- **API Gateway / ALB** → Synchronous HTTP-based invocation
- **S3** → Object-level events (PUT, DELETE) → Asynchronous
- **DynamoDB Streams / Kinesis** → Event source mapping (polling)
- **SQS** → Event source mapping; supports **batch processing**
- **SNS** → Asynchronous fan-out pattern
- **EventBridge (CloudWatch Events)** → Scheduled or rule-based triggers
- **Cognito** → Authentication lifecycle hooks

---

### Concurrency Concepts *(High-Frequency Exam Topic)*

| Concept | Description |
|---|---|
| **Unreserved Concurrency** | Shared pool across all functions in the account |
| **Reserved Concurrency** | Guarantees a function a specific concurrency limit; **throttles** beyond it |
| **Provisioned Concurrency** | Pre-warms execution environments; eliminates **cold starts** |

> **Cold Start**: The delay when Lambda initializes a new execution environment. Affects Java and .NET more than Python/Node.js. Provisioned concurrency solves this.

---

### Lambda Versions & Aliases

- **Versions** are immutable snapshots of your function (code + config).
- **Aliases** are pointers to a version; support **traffic shifting** (e.g., 90% → v1, 10% → v2) for canary/blue-green deployments.
- Use `$LATEST` for the unpublished, mutable version.

---

### Lambda Layers

- Shared libraries/dependencies packaged separately and reused across functions.
- Up to **5 layers** per function.
- Useful for: AWS SDK versions, common utilities, ML model binaries.

---

### Lambda in VPC

- By default, Lambda runs **outside your VPC** (has internet access).
- When placed **inside a VPC**, it loses internet access unless you add a **NAT Gateway**.
- Lambda creates **Elastic Network Interfaces (ENIs)** in your subnets.

> **Exam Tip:** "Lambda needs to access RDS in a private subnet" → Place Lambda in the VPC + use private subnet + security groups.

---

### Destinations & Dead Letter Queues (DLQ)

- **DLQ**: Sends failed **asynchronous** invocations to SQS or SNS.
- **Lambda Destinations** (newer, preferred): Route results of async invocations to SQS, SNS, EventBridge, or another Lambda — for both **success and failure**.

---

### Lambda@Edge vs CloudFront Functions

| Feature | Lambda@Edge | CloudFront Functions |
|---|---|---|
| **Runtime** | Node.js, Python | JavaScript (ES5) |
| **Max Execution Time** | 5–30 seconds | < 1 ms |
| **Triggers** | Viewer/Origin request/response | Viewer request/response only |
| **Use Case** | Auth, complex routing, A/B testing | URL rewrites, header manipulation |
| **Cost** | Higher | Lower |

---

### Key Exam Scenarios

- **High concurrency + throttling**: Use Reserved Concurrency or SQS as buffer.
- **No cold starts for critical API**: Use Provisioned Concurrency.
- **Long-running task > 15 min**: Lambda is **NOT suitable** → use ECS Fargate or EC2.
- **Lambda + RDS**: Use **RDS Proxy** to pool DB connections (Lambda's ephemeral nature can exhaust DB connections).
- **Cost optimization**: Lambda is ideal for **intermittent/spiky workloads**; EC2 is better for steady, continuous workloads.

---

---

## 2. AWS AppSync

### What Problem Does It Solve?
AppSync provides a **managed GraphQL API** service. Instead of building and maintaining a REST API layer, AppSync lets clients query exactly the data they need from multiple sources (DynamoDB, Lambda, RDS, HTTP APIs, OpenSearch) in a **single request**. It also provides **real-time subscriptions** and **offline data sync** for mobile/web apps.

---

### Key Characteristics

| Property | Value |
|---|---|
| **API Type** | GraphQL |
| **Real-time** | Yes — WebSocket-based subscriptions |
| **Offline Sync** | Yes — via Amplify DataStore |
| **Auth Methods** | API Key, IAM, Cognito User Pools, OpenID Connect, Lambda |
| **Data Sources** | DynamoDB, Lambda, RDS (Aurora Serverless), HTTP, OpenSearch, EventBridge |

---

### Core Concepts

#### Schema
- Defines **types**, **queries**, **mutations**, and **subscriptions** in GraphQL SDL.
- Acts as the contract between client and backend.

#### Resolvers
- Logic that maps a GraphQL operation to a data source.
- Two types:
  - **Unit Resolvers**: Single data source mapping.
  - **Pipeline Resolvers**: Chain multiple functions/data sources in sequence.
- Written in **Velocity Template Language (VTL)** or **JavaScript (APPSYNC_JS runtime)**.

#### Data Sources
AppSync natively integrates with:
- **DynamoDB** (most common for NoSQL data)
- **AWS Lambda** (custom business logic)
- **Aurora Serverless v2** (relational data via RDS Data API)
- **OpenSearch** (full-text search)
- **HTTP endpoints** (external REST APIs)
- **EventBridge** (event publishing)
- **None** (local resolver for mocking or pure Lambda orchestration)

---

### Authorization Modes

| Mode | Description | Use Case |
|---|---|---|
| **API Key** | Simple static key | Dev/testing, public APIs |
| **AWS IAM** | SigV4 signed requests | Server-to-server, AWS services |
| **Cognito User Pools** | JWT-based user auth | End-user mobile/web apps |
| **OpenID Connect** | 3rd party identity provider | Federated identity |
| **Lambda Authorizer** | Custom auth logic | Complex/custom auth rules |

> **Exam Tip:** Multiple auth modes can be enabled simultaneously, with one set as the **default**.

---

### Real-Time Subscriptions

- Clients subscribe to mutations via **WebSockets (MQTT over WebSocket)**.
- AppSync automatically pushes updates when data changes.
- **Use Case**: Chat apps, live dashboards, collaborative editing, notifications.

---

### Caching

- AppSync supports **server-side caching** at the API or resolver level.
- TTL configurable; reduces calls to backend data sources.
- Encryption at rest and in transit for cached data.

---

### AppSync vs API Gateway (REST)

| Feature | AppSync (GraphQL) | API Gateway (REST) |
|---|---|---|
| **Query flexibility** | Client defines shape of response | Fixed response structure |
| **Real-time** | Built-in WebSocket subscriptions | Requires separate WebSocket API |
| **Multiple data sources** | Single query, multiple sources | Separate endpoints per source |
| **Offline sync** | Native (Amplify DataStore) | Manual implementation |
| **Learning curve** | Higher (GraphQL schema) | Lower (REST familiarity) |
| **Best for** | Mobile/web apps, complex data graphs | Simple CRUD APIs, public APIs |

---

### AppSync vs API Gateway (WebSocket)

- **API Gateway WebSocket** is for general real-time two-way communication (custom protocols).
- **AppSync subscriptions** are tightly integrated with GraphQL mutations — cleaner for data-driven real-time updates.

---

### Key Exam Scenarios

- **Mobile app needs real-time data sync + offline support**: AppSync + Amplify DataStore.
- **Reduce over-fetching/under-fetching of data**: GraphQL via AppSync.
- **Aggregate data from DynamoDB + Lambda + external API in one call**: AppSync pipeline resolvers.
- **Fine-grained authorization per field**: AppSync with Cognito + VTL resolver logic.
- **AppSync is NOT suitable for**: Binary file transfers, simple CRUD with no real-time needs (use API Gateway), or streaming large datasets.

---

---

## 3. AWS Fargate

### What Problem Does It Solve?
Fargate is a **serverless compute engine for containers**. It removes the need to provision, manage, and scale EC2 instances for your container workloads. You define CPU and memory, and AWS handles the underlying infrastructure.

---

### Key Characteristics

| Property | Value |
|---|---|
| **Type** | Serverless container runtime |
| **Works With** | Amazon ECS and Amazon EKS |
| **Billing** | Per vCPU and memory, per second (minimum 1 min) |
| **OS** | Linux and Windows containers |
| **Networking** | Each task gets its own **ENI** (awsvpc mode) |

---

### Fargate Launch Type vs EC2 Launch Type (ECS)

| Feature | Fargate | EC2 Launch Type |
|---|---|---|
| **Server management** | None (serverless) | You manage EC2 instances |
| **Cost model** | Pay per task (CPU/memory) | Pay for EC2 instances (even if idle) |
| **Visibility** | No access to underlying host | Full EC2 access |
| **Startup time** | Slightly slower (cold provision) | Faster (pre-running instances) |
| **Use case** | Unpredictable/spiky workloads | Steady, high-density workloads |
| **GPU support** | No | Yes (P/G instances) |
| **Windows containers** | Yes | Yes |
| **Spot pricing** | Yes (**Fargate Spot**) | Yes (EC2 Spot) |

---

### Fargate with ECS vs EKS

| | ECS + Fargate | EKS + Fargate |
|---|---|---|
| **Orchestrator** | Amazon ECS (proprietary) | Kubernetes (open-source) |
| **Complexity** | Lower | Higher |
| **Portability** | AWS-specific | Portable across clouds |
| **Fargate Profile** | Task definitions | Fargate Profiles for namespaces |
| **Use Case** | AWS-native workloads | Multi-cloud / K8s expertise teams |

---

### Fargate Networking (awsvpc Mode)

- Every Fargate **task** gets its own **Elastic Network Interface (ENI)**.
- Supports **Security Groups** at the task level (not just instance level).
- Tasks can be placed in **public or private subnets**.
- For internet access from private subnet: **NAT Gateway** required.
- For AWS service access without internet: use **VPC Endpoints**.

---

### Fargate Storage

| Storage Type | Details |
|---|---|
| **Ephemeral storage** | 20 GB default, up to **200 GB** (configurable) |
| **EFS (Elastic File System)** | Persistent, shared storage across tasks |
| **EBS** | Not natively supported for Fargate (use ECS EC2 for EBS) |

> **Exam Tip:** If tasks need **persistent shared storage**, use **EFS**. EBS is not supported with Fargate.

---

### Fargate Spot

- Runs tasks on spare AWS capacity at **up to 70% discount**.
- Tasks can be **interrupted** with a 2-minute warning.
- Best for: Batch jobs, fault-tolerant workloads, CI/CD pipelines.
- **Not suitable** for: Stateful or latency-sensitive production workloads.

---

### Task Definition Key Components

- **CPU & Memory**: Defined at task level (Fargate) — e.g., 0.25 vCPU / 512 MB up to 16 vCPU / 120 GB.
- **IAM Task Role**: Permissions for the container to call AWS services.
- **IAM Task Execution Role**: Permissions for ECS agent (pull images from ECR, write logs to CloudWatch).
- **Container definitions**: Image URI, port mappings, environment variables, secrets.
- **Log driver**: `awslogs` for CloudWatch Logs.

> **Exam Tip:** Know the difference — **Task Role** = what the app does; **Task Execution Role** = what ECS does on your behalf.

---

### Fargate vs Lambda *(Very Common Exam Comparison)*

| Feature | Fargate | Lambda |
|---|---|---|
| **Max runtime** | Unlimited | 15 minutes |
| **Compute unit** | Container (any language/binary) | Function (limited runtimes) |
| **Startup** | Seconds (container pull) | Milliseconds–seconds |
| **State** | Can use persistent EFS | Stateless (ephemeral /tmp only) |
| **Scaling** | Task-level, slightly slower | Near-instant, per-request |
| **Packaging** | Docker image | Zip / container image |
| **Use case** | Long-running, stateful, any workload | Short-lived, event-driven, spiky |
| **Cost model** | Per second (vCPU + memory) | Per invocation + duration |

---

### Key Exam Scenarios

- **Containerized app, no server management**: Fargate over EC2 launch type.
- **Long-running batch job > 15 min**: Fargate (Lambda can't do this).
- **Need GPU for ML training**: Fargate does **NOT** support GPU → use EC2 launch type.
- **Shared persistent storage across tasks**: EFS + Fargate.
- **Cost optimization for batch**: Fargate Spot.
- **Microservices with auto-scaling**: ECS + Fargate + Application Auto Scaling.
- **Kubernetes workloads without managing nodes**: EKS + Fargate.

---

---

## Quick Comparison: Lambda vs AppSync vs Fargate

| Dimension | Lambda | AppSync | Fargate |
|---|---|---|---|
| **Category** | Serverless function | Managed GraphQL API | Serverless containers |
| **Primary use** | Event-driven compute | API layer for data access | Container orchestration |
| **Runtime limit** | 15 min | N/A (API service) | Unlimited |
| **Real-time** | Via SNS/WebSocket API | Native subscriptions | Via app implementation |
| **Packaging** | Zip / container | Schema + resolvers | Docker image |
| **Scaling** | Automatic, per request | Automatic | Task-based, auto scaling |
| **State** | Stateless | Stateless | Can be stateful (EFS) |
| **When NOT to use** | Tasks > 15 min, GPU | Simple REST APIs, file transfers | Short functions, GPU (on EKS) |

---

## Architecture Patterns to Know

### Pattern 1: Serverless Web App
```
Client → CloudFront → API Gateway → Lambda → DynamoDB
```

### Pattern 2: GraphQL Mobile App with Real-time
```
Mobile App → AppSync (GraphQL) → DynamoDB / Lambda
                ↓ (subscriptions)
           WebSocket push to clients
```

### Pattern 3: Containerized Microservices
```
Internet → ALB → ECS Fargate (Tasks) → RDS / ElastiCache
                        ↓
                  EFS (shared storage)
```

### Pattern 4: Hybrid Serverless
```
S3 Event → Lambda (< 15 min processing)
         → SQS → ECS Fargate (long-running job)
```

---

## Exam Tips Summary

### Lambda
- ✅ Max **15 minutes** timeout — hard limit
- ✅ Use **RDS Proxy** to avoid connection exhaustion
- ✅ **Provisioned Concurrency** = no cold starts
- ✅ Lambda in VPC → needs **NAT Gateway** for internet
- ✅ **Destinations** preferred over DLQ for async flows
- ✅ **Lambda@Edge** for CDN-level logic; **CloudFront Functions** for lightweight header/URL manipulation

### AppSync
- ✅ GraphQL = flexible queries, exact data shape
- ✅ **Subscriptions** = real-time via WebSocket
- ✅ **Pipeline resolvers** = chain multiple data sources
- ✅ Multiple auth modes can coexist
- ✅ Best for mobile apps needing offline + real-time sync (Amplify DataStore)

### Fargate
- ✅ No EC2 management; runs on ECS or EKS
- ✅ **awsvpc** mode = task-level security groups + ENI
- ✅ Persistent storage = **EFS** (not EBS)
- ✅ **Fargate Spot** for batch/fault-tolerant = ~70% savings
- ✅ **Task Role** ≠ **Task Execution Role** — know the difference
- ✅ No GPU support → use EC2 launch type for GPU workloads

---
---

<a name="compute"></a>
# PART 1.2 — Compute: EC2, Auto Scaling, Elastic Beanstalk, Batch, Outposts, VMware Cloud, Wavelength & SAR

# AWS Exam Study Guide: Compute Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** AWS Batch · Amazon EC2 · EC2 Auto Scaling · Elastic Beanstalk · AWS Outposts · Serverless Application Repository · VMware Cloud on AWS · AWS Wavelength

---

## 📋 Table of Contents

1. [Compute Services Quick-Reference Matrix](#quick-reference)
2. [Compute Selection Guide](#selection-guide)
3. [Amazon EC2](#ec2)
4. [Amazon EC2 Auto Scaling](#auto-scaling)
5. [AWS Elastic Beanstalk](#beanstalk)
6. [AWS Batch](#batch)
7. [AWS Outposts](#outposts)
8. [VMware Cloud on AWS](#vmware)
9. [AWS Wavelength](#wavelength)
10. [AWS Serverless Application Repository](#sar)
11. [Master Comparisons](#comparisons)
12. [Architecture Patterns](#patterns)
13. [Exam Tips Summary](#exam-tips)

---

<a name="quick-reference"></a>
## Compute Services Quick-Reference Matrix

| Service | Type | Management Level | Exam Trigger Words |
|---|---|---|---|
| **EC2** | Virtual Machines | You manage OS + above | "virtual server", "control over OS", "custom AMI", "any workload" |
| **EC2 Auto Scaling** | Scaling automation | You define policies | "scale in/out", "desired capacity", "lifecycle hooks", "target tracking" |
| **Elastic Beanstalk** | PaaS | AWS manages infra + platform | "deploy web app", "no infra management", "upload code", "PaaS" |
| **AWS Batch** | Batch processing | AWS manages compute | "batch jobs", "HPC", "hundreds of jobs", "queue-based compute" |
| **Outposts** | Hybrid on-prem cloud | AWS manages hardware | "on-premises AWS", "low latency local", "hybrid", "data residency" |
| **VMware Cloud on AWS** | Hybrid VMware | VMware + AWS co-managed | "VMware migration", "vSphere on AWS", "SDDC", "lift VMware" |
| **Wavelength** | Edge compute | AWS manages | "5G edge", "ultra-low latency", "telecom carrier", "mobile edge" |
| **SAR** | Serverless apps | Community / you publish | "serverless app repository", "reuse Lambda", "publish serverless" |

---

<a name="selection-guide"></a>
## Compute Selection Guide

```
Where does your workload run?

On AWS Cloud?
  ├─→ Full control over OS/infrastructure → EC2
  ├─→ Just upload code, don't manage servers → Elastic Beanstalk
  ├─→ Batch processing / HPC jobs → AWS Batch
  ├─→ Reuse existing serverless apps → Serverless Application Repository
  └─→ Need to scale EC2 automatically → EC2 Auto Scaling

On-Premises / Hybrid?
  ├─→ Run AWS services on-prem (AWS hardware) → AWS Outposts
  └─→ Run VMware workloads on AWS infrastructure → VMware Cloud on AWS

At the Edge (5G/Telecom)?
  └─→ Ultra-low latency for mobile 5G users → AWS Wavelength
```

---

<a name="ec2"></a>
## 1. Amazon EC2 (Elastic Compute Cloud)

### What Problem Does It Solve?
Traditionally, running a server required purchasing physical hardware, provisioning it in a data center, and managing it for years — with high upfront costs and limited flexibility. EC2 provides **resizable virtual compute capacity in the cloud** — launch a server in minutes, scale instantly, pay only for what you use, and terminate when done.

**Real-World Example:**
> A startup needs a web server. Instead of buying a $5,000 server, they launch a `t3.medium` EC2 instance for $0.0416/hour. When traffic grows, they scale to `c5.4xlarge`. When traffic drops, they scale back down. Total infrastructure cost for the first month: $30. No hardware procurement, no data center, no capacity planning.

---

### EC2 Instance Types

#### Instance Families

| Family | Optimized For | Example Types | Use Cases |
|---|---|---|---|
| **General Purpose (T, M, A)** | Balanced CPU/memory/network | t3, t4g, m5, m6i | Web servers, dev/test, small DBs |
| **Compute Optimized (C)** | High-performance CPU | c5, c6i, c7g | HPC, gaming, scientific modeling, batch |
| **Memory Optimized (R, X, Z)** | Large RAM | r5, r6i, x1e, z1d | In-memory DBs, real-time big data, SAP HANA |
| **Storage Optimized (I, D, H)** | High IOPS or throughput | i3, i4i, d3, h1 | NoSQL DBs, data warehousing, Hadoop |
| **Accelerated Computing (P, G, Inf, Trn)** | GPU / FPGA / custom chips | p4, g5, inf2, trn1 | ML training, inference, video rendering |
| **High Performance Computing (Hpc)** | HPC networking | hpc6a | Tightly coupled HPC |

#### T-Type Burstable Instances

| Feature | T3/T4g | T3/T4g Unlimited |
|---|---|---|
| **CPU credits** | Earn credits when idle, spend when busy | Unlimited burst (no CPU cap) |
| **Bill on burst** | Stops at credit limit | Additional charge for sustained burst |
| **Use case** | Variable workloads | Workloads needing sustained CPU |

> **Exam Tip:** T-type instances are **burstable** — great for development, small workloads with variable CPU. For sustained high CPU → use `c5` or `m5` instead.

---

### EC2 Purchasing Options

| Option | Description | Discount vs On-Demand | Use Case |
|---|---|---|---|
| **On-Demand** | Pay per second/hour; no commitment | 0% (baseline) | Short-term, unpredictable, dev/test |
| **Reserved Instances (1 or 3 yr)** | Commit to instance type/region | Up to **72%** | Steady-state production workloads |
| **Savings Plans** | Commit to $/hour spend | Up to **66%** (Compute) / **72%** (EC2 Instance) | Flexible steady-state workloads |
| **Spot Instances** | Bid on spare capacity | Up to **90%** | Fault-tolerant, flexible, batch jobs |
| **Dedicated Hosts** | Physical server for you | Depends | BYOL (Oracle, Windows Server), compliance |
| **Dedicated Instances** | Your instances on dedicated hardware | ~10% premium | Compliance requiring hardware isolation |
| **Capacity Reservations** | Reserve On-Demand capacity in AZ | 0% (paid at On-Demand rate) | Guaranteed capacity for critical workloads |
| **Scheduled Reserved** | Reserve capacity for time windows | Up to 5–10% | Predictable periodic workloads |

---

### Spot Instances — Deep Dive

- AWS gives you unused EC2 capacity at up to **90% discount**.
- Can be **interrupted with 2-minute warning** when AWS needs capacity back.
- **Spot Block**: reserved for 1–6 hours without interruption (being phased out).
- **Spot Fleet**: mix of instance types + On-Demand to achieve target capacity.
- **Spot Instance Advisor**: shows interruption frequency per instance type/AZ.

**Spot Best Practices:**
- Use for: batch jobs, ML training, big data, CI/CD, stateless web tiers.
- NOT for: databases, stateful apps, jobs < 2 minutes.
- Design for interruption: checkpoint regularly, use Spot interruption notices.

---

### EC2 Instance Lifecycle

```
Pending → Running → Stopping → Stopped → Terminated
                 ↓                          ↑
              Rebooting              (can restart)
```

| State | Billing | EBS | Instance Store |
|---|---|---|---|
| **Running** | ✅ Charged | Data preserved | Data preserved |
| **Stopped** | ❌ No compute charge | ✅ Data preserved | ❌ Data LOST |
| **Terminated** | ❌ No charge | Deleted (unless Delete on Termination = false) | Data LOST |
| **Rebooted** | ✅ Charged | Data preserved | Data preserved |

> **Exam Tip:** Stopping an EC2 instance **loses instance store data** but **preserves EBS data**. Terminating with default settings deletes the root EBS volume.

---

### AMI (Amazon Machine Image)

- A **template** containing OS, application server, applications, and configuration.
- Used to launch EC2 instances (defines initial state).
- Types:
  - **AWS provided**: Amazon Linux 2, Windows Server, Ubuntu, etc.
  - **AWS Marketplace**: pre-configured third-party software
  - **Community AMIs**: user-contributed
  - **Custom (private)**: AMIs you create from existing instances

**AMI Process:**
```
Running EC2 Instance → Create Image → AMI registered in region
                                              ↓
                               Launch new instance from AMI
                               (exact copy of original)
```

**AMI Copy:**
- Copy AMI to another region for cross-region launch.
- Copy AMI to another account (share AMI).
- Encrypted AMIs can be copied with re-encryption.

---

### EC2 User Data and Metadata

#### User Data
- **Bootstrap script** run at first launch (as root).
- Install packages, configure settings, start services.
- Passed as plain text or base64-encoded script.
- Runs **only once** at first boot by default.

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from $(hostname)</h1>" > /var/www/html/index.html
```

#### Instance Metadata
- Information about the running instance accessible from within the instance.
- Endpoint: `http://169.254.169.254/latest/meta-data/`
- IMDSv2 (recommended, more secure): requires session token (prevents SSRF attacks).

```bash
# IMDSv2 — Get instance ID
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id
```

---

### EC2 Placement Groups

| Type | Description | Use Case |
|---|---|---|
| **Cluster** | All instances in same AZ, same rack; low latency, high bandwidth | HPC, distributed computing, tightly coupled jobs |
| **Spread** | Each instance on different physical hardware; max 7 per AZ | Critical instances needing maximum isolation |
| **Partition** | Groups of instances in different partitions (racks); up to 7 partitions per AZ | Hadoop, Cassandra, Kafka — rack-aware distributed systems |

> **Exam Rule:**
> - "Lowest latency between instances" → **Cluster** placement group.
> - "Isolate critical instances from each other" → **Spread** placement group.
> - "Rack-aware distributed system" → **Partition** placement group.

---

### EC2 Hibernate

- Saves the **in-memory (RAM) state** to EBS root volume before stopping.
- On restart: RAM is restored from EBS → instance resumes where it left off.
- Faster startup (no OS boot, no app init).
- Requirements: EBS root volume must be encrypted; instance RAM ≤ 150 GB; not supported for all instance types.
- Use case: long-running processes, pre-warmed instances.

---

### EC2 Networking

| Feature | Description |
|---|---|
| **ENI (Elastic Network Interface)** | Virtual NIC; can have multiple private IPs, one EIP; attach/detach from instances |
| **Enhanced Networking (ENA)** | Up to 100 Gbps; lower latency; required for placement groups and HPC |
| **Elastic Fabric Adapter (EFA)** | For HPC and ML; OS-bypass networking; MPI/NCCL communication |
| **Elastic IP (EIP)** | Static public IPv4; can remap to another instance; 5 per region by default |

---

### EC2 Storage Options

| Storage | Type | Persistence | Use Case |
|---|---|---|---|
| **EBS** | Block | ✅ Persists beyond instance stop | OS, databases, persistent data |
| **Instance Store** | Block | ❌ Lost on stop/terminate | Temporary files, caches, buffers |
| **EFS** | File (NFS) | ✅ Shared, elastic | Shared workloads, Linux shared storage |
| **FSx** | File | ✅ Managed | Windows (SMB), HPC (Lustre) |
| **S3** | Object | ✅ | Static files, backups, data lake |

---

### Key EC2 Exam Scenarios

| Scenario | Answer |
|---|---|
| Lowest-latency networking between instances | Cluster placement group |
| 90% cost savings for fault-tolerant batch jobs | Spot Instances |
| Lock in lowest price, running 24/7 for 3 years | Reserved Instances (All Upfront, 3-year) |
| Physical server for Oracle BYOL licensing | Dedicated Host |
| Instance must have same IP after stop/start | Elastic IP |
| Accelerate EC2 ML training | P or Trn instance family |
| Share AMI across regions | Copy AMI to another region |
| Prevent SSRF attacks via metadata service | Enable IMDSv2 (require session token) |
| Instance maintains state across stop/start | EC2 Hibernate |
| Run HPC MPI jobs with OS-bypass networking | Elastic Fabric Adapter (EFA) |

---

<a name="auto-scaling"></a>
## 2. Amazon EC2 Auto Scaling

### What Problem Does It Solve?
Web traffic is unpredictable — a marketing campaign might triple your traffic in minutes, then drop back. Running enough EC2 instances for peak demand wastes money during quiet periods; running for average demand causes outages during peaks. EC2 Auto Scaling **automatically adjusts the number of EC2 instances** to match demand — maintaining performance while minimizing cost.

**Real-World Example:**
> An e-commerce site normally runs 4 EC2 instances. During a flash sale, traffic spikes 15x in 5 minutes. Auto Scaling automatically launches 56 more instances within 2 minutes (using pre-warmed AMIs + lifecycle hooks), handles the load, then terminates all extra instances when traffic subsides — total extra cost: $12 for 3 hours of burst capacity.

---

### Auto Scaling Group (ASG) Components

| Component | Description |
|---|---|
| **Launch Template** | Defines instance config: AMI, type, SG, key pair, user data, IAM role |
| **Minimum capacity** | ASG never scales below this number |
| **Maximum capacity** | ASG never scales above this number |
| **Desired capacity** | Target number of running instances |
| **Health checks** | EC2 status checks or ELB health checks |
| **Cooldown period** | Wait time after scaling before next action (default 300s) |

---

### Scaling Policies

| Policy Type | Trigger | Scaling Behavior | Best For |
|---|---|---|---|
| **Target Tracking** | CloudWatch metric deviates from target | Automatically adjusts to maintain target | Simplest; most common (e.g., CPU at 50%) |
| **Step Scaling** | CloudWatch alarm thresholds | Different step adjustments at different alarm levels | Need graduated response |
| **Simple Scaling** | CloudWatch alarm | One adjustment per alarm; waits for cooldown | Legacy; use step scaling instead |
| **Scheduled Scaling** | Time/date (cron) | Scale at defined times | Predictable traffic patterns |
| **Predictive Scaling** | ML forecast of future traffic | Proactively scales before load arrives | Daily/weekly recurring patterns |

---

### Launch Template vs Launch Configuration

| Feature | Launch Template (✅ Current) | Launch Configuration (Legacy) |
|---|---|---|
| **Versioning** | ✅ Multiple versions | ❌ No versioning |
| **Spot + On-Demand mix** | ✅ | ❌ |
| **Multiple instance types** | ✅ | ❌ |
| **Placement groups** | ✅ | Limited |
| **EBS encryption** | ✅ Native | Limited |
| **T2/T3 Unlimited** | ✅ | ❌ |
| **Recommendation** | ✅ Use this | Avoid for new ASGs |

---

### Instance Lifecycle Hooks

Hooks allow custom actions at scale-out and scale-in:

```
SCALE OUT:
  EC2 launches → Pending:Wait state
        ↓ (up to 3600 seconds)
  Custom action: install software, warm caches, register with discovery
        ↓ (send CompleteLifecycleAction)
  Pending:Proceed → InService → receives traffic

SCALE IN:
  Terminating:Wait state
        ↓ (up to 3600 seconds)
  Custom action: drain connections, save state, deregister from service mesh
        ↓ (send CompleteLifecycleAction)
  Terminating:Proceed → Terminated
```

**Notification targets:** SNS, SQS, EventBridge, Lambda.

---

### ASG Termination Policies

Default evaluation order:
1. Select AZ with most instances.
2. Within that AZ: oldest launch configuration/template.
3. Then: closest to billing hour.
4. Then: random.

**Custom policies:** `OldestInstance`, `NewestInstance`, `OldestLaunchTemplate`, `AllocationStrategy` (for Spot).

---

### ASG with Spot Instances

- Define **multiple instance types + sizes** in launch template.
- ASG uses **Capacity Rebalancing**: proactively replaces Spot instances at high interruption risk.
- **Allocation strategies**:
  - `lowest-price`: cheapest capacity
  - `capacity-optimized`: deepest available capacity pool (fewer interruptions)
  - `price-capacity-optimized`: balance of price and capacity (recommended)

---

### ASG with Load Balancers

```
Internet → ALB/NLB
                ↓
         Target Group
                ↓
         Auto Scaling Group
         (ASG registers/deregisters instances automatically)
```

- ASG automatically registers new instances with ALB target group.
- ELB health checks: ASG terminates unhealthy instances and launches new ones.
- **Connection Draining** (Deregistration Delay): wait for in-flight requests to complete before terminating.

---

### Key EC2 Auto Scaling Exam Scenarios

| Scenario | Answer |
|---|---|
| Scale based on ALB request count | Target Tracking with ALBRequestCountPerTarget |
| Ensure custom script runs before instance serves traffic | Lifecycle Hook (Pending:Wait) |
| Proactively scale before predicted Monday morning peak | Predictive Scaling |
| Scale at 8am, scale down at 8pm every weekday | Scheduled Scaling |
| Mix Spot and On-Demand for cost + reliability | ASG with mixed instances policy |
| Replace unhealthy instances automatically | ASG with ELB health check |
| Add new instances to ALB target group automatically | ASG registered with ALB Target Group |

---

<a name="beanstalk"></a>
## 3. AWS Elastic Beanstalk

### What Problem Does It Solve?
Developers want to focus on code, not infrastructure. Deploying a web application on AWS manually requires setting up EC2, ALB, Auto Scaling, VPC, security groups, IAM roles, CloudWatch — a week of work. Elastic Beanstalk is a **Platform-as-a-Service (PaaS)** that provisions and manages all this infrastructure automatically when you just **upload your application code**.

**Real-World Example:**
> A Python developer builds a Django web app. They run `eb deploy` with their code — Beanstalk automatically creates an ALB, Auto Scaling group of EC2 instances, RDS database, VPC, CloudWatch monitoring, and deploys the application. The developer never touches infrastructure. When they need to update, `eb deploy` again — zero downtime rolling deployment.

---

### Supported Platforms

| Language / Platform | Notes |
|---|---|
| **Python** | Django, Flask |
| **Node.js** | Express, Koa |
| **Java** | Spring Boot, Tomcat |
| **PHP** | Laravel, WordPress |
| **Ruby** | Rails, Sinatra |
| **.NET** | ASP.NET on Windows Server |
| **Go** | |
| **Docker** | Single + multi-container |
| **Packer** | Custom AMI |

---

### Beanstalk Components

| Component | Description |
|---|---|
| **Application** | Top-level container (like a project) |
| **Application version** | Specific deployment artifact (ZIP/WAR/Docker) |
| **Environment** | Running AWS resources serving a version (web or worker) |
| **Environment tier** | Web server tier OR Worker tier |
| **Configuration** | Instance type, scaling, VPC, env vars, etc. |

---

### Environment Tiers

#### Web Server Tier
- Runs a web application accessible over HTTP/HTTPS.
- Resources: ALB + EC2 Auto Scaling Group + EC2 instances.
- Handles HTTP requests from users.

#### Worker Tier
- Processes background tasks from an **SQS queue**.
- Resources: EC2 instances + SQS queue (Auto Scaling based on queue depth).
- Daemon reads from SQS and posts to a local HTTP endpoint on the instance.
- Use case: email sending, video processing, batch jobs.

```
Web Tier: User → ALB → EC2 (handles HTTP requests)
                              ↓ (sends task to SQS)
Worker Tier:            SQS Queue → EC2 Worker (processes task)
```

---

### Beanstalk Deployment Policies

| Policy | Downtime | Risk | Speed | Notes |
|---|---|---|---|---|
| **All at once** | ✅ Brief downtime | High | Fastest | Dev/test only |
| **Rolling** | ❌ None (gradual) | Medium | Slower | Reduced capacity during deploy |
| **Rolling with additional batch** | ❌ None | Low | Slower | Full capacity maintained |
| **Immutable** | ❌ None | Lowest | Slowest | New ASG; fastest rollback |
| **Traffic splitting (Canary)** | ❌ None | Very Low | Moderate | % traffic to new version |
| **Blue/Green** | ❌ None | Lowest | Moderate | Swap CNAMEs |

> **Exam Rule:**
> - "Zero downtime + fastest rollback" → **Immutable** deployment.
> - "Production with no capacity reduction + safe" → **Rolling with additional batch**.
> - "Canary test with small % of traffic" → **Traffic Splitting**.

---

### .ebextensions

- Customization files placed in a `.ebextensions/` directory in your application bundle.
- YAML/JSON configuration files with `.config` extension.
- Can configure: packages, files, services, commands, environment properties, resources.

```yaml
# .ebextensions/install-packages.config
packages:
  yum:
    git: []
    nodejs: []

commands:
  01_create_dir:
    command: mkdir -p /var/app/logs

container_commands:
  01_migrate:
    command: "python manage.py migrate"
    leader_only: true
```

---

### Beanstalk Managed Updates

- Beanstalk automatically applies platform updates (OS patches, runtime updates) during a maintenance window.
- Minor/patch updates only (configurable).
- Major version updates require manual action.

---

### Beanstalk vs EC2 vs ECS vs Lambda

| Feature | Beanstalk | Raw EC2 | ECS | Lambda |
|---|---|---|---|---|
| **Server management** | AWS handles | You manage | AWS handles (Fargate) | None |
| **Code packaging** | ZIP/WAR/Docker | AMI/user data | Docker image | ZIP/container |
| **Scaling** | Auto-configured | Manual ASG | Auto Scaling | Auto |
| **OS control** | Limited | Full | Limited | None |
| **Supported languages** | Predefined list | Any | Any (Docker) | Predefined list |
| **Cost** | No extra cost | Instance cost | Task cost | Per invocation |
| **Use case** | Web apps (traditional) | Full control | Microservices | Event-driven functions |

---

### Key Beanstalk Exam Scenarios

| Scenario | Answer |
|---|---|
| Deploy web app without managing EC2, ALB, ASG | Elastic Beanstalk |
| Process background tasks from SQS | Beanstalk Worker Tier |
| Zero-downtime deployment with instant rollback | Beanstalk Immutable deployment |
| Deploy Docker containers without K8s complexity | Beanstalk (Docker platform) |
| Customize EC2 instance during Beanstalk deploy | .ebextensions configuration files |
| Run DB migrations during deployment (once per deploy) | .ebextensions container_commands with leader_only: true |

---

<a name="batch"></a>
## 4. AWS Batch

### What Problem Does It Solve?
Organizations run large-scale batch jobs — genomics analysis, financial risk modeling, video rendering, ML training — that require hundreds or thousands of compute instances for finite periods. Managing the compute infrastructure, job scheduling, retries, and scaling for these workloads is complex and expensive. AWS Batch **fully manages batch computing workloads** — you submit jobs, it handles everything else.

**Real-World Example:**
> A pharmaceutical company runs genomics sequencing jobs that analyze 10,000 patient samples per night. Each analysis job requires 4 vCPUs and 16 GB RAM for ~30 minutes. AWS Batch automatically provisions hundreds of EC2 Spot instances, processes all 10,000 jobs in parallel (finishing in 2 hours instead of 13 days serially), then terminates all instances — costing 90% less than On-Demand instances left running 24/7.

---

### AWS Batch Core Components

| Component | Description |
|---|---|
| **Job** | Unit of work (Docker container); has a definition, parameters, dependencies |
| **Job Definition** | Template: Docker image, vCPU, memory, IAM role, retry strategy, timeout |
| **Job Queue** | Jobs wait here before execution; associated with compute environments; has a priority |
| **Compute Environment** | Set of compute resources (EC2 Spot/On-Demand, Fargate) that runs jobs |
| **Array Job** | Single job definition that runs N parallel copies (e.g., process 1,000 files with one submit) |
| **Job Dependencies** | Job B runs only after Job A completes successfully |

---

### Compute Environment Types

| Type | Description | Use Case |
|---|---|---|
| **Managed** | AWS manages EC2 fleet (launch, scale, terminate) | Most common; minimal ops |
| **Unmanaged** | You manage EC2 instances (just register to Batch) | Custom instance configurations |

**Within Managed:**
| Option | Description |
|---|---|
| **EC2 On-Demand** | Consistent, predictable capacity |
| **EC2 Spot** | Up to 90% cheaper; Batch handles interruptions + retries |
| **Fargate** | Serverless (no EC2); per-job compute; faster start for small jobs |
| **Fargate Spot** | Serverless + Spot discounts |

---

### AWS Batch vs AWS Lambda vs EC2

| Feature | AWS Batch | AWS Lambda | EC2 (manual) |
|---|---|---|---|
| **Job type** | Batch / HPC / long-running | Event-driven / short-lived | Any |
| **Max runtime** | No limit | 15 minutes | No limit |
| **Container support** | ✅ Docker required | ✅ (container option) | Any |
| **GPU support** | ✅ | ❌ | ✅ |
| **Cost model** | Per EC2/Fargate used | Per invocation + duration | Per EC2 instance-hour |
| **Scaling** | Automatic (up to 1000s of instances) | Automatic (per request) | Manual or ASG |
| **Orchestration** | Built-in queue + dependencies | Manual | Manual |

> **Exam Rule:** "Long-running batch jobs (> 15 min), HPC, GPU processing, hundreds of parallel jobs" → **AWS Batch**. "Short event-driven functions (< 15 min)" → **Lambda**.

---

### Batch Job Priority and Queues

```
High Priority Job Queue (priority: 10)
  └── Jobs: Critical analyses (run first)
  └── Compute Environment: On-Demand (always available)

Low Priority Job Queue (priority: 1)
  └── Jobs: Routine analyses (run when resources available)
  └── Compute Environment: Spot (cheap, can wait for capacity)
```

- Multiple queues with different priorities.
- Jobs from higher-priority queues are scheduled first.
- Each queue can be associated with multiple compute environments (ordered by priority).

---

### Batch with Spot Instances

- Batch automatically handles Spot interruptions: **checkpoints** job, retries on different instance.
- **Retry strategy**: define number of automatic retry attempts.
- **Automatic instance selection**: Batch chooses from multiple instance types to maximize Spot availability.

---

### Key Batch Exam Scenarios

| Scenario | Answer |
|---|---|
| Process 10,000 genomics samples overnight in parallel | AWS Batch |
| Run HPC jobs on hundreds of GPU instances | AWS Batch with EC2 GPU compute environment |
| Batch jobs need maximum cost savings | AWS Batch with Spot Instances (Managed CE) |
| Job B must run after Job A completes | AWS Batch job dependencies |
| Run 1,000 copies of the same job in parallel | AWS Batch Array Jobs |
| Run batch jobs without managing EC2 instances | AWS Batch with Fargate compute environment |
| Batch job is interrupted by Spot; must retry | AWS Batch retry strategy |

---

<a name="outposts"></a>
## 5. AWS Outposts

### What Problem Does It Solve?
Some workloads **cannot move to the public cloud** due to regulatory requirements, data residency laws, latency requirements, or dependencies on on-premises systems. Outposts brings **AWS infrastructure, services, APIs, and tools to your on-premises data center** — enabling a true hybrid experience where the same AWS services run both in the cloud and on-premises.

**Real-World Example:**
> A European bank has regulatory requirements that customer data must remain in Germany. AWS Outposts delivers physical AWS rack servers to their Frankfurt data center. They run RDS on Outposts, ECS containers on Outposts, and S3 on Outposts — using identical APIs as their cloud workloads. Data stays physically in Germany, but they use the same AWS console, CloudFormation templates, and IAM policies as their cloud environments.

---

### Outposts Form Factors

| Form Factor | Size | Use Case |
|---|---|---|
| **Outposts Rack** | 42U rack (or multi-rack) | Full data center deployments |
| **Outposts Servers** | 1U or 2U server | Space-constrained locations (factory floors, retail stores, hospitals) |

---

### Supported AWS Services on Outposts

| Service Category | Services Available |
|---|---|
| **Compute** | EC2, ECS, EKS |
| **Storage** | EBS, S3 (Outposts), FSx |
| **Database** | RDS, ElastiCache |
| **Networking** | VPC subnets, ALB, NLB |
| **Others** | SageMaker, EMR, MSK |

---

### Outposts Architecture

```
Your Data Center:
  ┌─────────────────────────────────────────┐
  │        AWS Outpost Rack/Server          │
  │  ┌──────────┐  ┌──────────┐            │
  │  │   EC2    │  │   RDS    │            │
  │  │instances │  │(Outposts)│            │
  │  └──────────┘  └──────────┘            │
  │     (on AWS hardware, your location)   │
  └─────────────────┬───────────────────────┘
                    │ Service Link (HTTPS)
                    │ (connects to AWS Region)
          ┌─────────▼────────────────┐
          │      AWS Region          │
          │  (control plane, IAM,    │
          │   console, CloudTrail)   │
          └──────────────────────────┘
```

**Key Points:**
- **Service Link**: encrypted connection from Outpost to parent AWS Region (for control plane).
- **Local Gateway (LGW)**: connects Outpost to your on-premises network.
- **Management**: done from AWS console/CLI just like cloud resources.
- **Internet**: Outposts do NOT come with internet access — you provide connectivity.

---

### Outposts Connectivity Requirements

| Requirement | Details |
|---|---|
| **Network** | Reliable, low-latency connection to parent AWS Region |
| **Bandwidth** | Minimum 1 Gbps recommended |
| **Latency** | < 100 ms to parent Region |
| **Power** | AWS specifies power and cooling requirements (you provide) |
| **Physical security** | You provide (your data center) |

---

### S3 on Outposts

- **Object storage running entirely on your Outpost** — data never leaves your premises.
- Separate service from S3 in the cloud (different endpoint, different buckets).
- **S3 Access Points** required (no public internet access by default).
- Use case: local application data that must remain on-premises.

---

### Outposts vs Local Zones vs Wavelength

| Feature | Outposts | Local Zones | Wavelength |
|---|---|---|---|
| **Location** | Your data center / facility | AWS-operated facility near cities | Telecom carrier network |
| **Who manages hardware** | AWS manages (your premises) | AWS manages | AWS manages |
| **Latency** | Single-digit ms (local network) | < 10 ms to city users | < 10 ms to 5G mobile users |
| **Use case** | Data residency, hybrid on-prem | City-level low latency | 5G mobile edge |
| **AWS services** | Selected services | Selected services | Selected services |
| **Internet access** | You provide | AWS provides | Via carrier |

---

### Key Outposts Exam Scenarios

| Scenario | Answer |
|---|---|
| Data residency laws require data to stay on-premises | AWS Outposts |
| Run AWS services in your factory for low-latency OT/IT integration | AWS Outposts (Servers for small spaces) |
| Use same AWS APIs on-premises and in the cloud | AWS Outposts |
| Migrate off-premises gradually while some workloads stay local | AWS Outposts |
| Single-digit millisecond latency to on-premises systems | AWS Outposts |
| Small retail locations needing local AWS compute | AWS Outposts Servers (1U/2U) |

---

<a name="vmware"></a>
## 6. VMware Cloud on AWS

### What Problem Does It Solve?
Organizations have massive investments in VMware infrastructure — thousands of VMs managed by vSphere, vSAN, NSX, and vCenter. Migrating these to AWS would require refactoring or re-platforming all workloads — an expensive multi-year effort. VMware Cloud on AWS lets organizations **run their existing VMware workloads on dedicated AWS infrastructure** without any refactoring, while gaining access to native AWS services.

**Real-World Example:**
> A global insurance company runs 5,000 VMs on VMware in their own data center. Their 3-year hardware refresh is due. Instead of buying new servers, they move to VMware Cloud on AWS — their vSphere VMs migrate (live migration via HCX) with zero code changes, vCenter looks exactly the same, and now they can access RDS, S3, Lambda directly from their VMware VMs with low-latency networking.

---

### VMware Cloud on AWS Architecture

```
VMware Cloud on AWS (SDDC):
  ┌────────────────────────────────────────────┐
  │         VMware Software-Defined Data        │
  │         Center (SDDC) on AWS Bare Metal     │
  │                                             │
  │  ┌──────────┐ ┌──────────┐ ┌──────────┐   │
  │  │ vSphere  │ │  vSAN    │ │   NSX    │   │
  │  │(VMs run) │ │(storage) │ │(network) │   │
  │  └──────────┘ └──────────┘ └──────────┘   │
  │            managed by vCenter              │
  └──────────────────┬─────────────────────────┘
                     │ High-bandwidth, low-latency
                     │ (ENI, same VPC)
          ┌──────────▼──────────────────────┐
          │        AWS Native Services       │
          │  (S3, RDS, Lambda, DynamoDB,     │
          │   ELB, Direct Connect, etc.)     │
          └─────────────────────────────────┘
```

---

### Key Features

| Feature | Description |
|---|---|
| **SDDC (Software-Defined Data Center)** | VMware stack (vSphere + vSAN + NSX) running on dedicated AWS bare metal i3 instances |
| **HCX (Hybrid Cloud Extension)** | Live migrate VMs between on-premises VMware and VMware Cloud on AWS |
| **vCenter** | Same vCenter interface — no retraining for VMware admins |
| **AWS Direct Connect** | Low-latency connection between SDDC and AWS services |
| **Elastic DRS** | Dynamically scale host count based on VM demand |
| **Stretch Cluster** | Span SDDC across two AZs for HA |

---

### Migration Path with HCX

```
On-Premises VMware VMs
        ↓ (HCX — live migration, no downtime)
VMware Cloud on AWS SDDC
  ├── vSphere VMs run unchanged
  ├── vCenter manages everything
  └── NSX provides networking
        ↓ (gradually modernize)
AWS Native Services (refactor VMs to containers, Lambda, RDS)
```

---

### VMware Cloud on AWS vs AWS Outposts

| Feature | VMware Cloud on AWS | AWS Outposts |
|---|---|---|
| **Where it runs** | AWS data center | Your data center |
| **Hypervisor** | VMware vSphere | AWS-native (Nitro) |
| **Management** | VMware + AWS console | AWS console |
| **Migration tool** | VMware HCX | AWS MGN (Application Migration Service) |
| **Use case** | Lift VMware VMs to AWS | Run AWS on-premises |
| **Data residency** | Data in AWS | Data in your facility |

---

### Key VMware Cloud on AWS Exam Scenarios

| Scenario | Answer |
|---|---|
| Migrate VMware VMs to AWS without refactoring | VMware Cloud on AWS |
| Data center hardware refresh, keep VMware operations | VMware Cloud on AWS |
| Live migrate VMs from on-premises vSphere to AWS | VMware Cloud on AWS with HCX |
| VMware admins continue using vCenter after migration | VMware Cloud on AWS |
| Access RDS and S3 from VMware VMs with low latency | VMware Cloud on AWS (direct ENI connection to AWS services) |

---

<a name="wavelength"></a>
## 7. AWS Wavelength

### What Problem Does It Solve?
5G networks deliver extremely high bandwidth and ultra-low latency to mobile devices — but only if the application server is physically close to the 5G base station. Traditional cloud regions are 50–100ms away from end users. Wavelength **embeds AWS compute and storage at the edge of 5G carrier networks** — enabling applications with single-digit millisecond latency to 5G devices.

**Real-World Example:**
> An autonomous vehicle company needs to process LIDAR sensor data from vehicles in under 10ms (too slow to round-trip to us-east-1). With Wavelength on Verizon's 5G network, compute runs at the 5G base station — LIDAR data is processed in 2ms, enabling real-time collision avoidance. The Wavelength Zone connects to AWS Region us-east-1 for non-latency-critical data.

---

### Wavelength Architecture

```
5G Mobile Device (car, phone, IoT)
        ↓ (5G Radio Network — sub-1ms)
Carrier Network (Verizon, AT&T, KDDI, SK Telecom)
        ↓ (same carrier network — no internet hop)
AWS Wavelength Zone (at carrier data center)
  ├── EC2 instances
  ├── ECS containers
  └── EBS storage
        ↓ (private connection)
AWS Parent Region (us-east-1, etc.)
  └── S3, RDS, Lambda, etc. (non-latency-critical)
```

---

### Wavelength Key Concepts

| Concept | Description |
|---|---|
| **Wavelength Zone** | AWS compute + storage at carrier location |
| **Carrier IP** | IP address routable within the carrier network (not internet) |
| **Carrier Gateway** | Connects Wavelength Zone to carrier network AND AWS Region |
| **Parent Region** | Full AWS Region that Wavelength Zone extends (e.g., us-east-1) |
| **VPC extension** | Wavelength Zone is a subnet extension of your VPC |

---

### Supported Carriers (Global)

| Region | Carriers |
|---|---|
| **US** | Verizon, AT&T, T-Mobile |
| **Japan** | KDDI, Softbank |
| **South Korea** | SK Telecom, KT |
| **UK** | Vodafone |
| **Germany** | Vodafone |
| **Canada** | Bell, Rogers |

---

### Use Cases for Wavelength

| Use Case | Why Wavelength |
|---|---|
| **Autonomous vehicles** | Real-time sensor processing at 5G edge |
| **AR/VR on mobile** | Sub-10ms render updates |
| **Live video game streaming** | Ultra-low latency gameplay |
| **Smart factory / Industry 4.0** | Real-time machine control over 5G |
| **Real-time media processing** | Live video transcode at edge |
| **Connected healthcare** | Real-time patient monitoring |

---

### Wavelength vs Local Zones vs Outposts

| Feature | Wavelength | Local Zones | Outposts |
|---|---|---|---|
| **Location** | Carrier 5G network | AWS-managed city facility | Your facility |
| **Latency** | < 10ms to 5G devices | < 10ms to city users | Single-digit ms to on-prem |
| **Use case** | 5G mobile applications | Low latency to metro users | Data residency + hybrid |
| **Internet access** | Via carrier IP | Yes | You provide |
| **Target user** | 5G mobile devices | Metro users (any device) | On-premises systems |

---

### Key Wavelength Exam Scenarios

| Scenario | Answer |
|---|---|
| Ultra-low latency application for 5G mobile users | AWS Wavelength |
| Process real-time data from connected vehicles on 5G | AWS Wavelength |
| AR/VR gaming with < 10ms latency for mobile | AWS Wavelength |
| Deploy EC2 compute at Verizon's 5G network edge | AWS Wavelength Zone |
| Reduce latency for users in a specific metro area | Local Zones (not Wavelength, if not 5G-specific) |

---

<a name="sar"></a>
## 8. AWS Serverless Application Repository (SAR)

### What Problem Does It Solve?
Teams often rebuild the same serverless patterns from scratch — image resizing Lambdas, API backends, Cognito auth flows, SQS processors. The Serverless Application Repository provides a **managed repository of serverless applications** that developers can discover, configure, and deploy in minutes — or publish their own for others to reuse.

**Real-World Example:**
> A developer needs to add image thumbnail generation to their app. Instead of writing Lambda code from scratch, they search SAR for "image resizing", find an AWS-published application that does exactly that, configure the S3 bucket name parameter, deploy it with one click — done in 5 minutes instead of days of development.

---

### SAR Key Concepts

| Concept | Description |
|---|---|
| **Application** | A packaged serverless app (SAM template + code + Lambda functions) |
| **SAM (Serverless Application Model)** | AWS framework for defining serverless apps (extension of CloudFormation) |
| **Publisher** | Developer or organization that publishes an app |
| **Consumer** | Developer that deploys an app from the repository |
| **Nested application** | Reference an SAR app from within another SAM template |

---

### SAR Application Visibility

| Visibility | Description |
|---|---|
| **Private** | Only your account can deploy |
| **Shared (specific accounts)** | Share with specific AWS account IDs |
| **Public** | Anyone can discover and deploy |

---

### Publishing to SAR

Requirements to publish:
1. Write Lambda function(s).
2. Define infrastructure with **AWS SAM template** (`.yaml`).
3. Write `README.md` and `LICENSE.txt`.
4. Package with `sam package`.
5. Publish with `aws serverlessrepo publish-application`.

---

### SAR vs GitHub / Docker Hub

| Feature | SAR | GitHub | Docker Hub |
|---|---|---|---|
| **Content** | Serverless apps (SAM) | Any code | Container images |
| **Deployment** | One-click deploy to AWS | Manual | Via container orchestration |
| **IAM integration** | ✅ Deploy with IAM | ❌ | Limited |
| **AWS native** | ✅ | ❌ | ❌ |
| **Versioning** | Semantic versioning | Git | Image tags |

---

### Key SAR Exam Scenarios

| Scenario | Answer |
|---|---|
| Quickly deploy a pre-built Lambda-based image resizer | AWS Serverless Application Repository |
| Share a serverless architecture pattern across teams | Publish to SAR (private visibility) |
| Reuse community-built serverless solutions | SAR public applications |
| Embed an SAR app inside a larger CloudFormation stack | SAR nested application reference |

---

<a name="comparisons"></a>
## Master Comparisons

### Compute Service Selection by Control Level

```
MOST CONTROL ←────────────────────────────────────────→ LEAST CONTROL

EC2 (bare metal)  EC2   Beanstalk   Batch    Lambda    SAR
(dedicated host) (VM)  (PaaS)   (managed  (function (pre-built
                                  batch)    as a      apps)
                                           service)
```

### On-Premises / Hybrid Decision

| Need | Service |
|---|---|
| Run AWS services on-prem, data residency | **Outposts** |
| Migrate VMware VMs to AWS without refactoring | **VMware Cloud on AWS** |
| 5G ultra-low latency mobile apps | **Wavelength** |
| Run EKS/ECS on-prem | **EKS Anywhere / ECS Anywhere** |
| Low-latency to metro city users | **Local Zones** |

### Batch Processing Decision

```
Job duration > 15 min OR GPU OR hundreds of parallel jobs?
  └─→ AWS Batch

Job duration < 15 min AND event-driven?
  └─→ Lambda

Need managed queue + auto-scaling workers?
  └─→ Beanstalk Worker Tier (SQS-backed)
      or AWS Batch
```

### EC2 Cost Optimization Decision

```
100% uptime, predictable workload (3 years):
  └─→ Reserved Instances (All Upfront, 3-year) → up to 72% savings

100% uptime, want flexibility (can change instance type):
  └─→ Compute Savings Plans → up to 66% savings

Fault-tolerant, flexible timing:
  └─→ Spot Instances → up to 90% savings

Short-term, unpredictable:
  └─→ On-Demand

Compliance, Oracle BYOL:
  └─→ Dedicated Hosts
```

---

<a name="patterns"></a>
## Architecture Patterns to Know

### Pattern 1: Highly Available Web Application with Auto Scaling
```
Route 53 (DNS + health check)
        ↓
CloudFront (CDN + WAF)
        ↓
ALB (multi-AZ)
        ↓
EC2 Auto Scaling Group (Target Tracking: CPU 50%)
  ├── us-east-1a: EC2 instances
  ├── us-east-1b: EC2 instances
  └── us-east-1c: EC2 instances
        ↓
RDS Multi-AZ (Aurora)  +  ElastiCache Redis
```

### Pattern 2: Batch Processing Pipeline
```
S3 (input data: 10,000 files uploaded)
        ↓
S3 Event → Lambda (submit Batch jobs)
        ↓
AWS Batch Job Queue (priority: normal)
        ↓
Managed Compute Environment (Spot + On-Demand mix)
        ↓
Batch jobs process each file in parallel (Docker containers)
        ↓
Results → S3 output bucket
        ↓
SNS notification on completion
```

### Pattern 3: Hybrid Cloud with Outposts
```
On-Premises (Data Center — Germany):
  ┌────────────────────────────────┐
  │      AWS Outposts Rack         │
  │  EC2 + RDS + ECS + EBS         │
  │  (patient data stays local)    │
  └──────────┬─────────────────────┘
             │ Service Link (HTTPS)
             │ Direct Connect
  ┌──────────▼─────────────────────────┐
  │        AWS Region (Frankfurt)       │
  │  S3 + Lambda + CloudWatch + IAM    │
  │  (non-sensitive workloads)          │
  └─────────────────────────────────────┘
```

### Pattern 4: VMware Migration Path
```
Phase 1 (Month 1-3): Lift and Shift
  On-prem VMware → VMware Cloud on AWS (HCX live migration)
  └── vCenter works the same; ops team unchanged

Phase 2 (Month 4-12): Optimize
  VMware VMs running on AWS + Direct Connect to AWS services
  └── Start using RDS, S3, Lambda from existing VMs

Phase 3 (Year 2+): Modernize (optional)
  Containerize workloads → ECS/EKS
  Refactor to serverless → Lambda
  Data layer → Aurora, DynamoDB
```

### Pattern 5: 5G Edge Application
```
Autonomous Vehicle (5G sensor data, 50ms SLA)
        ↓ (5G radio, <1ms)
Verizon 5G Network
        ↓ (carrier network, no internet hop)
AWS Wavelength Zone (at Verizon data center)
  └── EC2 g4dn.xlarge (GPU inference)
  └── Real-time object detection (2ms processing)
        ↓ (private connection)
AWS Region (us-east-1)
  └── S3 (store telemetry)
  └── SageMaker (retrain models)
  └── Kinesis (aggregate fleet data)
```

---

<a name="exam-tips"></a>
## Exam Tips Summary

### Amazon EC2
- ✅ Instance families: T=burstable, C=compute, R/X=memory, I/D/H=storage, P/G=GPU
- ✅ Cluster placement = lowest latency; Spread = max isolation; Partition = rack-aware (Kafka/Hadoop)
- ✅ Spot = up to 90% off, 2-min interruption warning; use for fault-tolerant batch
- ✅ Dedicated Host = physical server; Dedicated Instance = dedicated hardware (not full host)
- ✅ Stopping an EC2 loses **instance store** data; EBS persists
- ✅ Hibernate saves RAM to EBS; faster resume; root EBS must be encrypted
- ✅ IMDSv2 = session token required for metadata; prevents SSRF attacks
- ✅ EFA = HPC networking with OS-bypass (MPI/NCCL); Enhanced Networking = up to 100 Gbps

### EC2 Auto Scaling
- ✅ Target Tracking = simplest; Step Scaling = graduated; Scheduled = time-based; Predictive = ML-based proactive
- ✅ Lifecycle Hooks: Pending:Wait (before InService) and Terminating:Wait (before termination)
- ✅ Launch Template preferred over Launch Configuration (versioning + Spot+On-Demand mix)
- ✅ ASG automatically registers/deregisters instances with ALB target group
- ✅ Cooldown default = 300 seconds; prevents rapid scaling oscillation

### Elastic Beanstalk
- ✅ PaaS: upload code → Beanstalk manages EC2, ALB, ASG, VPC, monitoring
- ✅ Web tier = HTTP requests; Worker tier = SQS background processing
- ✅ Immutable deployment = new ASG, no downtime, fastest rollback
- ✅ Rolling with additional batch = full capacity throughout deployment
- ✅ .ebextensions = customize instance/platform configuration in deploy bundle
- ✅ No extra charge for Beanstalk itself (pay only for underlying resources)

### AWS Batch
- ✅ Job → Job Queue → Compute Environment → runs Docker containers
- ✅ Managed CE = AWS provisions/scales EC2; Unmanaged = you manage EC2
- ✅ Array Jobs = run N parallel copies of same job definition
- ✅ Job dependencies = sequential execution chains
- ✅ Batch vs Lambda: Batch = long-running (no timeout), GPU, hundreds of parallel; Lambda = < 15 min event-driven
- ✅ Spot + retry strategy = most cost-effective batch architecture

### AWS Outposts
- ✅ AWS-managed hardware in YOUR data center
- ✅ Service Link = control plane connection to parent AWS Region
- ✅ Local Gateway (LGW) = connects Outpost to your on-prem network
- ✅ Use for: data residency, latency to local systems, hybrid AWS APIs
- ✅ Outposts Rack (full rack) vs Outposts Servers (1U/2U for small spaces)
- ✅ S3 on Outposts = object storage that stays on-prem

### VMware Cloud on AWS
- ✅ VMware SDDC (vSphere + vSAN + NSX) on dedicated AWS bare metal
- ✅ HCX = live migration tool from on-prem to VMware Cloud on AWS
- ✅ vCenter interface unchanged; VMware admins need no retraining
- ✅ Direct ENI connection to AWS native services (S3, RDS, Lambda)
- ✅ Use when: data center hardware refresh, VMware workloads, no code refactoring
- ✅ VMware Cloud ≠ Outposts: VMware Cloud = runs in AWS DC; Outposts = runs in your DC

### AWS Wavelength
- ✅ AWS compute + storage embedded at telecom 5G carrier network
- ✅ Carrier IP = routable within carrier network (not public internet)
- ✅ Wavelength Zone = subnet extension of your VPC at carrier location
- ✅ Use for: 5G mobile apps, autonomous vehicles, AR/VR, real-time gaming
- ✅ Wavelength ≠ Local Zones: Wavelength = 5G carrier edge; Local Zones = city-level metro
- ✅ Wavelength ≠ Outposts: Wavelength = 5G network; Outposts = your data center

### Serverless Application Repository (SAR)
- ✅ Repository of pre-built serverless apps (Lambda + SAM templates)
- ✅ Visibility: private / shared (specific accounts) / public
- ✅ Nested applications = embed SAR app in your SAM/CloudFormation template
- ✅ No additional charge for SAR (pay only for deployed resources)
- ✅ Use SAR to share reusable serverless patterns across teams or with the community

---
---

<a name="containers"></a>
# PART 2 — Containers: ECS, EKS, ECR, ECS Anywhere, EKS Anywhere & EKS Distro

# AWS Exam Study Guide: Containers
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Amazon ECS · Amazon EKS · Amazon ECR · ECS Anywhere · EKS Anywhere · EKS Distro

---

## Container Services Quick-Reference Matrix

| Service | Where It Runs | Orchestrator | Manages Infra? | Exam Trigger Words |
|---|---|---|---|---|
| **Amazon ECS** | AWS Cloud | AWS proprietary | ✅ Fully managed | "containers on AWS", "ECS tasks", "Fargate", "no K8s" |
| **Amazon EKS** | AWS Cloud | Kubernetes | ✅ Control plane managed | "Kubernetes", "K8s", "open source orchestration" |
| **Amazon ECR** | AWS Cloud | Registry (not orchestrator) | ✅ Fully managed | "container registry", "store Docker images", "ECR" |
| **ECS Anywhere** | On-prem / other cloud | AWS ECS | ❌ You manage nodes | "run ECS on-premises", "hybrid containers" |
| **EKS Anywhere** | On-prem / other cloud | Kubernetes | ❌ You manage infra | "K8s on-prem", "EKS on bare metal", "hybrid K8s" |
| **EKS Distro** | Anywhere | Kubernetes | ❌ Fully self-managed | "K8s distribution", "DIY K8s", "same as EKS upstream" |

---

## Container Deployment Spectrum

```
MOST MANAGED ←─────────────────────────────────────────→ LEAST MANAGED

ECS + Fargate    ECS + EC2    EKS + Fargate    EKS + EC2    ECS/EKS Anywhere    EKS Distro
(zero infra)   (manage EC2)  (K8s no nodes)  (manage nodes)  (manage own HW)   (DIY everything)
```

---

---

## 1. Amazon ECS (Elastic Container Service)

### What Problem Does It Solve?
Running containers in production requires scheduling (which host runs which container?), health monitoring, auto-scaling, networking, load balancing, and service discovery. Amazon ECS is AWS's **native container orchestration service** that handles all of this — letting you focus on your application, not infrastructure.

**Real-World Example:**
> A SaaS company runs 30 microservices as Docker containers. ECS automatically schedules containers across a cluster, restarts failed containers, integrates with ALB for routing, scales service instances up/down based on CPU, and manages secrets injection from Secrets Manager — all without a single line of orchestration code.

---

### ECS Core Concepts

#### Task Definition
- A **blueprint** for your container application (JSON/YAML).
- Defines: Docker image, CPU/memory, port mappings, IAM roles, environment variables, logging, storage volumes, networking mode.
- **Immutable** once registered — new versions create new revisions.
- One task definition can define **multiple containers** (e.g., app + sidecar logger).

```json
{
  "family": "web-app",
  "containerDefinitions": [
    {
      "name": "web",
      "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/web-app:1.2.3",
      "cpu": 256,
      "memory": 512,
      "portMappings": [{ "containerPort": 80, "hostPort": 80 }],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/web-app",
          "awslogs-region": "us-east-1"
        }
      }
    }
  ],
  "taskRoleArn": "arn:aws:iam::123456789:role/ecsTaskRole",
  "executionRoleArn": "arn:aws:iam::123456789:role/ecsExecutionRole",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512"
}
```

#### Task
- A **running instance** of a task definition.
- Can be standalone (one-off job) or part of a Service.
- Has its own IP (in awsvpc mode) and lifecycle.

#### Service
- Maintains a **desired number of running tasks** from a task definition.
- Handles: task placement, health check replacement, rolling updates, integration with ALB/NLB.
- Types:
  - **REPLICA**: maintain N copies across cluster
  - **DAEMON**: one task per container instance (like DaemonSet in K8s)

#### Cluster
- **Logical grouping** of tasks and services.
- Contains: capacity providers (Fargate, EC2 ASG), service definitions.
- One cluster can have both Fargate and EC2 launch types.

---

### ECS Launch Types

#### Fargate Launch Type (Serverless)
- AWS manages **all underlying infrastructure** — no EC2 to provision, patch, or scale.
- You define CPU and memory per task; AWS runs it on managed compute.
- **Billing**: per-second based on vCPU + memory allocated.
- **Best for**: variable workloads, teams without ops expertise, microservices.

```
Task → Fargate → AWS manages where/how it runs
       (you never see the host)
```

#### EC2 Launch Type
- You provision and manage **EC2 instances** in the cluster.
- ECS Agent runs on each EC2 and reports to the ECS control plane.
- **Billing**: per-hour EC2 instance cost (regardless of task utilization).
- **Best for**: GPU workloads, Windows containers requiring specific instance families, cost optimization for steady-state workloads, spot instances.

```
Task → ECS Scheduler → EC2 Instance (in your cluster)
       (you manage EC2 patching, scaling, capacity)
```

#### External Launch Type (ECS Anywhere)
- Run ECS tasks on **on-premises or external infrastructure**.
- Discussed in detail in the ECS Anywhere section.

---

### ECS Fargate vs EC2 Launch Type

| Feature | Fargate | EC2 Launch Type |
|---|---|---|
| **Infrastructure management** | None (AWS manages) | You manage EC2 instances |
| **Scaling** | Per-task auto-scaling | EC2 + task scaling |
| **Cost model** | Per vCPU + memory/sec | Per EC2 instance-hour |
| **GPU support** | ❌ | ✅ |
| **Windows containers** | ✅ | ✅ |
| **Spot pricing** | ✅ Fargate Spot | ✅ EC2 Spot Instances |
| **Host access** | ❌ | ✅ SSH/SSM to EC2 |
| **Network mode** | awsvpc only | bridge, host, awsvpc |
| **Best for** | Spiky/unpredictable workloads | Steady-state, cost-optimized, GPU |

---

### ECS Networking Modes

| Mode | Description | Used With |
|---|---|---|
| **awsvpc** | Each task gets its own ENI and private IP | Fargate (required), EC2 (recommended) |
| **bridge** | Uses Docker bridge network; port mapping via NAT | EC2 only |
| **host** | Container shares host network namespace | EC2 only; maximum performance |
| **none** | No networking | EC2 only; batch jobs with no network |

> **Exam Tip:** Fargate requires **awsvpc** mode. Only awsvpc supports **task-level security groups** (SGs applied per task, not per instance).

---

### ECS IAM Roles — Two Critical Roles

| Role | Who Uses It | Purpose |
|---|---|---|
| **Task Execution Role** | ECS Agent (infra) | Pull image from ECR, push logs to CloudWatch, get secrets from Secrets Manager/SSM |
| **Task Role** | Your application code | Call DynamoDB, S3, SQS, etc. from within the container |

> **Exam Trap:** Getting confused between these two is common. "Container needs to read from S3" → **Task Role**. "ECS needs to pull image from private ECR" → **Task Execution Role**.

---

### ECS Service Auto Scaling

- Uses **Application Auto Scaling** (not EC2 ASG directly).
- Scaling metrics:
  - **ECSServiceAverageCPUUtilization** — average CPU across all tasks
  - **ECSServiceAverageMemoryUtilization** — average memory
  - **ALBRequestCountPerTarget** — requests per task via ALB
- Policies: Target Tracking, Step Scaling, Scheduled.

**For EC2 Launch Type — Two-Layer Scaling:**
```
Layer 1: ECS Service Auto Scaling (scale number of tasks)
Layer 2: EC2 ASG (scale number of EC2 instances — Cluster Auto Scaling / Capacity Provider)
```

---

### ECS Capacity Providers

- Define **where tasks run** within a cluster.
- **Managed scaling**: automatically adjusts EC2 ASG based on task demand.
- Types:
  - `FARGATE` — managed Fargate capacity
  - `FARGATE_SPOT` — Spot Fargate (up to 70% cheaper; can be interrupted)
  - Custom EC2 ASG capacity provider

---

### ECS Service Discovery

- ECS integrates with **AWS Cloud Map** for service-to-service discovery.
- Each service gets a DNS name (e.g., `payment.local`).
- Tasks register themselves on startup; deregister on shutdown.
- Works within a VPC using Route 53 private hosted zones.

---

### ECS Logging — awslogs Driver

```json
"logConfiguration": {
  "logDriver": "awslogs",
  "options": {
    "awslogs-group": "/ecs/my-service",
    "awslogs-region": "us-east-1",
    "awslogs-stream-prefix": "ecs"
  }
}
```

- Logs sent directly to **CloudWatch Logs**.
- FireLens: alternative log router using Fluentd/Fluent Bit (send to S3, Kinesis, OpenSearch).

---

### ECS Task Placement Strategies (EC2 Launch Type Only)

| Strategy | Description | Use Case |
|---|---|---|
| **Spread** | Distribute tasks evenly across AZs or instances | High availability |
| **Binpack** | Pack tasks on fewest instances (fill one before next) | Cost optimization |
| **Random** | Random placement | No preference |

**Task Placement Constraints:**
- `memberOf`: restrict to instances matching an attribute (e.g., only GPU instances)
- `distinctInstance`: each task on a different EC2 instance

---

### Key ECS Exam Scenarios

| Scenario | Answer |
|---|---|
| Run containers without managing servers | ECS + Fargate |
| Run containers on EC2 Spot for cost savings | ECS + EC2 launch type + Spot Instances |
| One monitoring agent per EC2 instance in ECS cluster | ECS Service with DAEMON scheduling strategy |
| Task needs to call DynamoDB | Task Role on task definition |
| ECS can't pull image from ECR | Check Task Execution Role permissions |
| Scale containers based on ALB request count | ECS Service Auto Scaling with ALBRequestCountPerTarget |
| Container A and B must run together on same host | Multi-container task definition |
| Run ECS tasks on-premises | ECS Anywhere |
| Fargate task needs persistent shared storage | Mount Amazon EFS volume |
| Blue/green deployment on ECS | ECS + CodeDeploy (blue/green deployment type) |

---

---

## 2. Amazon EKS (Elastic Kubernetes Service)

### What Problem Does It Solve?
Kubernetes is the industry-standard open-source container orchestration platform — but running Kubernetes yourself requires managing the control plane (API server, etcd, scheduler, controllers) which is complex, expensive, and operationally demanding. Amazon EKS provides a **fully managed Kubernetes control plane** so you get all of Kubernetes without managing the master nodes.

**Real-World Example:**
> A technology company already uses Kubernetes on-premises and wants to move to AWS without re-training their engineers or rewriting deployment manifests. EKS gives them the exact same Kubernetes API they know — AWS manages the control plane HA across 3 AZs — and they can run worker nodes on EC2 or Fargate, keeping all their existing Helm charts and kubectl workflows.

---

### EKS Architecture

```
AWS-Managed Control Plane (you never see these):
  ├── kube-apiserver (HA across 3 AZs)
  ├── etcd (distributed KV store, HA)
  ├── kube-scheduler
  ├── kube-controller-manager
  └── cloud-controller-manager

Your Account (you manage these):
  └── Worker Nodes (data plane):
      ├── EC2 Managed Node Groups (AWS manages node lifecycle)
      ├── Self-managed EC2 nodes (you manage everything)
      └── Fargate Profiles (serverless pods)
```

---

### EKS Node Types

#### 1. Managed Node Groups
- AWS creates, patches, and manages EC2 instances as Kubernetes worker nodes.
- **Rolling updates**: AWS drains nodes before terminating during updates.
- Supports: On-Demand and Spot Instances.
- Uses **Launch Templates** for customization.
- `eksctl` or console/CloudFormation to create.

#### 2. Self-Managed Nodes
- You provision EC2 instances and register them with the EKS cluster.
- Use for: custom AMIs, specific networking requirements, custom bootstrap scripts.
- You handle: patching, updates, scaling (via EC2 ASG).

#### 3. Fargate Profiles
- Run **individual Kubernetes pods** on Fargate (serverless).
- Define namespace + label selector → matching pods automatically run on Fargate.
- No nodes to manage; pods are isolated per task.
- Limitations: no DaemonSets, no stateful workloads without EFS, no GPU.

---

### EKS Node Types Comparison

| Feature | Managed Node Group | Self-Managed | Fargate |
|---|---|---|---|
| **Infrastructure** | AWS manages EC2 | You manage EC2 | Serverless |
| **Node patching** | ✅ AWS handles | ❌ You handle | N/A |
| **Custom AMI** | ✅ (via LT) | ✅ Full control | ❌ |
| **GPU support** | ✅ | ✅ | ❌ |
| **DaemonSets** | ✅ | ✅ | ❌ |
| **Spot Instances** | ✅ | ✅ | N/A |
| **Cost model** | Per EC2 hour | Per EC2 hour | Per pod vCPU+memory |

---

### EKS Networking

#### VPC CNI Plugin (aws-vpc-cni)
- Each pod gets an **IP address from the VPC CIDR** (not a virtual overlay network).
- Pods can communicate directly with other VPC resources (RDS, ElastiCache) using VPC IPs.
- Security groups can be applied at the **pod level** (via Security Groups for Pods feature).

#### EKS Cluster Endpoint Access

| Mode | Description | Use Case |
|---|---|---|
| **Public** | kubectl from anywhere on internet | Development (less secure) |
| **Private** | kubectl only from within VPC | Production (most secure) |
| **Public + Private** | Both (whitelist public CIDR) | Balanced; most common |

---

### EKS IAM Integration (IRSA — IAM Roles for Service Accounts)

- Kubernetes **Service Accounts** can be mapped to **IAM roles**.
- Pods using that Service Account automatically get temporary IAM credentials.
- Eliminates need for long-term access keys in pod environment variables.

```yaml
# Kubernetes Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-reader
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789:role/S3ReadOnlyRole
```

Pods using `s3-reader` service account automatically can read S3 — via temporary STS credentials.

> **Exam Tip:** IRSA (IAM Roles for Service Accounts) is the recommended way to grant AWS permissions to Kubernetes pods. It uses OIDC (OpenID Connect) federation between EKS and IAM.

---

### EKS Add-Ons (Managed)

| Add-On | Purpose |
|---|---|
| **CoreDNS** | Kubernetes DNS resolution |
| **kube-proxy** | Network rules on nodes |
| **VPC CNI** | Pod networking |
| **AWS Load Balancer Controller** | Provision ALB/NLB from Ingress/Service resources |
| **EBS CSI Driver** | Mount EBS volumes as PersistentVolumes |
| **EFS CSI Driver** | Mount EFS as PersistentVolumes |
| **Secrets Store CSI** | Mount Secrets Manager/SSM params as volumes |

---

### EKS with Fargate — Important Limitations

- ❌ **No DaemonSets** — Fargate doesn't support DaemonSets
- ❌ **No privileged containers**
- ❌ **No GPU** support
- ❌ **No EBS volumes** (use EFS for persistent storage)
- ✅ **EFS** is supported for persistent storage
- ✅ Each pod is **fully isolated** on its own compute (security boundary)

---

### EKS Cluster Autoscaler vs Karpenter

| Feature | Cluster Autoscaler | Karpenter |
|---|---|---|
| **What it scales** | EC2 ASG node groups | Provisions nodes directly (any instance type) |
| **Speed** | Minutes | Seconds |
| **Instance selection** | Limited to ASG config | Any EC2 instance (best fit) |
| **Cost optimization** | Basic | ✅ Advanced (right-sizes per pod) |
| **AWS native** | Open source | ✅ AWS-native open source |

> **Exam Tip:** **Karpenter** is the modern, faster replacement for Cluster Autoscaler on EKS. Karpenter directly provisions and deprovisions EC2 nodes — no ASG needed.

---

### Key EKS Exam Scenarios

| Scenario | Answer |
|---|---|
| Managed Kubernetes control plane in AWS | Amazon EKS |
| Run Kubernetes pods without managing nodes | EKS + Fargate Profiles |
| Give a K8s pod access to S3 without access keys | IRSA (IAM Roles for Service Accounts) |
| Provision ALB from Kubernetes Ingress resource | AWS Load Balancer Controller add-on |
| Persistent storage for stateful pods on EKS | EBS CSI Driver (RWO) or EFS CSI Driver (RWX) |
| Fast node provisioning based on pod requirements | Karpenter |
| Lock down kubectl access to VPC only | EKS private cluster endpoint |
| Use existing on-prem Kubernetes clusters in AWS console | EKS Anywhere (or EKS Connector) |

---

---

## 3. Amazon ECS vs Amazon EKS

> **The most important comparison for the exam.**

| Dimension | Amazon ECS | Amazon EKS |
|---|---|---|
| **Orchestrator** | AWS proprietary (ECS) | Kubernetes (open source) |
| **Kubernetes API** | ❌ | ✅ Full K8s compatibility |
| **Learning curve** | Lower | Higher (need K8s knowledge) |
| **Operational complexity** | Lower | Higher |
| **Fargate support** | ✅ | ✅ (Fargate Profiles) |
| **DaemonSet equivalent** | DAEMON service | DaemonSet |
| **Portability** | AWS-specific | ✅ Multi-cloud (K8s standard) |
| **Ecosystem** | AWS tools (CloudFormation, CDK) | K8s ecosystem (Helm, ArgoCD, etc.) |
| **Service mesh** | AWS App Mesh | Istio, Linkerd, AWS App Mesh |
| **GPU workloads** | ✅ (EC2) | ✅ (EC2 nodes) |
| **Cost** | No control plane cost | ~$0.10/hr per cluster (control plane) |
| **Use case** | AWS-native teams; simpler | K8s expertise; multi-cloud; complex workloads |

> **Exam Rule:**
> - "Already using Kubernetes / need K8s compatibility" → **EKS**
> - "Simplest managed container service on AWS" → **ECS**
> - "Serverless containers without any server management" → **ECS + Fargate** or **EKS + Fargate**

---

---

## 4. Amazon ECR (Elastic Container Registry)

### What Problem Does It Solve?
Container images need to be stored, versioned, scanned for vulnerabilities, and securely pulled by container runtimes. ECR is AWS's **fully managed Docker-compatible container image registry** — tightly integrated with ECS, EKS, Lambda, and App Runner.

**Real-World Example:**
> A CI/CD pipeline builds a Docker image for a new microservice version. CodeBuild pushes the image to ECR with a semantic version tag (`v2.1.4`). ECR automatically scans it for CVEs, stores it encrypted, and when the ECS task definition references `123456789.dkr.ecr.us-east-1.amazonaws.com/my-service:v2.1.4`, the task pulls the image privately without internet access.

---

### ECR Tiers

| Tier | Availability | Billing | Access |
|---|---|---|---|
| **Private Registry** | Per-account, per-region | Storage + data transfer | IAM-controlled |
| **Public Registry (ECR Public / Gallery)** | `public.ecr.aws` | Free storage; data transfer from public | Public (no auth) |

---

### ECR Key Features

| Feature | Description |
|---|---|
| **Image scanning** | Scan on push or on-demand for CVE vulnerabilities |
| **Encryption** | At rest with KMS (SSE-KMS); in transit with TLS |
| **Lifecycle policies** | Automatically clean up old/untagged images (reduce storage cost) |
| **Image tagging** | Mutable (default) or Immutable (tag cannot be overwritten) |
| **Cross-region replication** | Automatically replicate images to other regions |
| **Cross-account replication** | Replicate to other AWS accounts |
| **Pull-through cache** | Cache images from upstream registries (Docker Hub, quay.io, K8s registry) |
| **Resource-based policies** | Grant cross-account access to repositories |
| **OCI compliance** | Supports OCI images, Helm charts, other OCI artifacts |

---

### ECR Image Scanning

| Scan Type | Description | Trigger |
|---|---|---|
| **Basic Scanning** | Open-source Clair-based scanning | Manual or on push |
| **Enhanced Scanning** | Amazon Inspector integration; deeper CVE analysis; continuous | Automatic on push + continuous |

> **Exam Tip:** ECR Enhanced Scanning uses **Amazon Inspector** under the hood and provides **continuous scanning** — meaning existing images in ECR are re-evaluated as new CVEs are published.

---

### ECR Lifecycle Policies

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 10 images",
      "selection": {
        "tagStatus": "tagged",
        "tagPrefixList": ["v"],
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": { "type": "expire" }
    },
    {
      "rulePriority": 2,
      "description": "Delete untagged images older than 1 day",
      "selection": {
        "tagStatus": "untagged",
        "countType": "sinceImagePushed",
        "countUnit": "days",
        "countNumber": 1
      },
      "action": { "type": "expire" }
    }
  ]
}
```

---

### ECR Authentication

```bash
# Authenticate Docker to ECR private registry
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin \
  123456789.dkr.ecr.us-east-1.amazonaws.com

# Push an image
docker tag my-app:latest 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
docker push 123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:latest
```

---

### ECR + ECS/EKS Integration

- ECS Task Execution Role must have `ecr:GetDownloadUrlForLayer`, `ecr:BatchGetImage`, `ecr:GetAuthorizationToken`.
- EKS Managed Nodes use EC2 instance role; Fargate pods use pod execution role.
- **VPC Endpoint for ECR**: Pull images privately within VPC without internet (Interface Endpoint).
  - `com.amazonaws.region.ecr.api` — ECR API
  - `com.amazonaws.region.ecr.dkr` — Docker registry protocol
  - Also need S3 Gateway Endpoint (ECR stores layers in S3)

---

### ECR Cross-Region and Cross-Account Replication

```
Source: us-east-1 registry
        ↓ (automatic replication)
Destination: eu-west-1 registry (disaster recovery)
Destination: us-west-2 registry (region closer to some teams)
Destination: Account 987654321 (shared services account for partner teams)
```

---

### Key ECR Exam Scenarios

| Scenario | Answer |
|---|---|
| Store Docker images securely for ECS deployments | Amazon ECR private registry |
| Automatically scan new images for CVEs | ECR Enhanced Scanning (Inspector) |
| Prevent developers from overwriting a tagged production image | ECR Immutable Tags |
| Delete images older than 30 days automatically | ECR Lifecycle Policy |
| Pull images without internet access from private subnet | VPC Interface Endpoints for ECR (+ S3 Gateway Endpoint) |
| Replicate images to eu-west-1 for DR | ECR cross-region replication |
| Share images with a partner AWS account | ECR repository policy (resource-based) |
| Cache Docker Hub images in your ECR | ECR pull-through cache |
| Store Helm charts alongside container images | ECR OCI artifact support |

---

---

## 5. Amazon ECS Anywhere

### What Problem Does It Solve?
Companies have on-premises infrastructure they can't immediately move to the cloud — due to data sovereignty, latency requirements, existing hardware investments, or regulatory constraints. ECS Anywhere lets teams **use the same ECS APIs, task definitions, and tooling** to run containers on **any infrastructure** (on-prem, bare metal, VMs, other clouds) while still managing them from the AWS console and CLI.

**Real-World Example:**
> A hospital has patient monitoring systems that must remain on-premises for HIPAA latency requirements but wants to use a consistent container deployment methodology across on-prem and cloud. ECS Anywhere registers their on-prem servers as ECS external instances — the same `aws ecs run-task` CLI command deploys containers both on-prem and to Fargate in the cloud.

---

### How ECS Anywhere Works

```
On-Premises Server (Linux — any x86_64 or ARM)
        ↓ (install two agents)
  ├── SSM Agent (for secure connectivity back to AWS)
  └── ECS Agent (for task scheduling and reporting)
        ↓ (registers as external instance)
ECS Cluster (with EXTERNAL capacity provider)
        ↓
Deploy tasks using "launch type: EXTERNAL"
        ↓
Tasks run on your on-prem infrastructure, managed via AWS console/CLI/API
```

---

### ECS Anywhere Setup Process

```bash
# 1. Generate activation credentials (in AWS console or CLI)
aws ssm create-activation \
  --iam-role AmazonECSAnywhereIAMRolePolicy \
  --registration-limit 10

# 2. On the on-prem server, run the installation script
curl -O "https://amazon-ecs-agent-packages.s3.amazonaws.com/amazon-ecs-init.rpm"
bash amazon-ecs-anywhere-install.sh \
  --cluster my-cluster \
  --activation-id <activation_id> \
  --activation-code <activation_code> \
  --region us-east-1
```

---

### ECS Anywhere Key Properties

| Property | Details |
|---|---|
| **Supported OS** | Linux only (Amazon Linux 2, Ubuntu, CentOS, RHEL, SUSE) |
| **Connectivity** | SSM (outbound HTTPS only — no inbound ports needed) |
| **Network mode** | `bridge` and `host` only (no awsvpc on external instances) |
| **Fargate** | ❌ Not available on external instances |
| **Service Discovery** | ✅ (AWS Cloud Map) |
| **CloudWatch Logs** | ✅ (via awslogs driver) |
| **Task Execution Role** | ✅ Required (for ECR pulls, Secrets Manager) |
| **IAM** | Outbound HTTPS to AWS APIs |

---

### ECS Anywhere vs EKS Anywhere

| Feature | ECS Anywhere | EKS Anywhere |
|---|---|---|
| **Orchestrator** | AWS ECS | Kubernetes |
| **K8s compatibility** | ❌ | ✅ |
| **Infra complexity** | Lower | Higher |
| **Control plane location** | AWS-managed | Your infrastructure |
| **Supported infra** | Any Linux server | VMware vSphere, bare metal, CloudStack, Nutanix |
| **Best for** | Teams using ECS; simpler needs | K8s expertise; regulatory K8s on-prem |

---

### Key ECS Anywhere Exam Scenarios

| Scenario | Answer |
|---|---|
| Run ECS tasks on-premises with same ECS tooling | ECS Anywhere |
| Manage hybrid container fleet (cloud + on-prem) from one console | ECS Anywhere |
| Data sovereignty — containers must stay on-prem | ECS Anywhere |
| No inbound firewall ports but need AWS management | ECS Anywhere (uses SSM outbound only) |

---

---

## 6. Amazon EKS Anywhere

### What Problem Does It Solve?
Organizations need Kubernetes on-premises for compliance, latency, or data sovereignty reasons — but building and maintaining production-grade Kubernetes on bare metal or VMware is extremely complex. EKS Anywhere provides a **supported, validated Kubernetes distribution** based on EKS that runs on your own infrastructure, using AWS tooling for lifecycle management.

**Real-World Example:**
> A defense contractor must run their ML workload Kubernetes clusters in an air-gapped data center with no internet access. EKS Anywhere provides a fully functional, enterprise-supported Kubernetes cluster with the same EKS-validated components — running entirely on their VMware infrastructure, managed with the `eksctl anywhere` CLI.

---

### EKS Anywhere Infrastructure Options

| Environment | Provider | Notes |
|---|---|---|
| **VMware vSphere** | vSphere 6.7+ | Most common; production-ready |
| **Bare Metal** | Physical servers | Via Tinkerbell (PXE boot) |
| **AWS Snow** | Snowball Edge | Edge computing in disconnected environments |
| **Apache CloudStack** | CloudStack | Enterprise virtualization |
| **Nutanix** | Nutanix AHV | Hyperconverged infrastructure |

---

### EKS Anywhere Key Concepts

| Concept | Description |
|---|---|
| **Cluster Spec** | YAML file defining cluster config (node sizes, K8s version, networking) |
| **Management Cluster** | A bootstrap cluster used to create/manage workload clusters |
| **Workload Cluster** | The actual Kubernetes cluster running your applications |
| **Curated Packages** | AWS-supported add-ons (Harbor, MetalLB, Emissary, Prometheus) |
| **EKS Connector** | Register EKS Anywhere cluster in AWS console for visibility |
| **Bottlerocket** | AWS-hardened Linux OS option for EKS Anywhere nodes |

---

### EKS Anywhere Connectivity Options

| Mode | Description |
|---|---|
| **Connected** | Cluster has internet access; integrates with AWS services (IAM, ECR, CloudWatch) |
| **Disconnected (Air-gapped)** | No internet access; uses local artifact registry; no AWS service integration |

---

### EKS Anywhere vs EKS (Cloud)

| Feature | EKS (AWS Cloud) | EKS Anywhere |
|---|---|---|
| **Control plane** | AWS managed | You manage (on your infra) |
| **Kubernetes version** | AWS managed upgrades | You upgrade (eksctl anywhere upgrade) |
| **Worker nodes** | EC2, Fargate | VMware, bare metal, Snow |
| **AWS service integration** | ✅ Native | ✅ (connected) / ❌ (air-gapped) |
| **Cost** | $0.10/hr per cluster | Hardware cost + EKS Anywhere subscription (optional) |
| **Support** | AWS Support | AWS Support (with subscription) |

---

### Key EKS Anywhere Exam Scenarios

| Scenario | Answer |
|---|---|
| Run Kubernetes on VMware vSphere on-premises | EKS Anywhere (vSphere provider) |
| Air-gapped K8s cluster with no internet | EKS Anywhere (disconnected mode) |
| Kubernetes on AWS Snowball Edge (disconnected) | EKS Anywhere on Snow |
| View on-prem K8s clusters in AWS EKS console | EKS Connector (registers EKS Anywhere cluster) |
| Need enterprise support for on-prem K8s | EKS Anywhere with AWS support subscription |

---

---

## 7. Amazon EKS Distro (EKS-D)

### What Problem Does It Solve?
Organizations that want to run Kubernetes **entirely on their own infrastructure** — without AWS management, subscription, or tooling — but want **the exact same Kubernetes components and versions that power Amazon EKS** (validated, security-patched, tested). EKS Distro is a **downloadable Kubernetes distribution** (not a managed service).

**Real-World Example:**
> A cloud-native startup wants to run Kubernetes in their own co-location data center using their own tooling (Ansible, Terraform, custom scripts). They use EKS Distro to ensure they're running the same validated K8s version that EKS uses in AWS — with AWS's security patches and component versions — so when they eventually migrate to EKS, compatibility is guaranteed.

---

### What EKS Distro Includes

| Component | Description |
|---|---|
| **Kubernetes** | Same version as EKS (API server, scheduler, controller manager, etcd) |
| **AWS patches** | Security patches applied on top of upstream K8s |
| **Container images** | Pre-built and hosted in ECR Public Gallery |
| **Extended support** | Security patches beyond upstream K8s end-of-life |
| **Tested components** | CNI plugins, CoreDNS, metrics-server — version-locked and tested together |

---

### EKS Distro vs EKS Anywhere vs EKS

| Feature | EKS (Cloud) | EKS Anywhere | EKS Distro |
|---|---|---|---|
| **Where it runs** | AWS Cloud | Your infra (VMware, metal) | Anywhere you set it up |
| **Control plane managed** | ✅ AWS | ✅ AWS tooling | ❌ Fully you |
| **Lifecycle management** | AWS | eksctl anywhere | Fully manual |
| **AWS console integration** | ✅ | ✅ (via Connector) | ❌ None |
| **Same K8s as EKS** | ✅ | ✅ | ✅ |
| **Who uses it** | AWS cloud teams | On-prem K8s teams | DIY K8s experts |
| **Cost** | Cluster fee | Hardware + optional support | Free (open source) |
| **Complexity** | Lowest | Medium | Highest |

> **Exam Rule:**
> - "Kubernetes fully managed in AWS" → **EKS**
> - "Kubernetes on-prem with AWS tooling and support" → **EKS Anywhere**
> - "Same K8s components as EKS, run completely yourself" → **EKS Distro**

---

---

## Master Comparison: All Container Services

### Complete Decision Tree

```
Need to run containers?
    │
    ├─→ In AWS Cloud?
    │       │
    │       ├─→ Need Kubernetes?
    │       │       ├─→ Manage worker nodes yourself → EKS + Self-Managed Nodes
    │       │       ├─→ AWS manages node lifecycle → EKS + Managed Node Groups
    │       │       └─→ No nodes at all → EKS + Fargate Profiles
    │       │
    │       └─→ Don't need Kubernetes?
    │               ├─→ No server management → ECS + Fargate
    │               └─→ Control over EC2 → ECS + EC2 Launch Type
    │
    └─→ On-Premises / Hybrid?
            │
            ├─→ Using ECS → ECS Anywhere
            ├─→ Need Kubernetes with AWS tooling → EKS Anywhere
            └─→ DIY Kubernetes, same as EKS → EKS Distro
```

### Image Storage

```
Any container deployment above → Amazon ECR (private registry)
                                → ECR Public Gallery (public images)
```

---

### ECS vs EKS: When to Choose

| Choose ECS When | Choose EKS When |
|---|---|
| Team has no Kubernetes experience | Team already knows Kubernetes |
| Want simplest managed container service | Need Kubernetes API compatibility |
| AWS-only deployment (no multi-cloud plans) | Planning multi-cloud or hybrid |
| Cost is a primary concern (no K8s cluster fee) | Need rich K8s ecosystem (Helm, Istio, ArgoCD) |
| Rapid iteration with Fargate | Complex workloads needing fine-grained K8s control |
| Migrating from Docker Compose | Migrating from on-prem K8s |

---

## Architecture Patterns to Know

### Pattern 1: Complete ECS Microservices Platform
```
GitHub (code + Dockerfile)
        ↓ (CI/CD trigger)
AWS CodeBuild → build Docker image → push to ECR
        ↓
ECR (vulnerability scan on push)
        ↓
CodeDeploy Blue/Green deployment
        ↓
ECS Service (Fargate)
  ├── Tasks in us-east-1a + us-east-1b + us-east-1c
  ├── Task Execution Role → pull from ECR + push to CloudWatch
  ├── Task Role → read/write DynamoDB + publish to SNS
  └── awsvpc mode → task-level Security Groups
        ↓
ALB (path-based routing per microservice)
        ↓
Route 53 → Custom domain → CloudFront → ALB
```

### Pattern 2: EKS Multi-Tenant Platform
```
Developers → kubectl / Helm → EKS Cluster
                                    │
                    ┌───────────────┼────────────────┐
                    ↓               ↓                ↓
              Namespace: team-a  Namespace: team-b  Namespace: team-c
              (IRSA role A)      (IRSA role B)      (IRSA role C)
                    ↓               ↓                ↓
              Managed Node Group (Karpenter auto-provisions right-sized nodes)
                    ↓
              AWS Load Balancer Controller → ALB per namespace
                    ↓
              EBS CSI (RWO stateful) + EFS CSI (RWX shared)
```

### Pattern 3: Hybrid Container Architecture
```
AWS Cloud (ECS Fargate):              On-Premises (ECS Anywhere):
  └── Customer-facing APIs              └── Patient data processing
       └── Low latency, scalable              └── Must stay on-prem (HIPAA)

Both managed from AWS Console/CLI via same ECS API
Same task definitions, same ECR images, same CloudWatch logs
```

### Pattern 4: ECR-Centric Security Pipeline
```
Developer commits Dockerfile
        ↓
CodeBuild: docker build + docker push to ECR
        ↓
ECR Enhanced Scanning (Amazon Inspector)
        ↓
CVE found (severity: HIGH)?
    YES → CodePipeline blocked → SNS alert → developer must fix
    NO  → Proceed to deployment
        ↓
ECS/EKS pulls image from ECR (via VPC Interface Endpoint — no internet)
        ↓
Immutable tags prevent overwriting production image
        ↓
Lifecycle policy: delete untagged images after 1 day, keep 10 versioned images
```

### Pattern 5: EKS on Fargate — Pure Serverless Kubernetes
```
Kubernetes manifest (pod spec)
        ↓
EKS Fargate Profile: matches namespace=production
        ↓
Fargate provisions dedicated compute per pod
  ├── Pod A (isolated VM-level boundary)
  ├── Pod B (isolated VM-level boundary)
  └── Pod C (isolated VM-level boundary)
        ↓
No worker nodes to manage, patch, or scale
IRSA provides AWS credentials to each pod
EFS CSI driver for shared persistent storage
```

---

## Exam Tips Summary

### Amazon ECS
- ✅ Task Definition = container blueprint (image, CPU, mem, ports, roles, logging)
- ✅ **Task Execution Role** = ECS infra role (pull ECR, push CloudWatch logs, get secrets)
- ✅ **Task Role** = application role (call DynamoDB, S3, SQS from inside container)
- ✅ Fargate = serverless (no EC2); EC2 = you manage nodes
- ✅ **awsvpc** = required for Fargate; gives task its own ENI + task-level SGs
- ✅ DAEMON service = one task per EC2 instance (like K8s DaemonSet)
- ✅ Fargate Spot = up to 70% cheaper; interruptible (batch/fault-tolerant use)
- ✅ EFS for persistent shared storage on Fargate (not EBS)
- ✅ Capacity Providers = manage where/how tasks run; enable managed scaling
- ✅ `bridge` and `host` network modes = EC2 launch type only

### Amazon EKS
- ✅ AWS manages Kubernetes **control plane** (API server, etcd, scheduler) — you pay $0.10/hr/cluster
- ✅ Managed Node Groups = AWS patches EC2 worker nodes; Self-managed = you do it
- ✅ Fargate Profiles = serverless pods; no DaemonSets, no GPU, no EBS on Fargate
- ✅ **IRSA** (IAM Roles for Service Accounts) = give pods AWS permissions via OIDC; no access keys
- ✅ AWS Load Balancer Controller = create ALB/NLB from K8s Ingress/Service
- ✅ VPC CNI = pods get real VPC IPs; security groups per pod supported
- ✅ **Karpenter** = modern node autoscaler (faster than Cluster Autoscaler, no ASG needed)
- ✅ EBS CSI = RWO persistent volumes (one pod); EFS CSI = RWX (many pods)
- ✅ Private endpoint = kubectl only from within VPC (recommended for production)

### Amazon ECR
- ✅ Private registry per account per region; ECR Public for public images
- ✅ Enhanced Scanning = Amazon Inspector; continuous re-scanning as new CVEs published
- ✅ Immutable tags = prevent overwriting tagged images (production safety)
- ✅ Lifecycle policies = auto-delete old/untagged images
- ✅ Pull privately (no internet): need Interface Endpoints for ECR (API + DKR) + S3 Gateway Endpoint
- ✅ Cross-region + cross-account replication supported
- ✅ Pull-through cache = proxy Docker Hub / quay.io / k8s.gcr.io through ECR

### ECS Anywhere
- ✅ Run ECS on your own on-prem/other cloud servers (Linux only)
- ✅ Uses SSM Agent for connectivity (outbound HTTPS only — no inbound ports)
- ✅ Launch type = EXTERNAL; no Fargate; no awsvpc mode
- ✅ Same ECS APIs, task definitions, CloudWatch logs as cloud ECS

### EKS Anywhere
- ✅ Run Kubernetes on your own infra (VMware, bare metal, Snow, Nutanix)
- ✅ AWS manages K8s component versions + security patches; you manage infra
- ✅ Connected (internet) or disconnected (air-gapped) modes
- ✅ EKS Connector = register cluster in AWS console for visibility
- ✅ Curated Packages = AWS-supported add-ons for EKS Anywhere

### EKS Distro
- ✅ Just a K8s distribution (download and run yourself — no AWS tooling)
- ✅ Same components + versions as EKS; AWS security patches
- ✅ For DIY teams; maximum flexibility; zero AWS management
- ✅ Free; hosted on ECR Public Gallery
- ✅ EKS Distro ≠ EKS Anywhere: Distro = just the software; Anywhere = software + lifecycle management tooling

---
---

<a name="storage"></a>
# PART 3 — Storage: EBS, EFS, FSx, S3, S3 Glacier, Storage Gateway & AWS Backup

# AWS Exam Study Guide: Storage Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** AWS Backup · Amazon EBS · Amazon EFS · Amazon FSx · Amazon S3 · Amazon S3 Glacier · AWS Storage Gateway

---

## Storage Service Quick-Select Matrix

| Service | Type | Access | Persistence | Protocol | Best For |
|---|---|---|---|---|---|
| **EBS** | Block | Single EC2 (Multi-attach limited) | Persistent | iSCSI/NVMe | OS disks, databases |
| **EFS** | File | Multi-instance (Linux) | Persistent | NFS v4 | Shared Linux workloads |
| **FSx for Windows** | File | Multi-instance (Windows) | Persistent | SMB/CIFS | Windows apps, AD integration |
| **FSx for Lustre** | File | Multi-instance (Linux HPC) | Persistent | POSIX | HPC, ML, big data |
| **FSx for NetApp ONTAP** | File/Block | Multi-protocol | Persistent | NFS, SMB, iSCSI | Enterprise multi-protocol |
| **FSx for OpenZFS** | File | Multi-instance | Persistent | NFS | ZFS migrations |
| **S3** | Object | Internet / AWS services | Persistent | HTTP/REST | Unstructured data, backups |
| **S3 Glacier** | Object (Archive) | Internet | Persistent | HTTP/REST | Long-term archiving |
| **Storage Gateway** | Hybrid | On-premises | Persistent | NFS/SMB/iSCSI | On-prem to cloud bridge |
| **AWS Backup** | Managed backup | Cross-service | Persistent | N/A | Centralized backup policy |

---

---

## 1. Amazon Elastic Block Store (Amazon EBS)

### What Problem Does It Solve?
EBS provides **persistent, high-performance block storage** for EC2 instances — like a virtual hard drive. When an EC2 instance stops or terminates, EBS data persists (unlike instance store). Designed for workloads requiring **low-latency, random I/O** access: databases, OS boot volumes, transactional applications.

---

### EBS Volume Types *(Critical Exam Topic)*

#### SSD-Backed Volumes

| Type | Name | Max IOPS | Max Throughput | Use Case |
|---|---|---|---|---|
| **gp3** | General Purpose SSD v3 | 16,000 | 1,000 MB/s | Default choice; IOPS/throughput independent of size |
| **gp2** | General Purpose SSD v2 | 16,000 | 250 MB/s | Legacy; IOPS tied to size (3 IOPS/GB) |
| **io2 Block Express** | Provisioned IOPS SSD | **256,000** | 4,000 MB/s | Mission-critical DBs, sub-millisecond latency |
| **io1** | Provisioned IOPS SSD | 64,000 | 1,000 MB/s | I/O-intensive databases |

#### HDD-Backed Volumes

| Type | Name | Max IOPS | Max Throughput | Use Case |
|---|---|---|---|---|
| **st1** | Throughput Optimized HDD | 500 | 500 MB/s | Big data, Kafka, log processing (sequential reads) |
| **sc1** | Cold HDD | 250 | 250 MB/s | Infrequent access, lowest cost HDD |

> **Exam Rule:** HDD volumes (**st1, sc1**) **cannot** be used as boot volumes. Only SSD (gp2, gp3, io1, io2) can boot.

> **gp3 vs gp2:** gp3 is the modern default. With gp3, IOPS and throughput are configured independently of volume size. With gp2, IOPS = 3 × size in GB (burstable up to 3,000 IOPS for volumes < 1TB).

---

### EBS Key Properties

| Property | Details |
|---|---|
| **Availability Zone** | EBS volumes are **AZ-specific** — cannot attach to EC2 in another AZ |
| **Attachment** | One EC2 at a time (except io1/io2 Multi-Attach, same AZ, up to 16 instances) |
| **Snapshots** | Point-in-time, incremental, stored in **S3** (managed by AWS) |
| **Encryption** | AES-256; uses AWS KMS; snapshots of encrypted volumes are encrypted |
| **Max Size** | 64 TB (io2 Block Express), 16 TB (others) |

---

### EBS Snapshots

- **Incremental**: Only changed blocks since last snapshot are saved.
- **Cross-region copy**: Snapshots can be copied to another region for DR.
- **AMI creation**: Create a custom AMI from a snapshot.
- **Fast Snapshot Restore (FSR)**: Pre-warms snapshot so restored volume has full performance immediately (extra cost).
- **Snapshot Archive**: Move snapshots to archive tier — **75% cheaper**, but **24–72 hours** restore time.
- **Recycle Bin**: Recover accidentally deleted snapshots within a retention period.

---

### EBS Multi-Attach

- Only **io1/io2** volumes support Multi-Attach.
- Same AZ only, up to **16 Nitro-based EC2 instances**.
- Applications must manage concurrent write access (cluster-aware file systems).
- Use case: High-availability applications like clustered databases (e.g., Oracle RAC).

---

### Instance Store vs EBS

| Feature | Instance Store | EBS |
|---|---|---|
| **Persistence** | Lost on stop/terminate | Persists independently |
| **Performance** | Extremely high IOPS (NVMe) | High (up to 256K IOPS on io2) |
| **Cost** | Included in instance price | Additional cost |
| **Backup** | Manual (no snapshots) | Snapshots to S3 |
| **Use Case** | Temp data, buffers, caches | OS, databases, persistent data |

---

### Key Exam Scenarios

- **Database requiring 100K+ IOPS**: Use **io2 Block Express**.
- **Move EBS volume to another AZ**: Snapshot → Create volume in new AZ from snapshot.
- **Move EBS volume to another region**: Snapshot → Copy snapshot to region → Create volume.
- **Reduce cost for infrequently accessed large data**: Use **sc1**.
- **Big data / log processing (sequential)**: Use **st1**.
- **Boot volume**: Must be SSD (gp2, gp3, io1, io2).
- **Shared block storage across multiple EC2**: io1/io2 Multi-Attach (same AZ).

---

---

## 2. Amazon Elastic File System (Amazon EFS)

### What Problem Does It Solve?
EFS provides a **fully managed, elastic NFS file system** that multiple EC2 instances (and other compute) can mount simultaneously across multiple Availability Zones. It automatically grows and shrinks as files are added/removed — no capacity planning needed. Solves the problem of **shared file storage across Linux-based workloads**.

---

### EFS Performance Modes

| Mode | Description | Use Case |
|---|---|---|
| **General Purpose** (default) | Low latency, ideal for most workloads | Web servers, CMS, home dirs |
| **Max I/O** | Higher throughput, slightly higher latency | Big data, media processing (100s of instances) |

> **Exam Tip:** General Purpose is the default and suits most scenarios. Max I/O is for massively parallel workloads with 100+ clients.

---

### EFS Throughput Modes

| Mode | Description |
|---|---|
| **Elastic Throughput** (default) | Automatically scales throughput up/down; pay per GB transferred |
| **Bursting Throughput** | Throughput tied to file system size; earns burst credits |
| **Provisioned Throughput** | Specify throughput independently of storage size; for predictable high-throughput needs |

---

### EFS Storage Classes

| Class | Description | Retrieval |
|---|---|---|
| **Standard** | Frequently accessed files | Immediate |
| **Standard-IA (Infrequent Access)** | Lower cost for less-accessed files | Small retrieval fee |
| **One Zone** | Single AZ; lower cost than Standard | Immediate |
| **One Zone-IA** | Single AZ + IA pricing | Small retrieval fee |

- **Lifecycle Management**: Automatically move files between Standard and IA based on last-access time (e.g., move after 30 days).
- **Intelligent-Tiering**: EFS equivalent — automatically moves files to IA and back based on access patterns.

---

### EFS Key Properties

| Property | Details |
|---|---|
| **Protocol** | NFSv4.0 and NFSv4.1 |
| **OS Support** | **Linux only** (POSIX-compliant) |
| **Multi-AZ** | Standard tier spans multiple AZs |
| **Concurrent Access** | Thousands of instances simultaneously |
| **Encryption** | At-rest (KMS) and in-transit (TLS) |
| **Access** | EC2, ECS, EKS, Lambda, on-prem (via Direct Connect/VPN) |

---

### EFS vs EBS

| Feature | EFS | EBS |
|---|---|---|
| **Type** | File (NFS) | Block |
| **Multi-instance** | Yes (thousands) | No (except Multi-Attach io1/io2, same AZ) |
| **OS** | Linux only | Linux + Windows |
| **Availability** | Multi-AZ | Single AZ |
| **Capacity** | Elastic (auto-scales) | Fixed (must resize) |
| **Cost** | Higher per GB | Lower per GB |
| **Use Case** | Shared file system | Single-instance, high-performance disk |

---

### Key Exam Scenarios

- **Shared storage across multiple Linux EC2 instances**: EFS.
- **WordPress/CMS content shared across web servers**: EFS.
- **Container workloads (ECS/EKS) needing persistent shared storage**: EFS.
- **Lambda accessing a shared file system**: EFS (mount via VPC).
- **Windows file share**: Use **FSx for Windows**, not EFS.
- **Cost optimization for rarely accessed files**: Enable EFS IA lifecycle policy.

---

---

## 3. Amazon FSx (All Types)

### What Problem Does It Solve?
FSx provides **fully managed third-party file systems** on AWS, each optimized for specific use cases. It solves the problem of running specialized, high-performance file systems without managing the underlying infrastructure.

---

### FSx Variants Compared

| FSx Type | Protocol | OS | Use Case |
|---|---|---|---|
| **FSx for Windows File Server** | SMB / CIFS | Windows (+ Linux) | Windows apps, AD integration, DFSR |
| **FSx for Lustre** | POSIX / Lustre | Linux | HPC, ML training, genomics, video rendering |
| **FSx for NetApp ONTAP** | NFS, SMB, iSCSI | Windows + Linux | Enterprise multi-protocol, on-prem migration |
| **FSx for OpenZFS** | NFS | Linux | ZFS workload migration, dev/test |

---

### FSx for Windows File Server

- **SMB protocol**: Works natively with Windows applications.
- **Active Directory integration**: Supports AWS Managed AD or self-managed AD.
- **DFS Namespaces**: Consolidate multiple file shares under a single namespace.
- **Shadow Copies**: VSS-based point-in-time backups (user-recoverable).
- **Multi-AZ**: Active/standby for HA.
- **Deployment**: Single-AZ or Multi-AZ.

> **Exam Tip:** Any scenario involving **Windows file shares**, **SMB**, **Active Directory**, or **DFSR** → FSx for Windows. Never EFS (Linux-only NFS).

---

### FSx for Lustre

- **Parallel distributed file system** — designed for extreme performance.
- Hundreds of GB/s throughput, millions of IOPS, sub-millisecond latency.
- **Native S3 integration**: Lazily loads data from S3; writes back to S3 automatically.
- **Deployment Types**:

| Type | Description |
|---|---|
| **Scratch** | Temporary; high burst performance; no replication; cheapest |
| **Persistent** | Long-term; data replicated within AZ; HA within AZ |

> **Exam Tip:** HPC, ML training, video processing, big data analytics, or anything requiring **sub-millisecond latency at massive scale** → FSx for Lustre. If it also needs S3 as a data lake → FSx for Lustre is the answer.

---

### FSx for NetApp ONTAP

- Supports **NFS, SMB, and iSCSI** simultaneously — true multi-protocol.
- **SnapMirror**: Replicate data to on-premises ONTAP systems.
- **FlexClone**: Instantly clone volumes with zero additional storage cost.
- Supports **automatic tiering** to S3 for cold data.
- Compatible with Linux, Windows, macOS clients.

> **Exam Tip:** "Migrate on-premises NetApp ONTAP workloads to AWS" or "need NFS + SMB + iSCSI on same file system" → FSx for NetApp ONTAP.

---

### FSx for OpenZFS

- ZFS-based file system; NFS protocol.
- Up to **1 million IOPS**, sub-millisecond latency.
- **Point-in-time snapshots**, cloning.
- Best for migrating existing ZFS workloads to AWS without re-architecting.

---

### FSx Deployment Options

| Option | FSx for Windows | FSx for Lustre | FSx for ONTAP | FSx for OpenZFS |
|---|---|---|---|---|
| **Single-AZ** | ✅ | ✅ | ✅ | ✅ |
| **Multi-AZ** | ✅ | ❌ | ✅ | ❌ |
| **S3 integration** | ❌ | ✅ Native | ✅ (tiering) | ❌ |
| **AD integration** | ✅ Required | ❌ | ✅ | ❌ |

---

### Key Exam Scenarios

- **Windows-based file server in cloud**: FSx for Windows.
- **HPC / ML / genomics workloads**: FSx for Lustre.
- **On-prem NetApp migration**: FSx for NetApp ONTAP.
- **Lustre + S3 as data lake**: FSx for Lustre (S3 linked file system).
- **Multi-protocol enterprise storage**: FSx for NetApp ONTAP.
- **ZFS workload lift-and-shift**: FSx for OpenZFS.

---

---

## 4. Amazon S3

### What Problem Does It Solve?
S3 provides **infinitely scalable object storage** for any type of data. It decouples storage from compute, enabling durable and highly available storage for files, backups, data lakes, static websites, and more — without managing infrastructure.

---

### S3 Storage Classes *(Most Tested S3 Topic)*

| Storage Class | Availability | Min Duration | Retrieval | Use Case |
|---|---|---|---|---|
| **S3 Standard** | 99.99% | None | Immediate | Frequently accessed data |
| **S3 Intelligent-Tiering** | 99.9% | None | Immediate | Unknown/changing access patterns |
| **S3 Standard-IA** | 99.9% | 30 days | Immediate | Infrequent but rapid access needed |
| **S3 One Zone-IA** | 99.5% (1 AZ) | 30 days | Immediate | Infrequent; reproducible data |
| **S3 Glacier Instant Retrieval** | 99.9% | 90 days | Milliseconds | Archives accessed ~quarterly |
| **S3 Glacier Flexible Retrieval** | 99.99% | 90 days | 1–12 hours | Long-term archive, occasional access |
| **S3 Glacier Deep Archive** | 99.99% | 180 days | 12–48 hours | 7–10 year retention, compliance |

> **Exam Tip:** Know the **minimum storage duration** — you're billed for it even if you delete early. Also know that **One Zone-IA** is lost if the AZ fails.

---

### S3 Intelligent-Tiering Tiers

Automatically moves objects between tiers based on access:
- **Frequent Access** (default)
- **Infrequent Access** (after 30 days)
- **Archive Instant Access** (after 90 days)
- **Archive Access** (configurable: 90–730 days)
- **Deep Archive Access** (configurable: 180–730 days)

No retrieval fees, small monitoring fee per object.

---

### S3 Key Features

#### Versioning
- Once enabled, **cannot be disabled** (only suspended).
- Protects against accidental deletes; delete marker is created instead.
- Each version billed separately.

#### MFA Delete
- Requires MFA to permanently delete versions or disable versioning.
- Must be enabled by the **root account** via CLI.

#### Replication
| Feature | Same-Region Replication (SRR) | Cross-Region Replication (CRR) |
|---|---|---|
| **Requirement** | Versioning on both buckets | Versioning on both buckets |
| **Use case** | Log aggregation, compliance, same-region DR | Latency reduction, compliance, DR |
| **Replication** | Asynchronous | Asynchronous |
| **Existing objects** | Not replicated by default (use S3 Batch) | Not replicated by default |

#### Lifecycle Policies
- Transition objects between storage classes after defined periods.
- Expire (delete) objects after a period.
- Cannot transition **from IA back to Standard** via lifecycle (only forward).

#### Object Lock
- **WORM** (Write Once Read Many) — prevent deletion or overwrite for a period.
- **Governance mode**: Most users can't delete; specific IAM permission can override.
- **Compliance mode**: **No one** (including root) can delete until retention period expires.
- Required for SEC 17a-4, FINRA, CFTC compliance.

---

### S3 Security

#### Bucket Policies vs ACLs vs IAM

| Method | Scope | Use Case |
|---|---|---|
| **Bucket Policy** | Bucket/object level | Cross-account access, IP restrictions, enforce HTTPS |
| **IAM Policy** | User/role level | Control which AWS identities access S3 |
| **ACL** | Object level | Legacy; rarely used; disabled by default on new buckets |
| **Block Public Access** | Account/bucket level | Override all ACLs and policies to deny public |

> **Exam Tip:** "Deny all public access regardless of bucket policy" → Enable **Block Public Access** settings. This is account-wide or per-bucket.

#### Encryption

| Type | Description |
|---|---|
| **SSE-S3** | AWS-managed keys; AES-256; enabled by default |
| **SSE-KMS** | Customer-managed KMS keys; audit trail via CloudTrail; key rotation |
| **SSE-C** | Customer-provided keys; AWS uses key then discards it |
| **Client-Side Encryption** | Encrypted before upload; AWS never sees plaintext |

> **Exam Tip:** SSE-KMS has KMS API call limits — could cause throttling at scale. Use S3 Bucket Keys to reduce KMS API calls by up to **99%**.

---

### S3 Performance

- **Prefix-based**: 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD per prefix per second.
- **Multipart Upload**: Recommended for objects > 100 MB, required > 5 GB; enables parallel upload.
- **Transfer Acceleration**: Uses CloudFront edge locations to accelerate cross-region uploads.
- **S3 Select / Glacier Select**: Query CSV/JSON/Parquet data with SQL **without downloading entire object** — reduces data transfer and cost.
- **Byte-range fetches**: Parallelize downloads by fetching specific byte ranges.

---

### S3 Event Notifications

Can trigger:
- **SNS**, **SQS**, **Lambda** (for object create, delete, restore events)
- **EventBridge**: More advanced filtering, multiple destinations, archive/replay

---

### S3 Access Points

- Create **named access endpoints** with custom policies per use case or team.
- Simplify managing access for large buckets with many users/applications.
- Can restrict to specific VPCs (VPC Origin access points).

---

### S3 Object Lambda

- Modify data returned from S3 GET requests **on the fly** without storing modified versions.
- Use cases: Redact PII, watermark images, convert data formats.

---

### Key Exam Scenarios

- **Static website hosting**: S3 + CloudFront.
- **Compliance archiving (WORM)**: S3 Object Lock + Compliance Mode.
- **Unknown access patterns**: S3 Intelligent-Tiering.
- **Secure cross-account access**: Bucket policy with explicit account trust, or assume role.
- **Accelerate global uploads**: S3 Transfer Acceleration.
- **Query data in place**: S3 Select or Athena.
- **Audit S3 access**: Enable Server Access Logging or CloudTrail Data Events.
- **Prevent accidental deletion**: Enable Versioning + MFA Delete.
- **Replication to another account**: CRR with destination bucket policy granting source account permission.

---

---

## 5. Amazon S3 Glacier

### What Problem Does It Solve?
S3 Glacier provides **extremely low-cost archival storage** for data that is rarely accessed and can tolerate retrieval delays. It is the answer for long-term regulatory compliance, data archiving, and digital preservation where cost trumps access speed.

---

### Glacier Tiers *(Now Integrated into S3 Storage Classes)*

| Tier | Retrieval Time | Min Storage Duration | Cost |
|---|---|---|---|
| **Glacier Instant Retrieval** | Milliseconds | 90 days | Low |
| **Glacier Flexible Retrieval** | 1 min (Expedited) / 3–5 hrs (Standard) / 5–12 hrs (Bulk) | 90 days | Very Low |
| **Glacier Deep Archive** | 12 hrs (Standard) / 48 hrs (Bulk) | 180 days | Lowest |

---

### Glacier Flexible Retrieval Options

| Option | Time | Cost |
|---|---|---|
| **Expedited** | 1–5 minutes | Highest retrieval cost |
| **Standard** | 3–5 hours | Medium retrieval cost |
| **Bulk** | 5–12 hours | Lowest retrieval cost |

> **Exam Tip:** For guaranteed retrieval time (even during peak), use **Provisioned Capacity** with Expedited retrievals.

---

### Glacier Vault vs S3 Bucket

- **Vault Lock**: Similar to S3 Object Lock. Enforces compliance controls (WORM). Once locked, policy **cannot be changed**.
- Archives in Glacier (standalone, not S3 lifecycle) are accessed via **Vault** → **Archive** hierarchy.
- Most modern architectures use S3 storage classes with Glacier tiers rather than standalone Glacier vaults.

---

### Glacier Select

- Run SQL queries on data stored in Glacier **without restoring the full archive**.
- Supports CSV, JSON, Parquet (GZIP/BZIP2 compressed).
- Reduces retrieval cost and time for analytics on archived data.

---

### Key Exam Scenarios

- **7–10 year compliance retention, rarely accessed**: S3 Glacier Deep Archive.
- **Archives accessed quarterly, need instant retrieval**: S3 Glacier Instant Retrieval.
- **WORM compliance for archives**: Glacier Vault Lock.
- **Cost-effective query on archived data**: Glacier Select.
- **Guarantee retrieval SLA for Expedited**: Provisioned Capacity Units.

---

---

## 6. AWS Storage Gateway

### What Problem Does It Solve?
Storage Gateway is a **hybrid cloud storage service** that bridges on-premises environments with AWS cloud storage. It lets on-premises applications use AWS storage (S3, Glacier, EBS) without re-architecting, using familiar protocols (NFS, SMB, iSCSI). Solves the **on-premises to cloud migration** and **hybrid storage** problem.

---

### Storage Gateway Types *(Critical — Know All Four)*

#### 1. S3 File Gateway
- **Protocol**: NFS v3/v4.1, SMB
- **Backend**: Amazon S3
- **How it works**: On-premises app writes files via NFS/SMB → Gateway stores as **S3 objects**.
- **Caching**: Frequently accessed data cached locally on gateway appliance.
- **Use Case**: File shares backed by S3; archiving on-prem files to S3; migrating file data to S3.

#### 2. FSx File Gateway
- **Protocol**: SMB
- **Backend**: Amazon FSx for Windows File Server
- **How it works**: On-premises Windows clients use SMB → Gateway caches locally → syncs to FSx for Windows.
- **Use Case**: Windows file shares with low-latency local access + FSx backend. Better than S3 File Gateway for Windows/AD environments.

#### 3. Volume Gateway
- **Protocol**: iSCSI (block storage)
- **Backend**: Amazon S3 + EBS Snapshots
- **Sub-types**:

| Sub-type | Description | Use Case |
|---|---|---|
| **Cached Volumes** | Primary data in S3; frequently accessed data cached locally | Low-latency access to recently used data, minimize on-prem storage |
| **Stored Volumes** | Primary data stored locally; **asynchronously backed up to S3 as EBS snapshots** | Low-latency for ALL data; full on-prem copy with S3 backup |

> **Exam Tip:** Cached = S3 primary, local cache. Stored = Local primary, S3 backup.

#### 4. Tape Gateway (Virtual Tape Library — VTL)
- **Protocol**: iSCSI (tape emulation)
- **Backend**: S3 and S3 Glacier
- **How it works**: Presents virtual tape drives to backup software (Veeam, Backup Exec, Veritas). Data written to virtual tapes stored in S3; archived tapes moved to Glacier.
- **Use Case**: Replace physical tape infrastructure while keeping existing backup software workflows.

---

### Storage Gateway Summary Table

| Gateway Type | Protocol | AWS Backend | Local Cache | Use Case |
|---|---|---|---|---|
| **S3 File** | NFS, SMB | S3 | ✅ | File shares → S3 |
| **FSx File** | SMB | FSx for Windows | ✅ | Windows shares → FSx |
| **Volume (Cached)** | iSCSI | S3 (+ EBS snapshots) | ✅ Recent data | Minimal on-prem hardware |
| **Volume (Stored)** | iSCSI | S3 (EBS snapshots as backup) | ✅ All data | Full local + S3 backup |
| **Tape** | iSCSI VTL | S3 / Glacier | ✅ | Replace physical tape |

---

### Deployment Options

- **VM Appliance**: Deploy on VMware ESXi, Microsoft Hyper-V, or Linux KVM.
- **Hardware Appliance**: Physical AWS-provided appliance (for environments without virtualization).
- **EC2**: Deploy gateway as an EC2 instance (for cloud-side testing or migration).

---

### Key Exam Scenarios

- **On-prem NFS shares to be backed up in S3**: S3 File Gateway.
- **Windows SMB shares with local caching, backend in cloud**: FSx File Gateway.
- **Replace physical tape library**: Tape Gateway.
- **On-prem block storage with S3 backup**: Volume Gateway (Stored Volumes).
- **Minimize on-prem storage but need low latency for recent data**: Volume Gateway (Cached Volumes).
- **Disaster recovery — snapshots in AWS**: Volume Gateway → EBS Snapshots → launch EC2 with EBS from snapshot.

---

---

## 7. AWS Backup

### What Problem Does It Solve?
AWS Backup provides a **fully managed, centralized backup service** across multiple AWS services. Without it, each service has its own backup mechanism, making it hard to enforce consistent policies, audit compliance, or manage backups at scale.

---

### Supported Services

AWS Backup supports:
- **EC2** (AMI-based backups)
- **EBS** volumes (snapshots)
- **EFS** file systems
- **FSx** (all types)
- **RDS** / Aurora (automated snapshots)
- **DynamoDB** (on-demand and continuous backups)
- **S3** (continuous point-in-time restore)
- **DocumentDB**, **Neptune**, **VMware on AWS**, **Storage Gateway** volumes

---

### Key Concepts

#### Backup Plan
- Policy that defines **when** and **how** backups are taken.
- Components:
  - **Backup Rule**: Schedule (cron or frequency), retention period, lifecycle (move to cold storage).
  - **Backup Vault**: Target location for backups.
  - **Resource Assignment**: Which resources to back up (by tag or ARN).

#### Backup Vault
- Logical container for backups.
- **Vault Lock**: Prevents deletion of backups; enforces WORM. Cannot be undone after lock.
- Supports customer-managed KMS keys.

#### Backup Vault Lock
- **Governance mode**: Admin can delete with permission.
- **Compliance mode**: No one can delete until retention expires (similar to S3 Object Lock).

---

### Cross-Region & Cross-Account Backup

| Feature | Description |
|---|---|
| **Cross-Region** | Copy backups to another region for DR |
| **Cross-Account** | Copy backups to another AWS account (within an AWS Organization) |
| **AWS Organizations** | Apply backup policies centrally across all accounts via Service Control Policies (SCPs) |

> **Exam Tip (Professional level):** For enterprise-wide backup governance → AWS Backup + AWS Organizations + Backup Policies. This lets you enforce backup rules across all member accounts.

---

### AWS Backup Audit Manager

- Continuously audit backup compliance against defined controls.
- Pre-built frameworks: **AWS Backup Plan**, **AWS Backup Vault**, **Recovery Point Objective (RPO)**.
- Generate compliance reports for auditors.

---

### AWS Backup vs Native Service Backups

| Feature | AWS Backup | Native Backups (e.g., RDS automated) |
|---|---|---|
| **Central management** | ✅ Single pane of glass | ❌ Per-service |
| **Cross-service policy** | ✅ One plan for all | ❌ Configure each separately |
| **Cross-account copy** | ✅ | ❌ (manual) |
| **Compliance reporting** | ✅ Audit Manager | ❌ |
| **Vault Lock (WORM)** | ✅ | Limited (RDS has delete protection) |

---

### Key Exam Scenarios

- **Centralize backup across multiple AWS accounts**: AWS Backup + AWS Organizations.
- **Enforce backup compliance across an organization**: AWS Backup Policies (via Organizations).
- **WORM protection for backups**: Backup Vault Lock.
- **RPO/RTO compliance auditing**: AWS Backup Audit Manager.
- **Cross-region DR backup**: AWS Backup cross-region copy rule.
- **Backup EC2 including all EBS volumes**: AWS Backup EC2 resource assignment → creates AMI + EBS snapshots.

---

---

## Master Comparison: All Storage Services

### By Access Pattern

| Need | Service |
|---|---|
| Single EC2 block storage (OS/DB) | EBS |
| Shared block storage (clustered DB) | EBS Multi-Attach (io1/io2) |
| Shared file storage (Linux, multi-instance) | EFS |
| Shared file storage (Windows, SMB) | FSx for Windows |
| HPC / ML high-performance parallel FS | FSx for Lustre |
| Object storage (general) | S3 |
| Long-term archive | S3 Glacier |
| On-prem to cloud bridge | Storage Gateway |
| Centralized backup policy | AWS Backup |

---

### By Cost (Approximate, Low → High per GB)

```
Deep Archive < Glacier Flexible < S3 One Zone-IA < S3 Standard-IA
< st1/sc1 (EBS HDD) < S3 Standard < gp3 (EBS SSD) < io2 (EBS PIOPS) < EFS Standard
```

---

### S3 vs EFS vs EBS — The Classic Exam Triangle

| | S3 | EFS | EBS |
|---|---|---|---|
| **Storage type** | Object | File | Block |
| **EC2 required** | No | No | Yes |
| **Multi-instance** | Yes (via HTTP) | Yes (NFS mount) | No (except Multi-Attach) |
| **Max throughput** | Very high | High | High (io2: 4 GB/s) |
| **Latency** | ms (higher) | ms | Sub-ms |
| **Protocol** | HTTP/REST | NFS | iSCSI |
| **OS** | Any | Linux | Linux + Windows |

---

## Architecture Patterns to Know

### Pattern 1: Backup & DR
```
EC2 / RDS / EFS → AWS Backup → Backup Vault (Region A)
                                        ↓ Cross-region copy
                               Backup Vault (Region B, DR)
```

### Pattern 2: Hybrid Storage
```
On-premises servers → Storage Gateway (S3 File) → Amazon S3
                                                         ↓ Lifecycle Policy
                                                   S3 Glacier Deep Archive
```

### Pattern 3: HPC Data Pipeline
```
S3 (raw data lake) → FSx for Lustre (linked FS) → HPC Cluster (EC2)
                             ↓ (results)
                          S3 (output)
```

### Pattern 4: Disaster Recovery with EBS
```
EC2 + EBS (Primary Region) → EBS Snapshot → Copy to DR Region
                                                   ↓
                                        Restore EBS → Launch EC2 (DR)
```

### Pattern 5: Data Lifecycle
```
S3 Standard (0–30 days, active)
    → S3 Standard-IA (30–90 days)
        → S3 Glacier Instant Retrieval (90–180 days)
            → S3 Glacier Deep Archive (180+ days, compliance)
```

---

## Exam Tips Summary

### EBS
- ✅ **gp3** = default; IOPS independent of size
- ✅ **io2 Block Express** = highest IOPS (256K); mission-critical DBs
- ✅ HDD (st1/sc1) = **cannot boot**
- ✅ AZ-locked; move via snapshot → restore in new AZ
- ✅ Multi-Attach = io1/io2 only, same AZ, up to 16 instances
- ✅ **RDS Proxy** if Lambda uses RDS (connection pooling)

### EFS
- ✅ NFS, Linux only, multi-AZ, elastic capacity
- ✅ **General Purpose** for latency-sensitive; **Max I/O** for massively parallel
- ✅ Enable Lifecycle to move cold files to IA tier
- ✅ Mount from on-prem via Direct Connect or VPN

### FSx
- ✅ **Windows** = SMB + AD; **Lustre** = HPC + S3; **ONTAP** = multi-protocol; **OpenZFS** = ZFS migration
- ✅ Lustre: **Scratch** = temp/fast; **Persistent** = replicated
- ✅ ONTAP = only FSx supporting iSCSI
- ✅ Lustre integrates natively with S3 as a linked data repository

### S3
- ✅ Know all storage classes + retrieval times + min durations
- ✅ **Object Lock Compliance** = even root can't delete
- ✅ **S3 Select** = SQL query without full download
- ✅ **Transfer Acceleration** = CloudFront edge for uploads
- ✅ **Bucket Key** = reduces KMS API calls by 99%
- ✅ **SRR** = same region; **CRR** = cross region; both need versioning

### S3 Glacier
- ✅ **Deep Archive** = cheapest; 12–48 hr retrieval; 180-day min
- ✅ **Vault Lock** = WORM; irreversible once locked
- ✅ **Provisioned Capacity** = guarantees Expedited retrieval
- ✅ **Glacier Select** = query without full restore

### Storage Gateway
- ✅ **S3 File** = NFS/SMB → S3
- ✅ **FSx File** = SMB → FSx for Windows
- ✅ **Volume Cached** = S3 primary + local cache
- ✅ **Volume Stored** = local primary + S3 backup (EBS snapshots)
- ✅ **Tape** = replace physical tape with virtual tapes → S3/Glacier

### AWS Backup
- ✅ Central policy for EBS, EFS, RDS, DynamoDB, S3, FSx, EC2
- ✅ Cross-region + cross-account copy
- ✅ **Vault Lock** = WORM for backup retention
- ✅ Use with **AWS Organizations** for enterprise-wide enforcement
- ✅ **Audit Manager** = compliance reporting for RPO/RTO

---
---

<a name="databases"></a>
# PART 4 — Databases: Aurora, Aurora Serverless, DocumentDB, DynamoDB, ElastiCache, Keyspaces, Neptune, QLDB, RDS & Redshift

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

---
---

<a name="networking"></a>
# PART 5 — Networking & Content Delivery: VPC, ELB, Route 53, CloudFront, Global Accelerator, Direct Connect, VPNs, Transit Gateway & PrivateLink

# AWS Exam Study Guide: Networking & Content Delivery
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Client VPN · CloudFront · Direct Connect · ELB · Global Accelerator · PrivateLink · Route 53 · Site-to-Site VPN · Transit Gateway · Amazon VPC

---

## Networking Quick-Reference Matrix

| Service | Layer | Scope | Primary Purpose | Exam Trigger Words |
|---|---|---|---|---|
| **VPC** | L3/L4 | Regional | Isolated network environment | "private network", "subnets", "routing" |
| **ELB** | L4/L7 | Regional/AZ | Distribute traffic across targets | "load balance", "high availability", "health check" |
| **Route 53** | L7 (DNS) | Global | DNS, routing, health checks | "DNS", "failover", "latency routing", "domain" |
| **CloudFront** | L7 | Global (Edge) | CDN, cache, edge delivery | "cache", "low latency global", "edge", "static content" |
| **Global Accelerator** | L4 | Global (Edge) | TCP/UDP acceleration via AWS backbone | "non-HTTP", "static IP", "global TCP/UDP acceleration" |
| **Direct Connect** | L2 | Hybrid | Dedicated private line to AWS | "dedicated bandwidth", "consistent latency", "on-prem to AWS" |
| **Site-to-Site VPN** | L3 | Hybrid | Encrypted tunnel over internet | "IPsec VPN", "quick setup", "on-prem to VPC" |
| **Client VPN** | L3 | Hybrid | Remote user VPN to AWS | "remote employees", "OpenVPN", "user VPN" |
| **Transit Gateway** | L3 | Regional/Global | Hub-and-spoke network hub | "many VPCs", "transitive routing", "centralized routing" |
| **PrivateLink** | L4 | Regional | Private service access without IGW | "private endpoint", "expose SaaS service", "no public internet" |

---

---

## 1. Amazon VPC (Virtual Private Cloud)

### What Problem Does It Solve?
VPC lets you provision a **logically isolated section of the AWS cloud** where you define your own IP address range, subnets, route tables, and network gateways — essentially your own private data center in AWS.

---

### VPC Core Components

| Component | Description |
|---|---|
| **VPC** | Logical network boundary; spans all AZs in a region |
| **Subnet** | Range of IPs within an AZ; can be public or private |
| **Route Table** | Rules that determine where network traffic is directed |
| **Internet Gateway (IGW)** | Enables internet access for public subnets; horizontally scaled, HA |
| **NAT Gateway** | Enables outbound-only internet for private subnets; managed; per-AZ |
| **NAT Instance** | Self-managed EC2 acting as NAT; legacy; not HA by default |
| **Security Group** | Stateful, instance-level firewall; allow-only rules |
| **Network ACL (NACL)** | Stateless, subnet-level firewall; allow + deny rules |
| **VPC Peering** | Private connection between two VPCs |
| **VPC Endpoints** | Private connectivity to AWS services without internet |
| **Elastic IP (EIP)** | Static public IPv4 address |

---

### Subnet Types

| Type | Route Table Has | Use Case |
|---|---|---|
| **Public subnet** | Route to IGW (`0.0.0.0/0 → igw-xxx`) | Web servers, ALBs, NAT Gateways |
| **Private subnet** | Route to NAT Gateway | App servers, databases |
| **Isolated subnet** | No internet route at all | Sensitive databases, internal services |

---

### Security Groups vs Network ACLs *(Heavily Tested)*

| Feature | Security Group | Network ACL |
|---|---|---|
| **Level** | Instance (ENI) | Subnet |
| **State** | Stateful (return traffic auto-allowed) | Stateless (must allow both directions) |
| **Rules** | Allow only | Allow + Deny |
| **Rule evaluation** | All rules evaluated together | Rules evaluated in number order; first match wins |
| **Default** | Deny all inbound, allow all outbound | Allow all in/out |
| **Applies to** | EC2, RDS, Lambda in VPC, etc. | All traffic to/from subnet |

> **Exam Tip:** Security Groups are stateful — if you allow inbound 443, the response is automatically allowed outbound. NACLs require explicit rules for both directions.

---

### NAT Gateway vs NAT Instance

| Feature | NAT Gateway | NAT Instance |
|---|---|---|
| **Management** | Fully managed by AWS | You manage EC2 |
| **HA** | Redundant within AZ | Single point of failure |
| **Bandwidth** | Up to 100 Gbps | Depends on instance type |
| **Cost** | Per hour + per GB | EC2 instance cost |
| **Security Groups** | Cannot apply | Can apply |
| **Placement** | Public subnet | Public subnet |
| **Multi-AZ HA** | Deploy one per AZ | Requires Auto Scaling + scripting |
| **Port forwarding** | ❌ | ✅ |

> **Exam Rule:** Always use NAT Gateway in production. NAT Instance is legacy and only appears in trick questions.

---

### VPC Peering

- Private, direct connection between two VPCs (same or different accounts/regions).
- Uses AWS backbone — no internet transit.
- **Not transitive**: A↔B and B↔C does NOT mean A↔C.
- No overlapping CIDR blocks allowed.
- Must update route tables in both VPCs.

---

### VPC Endpoints

| Type | Description | Use Case |
|---|---|---|
| **Gateway Endpoint** | Route-table-based; free | **S3 and DynamoDB only** |
| **Interface Endpoint (PrivateLink)** | ENI in subnet; hourly + data charge | All other AWS services, SaaS |

> **Exam Tip:** "EC2 in private subnet needs to access S3 without internet" → **VPC Gateway Endpoint** (free, route-table-based).

---

### VPC Flow Logs

- Capture metadata (not payload) of IP traffic to/from ENIs, subnets, or VPCs.
- Destination: CloudWatch Logs, S3, Kinesis Data Firehose.
- Fields: source IP, dest IP, port, protocol, bytes, action (ACCEPT/REJECT).
- Does NOT capture: DNS queries (use Route 53 Resolver logs), NTP, DHCP, instance metadata.

---

### CIDR and IP Planning

- VPC CIDR: `/16` to `/28`.
- AWS **reserves 5 IPs per subnet**: network, VPC router, DNS, future use, broadcast.
- Example: `/24` subnet = 256 − 5 = **251 usable IPs**.
- IPv6: `/56` for VPC, `/64` for subnets; all IPv6 are public by default.

---

### Key Exam Scenarios

- **Private EC2 → internet (outbound only)**: NAT Gateway in public subnet.
- **Private EC2 → S3 without internet**: VPC Gateway Endpoint for S3.
- **Private EC2 → any AWS service privately**: Interface Endpoint (PrivateLink).
- **Connect two VPCs (no transitive routing needed)**: VPC Peering.
- **Connect many VPCs (transitive routing needed)**: Transit Gateway.
- **Block specific IP at subnet level**: NACL deny rule.
- **Capture traffic for security analysis**: VPC Flow Logs.
- **EC2 needs static public IP**: Elastic IP.

---

---

## 2. Elastic Load Balancing (ELB)

### What Problem Does It Solve?
ELB distributes **incoming application traffic** across multiple targets (EC2, ECS, Lambda, IPs) in multiple AZs, providing fault tolerance, automatic scaling, and health checking.

---

### ELB Types — The Big Four

#### Application Load Balancer (ALB) — Layer 7
- HTTP/HTTPS/gRPC traffic.
- **Content-based routing**: path (`/api/*`), host (`api.example.com`), headers, query strings, IP.
- Supports **WebSockets**, HTTP/2.
- Targets: EC2, ECS, Lambda, IP addresses.
- **Sticky sessions**: via cookies (AWSALBCOOKIE or custom).
- **SSL termination**: offloads TLS from backend.
- Returns **fixed response** or **redirects** (e.g., HTTP → HTTPS).

#### Network Load Balancer (NLB) — Layer 4
- TCP, UDP, TLS traffic.
- Handles **millions of requests per second** with ultra-low latency.
- **Static IP per AZ** (or use Elastic IP) — critical for whitelisting.
- Preserves **client source IP** (unlike ALB which uses X-Forwarded-For).
- Targets: EC2, IP addresses, ALB (ALB behind NLB).
- Supports **PrivateLink** (expose services privately).
- **No security groups** on NLB itself (but target SGs must allow NLB traffic from VPC CIDR).

#### Gateway Load Balancer (GWLB) — Layer 3
- For deploying, scaling, and managing **third-party virtual network appliances** (firewalls, IDS/IPS, deep packet inspection).
- Uses **GENEVE protocol** (port 6081).
- Traffic flow: user → GWLB → appliance (inspect/modify) → GWLB → destination.
- Operates at VPC boundary using **Gateway Load Balancer Endpoints**.

#### Classic Load Balancer (CLB) — Layer 4/7 (Legacy)
- Original ELB; supports HTTP/HTTPS/TCP/SSL.
- Does not support content-based routing.
- Being phased out; avoid in new designs.

---

### ALB vs NLB vs GWLB Comparison

| Feature | ALB | NLB | GWLB |
|---|---|---|---|
| **Layer** | 7 (HTTP) | 4 (TCP/UDP) | 3 (IP) |
| **Protocol** | HTTP, HTTPS, gRPC | TCP, UDP, TLS | IP (GENEVE) |
| **Routing** | Content-based | Connection-based | Transparent pass-through |
| **Static IP** | ❌ (use Global Accelerator) | ✅ (per AZ) | N/A |
| **Source IP preserve** | Via header | ✅ Native | ✅ |
| **PrivateLink** | ❌ | ✅ | ✅ |
| **Lambda target** | ✅ | ❌ | ❌ |
| **Use case** | Web apps, microservices | High-perf, gaming, IoT, VPN | Security appliances |

---

### ELB Health Checks

- ALB: HTTP/HTTPS health check (must return 2xx/3xx).
- NLB: TCP, HTTP, HTTPS health checks.
- Unhealthy targets are deregistered from routing until healthy.
- **Deregistration delay** (connection draining): time to complete in-flight requests when deregistering target (default 300s, range 0–3600s).

---

### Cross-Zone Load Balancing

| ELB Type | Default | Cost |
|---|---|---|
| **ALB** | Enabled | Free |
| **NLB** | Disabled | Charged if enabled |
| **GWLB** | Disabled | Charged if enabled |

> **Exam Tip:** With cross-zone LB disabled, each LB node only distributes traffic among targets in its own AZ. With it enabled, traffic distributes evenly across all AZs.

---

### ALB Listener Rules

Priority-ordered rules evaluated top-down:
1. **Conditions**: host header, path, HTTP method, query string, source IP, header value
2. **Actions**: forward to target group, redirect, fixed response, authenticate (Cognito / OIDC)

---

### ELB Access Logs & Monitoring

- **Access logs**: Stored in S3; detailed per-request logging (source IP, latency, response code).
- **CloudWatch metrics**: RequestCount, TargetResponseTime, HTTPCode_ELB_5XX, HealthyHostCount.
- **Request tracing**: ALB adds `X-Amzn-Trace-Id` header; integrate with X-Ray.

---

### Key Exam Scenarios

- **Route `/api` to one service, `/web` to another**: ALB path-based routing.
- **Whitelist load balancer IP in firewall**: NLB (static IP per AZ).
- **Expose internal service to another VPC via PrivateLink**: NLB as PrivateLink service.
- **Third-party firewall appliance in traffic path**: GWLB.
- **Lambda as backend for HTTP API**: ALB with Lambda target group.
- **Redirect HTTP to HTTPS**: ALB listener rule (redirect action, port 443).
- **Blue/green deployment with traffic shift**: ALB weighted target groups.
- **Sticky sessions for stateful app**: ALB with cookie-based stickiness.

---

---

## 3. Amazon Route 53

### What Problem Does It Solve?
Route 53 is AWS's **highly available, scalable DNS service** that also provides domain registration, DNS-based health checks, and intelligent traffic routing.

---

### Route 53 Record Types

| Record | Description |
|---|---|
| **A** | IPv4 address |
| **AAAA** | IPv6 address |
| **CNAME** | Alias to another hostname; **cannot be used for zone apex (root domain)** |
| **Alias** | AWS-specific; points to AWS resources (ALB, CloudFront, S3, etc.); **works at zone apex**; free of charge |
| **MX** | Mail server |
| **TXT** | Text records; domain verification |
| **NS** | Name servers for hosted zone |
| **PTR** | Reverse DNS lookup |
| **SRV** | Service location |

> **Critical Exam Rule:** `CNAME` cannot be at the zone apex (`example.com`). Use **Alias** record instead. Alias records are free; CNAME queries are charged.

---

### Hosted Zones

| Type | Description |
|---|---|
| **Public Hosted Zone** | Resolves queries from the internet |
| **Private Hosted Zone** | Resolves queries within VPCs; must associate VPC |

---

### Route 53 Routing Policies *(Most Tested Topic)*

#### 1. Simple Routing
- One record → one or more IP values.
- If multiple values: **client randomly picks one**.
- No health checks on individual values.

#### 2. Weighted Routing
- Multiple records with **weight values** (0–255).
- Traffic proportional to weight: weight of record / total weights.
- Use for: A/B testing, gradual deployments, blue/green.
- Health check supported.

#### 3. Latency-Based Routing
- Routes to the **AWS region with lowest network latency** for the user.
- Not geographic — based on measured latency.
- Health check supported.

#### 4. Failover Routing
- **Active-passive** failover.
- Primary record used when healthy; Route 53 fails over to secondary if primary health check fails.
- Requires health check on primary.

#### 5. Geolocation Routing
- Routes based on **user's geographic location** (continent, country, state).
- Must define a **default record** for unmatched locations.
- Use for: localized content, compliance (GDPR), language routing.
- Different from latency routing — based on location, not latency.

#### 6. Geoproximity Routing (Traffic Flow Only)
- Routes based on geographic location of **users AND resources**.
- **Bias value**: Expand (+) or shrink (−) geographic area served by a resource.
- Requires **Route 53 Traffic Flow**.

#### 7. Multi-Value Answer Routing
- Returns **multiple healthy IP values** (up to 8) — clients randomly pick.
- Health checks supported; unhealthy records excluded.
- NOT a replacement for a load balancer; client-side "load balancing".

#### 8. IP-Based Routing
- Route based on the **client's IP CIDR** (you provide CIDR → endpoint mapping).
- Use for: ISP-based routing, known office IP routing.

---

### Route 53 Health Checks

| Type | Description |
|---|---|
| **Endpoint health check** | HTTP/HTTPS/TCP check against IP or hostname |
| **Calculated health check** | Combines results of multiple child health checks (AND/OR logic) |
| **CloudWatch alarm health check** | Healthy = alarm state is OK |

- Health checkers are globally distributed; **18% threshold** by default must report healthy.
- Private endpoints require **CloudWatch alarm** health check (health checkers can't reach private IPs).

---

### Route 53 Resolver

| Feature | Description |
|---|---|
| **Inbound Endpoint** | On-prem DNS can query Route 53 private hosted zones |
| **Outbound Endpoint** | Route 53 forwards queries to on-prem DNS |
| **Resolver Rules** | Forwarding rules: which domains go to on-prem DNS |

---

### Route 53 vs CloudFront for Routing

| Scenario | Service |
|---|---|
| DNS-level failover between regions | Route 53 Failover routing |
| Serve cached content from edge | CloudFront |
| Low-latency API for global users | Route 53 Latency + Global Accelerator |

---

### Key Exam Scenarios

- **Root domain pointing to ALB**: Alias A record (not CNAME).
- **Blue/green with 10% traffic to new version**: Weighted routing.
- **Serve users from nearest AWS region**: Latency-based routing.
- **Failover to S3 static website if primary down**: Failover routing with health check.
- **Route EU users to EU endpoint for GDPR**: Geolocation routing.
- **On-prem DNS resolve AWS private hosted zone**: Route 53 Resolver inbound endpoint.
- **Route 53 forward on-prem domain resolution**: Route 53 Resolver outbound endpoint.
- **Health check for private ALB**: CloudWatch alarm → Route 53 CloudWatch alarm health check.

---

---

## 4. Amazon CloudFront

### What Problem Does It Solve?
CloudFront is AWS's **Content Delivery Network (CDN)** that caches content at **450+ edge locations** globally, reducing latency for end users and offloading origin servers.

---

### CloudFront Components

| Component | Description |
|---|---|
| **Distribution** | The CloudFront deployment (one per origin/behavior config) |
| **Origin** | Source of content: S3, ALB, EC2, API Gateway, custom HTTP |
| **Edge Location** | Cache node closest to the user |
| **Regional Edge Cache** | Larger cache between edge locations and origin (fewer origin fetches) |
| **Cache Behavior** | Rules per path pattern: which origin, TTL, headers, cookies |

---

### CloudFront Origins

| Origin Type | Notes |
|---|---|
| **S3 bucket** | Use Origin Access Control (OAC) to restrict S3 to CloudFront only |
| **S3 static website endpoint** | Used for website hosting with custom error pages |
| **ALB** | Must allow CloudFront IP ranges in SG |
| **EC2** | Must be publicly accessible |
| **API Gateway** | Common for serverless APIs |
| **Custom HTTP origin** | Any HTTP/HTTPS endpoint |

---

### Origin Access Control (OAC) vs Origin Access Identity (OAI)

| Feature | OAC (New) | OAI (Legacy) |
|---|---|---|
| **S3 SSE-KMS** | ✅ Supported | ❌ |
| **All S3 operations** | ✅ | Limited |
| **Recommended** | ✅ | Deprecated |

> **Exam Tip:** Use **OAC** (not OAI) for restricting S3 bucket access to CloudFront only. Set bucket policy to allow CloudFront service principal.

---

### CloudFront Cache Behavior

- **TTL settings**: min, max, default TTL.
- **Cache key**: URL path + selected headers, cookies, query strings.
- **Cache invalidation**: `/*` invalidates everything (first 1,000 paths/month free).
- **Cache-Control / Expires headers** from origin override default TTL.

---

### CloudFront Security

| Feature | Description |
|---|---|
| **HTTPS** | Enforce HTTPS between viewer-CloudFront and CloudFront-origin |
| **WAF** | Attach WAF Web ACL to distribution |
| **Geo-restriction** | Allow/block by country (allowlist or blocklist) |
| **Signed URLs** | Grant time-limited access to a single file |
| **Signed Cookies** | Grant time-limited access to multiple files |
| **Field-Level Encryption** | Encrypt specific POST fields at edge using public key |

#### Signed URL vs Signed Cookie

| Feature | Signed URL | Signed Cookie |
|---|---|---|
| **Scope** | One file | Multiple files |
| **Client support** | Any client | Must support cookies |
| **Use case** | Individual file download | Premium video streaming, content library |

---

### CloudFront Functions vs Lambda@Edge

| Feature | CloudFront Functions | Lambda@Edge |
|---|---|---|
| **Runtime** | JavaScript (ES5.1) | Node.js, Python |
| **Max execution** | < 1 ms | 5 sec (viewer) / 30 sec (origin) |
| **Triggers** | Viewer request/response only | Viewer + origin request/response |
| **Memory** | 2 MB | 128 MB – 10 GB |
| **Network access** | ❌ | ✅ |
| **Cost** | ~1/6th of Lambda@Edge | Higher |
| **Use case** | URL rewrites, header manipulation, simple redirects | Auth, A/B testing, complex routing, API calls |

---

### CloudFront Pricing Classes

| Class | Edge Locations | Cost |
|---|---|---|
| **Price Class All** | All regions | Highest |
| **Price Class 200** | Most regions (excl. most expensive) | Medium |
| **Price Class 100** | Cheapest regions (US, EU, Asia) | Lowest |

---

### CloudFront vs S3 Transfer Acceleration

| Feature | CloudFront | S3 Transfer Acceleration |
|---|---|---|
| **Purpose** | Cache + deliver content | Accelerate S3 uploads |
| **Direction** | Download (mostly) | Upload to S3 |
| **Caching** | ✅ | ❌ |
| **Use case** | Serve static/dynamic content | Upload large files to S3 globally |

---

### Key Exam Scenarios

- **Serve S3 static website globally with low latency**: CloudFront + S3 OAC.
- **Prevent direct S3 access; only via CloudFront**: S3 bucket policy + OAC.
- **Serve premium video only to subscribers**: CloudFront Signed Cookies.
- **Block users from specific countries**: CloudFront Geo-restriction.
- **Add security headers to all responses**: CloudFront Functions (viewer response trigger).
- **A/B test new origin**: CloudFront weighted origin groups.
- **Force HTTPS**: Redirect HTTP to HTTPS in CloudFront viewer protocol policy.
- **Real-time logs for monitoring**: CloudFront Real-time Logs → Kinesis.

---

---

## 5. AWS Global Accelerator

### What Problem Does It Solve?
Global Accelerator improves performance of **global TCP and UDP applications** by routing traffic through the **AWS global backbone network** instead of the public internet, reducing latency and packet loss.

---

### Key Properties

| Property | Details |
|---|---|
| **Entry points** | **2 static Anycast IPs** (globally announced from all edge locations) |
| **Protocol** | TCP, UDP (not HTTP-aware) |
| **Backend targets** | ALB, NLB, EC2, Elastic IP |
| **Health checks** | Built-in; automatic failover in < 30 seconds |
| **Client affinity** | Route same client to same endpoint |

---

### How It Works

```
User → Nearest AWS Edge Location (Anycast IP)
              ↓
    AWS Global Backbone Network (private, fast)
              ↓
    Endpoint (ALB/NLB/EC2 in target region)
```

---

### Global Accelerator vs CloudFront *(Most Tested Comparison)*

| Feature | Global Accelerator | CloudFront |
|---|---|---|
| **Layer** | L4 (TCP/UDP) | L7 (HTTP/HTTPS) |
| **Caching** | ❌ No caching | ✅ Caches content |
| **Static IP** | ✅ 2 static Anycast IPs | ❌ (uses CloudFront domain) |
| **Protocol** | TCP, UDP | HTTP, HTTPS |
| **Use case** | Gaming, IoT, VoIP, non-HTTP | Web, API, media delivery |
| **Routing logic** | Latency + health-based | Cache + behavior rules |
| **Origin** | ALB, NLB, EC2, EIP | S3, ALB, API GW, custom |
| **DDoS** | Shield Standard included | Shield Standard included |

> **Key Exam Rule:** If the question mentions **HTTP/HTTPS with caching** → CloudFront. If it mentions **static IPs**, **non-HTTP (gaming/IoT/VoIP)**, or **TCP/UDP global acceleration** → Global Accelerator.

---

### Global Accelerator Endpoint Groups

- One endpoint group per AWS region.
- Traffic dial: control % of traffic to each region (0–100%).
- Useful for gradual region migrations and blue/green deployments.

---

### Key Exam Scenarios

- **Static IPs for global application (firewall whitelisting)**: Global Accelerator.
- **Multiplayer gaming with low latency globally**: Global Accelerator.
- **Instant failover between regions for TCP app**: Global Accelerator (< 30 sec).
- **HTTP REST API with caching globally**: CloudFront.
- **Gradual traffic shift between regions**: Global Accelerator traffic dial.

---

---

## 6. AWS Direct Connect

### What Problem Does It Solve?
Direct Connect provides a **dedicated private network connection** from your on-premises data center to AWS — bypassing the public internet for **consistent bandwidth, lower latency, and reduced data transfer costs**.

---

### Connection Types

| Type | Speed | Description |
|---|---|---|
| **Dedicated Connection** | 1, 10, 100 Gbps | Physical port at AWS Direct Connect location |
| **Hosted Connection** | 50 Mbps – 10 Gbps | Provisioned by AWS Partner; sub-1G speeds available |

---

### Virtual Interfaces (VIFs)

| VIF Type | Description | Connects To |
|---|---|---|
| **Private VIF** | Access resources in your VPC via private IPs | VPC (via VGW or TGW) |
| **Public VIF** | Access AWS public services (S3, DynamoDB) via AWS IPs (not internet) | AWS public endpoints |
| **Transit VIF** | Connect to Transit Gateway | Transit Gateway (for many VPCs) |

---

### Direct Connect Gateway

- Connect **one or more VPCs across multiple regions** using a single Direct Connect connection.
- Does NOT allow VPC-to-VPC communication through the gateway.
- Use with: Private VIF or Transit VIF.

```
On-Premises ──DX──→ Direct Connect Gateway → VPC (us-east-1)
                                            → VPC (eu-west-1)
                                            → Transit Gateway
```

---

### Direct Connect Resilience

| Pattern | Description | SLA |
|---|---|---|
| **Single connection** | One DX connection | No redundancy |
| **Dual connections (same location)** | Redundant at device level | Device failure protection |
| **Dual locations** | Two DX locations | Location failure protection |
| **Maximum resilience** | 2 connections at 2 locations | Highest availability |

> **Exam Tip:** For high availability, **always pair Direct Connect with a Site-to-Site VPN as backup**.

---

### Direct Connect + VPN (Backup)

- DX as primary; Site-to-Site VPN as failover.
- Use BGP routing: DX has lower cost metric (preferred); VPN activates when DX fails.

---

### MACsec

- Layer 2 encryption on Direct Connect.
- Encrypts data in transit at the link layer (10G and 100G dedicated connections).
- For compliance requirements needing encryption at L2.

---

### Direct Connect vs Site-to-Site VPN

| Feature | Direct Connect | Site-to-Site VPN |
|---|---|---|
| **Connection** | Dedicated private line | Encrypted tunnel over internet |
| **Bandwidth** | Up to 100 Gbps | Up to 1.25 Gbps per tunnel |
| **Latency** | Consistent, low | Variable (internet) |
| **Setup time** | Weeks to months | Minutes to hours |
| **Cost** | Higher | Lower |
| **Encryption** | Not by default (MACsec optional) | Always encrypted (IPsec) |
| **Use case** | Consistent, high-volume, compliance | Quick setup, backup, lower cost |

---

### Key Exam Scenarios

- **Dedicated high-bandwidth to AWS**: Direct Connect (dedicated).
- **Sub-1 Gbps connection via partner**: Direct Connect (hosted).
- **Access multiple VPCs across regions from on-prem**: Direct Connect Gateway.
- **Access S3/DynamoDB without internet from on-prem**: Direct Connect Public VIF.
- **Maximum resiliency**: Two connections at two DX locations.
- **DX backup**: Site-to-Site VPN as failover.
- **Consistent latency + compliance**: Direct Connect (over VPN).

---

---

## 7. AWS Site-to-Site VPN

### What Problem Does It Solve?
Site-to-Site VPN creates an **encrypted IPsec tunnel over the public internet** connecting your on-premises network to an AWS VPC — quickly and at low cost.

---

### Components

| Component | Description |
|---|---|
| **Virtual Private Gateway (VGW)** | AWS side of VPN; attached to a VPC |
| **Transit Gateway (TGW)** | Alternative to VGW for connecting many VPCs |
| **Customer Gateway (CGW)** | Represents your on-prem VPN device (IP + BGP/static config) |
| **VPN Connection** | Two IPsec tunnels (HA); one per AZ |

---

### VPN Tunnel Details

- **2 tunnels per VPN connection** for redundancy (different AZs on AWS side).
- Each tunnel: up to **1.25 Gbps**.
- Protocols: IKEv1 / IKEv2, AES-128/256, SHA-1/2.
- Routing: **Static** or **BGP (dynamic)**.

---

### Accelerated Site-to-Site VPN

- Routes VPN traffic through **Global Accelerator** edge locations.
- Reduces internet hops → lower latency for VPN tunnels.
- Requires Transit Gateway (not VGW).

---

### VPN CloudHub

- Connect **multiple on-premises sites** to a single VGW (hub-and-spoke).
- On-prem sites can communicate with each other via AWS hub.
- Each site needs its own Customer Gateway.
- Low-cost WAN alternative.

---

### Key Exam Scenarios

- **Quick encrypted connection to VPC**: Site-to-Site VPN.
- **Multiple branch offices to AWS**: VPN CloudHub.
- **VPN for multiple VPCs**: TGW + VPN attachment.
- **Reduce VPN latency globally**: Accelerated Site-to-Site VPN + TGW.
- **Backup for Direct Connect**: Site-to-Site VPN (automatic BGP failover).

---

---

## 8. AWS Client VPN

### What Problem Does It Solve?
Client VPN provides **remote access VPN for individual users** (employees, developers) to securely connect to AWS VPCs and on-premises networks from any location using an **OpenVPN-based client**.

---

### Key Properties

| Property | Details |
|---|---|
| **Protocol** | OpenVPN (TLS) |
| **Authentication** | Active Directory, SAML-based IdP, certificate-based (mutual TLS) |
| **Authorization** | Network-based authorization rules (CIDR ranges) |
| **Split tunneling** | Route only AWS-bound traffic through VPN (reduces bandwidth) |
| **Logging** | CloudWatch Logs |
| **Scaling** | Auto-scales based on connected users |

---

### Client VPN vs Site-to-Site VPN

| Feature | Client VPN | Site-to-Site VPN |
|---|---|---|
| **Connects** | Individual user device | Entire on-prem network |
| **Protocol** | OpenVPN | IPsec |
| **Use case** | Remote workers | Branch office/DC to AWS |
| **Auth** | AD, SAML, certificates | Shared key or certificates |
| **Scale** | Per user | Per site |

---

### Key Exam Scenarios

- **Remote employees access VPC resources**: Client VPN.
- **Developer accessing private RDS from home**: Client VPN.
- **Authenticate VPN users with corporate AD**: Client VPN + Active Directory authentication.
- **Reduce client VPN bandwidth usage**: Enable split tunneling.

---

---

## 9. AWS Transit Gateway

### What Problem Does It Solve?
Transit Gateway acts as a **regional network hub** that simplifies complex VPC-to-VPC and VPC-to-on-premises connectivity — replacing the unmanageable mesh of VPC peering connections with a single hub-and-spoke model.

---

### What Can Connect to TGW

| Attachment Type | Description |
|---|---|
| **VPC** | Attach VPCs in same or different accounts (via RAM) |
| **Site-to-Site VPN** | Connect on-prem via IPsec VPN |
| **Direct Connect Gateway** | Connect on-prem via DX |
| **TGW Peering** | Connect TGWs in different regions |
| **SD-WAN appliance** | Via VPN attachment |

---

### Transit Gateway Route Tables

- Each TGW has one or more **route tables**.
- Attachments are associated with one route table (for inbound route lookup).
- Attachments can propagate routes to a route table.
- **Route isolation**: Create separate route tables to prevent certain VPCs from communicating.

---

### TGW vs VPC Peering

| Feature | Transit Gateway | VPC Peering |
|---|---|---|
| **Transitive routing** | ✅ | ❌ |
| **Scale** | Thousands of VPCs | Point-to-point (N²) |
| **Cross-region** | ✅ (TGW peering) | ✅ (inter-region peering) |
| **Cross-account** | ✅ (RAM) | ✅ |
| **Bandwidth** | Up to 50 Gbps per attachment | Effectively unlimited |
| **Centralized control** | ✅ | ❌ |
| **Cost** | Per attachment + data | Per GB (no hourly) |

---

### TGW Network Topologies

#### Hub-and-Spoke (Default)
```
Spoke VPC A ──┐
Spoke VPC B ──┤── Transit Gateway ── On-Premises (VPN/DX)
Spoke VPC C ──┘
```

#### Isolated VPCs (Multiple Route Tables)
```
Prod VPCs ──→ Prod Route Table ──→ Shared Services VPC
Dev VPCs ──→ Dev Route Table ──→ Shared Services VPC
(Prod ↔ Dev communication blocked by separate route tables)
```

#### Centralized Inspection
```
All VPCs → TGW → Inspection VPC (Network Firewall / GWLB)
                       ↓
                  Internet / On-Prem
```

---

### TGW Multicast

- Supports **IP multicast** across VPCs — for video streaming, financial tick data.
- Only TGW supports multicast (VPC Peering does not).

---

### Key Exam Scenarios

- **Connect 10+ VPCs without N² peering**: Transit Gateway.
- **On-prem to multiple VPCs via one DX**: DX Gateway + TGW + Transit VIF.
- **Isolate prod and dev VPCs but share services**: TGW with separate route tables.
- **Centralized firewall inspection for all VPC traffic**: TGW + Inspection VPC + Network Firewall.
- **Share TGW across accounts in org**: TGW + RAM.
- **Connect TGWs in us-east-1 and eu-west-1**: TGW peering.

---

---

## 10. AWS PrivateLink

### What Problem Does It Solve?
PrivateLink provides **private connectivity between VPCs, AWS services, and on-premises** using **Interface Endpoints** — without exposing traffic to the public internet, VPC peering, NAT, or IGW.

---

### PrivateLink Architecture

```
Consumer VPC ──Interface Endpoint (ENI)──→ PrivateLink ──→ Provider Service
                        (private IP)                         (behind NLB)
```

---

### Use Cases

| Use Case | Description |
|---|---|
| **AWS service access** | Access S3, DynamoDB, SSM, Secrets Manager, etc. privately |
| **Expose your own service** | Put NLB in front of your service → share as PrivateLink endpoint service |
| **SaaS integration** | Third-party SaaS accessible via PrivateLink (no internet) |
| **Cross-account service** | Share service between accounts without VPC peering |

---

### PrivateLink vs VPC Peering vs VPC Gateway Endpoint

| Feature | PrivateLink (Interface Endpoint) | VPC Peering | Gateway Endpoint |
|---|---|---|---|
| **Services** | Any AWS/custom service | VPCs only | S3 and DynamoDB only |
| **Overlapping CIDR** | Allowed | ❌ Not allowed | N/A |
| **Transitive** | No (point-to-point) | ❌ | N/A |
| **Cost** | Hourly + data | Per GB | **Free** |
| **On-prem access** | ✅ (via DX/VPN) | ❌ | ❌ |
| **Security** | ENI + Security Group | Route table | Route table |

---

### PrivateLink for On-Prem

- On-prem can access services via Interface Endpoint **through Direct Connect or VPN** → no internet needed.

---

### Key Exam Scenarios

- **Access SSM/Secrets Manager from private EC2 without NAT**: Interface Endpoint (PrivateLink).
- **Expose internal microservice to partner account without peering**: PrivateLink endpoint service (NLB).
- **SaaS provider expose service to customers privately**: PrivateLink.
- **Overlapping CIDRs between consumer and provider**: PrivateLink (works; peering doesn't).
- **On-prem access to S3 privately via DX**: Interface Endpoint (in VPC) + Direct Connect.

---

---

## Master Topology: All Networking Services Together

```
                        INTERNET
                            │
                    Route 53 (DNS)
                            │
                 CloudFront (CDN/WAF)
                            │
              Global Accelerator (Static IPs)
                            │
                   ┌────────┴────────┐
              ALB (L7)          NLB (L4)
                   │                │
            ┌──────┴──────┐         │
          EC2/ECS      Lambda    EC2/ECS
            │
         ──── VPC ────────────────────────────────────
         │  Public Subnet                             │
         │  [ALB][NAT GW][Bastion]                   │
         │                                            │
         │  Private Subnet                            │
         │  [EC2][RDS][ECS]                          │
         │     │                                      │
         │  VPC Endpoint (S3/DynamoDB)               │
         │     │                                      │
         └─────┼──────────────────────────────────────┘
               │
         ┌─────┴─────────────────────────────────┐
         │          Transit Gateway               │
         ├─── VPC (Prod)                          │
         ├─── VPC (Dev)                           │
         ├─── Inspection VPC (Network Firewall)   │
         ├─── Site-to-Site VPN                    │
         └─── Direct Connect Gateway              │
                            │                     │
                     On-Premises DC               │
                     [Client VPN users] ──────────┘
```

---

## Critical Comparisons for the Exam

### Connectivity: On-Premises to AWS

| Scenario | Solution |
|---|---|
| Quick encrypted connection | Site-to-Site VPN |
| High bandwidth, consistent latency | Direct Connect |
| Highly available hybrid | Direct Connect + VPN backup |
| Multiple branch offices | VPN CloudHub |
| Remote users | Client VPN |
| On-prem → many VPCs | Direct Connect + TGW + Transit VIF |

### VPC-to-VPC Connectivity

| Scenario | Solution |
|---|---|
| Two VPCs, simple | VPC Peering |
| Many VPCs, transitive | Transit Gateway |
| Expose service (no peering) | PrivateLink |
| Overlapping CIDRs | PrivateLink |
| Share subnets across accounts | RAM |

### Global Traffic Distribution

| Scenario | Solution |
|---|---|
| DNS-level routing | Route 53 (latency / failover / weighted) |
| Content caching at edge | CloudFront |
| Static IPs, non-HTTP global | Global Accelerator |
| HTTP API globally | CloudFront or Route 53 + ALB |

### Load Balancer Selection

| Scenario | Load Balancer |
|---|---|
| HTTP/HTTPS routing by path/host | ALB |
| High-performance, static IP, TCP/UDP | NLB |
| Third-party firewall appliance | GWLB |
| Lambda as backend | ALB |
| PrivateLink exposure | NLB |

---

## Architecture Patterns to Know

### Pattern 1: Multi-Region Active-Active
```
Route 53 (Latency routing)
    ├── us-east-1: CloudFront → ALB → EC2 → RDS
    └── eu-west-1: CloudFront → ALB → EC2 → RDS (replica)
```

### Pattern 2: Hybrid Cloud Network
```
On-Prem DC ──Direct Connect (primary)──┐
            ──Site-to-Site VPN (backup)─┤
                                        └→ Transit Gateway
                                              ├── Prod VPC
                                              ├── Dev VPC
                                              └── Shared Services VPC
```

### Pattern 3: Centralized Ingress/Egress
```
Internet → ALB (in Ingress VPC)
                → Transit Gateway
                      ├── App VPC A
                      ├── App VPC B
                      └── Egress VPC (NAT Gateway → Internet)
```

### Pattern 4: Private SaaS Service
```
Consumer Account VPC
  └── Interface Endpoint (ENI, private IP)
            ↓ PrivateLink
Provider Account VPC
  └── NLB → EC2 (SaaS Service)
```

### Pattern 5: Serverless Global API
```
Users globally → Route 53 (latency routing)
                      ↓
                 CloudFront (cache GET responses)
                      ↓
                 API Gateway → Lambda → DynamoDB Global Tables
```

---

## Exam Tips Summary

### VPC
- ✅ SGs = stateful, instance-level, allow-only; NACLs = stateless, subnet-level, allow+deny
- ✅ NAT Gateway in public subnet; private subnet instances route to it
- ✅ Gateway Endpoint = free; S3/DynamoDB only; route-table-based
- ✅ Interface Endpoint = PrivateLink; all services; hourly cost
- ✅ VPC peering = no transitive routing; no overlapping CIDRs
- ✅ AWS reserves 5 IPs per subnet
- ✅ Flow Logs = metadata only; not payload; goes to S3/CloudWatch/Firehose

### ELB
- ✅ ALB = L7; path/host routing; Lambda target; Cognito auth
- ✅ NLB = L4; static IP; source IP preserve; PrivateLink; ultra-low latency
- ✅ GWLB = security appliances; GENEVE protocol
- ✅ ALB: cross-zone LB free; NLB/GWLB: cross-zone LB charged
- ✅ Deregistration delay = drain in-flight requests (0–3600s, default 300s)

### Route 53
- ✅ Alias = works at zone apex (root domain); free; AWS resources only
- ✅ CNAME = cannot be at zone apex; charged
- ✅ Latency routing ≠ geolocation routing
- ✅ Failover = active-passive; requires health check on primary
- ✅ Private hosted zone health checks → must use CloudWatch alarm
- ✅ Resolver inbound = on-prem queries Route 53; outbound = Route 53 queries on-prem

### CloudFront
- ✅ OAC (not OAI) for restricting S3 access
- ✅ Signed URL = single file; Signed Cookie = multiple files
- ✅ CloudFront Functions = < 1ms, viewer triggers only
- ✅ Lambda@Edge = heavier, all 4 trigger types, network access
- ✅ Price Class 100 = cheapest, US/EU/Asia only
- ✅ ACM cert for CloudFront must be in us-east-1

### Global Accelerator
- ✅ 2 static Anycast IPs; TCP/UDP; no caching
- ✅ Use when: non-HTTP, static IPs needed, gaming/IoT/VoIP
- ✅ CloudFront = HTTP caching; Global Accelerator = TCP/UDP acceleration

### Direct Connect
- ✅ Dedicated = 1/10/100 Gbps; Hosted = 50 Mbps–10 Gbps (via partner)
- ✅ Private VIF → VPC; Public VIF → AWS public services; Transit VIF → TGW
- ✅ DX Gateway = multi-region multi-VPC from one DX
- ✅ Not encrypted by default; use MACsec or IPsec VPN over DX
- ✅ Maximum resiliency = 2 connections × 2 locations

### Site-to-Site VPN
- ✅ 2 tunnels per connection; 1.25 Gbps each
- ✅ VPN CloudHub = multiple on-prem sites via single VGW
- ✅ Accelerated VPN = Global Accelerator + TGW
- ✅ Always use VPN as DX backup

### Transit Gateway
- ✅ Hub-and-spoke; transitive routing; replaces mesh peering
- ✅ Multiple route tables = traffic isolation between VPCs
- ✅ Share via RAM across accounts
- ✅ TGW peering = connect TGWs across regions
- ✅ Only TGW supports multicast

### PrivateLink
- ✅ Interface Endpoint = ENI in your subnet; PrivateLink under the hood
- ✅ Works with overlapping CIDRs (unlike peering)
- ✅ Expose own service: put NLB in front → create endpoint service
- ✅ On-prem access via DX/VPN through Interface Endpoint
- ✅ Gateway Endpoint (S3/DynamoDB) is free; Interface Endpoint costs money

---
---

<a name="security"></a>
# PART 6 — Security, Identity & Compliance: IAM, Cognito, KMS, WAF, Shield, GuardDuty, Inspector, Macie, Detective, Security Hub & more

# AWS Exam Study Guide: Security, Identity & Compliance
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Artifact · Audit Manager · ACM · CloudHSM · Cognito · Detective · Directory Service · Firewall Manager · GuardDuty · IAM Identity Center · IAM · Inspector · KMS · Macie · Network Firewall · RAM · Secrets Manager · Security Hub · Shield · WAF

---

## Security Service Quick-Reference Matrix

| Service | Category | What It Does | Exam Trigger Words |
|---|---|---|---|
| **IAM** | Identity & Access | Users, roles, policies, permissions | "who can access what", "least privilege" |
| **IAM Identity Center** | SSO/Federation | Single sign-on across accounts/apps | "SSO", "multi-account access", "SAML federation" |
| **Cognito** | App Identity | User auth for web/mobile apps | "user sign-up/login", "social login", "app authentication" |
| **Directory Service** | Directory | Managed AD in AWS | "Active Directory", "LDAP", "domain join" |
| **KMS** | Encryption Keys | Managed encryption key service | "encryption", "key rotation", "CMK" |
| **CloudHSM** | Hardware Keys | Dedicated HSM appliance | "FIPS 140-2 Level 3", "dedicated hardware", "custom key store" |
| **ACM** | Certificates | TLS/SSL certificate management | "HTTPS", "SSL cert", "certificate renewal" |
| **Secrets Manager** | Secrets | Store and rotate secrets | "DB password", "API key rotation", "secret rotation" |
| **WAF** | Web Protection | Block malicious web requests | "SQL injection", "XSS", "Layer 7 protection" |
| **Shield** | DDoS Protection | Protect against DDoS attacks | "DDoS", "volumetric attack", "SYN flood" |
| **Network Firewall** | Network Protection | VPC-level stateful firewall | "stateful inspection", "IDS/IPS", "VPC firewall" |
| **Firewall Manager** | Policy Mgmt | Central firewall policy across accounts | "central security policy", "Organizations WAF rules" |
| **GuardDuty** | Threat Detection | ML-based threat detection | "unusual behavior", "compromised credentials", "crypto mining" |
| **Inspector** | Vulnerability Scan | Automated vulnerability assessment | "CVE", "EC2/ECR scanning", "vulnerability findings" |
| **Macie** | Data Discovery | Find/protect sensitive data in S3 | "PII", "sensitive data in S3", "GDPR data discovery" |
| **Detective** | Investigation | Root cause analysis of security events | "investigate", "visualize", "drill into finding" |
| **Security Hub** | Aggregation | Central security findings dashboard | "single pane of glass", "compliance score", "aggregate findings" |
| **Audit Manager** | Compliance | Map AWS usage to compliance frameworks | "PCI DSS", "SOC 2", "HIPAA audit evidence" |
| **Artifact** | Compliance Docs | AWS compliance reports/agreements | "AWS compliance report", "SOC report", "BAA" |
| **RAM** | Resource Sharing | Share AWS resources across accounts | "share subnets", "cross-account resource", "Transit Gateway share" |

---

---

## 1. AWS Identity and Access Management (IAM)

### What Problem Does It Solve?
IAM controls **who can do what** in your AWS account. It provides authentication (who you are) and authorization (what you're allowed to do) for every AWS API call.

---

### Core Components

| Component | Description |
|---|---|
| **User** | Permanent identity for a person or application; has long-term credentials |
| **Group** | Collection of users; attach policies to groups |
| **Role** | Temporary identity assumed by users, services, or external identities; no long-term credentials |
| **Policy** | JSON document defining permissions (Allow/Deny + Actions + Resources + Conditions) |

---

### Policy Types

| Type | Scope | Description |
|---|---|---|
| **Identity-based** | User/Group/Role | Grants permissions to an identity |
| **Resource-based** | Resource (S3, KMS, etc.) | Grants permissions on the resource itself; supports cross-account |
| **Permissions boundary** | User/Role | Maximum permissions an identity can have (ceiling, not grant) |
| **Service Control Policy (SCP)** | OU/Account (Organizations) | Max permissions for entire account; overrides all IAM |
| **Session policy** | Assumed role session | Limits permissions for a specific session |
| **ACL** | Resource | Legacy; cross-account only; limited to S3, VPC |

> **Exam Tip — Policy Evaluation Logic:**
> 1. Explicit **Deny** always wins
> 2. SCP must allow (Organizations)
> 3. Resource-based policy OR identity-based policy must allow
> 4. Permissions boundary must allow
> 5. Default = **implicit Deny**

---

### IAM Roles — Key Patterns

| Pattern | Description |
|---|---|
| **EC2 Instance Profile** | Attach role to EC2; apps get temp credentials via metadata |
| **Cross-account role** | Account A assumes role in Account B |
| **Service role** | Lambda, ECS task, etc. assume role to call AWS services |
| **Web Identity / OIDC** | Federated users (Google, GitHub Actions, EKS pods) assume role |

---

### IAM Trust Policy vs Permission Policy

- **Trust Policy**: Who can **assume** this role (principal: service, account, user).
- **Permission Policy**: What the role **can do** (actions on resources).

---

### IAM Best Practices (Exam-Tested)

- Enable **MFA** for root and privileged users.
- Never use root account for day-to-day tasks.
- Use **roles** for EC2/Lambda/ECS, never embed access keys.
- Use **Permissions Boundaries** to delegate safe role creation.
- Use **IAM Access Analyzer** to find unintended external access.
- Rotate access keys; use **Credential Report** for auditing.

---

### IAM Conditions (Common Exam Examples)

```json
"Condition": {
  "StringEquals": { "aws:RequestedRegion": "us-east-1" },
  "Bool": { "aws:MultiFactorAuthPresent": "true" },
  "IpAddress": { "aws:SourceIp": "203.0.113.0/24" },
  "StringEquals": { "s3:prefix": "home/${aws:username}/" }
}
```

---

### Key Exam Scenarios

- **EC2 needs to access S3**: Attach IAM role via instance profile — never use access keys.
- **Developer can create roles but not escalate privileges**: Use Permissions Boundary.
- **Restrict actions to a specific region**: Use `aws:RequestedRegion` condition.
- **Audit all IAM users and their key age**: IAM Credential Report.
- **Find who has external access to resources**: IAM Access Analyzer.
- **Cross-account access without new user**: Cross-account IAM role + sts:AssumeRole.

---

---

## 2. AWS IAM Identity Center (AWS Single Sign-On)

### What Problem Does It Solve?
IAM Identity Center provides **centralized SSO** across multiple AWS accounts (in an AWS Organization) and business applications (Salesforce, Office 365, etc.). Replaces the need to create IAM users in every account.

---

### Key Concepts

| Concept | Description |
|---|---|
| **Permission Sets** | Define what access a user gets in an AWS account (like IAM roles) |
| **Identity Source** | Where users/groups come from |
| **AWS Account Access** | Assign users/groups to accounts with a permission set |
| **Application Assignments** | SAML 2.0 app integrations |

---

### Identity Sources

| Source | Description |
|---|---|
| **IAM Identity Center directory** | Built-in user directory (default) |
| **Active Directory** | AWS Managed AD or AD Connector |
| **External IdP** | Okta, Azure AD, Ping — via SAML 2.0 / SCIM |

---

### IAM Identity Center vs IAM Roles (Cross-Account)

| Feature | IAM Identity Center | IAM Cross-Account Roles |
|---|---|---|
| **Scale** | Manage all accounts centrally | Configure per account |
| **MFA** | Centralized | Per-account |
| **User source** | External IdP, AD, built-in | IAM users only |
| **Audit** | CloudTrail + unified access portal | CloudTrail per account |
| **Use Case** | Multi-account, enterprise | Simple cross-account |

---

### Key Exam Scenarios

- **Single sign-on across 50 AWS accounts**: IAM Identity Center + AWS Organizations.
- **Employees log in with corporate AD to access AWS**: IAM Identity Center + AD Connector or Managed AD.
- **Federate with Okta/Azure AD**: IAM Identity Center with external IdP (SAML 2.0 + SCIM).
- **Least privilege across all accounts from one place**: IAM Identity Center + Permission Sets.

---

---

## 3. Amazon Cognito

### What Problem Does It Solve?
Cognito provides **authentication, authorization, and user management for web and mobile applications** — without building your own auth system. It supports sign-up, sign-in, social login, MFA, and token-based access to AWS services.

---

### Two Main Components

#### Cognito User Pools
- **User directory** for app authentication.
- Handles sign-up, sign-in, MFA, password reset.
- Integrates with social IdPs (Google, Facebook, Apple) and enterprise IdPs (SAML).
- Issues **JWT tokens** (ID token, access token, refresh token).
- Use with: API Gateway, AppSync for app-level auth.

#### Cognito Identity Pools (Federated Identities)
- Provides **temporary AWS credentials** to authenticated (or unauthenticated) users.
- Input: JWT from User Pool, social IdP token, SAML assertion.
- Output: STS credentials → access AWS services directly (S3, DynamoDB).
- Supports **unauthenticated (guest) access**.

---

### Cognito Flow

```
User → User Pool (authenticate) → JWT token
                                       ↓
                              Identity Pool → STS → Temp AWS credentials
                                       ↓
                               Access S3 / DynamoDB directly
```

---

### Cognito vs IAM Identity Center

| Feature | Cognito | IAM Identity Center |
|---|---|---|
| **Target** | App users (customers) | Workforce users (employees) |
| **Protocol** | OAuth 2.0, OIDC, SAML | SAML 2.0, SCIM |
| **Tokens** | JWT (ID/access/refresh) | AWS temporary credentials |
| **Use Case** | Mobile/web app auth | AWS console / CLI access |
| **Scale** | Millions of users | Thousands of employees |

---

### Key Exam Scenarios

- **Mobile app user sign-in**: Cognito User Pool.
- **Mobile app user needs to upload to S3 directly**: Cognito User Pool → Identity Pool → STS credentials.
- **Social login (Google/Facebook) for app**: Cognito User Pool with social IdP.
- **Guest access to limited AWS resources**: Cognito Identity Pool unauthenticated role.
- **API Gateway authorization with app users**: Cognito User Pool authorizer.

---

---

## 4. AWS Directory Service

### What Problem Does It Solve?
Directory Service brings **Microsoft Active Directory (AD) capabilities into AWS**, enabling domain-joined EC2 instances, LDAP authentication, and AD-integrated applications without maintaining on-premises domain controllers.

---

### Directory Service Types

#### 1. AWS Managed Microsoft AD
- **Full Microsoft AD** (2019) hosted and managed by AWS.
- Multi-AZ; automatic patching, backups, snapshots.
- Supports Group Policy, trusts with on-prem AD, MFA (RADIUS).
- Best for: New AD in AWS or extending on-prem AD to AWS.

#### 2. AD Connector
- **Proxy** that redirects AD authentication requests to on-premises AD.
- No AD data stored in AWS — all queries go to on-prem.
- Use when you want to reuse existing on-prem AD without syncing data.
- Cannot function if on-prem AD is unavailable (single point of failure).

#### 3. Simple AD
- **Standalone, Samba-based** lightweight AD-compatible directory.
- No trust relationships; no MFA; no AD-specific features.
- Low cost; suitable for small, simple workloads (< 5,000 users).
- Cannot join with on-prem AD.

---

### Directory Service Comparison

| Feature | Managed Microsoft AD | AD Connector | Simple AD |
|---|---|---|---|
| **Full AD** | ✅ | Proxy only | ✅ (limited) |
| **On-prem trust** | ✅ (forest/domain trust) | ✅ (redirects) | ❌ |
| **Data in AWS** | ✅ | ❌ | ✅ |
| **MFA** | ✅ (RADIUS) | ✅ (RADIUS) | ❌ |
| **Scale** | Up to 500K objects | Any (on-prem) | Up to 30K objects |
| **Use case** | Full AD in cloud | Extend on-prem | Small/simple |

---

### Key Exam Scenarios

- **EC2 instances need to join on-prem AD domain**: AD Connector.
- **New AD in AWS, no on-prem**: AWS Managed Microsoft AD.
- **SSO via IAM Identity Center with existing corporate AD**: AD Connector or Managed AD.
- **Small application needing basic LDAP**: Simple AD.
- **On-prem AD + AWS AD trust**: Managed Microsoft AD with forest trust.

---

---

## 5. AWS Key Management Service (AWS KMS)

### What Problem Does It Solve?
KMS provides **centralized, managed cryptographic key creation, storage, and control**. It integrates with virtually all AWS services to encrypt data at rest. Keys never leave KMS unencrypted; all encryption/decryption happens inside the service.

---

### KMS Key Types

| Type | Description | Control |
|---|---|---|
| **AWS Managed Keys** | Created/managed by AWS for each service (`aws/s3`, `aws/ebs`) | Limited; automatic rotation |
| **Customer Managed Keys (CMK)** | Created by you in KMS | Full control; manual or automatic rotation |
| **Customer-Provided Keys (SSE-C)** | You provide keys (S3 only) | Full control; AWS uses and discards |
| **AWS Owned Keys** | Fully managed by AWS; not visible to you | None |

---

### KMS Key Material Origin

| Origin | Description |
|---|---|
| **KMS** | AWS generates key material |
| **External (BYOK)** | You import your own key material |
| **Custom Key Store (CloudHSM)** | Key material in your dedicated HSM |

---

### Key Rotation

- **AWS Managed Keys**: Auto-rotated every **1 year** (cannot change).
- **CMK**: Auto-rotation optional; rotates every **1 year** if enabled. Old key material retained for decryption.
- **Imported key material**: **No automatic rotation**; must rotate manually.

---

### KMS Operations

| Operation | Description |
|---|---|
| **Encrypt** | Encrypt up to **4 KB** of data directly |
| **Decrypt** | Decrypt data encrypted by KMS |
| **GenerateDataKey** | Returns plaintext + encrypted data key (envelope encryption) |
| **GenerateDataKeyWithoutPlaintext** | Returns only encrypted data key (used for async workflows) |

---

### Envelope Encryption *(High-Frequency Exam Topic)*

1. KMS generates a **Data Encryption Key (DEK)**.
2. Plaintext DEK encrypts your data locally.
3. KMS encrypts the DEK with the CMK.
4. Encrypted data + encrypted DEK stored together.
5. To decrypt: KMS decrypts the DEK → DEK decrypts data locally.

> **Why?** KMS can only encrypt 4 KB directly. Envelope encryption allows encrypting large data without sending it to KMS.

---

### KMS Key Policies

- **Every KMS key has a key policy** (resource-based policy).
- By default, key policy must explicitly allow the account root.
- Without `kms:*` in key policy for root, even account admin can be locked out.
- Use key policy + IAM policy for fine-grained access.

---

### Multi-Region Keys

- Replicate a CMK across regions with the **same key ID, key material, and automatic rotation**.
- Use case: Encrypt in us-east-1, decrypt in eu-west-1 (DR, global apps).
- Keys are independent but interoperable.

---

### KMS vs CloudHSM

| Feature | KMS | CloudHSM |
|---|---|---|
| **Management** | Fully managed by AWS | You manage the HSM |
| **FIPS compliance** | FIPS 140-2 Level 2 | **FIPS 140-2 Level 3** |
| **Key control** | Shared HSM (logical isolation) | **Dedicated hardware** |
| **Integration** | Native with all AWS services | Requires custom app integration |
| **Performance** | High (shared) | Higher (dedicated) |
| **Cost** | Per API call | Per HSM-hour (~$1.45/hr) |
| **Use Case** | General encryption | Regulatory HSM requirement, custom CA |

---

### Key Exam Scenarios

- **Encrypt EBS/S3/RDS at rest**: KMS CMK with SSE-KMS.
- **Large data encryption**: Envelope encryption (GenerateDataKey).
- **Compliance requires dedicated hardware HSM (FIPS Level 3)**: CloudHSM.
- **Share encrypted snapshot across accounts**: Share CMK key policy + share snapshot.
- **Prevent key deletion by anyone**: KMS key deletion has mandatory 7–30 day waiting period.
- **Reduce KMS API throttling on S3**: Enable S3 Bucket Keys.

---

---

## 6. AWS CloudHSM

### What Problem Does It Solve?
CloudHSM provides a **dedicated Hardware Security Module** in AWS, giving you exclusive, single-tenant control over cryptographic keys at **FIPS 140-2 Level 3** — the highest level for regulatory compliance.

---

### Key Properties

| Property | Details |
|---|---|
| **Hardware** | Dedicated, single-tenant HSM |
| **FIPS Level** | 140-2 Level 3 |
| **Key ownership** | You; AWS has no access to keys |
| **Integration** | Custom apps via PKCS#11, JCE, CNG/KSP APIs |
| **HA** | Deploy HSM cluster across multiple AZs |
| **Use with KMS** | KMS Custom Key Store backed by CloudHSM |

---

### Key Exam Scenarios

- **FIPS 140-2 Level 3 required**: CloudHSM (KMS is Level 2).
- **Custom PKI / Certificate Authority**: CloudHSM.
- **Oracle TDE with customer-controlled keys**: CloudHSM.
- **SSL offloading with key on dedicated HSM**: CloudHSM.
- **KMS + dedicated hardware**: KMS Custom Key Store → CloudHSM.

---

---

## 7. AWS Certificate Manager (ACM)

### What Problem Does It Solve?
ACM provides **free, managed TLS/SSL certificates** for AWS services, with automatic renewal — eliminating the manual work of certificate procurement, deployment, and renewal.

---

### ACM Certificate Types

| Type | Description |
|---|---|
| **ACM-issued (public)** | Free; auto-renewed; domain validation (DV) |
| **ACM-issued (private)** | Private CA (ACM PCA); internal services |
| **Imported certificates** | Bring your own certificate; **no auto-renewal** |

---

### ACM Integration

ACM certificates can be used with:
- **Elastic Load Balancers** (ALB, NLB, CLB)
- **CloudFront distributions**
- **API Gateway**
- **Elastic Beanstalk**
- **AppSync**

> **Critical Exam Tip:** ACM certificates **cannot be used directly on EC2 instances**. They must be attached to supported AWS services. For EC2, export/install a certificate manually or use a third-party cert.

---

### ACM Private CA (AWS Private Certificate Authority)

- Create internal PKI hierarchy for private certificates.
- Issue certs for internal services, IoT devices, containers.
- Costs per CA per month + per certificate issued.

---

### ACM vs CloudHSM for Certificates

- **ACM**: Managed certs for AWS services; automatic renewal; free public certs.
- **CloudHSM**: Use when you need private keys in dedicated hardware; custom CA; bring-your-own PKI.

---

### Key Exam Scenarios

- **HTTPS for ALB/CloudFront**: ACM public certificate (free, auto-renews).
- **Internal microservices TLS**: ACM Private CA.
- **Certificate on EC2 directly**: Not ACM; use self-signed or third-party, install manually.
- **Centrally manage certs across regions**: ACM certificates are region-specific; provision separately per region.
- **CloudFront certificate**: Must be in **us-east-1** (N. Virginia) regardless of distribution origin.

---

---

## 8. AWS Secrets Manager

### What Problem Does It Solve?
Secrets Manager securely **stores, rotates, and retrieves secrets** (database credentials, API keys, OAuth tokens) — replacing hardcoded credentials in application code with API calls.

---

### Key Features

| Feature | Details |
|---|---|
| **Automatic rotation** | Lambda-based; built-in support for RDS, Redshift, DocumentDB |
| **Encryption** | KMS (SSE-KMS); always encrypted at rest |
| **Cross-account access** | Resource-based policy |
| **Versioning** | AWSCURRENT, AWSPENDING, AWSPREVIOUS labels |
| **Replication** | Replicate secrets across regions |

---

### Secrets Manager vs SSM Parameter Store

| Feature | Secrets Manager | SSM Parameter Store |
|---|---|---|
| **Cost** | $0.40/secret/month + API calls | Free (Standard); $0.05/param/month (Advanced) |
| **Auto-rotation** | ✅ Native (Lambda) | ❌ (manual Lambda) |
| **Encryption** | Always KMS | Optional KMS (SecureString) |
| **Cross-account** | ✅ | Limited |
| **Max size** | 65 KB | 4 KB (Standard), 8 KB (Advanced) |
| **Versioning** | ✅ | ✅ |
| **Use Case** | DB credentials, API keys needing rotation | Config values, non-sensitive params, feature flags |

> **Exam Rule:** "Auto-rotate database password" → **Secrets Manager**. "Store application config with optional encryption" → **SSM Parameter Store**.

---

### Rotation Mechanism

1. Secrets Manager invokes a **Lambda rotation function**.
2. Lambda creates new credentials on the DB → stores as AWSPENDING.
3. Lambda tests new credentials.
4. Lambda marks new as AWSCURRENT; old as AWSPREVIOUS.

---

### Key Exam Scenarios

- **RDS password auto-rotation every 30 days**: Secrets Manager.
- **Inject secret into ECS task**: Reference Secrets Manager ARN in task definition.
- **Lambda accessing DB without hardcoded password**: Secrets Manager API call at runtime.
- **Multi-region app needing same secret**: Secrets Manager replication to replica regions.
- **Config values, non-sensitive**: SSM Parameter Store (cheaper).

---

---

## 9. AWS WAF (Web Application Firewall)

### What Problem Does It Solve?
WAF protects web applications from **common web exploits** (SQL injection, XSS, bad bots, scrapers) at **Layer 7 (HTTP/HTTPS)** by inspecting and filtering web requests.

---

### What WAF Protects

| Resource | Notes |
|---|---|
| **CloudFront** | Global; protects at edge |
| **ALB** | Regional |
| **API Gateway** | Regional |
| **AppSync** | Regional |
| **Cognito User Pools** | Regional |

---

### WAF Components

| Component | Description |
|---|---|
| **Web ACL** | Container for rules; attached to a resource |
| **Rule** | Condition + action (Allow/Block/Count/CAPTCHA) |
| **Rule Group** | Reusable set of rules |
| **Managed Rule Groups** | AWS or marketplace pre-built rules (OWASP Top 10, bot control, etc.) |
| **IP Set** | List of IP addresses/CIDRs |
| **Regex Pattern Set** | Match patterns in request body/headers/URI |

---

### WAF vs Security Groups vs NACLs

| Layer | Service | Operates At |
|---|---|---|
| **Layer 7 (HTTP)** | WAF | HTTP headers, body, URI, query strings |
| **Layer 4 (TCP/IP)** | Security Groups, NACLs | IP, port, protocol |
| **Layer 3/4 (DDoS)** | Shield | Network/transport volumetric attacks |

---

### Key Exam Scenarios

- **Block SQL injection on ALB**: WAF with SQL injection match rule.
- **Rate limiting on API Gateway (100 req/5 min per IP)**: WAF rate-based rule.
- **Block requests from specific countries**: WAF geo-match rule.
- **Protect against OWASP Top 10**: WAF with AWS Managed Rules.
- **WAF across 20 accounts centrally**: AWS Firewall Manager.

---

---

## 10. AWS Shield

### What Problem Does It Solve?
Shield protects AWS applications from **Distributed Denial of Service (DDoS)** attacks at Layers 3 and 4 (network/transport).

---

### Shield Tiers

| Feature | Shield Standard | Shield Advanced |
|---|---|---|
| **Cost** | **Free** for all AWS customers | $3,000/month per organization |
| **Protection** | L3/L4 common DDoS attacks | L3/L4/L7 sophisticated attacks |
| **Auto mitigation** | ✅ | ✅ Enhanced |
| **DDoS Response Team (DRT)** | ❌ | ✅ 24/7 access |
| **Cost protection** | ❌ | ✅ DDoS scaling cost credits |
| **Attack diagnostics** | ❌ | ✅ Real-time metrics + reports |
| **WAF integration** | ❌ | ✅ Automatic WAF rules during attack |
| **Health-based detection** | ❌ | ✅ |

---

### Protected Resources (Shield Advanced)

- EC2 Elastic IPs
- ELB (ALB, NLB, CLB)
- CloudFront
- Route 53
- Global Accelerator

---

### Key Exam Scenarios

- **Basic DDoS protection at no cost**: Shield Standard (automatic for all).
- **24/7 DDoS response team + financial protection**: Shield Advanced.
- **Protect Route 53 from DNS DDoS**: Shield Advanced.
- **DDoS + WAF protection together**: Shield Advanced + WAF (integrated).
- **Financial protection from DDoS-induced scaling costs**: Shield Advanced cost protection.

---

---

## 11. AWS Network Firewall

### What Problem Does It Solve?
Network Firewall provides a **stateful, managed network firewall and IDS/IPS** at the **VPC level** — filtering traffic between VPCs, to/from the internet, and to AWS services with deep packet inspection.

---

### Key Features

| Feature | Details |
|---|---|
| **Traffic inspection** | Stateful + stateless rules |
| **IDS/IPS** | Suricata-compatible rules; signature-based detection |
| **Deployment** | Firewall endpoint per AZ in dedicated subnet |
| **Protocols** | HTTP/HTTPS, DNS, TCP, UDP, ICMP |
| **TLS inspection** | Decrypt and inspect TLS traffic |
| **Logging** | Flow logs, alert logs → S3, CloudWatch, Kinesis |

---

### Network Firewall vs WAF vs Security Groups vs NACLs

| Service | Layer | Scope | Use Case |
|---|---|---|---|
| **Security Groups** | L4 | Instance level | Allow/deny by IP/port per ENI |
| **NACLs** | L3/L4 | Subnet level | Stateless subnet-level filtering |
| **WAF** | L7 | CloudFront/ALB/API GW | HTTP request filtering |
| **Network Firewall** | L3–L7 | VPC level | Deep packet inspection, IDS/IPS, DNS filtering |
| **Shield** | L3/L4 | Edge | DDoS protection |

---

### Key Exam Scenarios

- **Block outbound connections to known malware domains**: Network Firewall with domain list rules.
- **IDS/IPS for VPC traffic**: Network Firewall with Suricata rules.
- **Inspect all traffic between VPCs in a hub-spoke**: Network Firewall in inspection VPC + Transit Gateway.
- **Centrally manage firewall rules across accounts**: AWS Firewall Manager + Network Firewall.

---

---

## 12. AWS Firewall Manager

### What Problem Does It Solve?
Firewall Manager provides **centralized security policy management** across accounts in an AWS Organization — ensuring WAF rules, Shield Advanced protections, Security Groups, and Network Firewall rules are consistently applied.

---

### Supported Policies

| Policy Type | Description |
|---|---|
| **WAF** | Apply WAF Web ACLs to ALBs, API GW, CloudFront across accounts |
| **Shield Advanced** | Enroll resources in Shield Advanced organization-wide |
| **Security Groups** | Audit and enforce Security Group rules |
| **Network Firewall** | Deploy Network Firewall policies across VPCs |
| **Route 53 Resolver DNS Firewall** | Apply DNS filtering policies centrally |

---

### Prerequisites

- AWS Organizations must be enabled.
- Firewall Manager administrator account must be designated.
- AWS Config must be enabled in all member accounts.

---

### Key Exam Scenarios

- **Apply WAF rules to all ALBs across 50 accounts**: Firewall Manager WAF policy.
- **Ensure all new accounts auto-enroll in Shield Advanced**: Firewall Manager Shield policy.
- **Centrally enforce Security Group rules**: Firewall Manager Security Group policy.
- **Detect non-compliant SGs across organization**: Firewall Manager audit policy.

---

---

## 13. Amazon GuardDuty

### What Problem Does It Solve?
GuardDuty is a **managed threat detection service** using ML, anomaly detection, and threat intelligence to identify malicious activity and unauthorized behavior in your AWS environment.

---

### Data Sources

| Source | What It Analyzes |
|---|---|
| **VPC Flow Logs** | Network traffic anomalies |
| **CloudTrail Events** | Unusual API calls, credential abuse |
| **DNS Logs** | C2 callbacks, crypto mining domains |
| **EKS Audit Logs** | Kubernetes API activity |
| **S3 Data Events** | Unusual S3 access patterns |
| **EBS Malware Scanning** | Malware in EC2/EBS workloads |
| **RDS Login Activity** | Unusual DB authentication |
| **Lambda Network Activity** | Suspicious Lambda network calls |

> **Exam Tip:** GuardDuty does **not** require you to enable these logs separately — it analyzes them independently. Enabling GuardDuty does not affect CloudTrail/VPC Flow Logs configuration.

---

### Finding Types (Examples)

- `UnauthorizedAccess:IAMUser/ConsoleLoginSuccess.B` — Login from unusual location.
- `CryptoCurrency:EC2/BitcoinTool.B` — Crypto mining detected.
- `Trojan:EC2/BlackholeTraffic` — Traffic to known malicious IPs.
- `CredentialAccess:IAMUser/AnomalousBehavior` — Unusual credential use.

---

### GuardDuty Multi-Account

- **Delegated Administrator**: One account manages GuardDuty for all org accounts.
- Findings aggregated centrally.
- Cannot be disabled by member accounts.

---

### Key Exam Scenarios

- **Detect compromised EC2 instance (crypto mining)**: GuardDuty.
- **Unusual IAM API calls at 3am from new IP**: GuardDuty + CloudTrail.
- **Centralized threat detection across org**: GuardDuty with Organizations delegated admin.
- **Automate response to GuardDuty finding**: GuardDuty → EventBridge → Lambda (isolate instance).
- **GuardDuty finding → SIEM**: GuardDuty → EventBridge → Kinesis → SIEM.

---

---

## 14. Amazon Inspector

### What Problem Does It Solve?
Inspector performs **automated, continuous vulnerability assessments** on EC2 instances, container images in ECR, and Lambda functions — identifying CVEs, network exposure, and software vulnerabilities.

---

### What Inspector Scans

| Target | What Is Checked |
|---|---|
| **EC2 instances** | CVEs, network reachability, OS/software vulnerabilities |
| **ECR container images** | CVEs in packages; on push or continuously |
| **Lambda functions** | CVE vulnerabilities in function code and layers |

---

### Inspector vs GuardDuty vs Macie

| Service | Focus | What It Does |
|---|---|---|
| **Inspector** | Vulnerabilities | Finds CVEs, misconfigs in EC2/ECR/Lambda |
| **GuardDuty** | Threats | Detects active malicious behavior |
| **Macie** | Data security | Finds sensitive data (PII) in S3 |

---

### Key Exam Scenarios

- **Find unpatched CVEs on EC2**: Amazon Inspector.
- **Scan container images for vulnerabilities before deployment**: Inspector + ECR integration.
- **Continuously assess Lambda for known vulnerabilities**: Inspector Lambda scanning.
- **Prioritize findings by severity and exploitability**: Inspector risk score.

---

---

## 15. Amazon Macie

### What Problem Does It Solve?
Macie uses **ML and pattern matching to discover and protect sensitive data in Amazon S3** — automatically identifying PII, financial data, credentials, and other sensitive content.

---

### Key Features

| Feature | Details |
|---|---|
| **Data discovery** | Scan S3 buckets for PII, PHI, financial data, credentials |
| **Managed identifiers** | 100+ built-in sensitive data types (credit cards, SSNs, passports) |
| **Custom identifiers** | Regex + keywords for proprietary data |
| **Bucket inventory** | Visibility into all S3 buckets (public, encrypted, shared) |
| **Policy findings** | Detect S3 misconfigurations (public bucket, no encryption) |

---

### Macie Finding Types

| Type | Description |
|---|---|
| **Policy findings** | Bucket became public, encryption disabled, cross-account access added |
| **Sensitive data findings** | PII, financial data, credentials found in objects |

---

### Key Exam Scenarios

- **Find PII in S3 data lake (GDPR compliance)**: Amazon Macie.
- **Alert when S3 bucket becomes public**: Macie policy finding → EventBridge → SNS.
- **Classify all S3 data for compliance**: Macie data discovery job.
- **Find hardcoded AWS credentials in S3**: Macie credential detection.

---

---

## 16. Amazon Detective

### What Problem Does It Solve?
Detective **automatically collects and analyzes log data** to help you **investigate security incidents** — visualizing relationships between AWS resources, users, and IP addresses to identify the root cause of a finding.

---

### Data Sources

- VPC Flow Logs
- CloudTrail
- GuardDuty findings
- EKS Audit Logs

---

### Detective vs GuardDuty vs Security Hub

| Service | Role | Analogy |
|---|---|---|
| **GuardDuty** | Detects threats | Security alarm |
| **Security Hub** | Aggregates findings | Dashboard |
| **Detective** | Investigates findings | Forensic analyst |

---

### Key Exam Scenarios

- **GuardDuty finding — need to investigate root cause**: Amazon Detective.
- **Visualize which IP was communicating with compromised EC2**: Detective.
- **Triage security incident across CloudTrail + VPC logs**: Detective.

---

---

## 17. AWS Security Hub

### What Problem Does It Solve?
Security Hub provides a **single pane of glass** for security findings across AWS accounts and services — aggregating, normalizing, and prioritizing findings from GuardDuty, Inspector, Macie, Firewall Manager, IAM Access Analyzer, and third-party tools.

---

### Key Features

| Feature | Details |
|---|---|
| **Finding aggregation** | ASFF (Amazon Security Finding Format) standardizes findings |
| **Compliance standards** | CIS AWS Foundations, PCI DSS, AWS Foundational Best Practices, NIST |
| **Cross-account** | Aggregator account in Organizations |
| **Automated response** | Security Hub → EventBridge → Lambda |
| **Integrations** | 60+ AWS and partner integrations |

---

### Security Hub vs GuardDuty vs Macie vs Inspector

```
GuardDuty ─────────┐
Inspector ─────────┤──→ Security Hub (aggregate + score) ──→ EventBridge ──→ Response
Macie ─────────────┤
Firewall Manager ──┘
IAM Access Analyzer┘
```

---

### Key Exam Scenarios

- **Aggregate security findings from all accounts**: Security Hub with Organizations.
- **Compliance check against CIS benchmarks**: Security Hub CIS standard.
- **Automatically remediate non-compliant resources**: Security Hub → EventBridge → Lambda/SSM.
- **Single view of all security posture**: Security Hub.

---

---

## 18. AWS Audit Manager

### What Problem Does It Solve?
Audit Manager **continuously collects evidence** from AWS services and maps it to compliance frameworks (PCI DSS, HIPAA, SOC 2, GDPR, ISO 27001) — automating the manual work of gathering audit evidence.

---

### Key Concepts

| Concept | Description |
|---|---|
| **Framework** | Compliance framework (PCI DSS, SOC 2, HIPAA, etc.) |
| **Control** | Specific requirement within a framework |
| **Assessment** | Active evaluation against a framework |
| **Evidence** | AWS config snapshots, CloudTrail logs, etc. auto-collected |

---

### Audit Manager vs AWS Artifact

| Service | Purpose |
|---|---|
| **Audit Manager** | Continuously collect YOUR evidence for compliance audits |
| **AWS Artifact** | Download AWS's own compliance reports (SOC, PCI, ISO) |

---

### Key Exam Scenarios

- **Automate HIPAA audit evidence collection**: Audit Manager.
- **Prove PCI DSS compliance for your own environment**: Audit Manager.
- **Download AWS's SOC 2 Type II report**: AWS Artifact.

---

---

## 19. AWS Artifact

### What Problem Does It Solve?
Artifact provides **on-demand access to AWS compliance documentation** — audit reports, certifications, and legal agreements — so you can demonstrate AWS's own compliance posture to auditors.

---

### Two Components

| Component | Description |
|---|---|
| **Artifact Reports** | AWS compliance reports: SOC 1/2/3, PCI DSS, ISO 27001, FedRAMP, etc. |
| **Artifact Agreements** | Legal agreements: Business Associate Addendum (BAA) for HIPAA, GDPR DPA |

---

### Key Exam Scenarios

- **Download AWS ISO 27001 certification**: AWS Artifact Reports.
- **Sign HIPAA BAA with AWS**: AWS Artifact Agreements.
- **Auditor needs AWS's PCI DSS Attestation of Compliance**: AWS Artifact.
- **Prove AWS's compliance to your own auditor**: AWS Artifact.

---

---

## 20. Amazon Cognito vs IAM vs IAM Identity Center

*(This triad is heavily tested — know when to use each)*

| Dimension | IAM | IAM Identity Center | Cognito |
|---|---|---|---|
| **Who** | AWS-internal users/services | Workforce (employees) | App end-users (customers) |
| **Scale** | Thousands | Thousands | Millions |
| **Auth method** | Access keys, console | SSO, SAML | OAuth2, OIDC, SAML |
| **Token type** | SigV4 / STS | STS temp credentials | JWT (User Pool) or STS (Identity Pool) |
| **Use case** | AWS resource access | Multi-account AWS access | App user auth |
| **MFA** | Per IAM user | Centrally managed | Per user pool config |

---

---

## 21. AWS Resource Access Manager (AWS RAM)

### What Problem Does It Solve?
RAM enables **sharing of AWS resources across AWS accounts** (within or outside an AWS Organization) — without duplicating resources or managing complex VPC peering for every account.

---

### Shareable Resources (Key Ones)

- **VPC Subnets** (most common exam topic)
- Transit Gateways
- Route 53 Resolver rules
- License Manager configurations
- Aurora DB clusters
- EC2 Capacity Reservations
- AWS Glue tables / catalogs
- CodeBuild projects

---

### RAM Key Properties

| Property | Details |
|---|---|
| **Within Organization** | Sharing auto-accepted; no invitation needed |
| **Outside Organization** | Invitation required; recipient must accept |
| **VPC subnet sharing** | Accounts can launch resources into shared subnets; they see the subnet but don't own it |
| **No resource duplication** | Resources exist once; shared logically |

---

### RAM vs VPC Peering vs Transit Gateway

| Feature | RAM (Subnet Share) | VPC Peering | Transit Gateway |
|---|---|---|---|
| **Complexity** | Low | Medium | High (but scalable) |
| **Cross-account** | ✅ | ✅ | ✅ |
| **Transitive routing** | N/A | ❌ | ✅ |
| **Centralized networking** | Partial | ❌ | ✅ |
| **Use case** | Share subnets in central VPC | Connect 2 VPCs | Hub-spoke, many VPCs |

---

### Key Exam Scenarios

- **Multiple accounts deploy into one centralized VPC**: RAM + Subnet sharing.
- **Share Transit Gateway across accounts in org**: RAM.
- **Share Route 53 Resolver rules**: RAM.
- **Avoid duplicate resources per account**: RAM resource sharing.

---

---

## Architecture Patterns to Know

### Pattern 1: Defense in Depth (Web Application)
```
Internet → Route 53 → CloudFront (WAF + Shield Advanced)
                              ↓
                    ALB (WAF + ACM cert)
                              ↓
                    EC2 / ECS (Security Groups)
                              ↓
                    RDS (KMS encryption + Secrets Manager)
```

### Pattern 2: Centralized Security Monitoring
```
All Accounts:
  GuardDuty ──┐
  Inspector ──┤──→ Security Hub (Delegated Admin Account)
  Macie ──────┤           ↓
  Config ─────┘    EventBridge → Lambda (Auto-remediation)
                                → SNS (Alerts)
                                → Detective (Investigation)
```

### Pattern 3: Multi-Account Identity
```
Corporate AD → IAM Identity Center → AWS Organizations
                     ↓                    ├── Account A (Permission Set 1)
               SAML/SCIM sync             ├── Account B (Permission Set 2)
               (Okta / Azure AD)          └── Account C (Permission Set 3)
```

### Pattern 4: Encryption Architecture
```
Application → KMS (CMK) → Envelope Encryption
                 ↓
          CloudHSM (Custom Key Store) ← If FIPS 140-2 L3 required
                 ↓
   S3 (SSE-KMS) / EBS / RDS / Secrets Manager
```

### Pattern 5: Incident Response Automation
```
GuardDuty Finding
      ↓
EventBridge Rule
      ↓
Lambda Function:
  - Isolate EC2 (change Security Group)
  - Revoke IAM credentials
  - Take EBS snapshot for forensics
  - Notify via SNS
      ↓
Detective (root cause investigation)
```

---

## Exam Tips Summary

### IAM
- ✅ Explicit Deny always wins; default is implicit Deny
- ✅ SCP = ceiling for entire account; doesn't grant permissions
- ✅ Permissions Boundary = ceiling for a user/role
- ✅ Instance profile = IAM role for EC2 (never embed access keys)
- ✅ Access Analyzer = find unintended external access
- ✅ Credential Report = audit all users + key age

### IAM Identity Center
- ✅ Workforce SSO across multi-account; replaces per-account IAM users
- ✅ Permission Sets = portable role definitions across accounts
- ✅ Integrates with Okta, Azure AD via SAML/SCIM
- ✅ Requires AWS Organizations

### Cognito
- ✅ User Pool = authentication (JWT tokens)
- ✅ Identity Pool = AWS credentials via STS
- ✅ Use both together: User Pool auth → Identity Pool for AWS access
- ✅ Supports unauthenticated (guest) access via Identity Pool

### KMS
- ✅ CMK max encrypt = 4 KB; use envelope encryption for larger data
- ✅ Imported key material = NO auto-rotation
- ✅ Multi-region keys = same key ID across regions
- ✅ CloudHSM = FIPS Level 3; KMS = FIPS Level 2
- ✅ S3 Bucket Keys reduce KMS API calls by 99%
- ✅ Key deletion = 7–30 day mandatory waiting period

### ACM
- ✅ Cannot use ACM cert directly on EC2
- ✅ CloudFront cert MUST be in us-east-1
- ✅ Imported certs = no auto-renewal
- ✅ Free for public certs on supported services

### Secrets Manager
- ✅ Use for DB passwords + auto-rotation
- ✅ SSM Parameter Store = cheaper, no native rotation, for configs
- ✅ Rotation uses Lambda function; built-in for RDS/Redshift/DocumentDB

### WAF / Shield / Network Firewall
- ✅ WAF = L7 HTTP filtering (SQL injection, XSS, geo-block, rate limit)
- ✅ Shield Standard = free, automatic L3/L4
- ✅ Shield Advanced = $3K/month, DRT, cost protection, L7
- ✅ Network Firewall = VPC-level stateful + IDS/IPS
- ✅ Firewall Manager = central policy across org accounts

### Threat Detection & Investigation
- ✅ GuardDuty = threat detection (behavior, ML)
- ✅ Inspector = vulnerability assessment (CVEs)
- ✅ Macie = sensitive data in S3 (PII)
- ✅ Detective = root cause investigation
- ✅ Security Hub = aggregate all findings

### Compliance
- ✅ Audit Manager = YOUR evidence for compliance frameworks
- ✅ Artifact = AWS's compliance reports + legal agreements (BAA)
- ✅ CloudHSM = FIPS 140-2 Level 3

### RAM
- ✅ Share subnets across accounts (most tested)
- ✅ Within org = auto-accepted; outside org = invitation
- ✅ Shared subnet: other accounts launch resources; owner retains control

---
---

<a name="app-integration"></a>
# PART 7 — Application Integration: SQS, SNS, EventBridge, Step Functions, Amazon MQ, AppSync & AppFlow

# AWS Exam Study Guide: Application Integration Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** AppFlow · AppSync · EventBridge · Amazon MQ · SNS · SQS · Step Functions

---

## 📋 Table of Contents

1. [Application Integration Quick-Reference Matrix](#quick-reference)
2. [Integration Patterns Overview](#patterns-overview)
3. [Amazon SQS](#sqs)
4. [Amazon SNS](#sns)
5. [Amazon EventBridge](#eventbridge)
6. [AWS Step Functions](#step-functions)
7. [Amazon MQ](#amazon-mq)
8. [AWS AppSync](#appsync)
9. [Amazon AppFlow](#appflow)
10. [Master Comparisons](#comparisons)
11. [Architecture Patterns](#architecture-patterns)
12. [Exam Tips Summary](#exam-tips)

---

<a name="quick-reference"></a>
## Application Integration Quick-Reference Matrix

| Service | Type | Communication | Exam Trigger Words |
|---|---|---|---|
| **SQS** | Message Queue | Async, pull-based, point-to-point | "decouple", "queue", "buffer", "dead letter queue", "exactly-once" |
| **SNS** | Pub/Sub Notification | Async, push-based, fan-out | "fan-out", "broadcast", "push notification", "publish/subscribe" |
| **EventBridge** | Event Bus | Async, event-driven, rule-based routing | "event routing", "SaaS events", "schema registry", "event patterns" |
| **Step Functions** | Workflow Orchestration | Sync/Async, state machine | "workflow", "orchestrate", "long-running process", "retry logic", "state machine" |
| **Amazon MQ** | Managed Message Broker | Async, AMQP/MQTT/OpenWire | "ActiveMQ", "RabbitMQ", "AMQP", "MQTT", "migrate message broker" |
| **AppSync** | GraphQL API | Sync/Async, WebSocket | "GraphQL", "real-time subscriptions", "multiple data sources", "mobile API" |
| **AppFlow** | SaaS Data Integration | Batch/event-triggered transfer | "Salesforce to S3", "SaaS integration", "no-code data transfer", "connector" |

---

<a name="patterns-overview"></a>
## Integration Patterns Overview

### The Core Integration Problem
Modern applications are made up of **loosely coupled services** that need to communicate without tight dependencies. Integration services provide the **glue** between components.

```
SYNCHRONOUS (tight coupling — avoid where possible):
  Service A → directly calls → Service B
  (A waits for B; if B is down, A fails)

ASYNCHRONOUS (loose coupling — preferred):
  Service A → puts message in queue/bus → Service B reads when ready
  (A doesn't wait; B can be down temporarily; system is resilient)
```

### Which Service for Which Pattern?

```
Need to...
  ├─→ Buffer/decouple point-to-point messages → SQS
  ├─→ Broadcast to multiple subscribers → SNS
  ├─→ Route events from AWS/SaaS/custom sources → EventBridge
  ├─→ Orchestrate multi-step workflows with state → Step Functions
  ├─→ Migrate existing ActiveMQ/RabbitMQ broker → Amazon MQ
  ├─→ Build GraphQL API with real-time subscriptions → AppSync
  └─→ Move data between SaaS and AWS (no code) → AppFlow
```

---

<a name="sqs"></a>
## 1. Amazon SQS (Simple Queue Service)

### What Problem Does It Solve?
When a fast producer (e.g., a web server receiving orders) generates work faster than a slow consumer (e.g., a payment processor) can handle it, the consumer gets overwhelmed. SQS provides a **fully managed message queue** that buffers messages between components, enabling them to operate at their own pace without losing messages.

**Real-World Example:**
> An e-commerce site receives 10,000 orders per minute during a flash sale. The order processing service can only handle 1,000 per minute. Without SQS, 9,000 orders per minute would be lost. With SQS: all 10,000 orders are safely buffered in the queue. The order processor consumes them at its own pace over 10 minutes. No orders lost. System never overwhelmed.

---

### SQS Queue Types

#### Standard Queue
| Feature | Details |
|---|---|
| **Throughput** | Unlimited (nearly infinite messages/second) |
| **Ordering** | Best-effort (NOT guaranteed in order) |
| **Delivery** | At-least-once (message may be delivered more than once) |
| **Duplicates** | Possible (design consumers to be idempotent) |
| **Use case** | Maximum throughput; order not critical |

#### FIFO Queue
| Feature | Details |
|---|---|
| **Throughput** | 300 TPS (transactions/sec) without batching; 3,000 TPS with batching |
| **Ordering** | ✅ Strict FIFO (exactly in order sent) |
| **Delivery** | Exactly-once (deduplication) |
| **Duplicates** | ❌ None (message deduplication ID) |
| **Use case** | Financial transactions, order processing where order matters |
| **Name suffix** | Queue name must end in `.fifo` |

> **Exam Rule:** "Exactly-once processing" + "strict order" → **FIFO Queue**. "Maximum throughput" + "order doesn't matter" → **Standard Queue**.

---

### SQS Key Concepts

#### Message Retention
- Default: **4 days**; minimum: 60 seconds; maximum: **14 days**.
- Messages not consumed within retention period are automatically deleted.

#### Visibility Timeout
- When a consumer reads a message, it becomes **invisible** to other consumers for the visibility timeout period.
- Default: **30 seconds**; range: 0 seconds to **12 hours**.
- Consumer must **delete** the message after successful processing.
- If consumer fails (crashes), message becomes visible again after timeout → retried.

```
Message in Queue → Consumer A reads it
                         ↓ (message invisible for 30 sec)
Consumer A processes successfully → Delete message ✅
         OR
Consumer A crashes → timeout expires → message visible again → Consumer B picks it up
```

> **Exam Tip:** If messages are being processed multiple times, increase the **visibility timeout** to match actual processing time.

#### Long Polling vs Short Polling

| Type | Behavior | Wait time | Cost |
|---|---|---|---|
| **Short Polling** | Returns immediately (even if empty) | 0 seconds | Higher (more API calls) |
| **Long Polling** | Waits up to 20 seconds for messages | 1–20 seconds | Lower (fewer empty responses) |

> **Exam Rule:** "Reduce SQS costs" + "reduce empty responses" → **Enable Long Polling** (WaitTimeSeconds = 20).

#### Dead Letter Queue (DLQ)
- After a message fails processing N times (**maxReceiveCount**), it's moved to the DLQ.
- DLQ must be same type as source queue (Standard → Standard DLQ, FIFO → FIFO DLQ).
- Use DLQ to: isolate failed messages, debug processing failures, alert on failures.
- **DLQ Redrive**: move messages from DLQ back to source queue after fixing the bug.

```
Message received → processed → delete ✅ (normal path)
Message received → fails → back to queue (invisible timeout)
                → fails again (maxReceiveCount times)
                → moved to Dead Letter Queue (DLQ)
                → alert via CloudWatch → developer investigates
```

#### Message Size
- Maximum: **256 KB**.
- For larger payloads: store in S3, put S3 reference in SQS message (**SQS Extended Client Library**).

---

### SQS Delay Queues
- **Delay entire queue**: all messages invisible for 0–900 seconds on arrival.
- **Message-level delay**: set per-message delay with `DelaySeconds`.
- Use case: delay processing by a specific amount of time (e.g., send email 5 min after order).

---

### SQS Security

| Feature | Details |
|---|---|
| **Encryption** | SSE-SQS (AWS-managed) or SSE-KMS (customer-managed) |
| **Access control** | IAM policies + SQS resource policy (queue policy) |
| **Cross-account access** | SQS resource policy grants another account permission |
| **VPC Endpoint** | Interface endpoint (PrivateLink) for private access |

---

### SQS with Lambda (Event Source Mapping)
- Lambda **polls** SQS; no separate SQS trigger needed.
- Lambda reads in **batches** (configurable: 1–10,000 messages per batch).
- On batch failure: entire batch retried; use DLQ on SQS to isolate poison messages.
- **FIFO + Lambda**: in-order processing; 1 concurrent Lambda per message group.

---

### SQS FIFO — Message Groups
- `MessageGroupId`: messages with the same group ID are processed in order.
- Different group IDs can be processed in parallel.

```
Group: ORDER-001 → messages processed in strict order (1 consumer at a time)
Group: ORDER-002 → processed in parallel with ORDER-001
Group: ORDER-003 → processed in parallel with ORDER-001 and ORDER-002
```

---

### Key SQS Exam Scenarios

| Scenario | Answer |
|---|---|
| Decouple fast producer from slow consumer | SQS Standard Queue |
| Financial transactions must be processed in exact order | SQS FIFO Queue |
| Reduce SQS API costs from empty poll responses | Enable Long Polling (WaitTimeSeconds=20) |
| Message fails processing repeatedly | Dead Letter Queue (DLQ) |
| Message larger than 256 KB | SQS Extended Client Library (store in S3) |
| Lambda processes SQS messages in batches | SQS + Lambda Event Source Mapping |
| Delay all messages in queue by 5 minutes | SQS Delay Queue (DelaySeconds=300) |
| Messages being processed twice | Increase visibility timeout |
| Fan-out SQS to multiple consumers | SNS → multiple SQS queues |

---

<a name="sns"></a>
## 2. Amazon SNS (Simple Notification Service)

### What Problem Does It Solve?
When an event occurs in your system (e.g., a new order), multiple services may need to know — fraud detection, inventory, email, analytics. Calling each service directly creates tight coupling. SNS provides **publish-subscribe (pub/sub) messaging** where a single message is delivered to all interested subscribers simultaneously (fan-out).

**Real-World Example:**
> An e-commerce order is placed. One SNS `OrderPlaced` topic notifies:
> - SQS queue → Order fulfillment service
> - SQS queue → Inventory management service
> - SQS queue → Analytics pipeline
> - Email → Customer confirmation
> - Lambda → Fraud detection
>
> All 5 subscribers receive the same message simultaneously. Adding a new subscriber requires no change to the producer.

---

### SNS Key Concepts

| Concept | Description |
|---|---|
| **Topic** | Named channel for publishing messages |
| **Publisher** | Sends a message to a topic |
| **Subscriber** | Receives messages from a topic |
| **Subscription** | Association between a subscriber endpoint and a topic |
| **Message filtering** | Subscribers receive only messages matching their filter policy |

---

### SNS Topic Types

| Type | Description | Ordering | Dedup | Use Case |
|---|---|---|---|---|
| **Standard** | Best-effort delivery; unlimited throughput | No | No | High-throughput fan-out |
| **FIFO** | Ordered delivery; deduplication | ✅ Strict | ✅ | Ordered fan-out to FIFO SQS |

> **FIFO SNS + FIFO SQS**: Only FIFO SQS queues can subscribe to FIFO SNS topics. Used when you need ordered fan-out.

---

### SNS Subscription Protocols

| Protocol | Description | Use Case |
|---|---|---|
| **SQS** | Delivers to an SQS queue | Decouple services; queue for processing |
| **Lambda** | Invokes a Lambda function | Serverless processing |
| **HTTP/HTTPS** | POST to a web endpoint | Webhooks; external services |
| **Email / Email-JSON** | Sends email | Human notifications; alerts |
| **SMS** | Text message to phone | OTP; critical alerts to phones |
| **Mobile Push** | Push to APNs (iOS), FCM (Android) | App notifications |
| **Kinesis Data Firehose** | Stream to S3/Redshift/OpenSearch | Analytics pipelines |

---

### SNS Message Filtering

- Each subscription can have a **filter policy** (JSON) that selects which messages to receive.
- Messages not matching the filter are **not delivered** to that subscriber.
- Filters based on message attributes.

```json
// Only receive messages for NYC orders over $100
{
  "region": ["NYC", "Brooklyn"],
  "orderValue": [{"numeric": [">", 100]}]
}
```

**Example — Different teams subscribe to same topic with filters:**
```
OrderTopic publisher:
  Message: { "type": "order", "region": "US", "value": 250 }

Subscriber A (filter: region=US) → ✅ receives
Subscriber B (filter: region=EU) → ❌ filtered out
Subscriber C (filter: value > 500) → ❌ filtered out
Subscriber D (no filter) → ✅ receives
```

---

### SNS Fan-Out Pattern (Critical Exam Pattern)

```
Producer
    ↓
SNS Topic (1 publish)
    ├── SQS Queue 1 → Service A (order fulfillment)
    ├── SQS Queue 2 → Service B (inventory update)
    ├── SQS Queue 3 → Service C (analytics)
    └── Lambda → Service D (fraud detection)
```

**Why SNS + SQS and not SNS alone?**
- SQS provides **durability** (messages persist if consumer is down).
- SQS provides **retry logic** (visibility timeout + DLQ).
- SNS alone: if the subscriber endpoint is down, message is lost.
- SNS + SQS: if consumer is down, messages wait in SQS queue.

---

### SNS Message Delivery Retry Policy

- SNS retries delivery on failure:
  - Immediate retries: up to 3 times.
  - Then exponential backoff.
  - Total retry duration: up to **23 days** for HTTP/HTTPS endpoints.
  - Failed deliveries → **Dead Letter Queue** (SNS DLQ, separate from SQS DLQ).

---

### SNS Security

| Feature | Details |
|---|---|
| **Encryption** | SSE-SNS (AWS-managed) or SSE-KMS |
| **Access policy** | Resource-based policy controlling who can publish/subscribe |
| **Cross-account** | Resource policy grants another account publish access |
| **VPC Endpoint** | Interface endpoint for private SNS access |

---

### SNS vs SQS vs EventBridge

| Feature | SQS | SNS | EventBridge |
|---|---|---|---|
| **Pattern** | Queue (point-to-point) | Pub/Sub (fan-out) | Event bus (routing) |
| **Consumers** | 1 (or multiple competing) | Multiple simultaneous | Multiple per rule |
| **Message persistence** | ✅ (up to 14 days) | ❌ (push-only) | ❌ (push-only, 24h retry) |
| **Filtering** | ❌ (all messages) | ✅ (attribute filter) | ✅ (rich pattern matching) |
| **SaaS sources** | ❌ | ❌ | ✅ (Salesforce, Zendesk, etc.) |
| **Schema registry** | ❌ | ❌ | ✅ |
| **Ordering** | FIFO option | FIFO option | ❌ |
| **Max message size** | 256 KB | 256 KB | 256 KB |

---

### Key SNS Exam Scenarios

| Scenario | Answer |
|---|---|
| Notify 5 different services when an event occurs | SNS (publish once, fan-out to all) |
| Send email + trigger Lambda when alarm fires | SNS with Email + Lambda subscriptions |
| Ensure subscriber only gets relevant messages | SNS message filtering |
| Fan-out with durability (consumer might be down) | SNS + SQS (fan-out to multiple SQS queues) |
| Send mobile push notification to iOS/Android | SNS Mobile Push (APNs/FCM) |
| Fan-out with strict ordering | SNS FIFO + SQS FIFO |

---

<a name="eventbridge"></a>
## 3. Amazon EventBridge

### What Problem Does It Solve?
Traditional messaging (SQS, SNS) requires custom code to connect producers and consumers. EventBridge provides a **serverless event bus** that automatically routes events from AWS services, your own applications, and third-party SaaS applications — to any target — based on rules with rich pattern matching.

**Real-World Example:**
> A DevOps team wants to:
> - Alert Slack when any EC2 instance state changes to "stopped"
> - Trigger a Lambda when CodePipeline fails
> - Sync Salesforce contacts to DynamoDB when updated
> - Run a Lambda every 5 minutes for health checks
>
> All configured in EventBridge with zero custom integration code — rules define what events to match and where to send them.

---

### EventBridge Event Buses

| Bus Type | Description | Sources |
|---|---|---|
| **Default (AWS) Bus** | Pre-existing; AWS services publish here automatically | EC2, S3, RDS, CodePipeline, etc. |
| **Custom Bus** | Create your own for application events | Your microservices, apps |
| **Partner Bus** | Pre-integrated SaaS sources | Salesforce, Zendesk, Datadog, PagerDuty, GitHub, etc. |

---

### EventBridge Components

#### Rules
- Define **which events** to match and **where to send them**.
- Two types:
  - **Event pattern rules**: match events based on JSON structure/content.
  - **Scheduled rules**: cron or rate expression (replaces CloudWatch Events cron).

```json
// Event Pattern Rule: match any EC2 instance state change
{
  "source": ["aws.ec2"],
  "detail-type": ["EC2 Instance State-change Notification"],
  "detail": {
    "state": ["stopped", "terminated"]
  }
}
```

#### Targets
One rule can route to **up to 5 targets** simultaneously:

| Target Type | Examples |
|---|---|
| **Compute** | Lambda, ECS task, Batch job, Step Functions |
| **Messaging** | SNS, SQS, Kinesis Data Streams, Kinesis Firehose |
| **Monitoring** | CloudWatch Logs |
| **Integration** | API Gateway, AppSync, EventBridge API Destinations |
| **AWS Services** | CodeBuild, CodePipeline, Systems Manager, Inspector |

#### Event Pattern Matching
EventBridge uses JSON pattern matching with operators:

```json
{
  "source": ["aws.s3"],           // exact match
  "detail": {
    "eventName": ["PutObject"],   // list = OR match
    "bucket": {
      "name": [{"prefix": "prod-"}]  // prefix match
    },
    "object": {
      "size": [{"numeric": [">", 1000000]}]  // numeric
    }
  }
}
```

---

### EventBridge Pipes

- **Point-to-point** event processing pipeline with built-in filtering, enrichment, and transformation.
- Source → (optional filter) → (optional enrichment: Lambda/API GW/Step Functions) → Target.
- Sources: SQS, Kinesis, DynamoDB Streams, Kafka, MQ.
- Targets: Lambda, Step Functions, API Gateway, SQS, SNS, EventBridge, etc.

```
SQS Queue
    ↓ (filter: only error events)
Lambda enrichment (add context from DynamoDB)
    ↓ (transform: reshape JSON)
Step Functions (start workflow)
```

---

### EventBridge Schema Registry

- Automatically discovers and records the **schema** of events flowing through your bus.
- Generates **code bindings** (Java, Python, TypeScript, Go) for event deserialization.
- Enables IDE auto-completion for event handling code.

---

### EventBridge API Destinations

- Send events to **any HTTP endpoint** (external APIs, webhooks, SaaS services).
- Supports OAuth, API key, and Basic auth.
- Rate limiting and retry built-in.
- Use case: Send AWS events to Datadog, PagerDuty, Jira, etc.

---

### EventBridge Archive and Replay

- **Archive**: automatically store all (or filtered) events for a configurable period.
- **Replay**: re-run archived events against current or updated rules.
- Use case: Debug production issues by replaying historical events; test new rule logic.

---

### EventBridge vs CloudWatch Events

> CloudWatch Events is the **predecessor** to EventBridge — they use the same underlying service. EventBridge is the current name with additional capabilities (partner sources, schema registry, pipes, API destinations). All CloudWatch Events features work in EventBridge.

---

### EventBridge Scheduler

- Standalone **task scheduler** (cron + one-time schedules) that invokes targets directly.
- Supports flexible time windows (run within X minutes of scheduled time).
- Manages millions of schedules (scales better than individual EventBridge rules).
- Targets: Lambda, SQS, SNS, Step Functions, 270+ AWS services.

---

### Key EventBridge Exam Scenarios

| Scenario | Answer |
|---|---|
| Trigger Lambda whenever any EC2 instance stops | EventBridge rule on default bus (EC2 state change event) |
| Route Salesforce events to Lambda | EventBridge partner bus (Salesforce) |
| Run a Lambda every 5 minutes | EventBridge scheduled rule (rate(5 minutes)) |
| React to S3 object creation with complex routing | EventBridge rule + S3 CloudTrail events |
| Send AWS events to PagerDuty webhook | EventBridge API Destinations |
| Replay events after fixing a bug | EventBridge Archive + Replay |
| Point-to-point pipeline with enrichment | EventBridge Pipes |

---

<a name="step-functions"></a>
## 4. AWS Step Functions

### What Problem Does It Solve?
Complex workflows involve multiple services called in sequence or parallel, with retry logic, error handling, timeouts, and conditional branching. Implementing this in application code leads to spaghetti logic that's hard to debug and maintain. Step Functions lets you **define workflows as state machines** (visual flowcharts) with built-in retry, error handling, and parallel execution — without writing coordination code.

**Real-World Example:**
> An order fulfillment workflow:
> 1. Validate order (Lambda) — if invalid, send error email
> 2. Process payment (Lambda) — retry 3 times on failure; if all fail, send to manual review queue
> 3. Check inventory (Lambda) — if out of stock, pause and wait for restock notification
> 4. Notify warehouse AND fraud detection simultaneously (parallel)
> 5. Send confirmation email
>
> All of this in a visual state machine with zero coordination code in Lambda functions.

---

### Step Functions Workflow Types

| Type | Description | Max Duration | Pricing | Use Case |
|---|---|---|---|---|
| **Standard Workflow** | Exactly-once execution; full history stored | **1 year** | Per state transition | Long-running workflows, auditable |
| **Express Workflow** | At-least-once; optimized for high volume | **5 minutes** | Per execution + duration | High-volume, short-duration IoT/streaming |

---

### State Types

| State Type | Purpose | Example |
|---|---|---|
| **Task** | Execute work (Lambda, API call, SDK call) | Call Lambda to process payment |
| **Choice** | Conditional branching (if/else/switch) | If payment_status == "success" → ship; else → refund |
| **Wait** | Pause for time period or specific timestamp | Wait until 2026-12-01 for scheduled send |
| **Parallel** | Execute multiple branches simultaneously | Notify warehouse + fraud detection at same time |
| **Map** | Iterate over array, running same workflow for each item | Process each item in a list of files |
| **Pass** | Pass input to output; optionally transform | Add metadata to state |
| **Succeed** | End execution successfully | Terminal state on success |
| **Fail** | End execution with error | Terminal state on failure |

---

### Step Functions State Machine Example

```json
{
  "Comment": "Order Processing Workflow",
  "StartAt": "ValidateOrder",
  "States": {
    "ValidateOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:ValidateOrder",
      "Next": "ProcessPayment",
      "Catch": [{
        "ErrorEquals": ["InvalidOrderError"],
        "Next": "SendErrorEmail"
      }]
    },
    "ProcessPayment": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:...:function:ProcessPayment",
      "Retry": [{
        "ErrorEquals": ["States.TaskFailed"],
        "IntervalSeconds": 5,
        "MaxAttempts": 3,
        "BackoffRate": 2
      }],
      "Next": "ParallelNotifications"
    },
    "ParallelNotifications": {
      "Type": "Parallel",
      "Branches": [
        { "StartAt": "NotifyWarehouse", "States": {"NotifyWarehouse": {"Type": "Task", ...}} },
        { "StartAt": "FraudDetection", "States": {"FraudDetection": {"Type": "Task", ...}} }
      ],
      "Next": "SendConfirmationEmail"
    },
    "SendConfirmationEmail": {"Type": "Task", ...},
    "SendErrorEmail": {"Type": "Task", ...}
  }
}
```

---

### Step Functions Service Integrations

Step Functions can **directly integrate** with AWS services (without Lambda):

| Integration Type | Description | Services |
|---|---|---|
| **Request-Response** | Call and immediately proceed | SQS, SNS, Lambda, DynamoDB |
| **Sync** | Wait for service to complete | Lambda, ECS, CodeBuild, Glue, SageMaker |
| **Wait for Token** | Pause workflow, wait for external signal | Any service (send task token; resume with SendTaskSuccess) |

**Wait for Callback (Task Token) Pattern:**
```
Step Functions → sends message to SQS with taskToken
        ↓ (workflow paused for days/weeks)
Human reviews the case in a portal
        ↓ (human approves/denies)
Portal calls SendTaskSuccess(taskToken, result)
        ↓
Step Functions workflow resumes
```

---

### Step Functions Error Handling

#### Retry
```json
"Retry": [{
  "ErrorEquals": ["Lambda.ServiceException", "States.TaskFailed"],
  "IntervalSeconds": 2,    // initial wait
  "MaxAttempts": 3,        // try 3 more times
  "BackoffRate": 2.0       // double wait each time: 2, 4, 8 seconds
}]
```

#### Catch (Fallback State)
```json
"Catch": [{
  "ErrorEquals": ["PaymentDeclinedError"],
  "Next": "RefundState",
  "ResultPath": "$.error"
}]
```

---

### Step Functions vs Lambda vs SQS for Orchestration

| Scenario | Best Service |
|---|---|
| Simple queue of tasks | SQS |
| Event triggers a single function | Lambda |
| Multi-step sequential workflow with error handling | Step Functions |
| Long-running workflow (days/months) with human approval | Step Functions (Wait for Token) |
| High-volume short workflows (IoT) | Step Functions Express |

---

### Key Step Functions Exam Scenarios

| Scenario | Answer |
|---|---|
| Orchestrate multi-step order workflow with retry logic | AWS Step Functions (Standard) |
| Workflow must pause and wait for human approval | Step Functions + Wait for Callback (Task Token) |
| Process each item in a DynamoDB scan result | Step Functions Map state |
| Run fraud detection AND inventory check simultaneously | Step Functions Parallel state |
| Workflow can run up to 1 year | Step Functions Standard Workflow |
| High-volume IoT event processing < 5 minutes | Step Functions Express Workflow |
| Coordinate Lambda + ECS + SageMaker in sequence | Step Functions with SDK integrations |

---

<a name="amazon-mq"></a>
## 5. Amazon MQ

### What Problem Does It Solve?
Organizations migrating from on-premises applications to AWS often have existing applications using traditional message brokers — Apache ActiveMQ or RabbitMQ — with industry-standard protocols (AMQP, MQTT, OpenWire, STOMP). Rewriting these applications to use SNS/SQS requires significant code changes. Amazon MQ provides **managed ActiveMQ or RabbitMQ brokers** that existing applications can connect to with zero code changes.

**Real-World Example:**
> A manufacturing company's factory floor systems use ActiveMQ with MQTT for machine-to-machine communication. Moving to AWS: they point their existing MQTT clients at Amazon MQ instead of their on-premises ActiveMQ broker — zero application code changes, fully managed by AWS (patching, HA, backups), and connected to their factory systems via Direct Connect.

---

### Amazon MQ Broker Types

| Broker | Description | Protocols | Use Case |
|---|---|---|---|
| **ActiveMQ** | Apache ActiveMQ broker | AMQP, MQTT, OpenWire, STOMP, WebSocket | Legacy Java applications, IoT |
| **RabbitMQ** | RabbitMQ broker | AMQP 0-9-1, MQTT (plugin), STOMP | Node.js/Python apps, complex routing |

---

### Amazon MQ Deployment Modes

| Mode | Description | HA |
|---|---|---|
| **Single-instance** | One broker; dev/test | ❌ |
| **Active/Standby (ActiveMQ)** | Two brokers across AZs; automatic failover | ✅ (manual DNS switch) |
| **Cluster (RabbitMQ)** | Multiple nodes across AZs; automatic failover | ✅ (automatic) |

---

### Amazon MQ Key Properties

| Property | Details |
|---|---|
| **Protocols** | AMQP, MQTT, OpenWire, STOMP (ActiveMQ); AMQP 0-9-1 (RabbitMQ) |
| **Storage** | Amazon EFS (ActiveMQ shared storage for HA) |
| **Network** | Deployed in VPC; not public internet by default |
| **Encryption** | At rest (AES-256 by default); in transit (TLS) |
| **Monitoring** | CloudWatch metrics; ActiveMQ web console |

---

### Amazon MQ vs SQS vs SNS

| Feature | Amazon MQ | SQS | SNS |
|---|---|---|---|
| **Protocol** | AMQP, MQTT, STOMP, OpenWire | AWS SDK / HTTP | AWS SDK / HTTP |
| **Migration** | ✅ Drop-in for existing brokers | Requires code rewrite | Requires code rewrite |
| **Topics + Queues** | ✅ Both natively | Queue only | Topic only |
| **Message routing** | ✅ Complex (topic-based, header-based) | Simple (point-to-point) | Simple (pub/sub) |
| **Scalability** | Limited (broker-based) | Infinite | Infinite |
| **Managed** | ✅ | ✅ | ✅ |
| **Use case** | Migrate existing broker | New cloud-native queuing | New cloud-native pub/sub |

> **Exam Rule:** "Migrate existing ActiveMQ/RabbitMQ" or "need AMQP/MQTT protocol" → **Amazon MQ**. "New cloud-native application" → **SQS or SNS** (simpler, more scalable).

---

### Key Amazon MQ Exam Scenarios

| Scenario | Answer |
|---|---|
| Migrate on-premises ActiveMQ to AWS without code changes | Amazon MQ (ActiveMQ broker) |
| IoT devices using MQTT protocol need managed broker | Amazon MQ |
| Application uses AMQP protocol and can't change it | Amazon MQ |
| New microservices application needs a message queue | SQS (not Amazon MQ) |
| RabbitMQ cluster needed for existing Python app | Amazon MQ (RabbitMQ) |
| Maximum scalability for new cloud-native workload | SQS (not Amazon MQ) |

---

<a name="appsync"></a>
## 6. AWS AppSync

### What Problem Does It Solve?
Modern mobile and web applications need APIs that: (1) let clients request exactly the data they need, (2) aggregate data from multiple backends in one call, (3) push updates to clients in real time. REST APIs over-fetch or under-fetch data. AppSync provides a **managed GraphQL service** that addresses all three problems.

**Real-World Example:**
> A social media mobile app needs: user profile (from DynamoDB), post feed (from DynamoDB), friend recommendations (from Lambda using Neptune), and real-time notifications (via WebSocket). With AppSync GraphQL, the mobile client makes **one API call** specifying exactly which fields it needs from which sources — AppSync routes to each backend and returns the combined result. When a new post is created, all subscribed clients receive it instantly via WebSocket.

---

### AppSync Core Concepts

| Concept | Description |
|---|---|
| **Schema** | GraphQL type definitions (types, queries, mutations, subscriptions) |
| **Resolver** | Maps GraphQL field to data source; written in VTL or JavaScript |
| **Data Source** | Backend connected to AppSync |
| **Pipeline Resolver** | Chain multiple functions/data sources in sequence |
| **Subscription** | Real-time WebSocket connection; clients subscribe to mutations |

---

### AppSync Data Sources

| Data Source | Description |
|---|---|
| **Amazon DynamoDB** | Most common; direct DynamoDB table access |
| **AWS Lambda** | Custom business logic; any backend |
| **Amazon RDS (Aurora Serverless)** | SQL database via RDS Data API |
| **HTTP endpoint** | Any REST/HTTP API |
| **Amazon OpenSearch** | Full-text search |
| **Amazon EventBridge** | Publish GraphQL mutations as events |
| **None (local)** | Pure client-side mock or transformation |

---

### AppSync Authorization Modes

| Mode | Description | Use Case |
|---|---|---|
| **API Key** | Static key; limited to 1 year | Development, public read-only |
| **AWS IAM** | SigV4; for AWS services | Backend-to-backend, Cognito Identity Pool |
| **Cognito User Pools** | JWT tokens | End-user authenticated apps |
| **OIDC** | Third-party identity provider | Custom IdP (Okta, Auth0) |
| **Lambda Authorizer** | Custom auth logic in Lambda | Complex authorization rules |

> Multiple auth modes can coexist (one default + additional modes).

---

### AppSync Real-Time Subscriptions

```
Client subscribes: subscription onNewPost { id title author }
        ↓ (WebSocket maintained)
Another client creates a post (mutation: createPost)
        ↓ (AppSync detects mutation)
All subscribed clients receive the new post instantly
```

- Real-time via **WebSocket** (MQTT over WebSocket).
- No polling needed — server pushes to client.
- Use case: chat apps, live dashboards, collaborative editing, notifications.

---

### AppSync Caching

- **Server-side caching** at the API or resolver level.
- Cache TTL: configurable per API or per resolver.
- Cache encryption at rest and in transit.
- Reduces backend calls; improves response times.

---

### AppSync vs API Gateway

| Feature | AppSync | API Gateway |
|---|---|---|
| **Protocol** | GraphQL | REST / HTTP / WebSocket |
| **Data fetching** | Client specifies exact fields | Fixed response structure |
| **Real-time** | ✅ Built-in subscriptions | WebSocket API (requires custom) |
| **Multiple sources** | ✅ One query, multiple backends | One endpoint, one backend |
| **Offline sync** | ✅ (Amplify DataStore) | Manual |
| **Best for** | Mobile apps, complex data needs | REST APIs, microservices |

---

### Key AppSync Exam Scenarios

| Scenario | Answer |
|---|---|
| Mobile app needs real-time data updates | AppSync with GraphQL subscriptions |
| Single API call fetching data from DynamoDB + Lambda | AppSync with pipeline resolvers |
| Offline-first mobile app with sync on reconnect | AppSync + Amplify DataStore |
| Fine-grained authorization at field level | AppSync with Cognito + VTL resolver logic |
| Replace REST over-fetching with efficient queries | AppSync (GraphQL) |

---

<a name="appflow"></a>
## 7. Amazon AppFlow

### What Problem Does It Solve?
Organizations need to regularly transfer data between SaaS applications (Salesforce, ServiceNow, Slack, Google Analytics) and AWS services (S3, Redshift, DynamoDB). Building custom ETL pipelines with API integrations, authentication, scheduling, and error handling for every SaaS-to-AWS connection is time-consuming and fragile. AppFlow provides **no-code/low-code data integration flows** between SaaS applications and AWS — set up in minutes.

**Real-World Example:**
> A sales operations team needs Salesforce opportunity data in Redshift for BI reporting. Traditionally this requires: a developer writing a Salesforce API integration, handling OAuth, scheduling, pagination, error handling, and loading to Redshift — weeks of work. AppFlow: select Salesforce source + Redshift target + field mappings + daily schedule → flowing data in 30 minutes with zero code.

---

### AppFlow Key Concepts

| Concept | Description |
|---|---|
| **Flow** | Defines source, destination, mapping, triggers, transformation |
| **Connection** | Authenticated link to a SaaS or AWS service |
| **Trigger** | When the flow runs (scheduled, event-based, on-demand) |
| **Mapping** | How source fields map to destination fields |
| **Transformation** | Masking, truncation, validation, custom values |
| **Filter** | Only transfer records matching criteria |

---

### Supported Connectors

**SaaS Sources/Destinations:**
Salesforce, ServiceNow, Slack, Google Analytics, Marketo, Zendesk, HubSpot, SAP, Veeva, Trend Micro, Snowflake, Datadog, Dynatrace, Facebook Ads, Google Ads, LinkedIn Ads, Amplitude, and more (50+ connectors).

**AWS Destinations:**
Amazon S3, Amazon Redshift, Amazon DynamoDB, Amazon OpenSearch, Salesforce, Snowflake, EventBridge.

---

### AppFlow Trigger Types

| Trigger | Description | Use Case |
|---|---|---|
| **Scheduled** | Run at defined intervals (hourly, daily, weekly) | Regular data sync batch |
| **Event-based** | Trigger on SaaS event (new Salesforce record) | Near-real-time sync |
| **On-demand** | Manual run | Ad-hoc data transfer |

---

### AppFlow Data Transformations

| Transformation | Description |
|---|---|
| **Masking** | Replace sensitive field values (credit cards, SSNs) |
| **Truncation** | Limit field length |
| **Validation** | Reject records not matching rules |
| **Custom value mapping** | Map source values to destination values |
| **Concatenation** | Combine multiple fields into one |
| **Arithmetic** | Add, subtract, multiply fields |

---

### AppFlow vs AWS Glue vs DataSync

| Feature | AppFlow | AWS Glue | AWS DataSync |
|---|---|---|---|
| **Target data** | SaaS applications | Any data source/format | File systems (NFS/SMB/S3/EFS) |
| **Coding required** | ❌ (no-code) | ✅ (PySpark/SQL) | ❌ (configuration) |
| **SaaS connectors** | ✅ 50+ built-in | Limited (custom needed) | ❌ |
| **Transformation** | Basic | ✅ Complex ETL | ❌ |
| **Use case** | SaaS → AWS data sync | Complex ETL pipelines | File/object data migration |

---

### Key AppFlow Exam Scenarios

| Scenario | Answer |
|---|---|
| Transfer Salesforce data to S3 daily without coding | Amazon AppFlow |
| Sync ServiceNow incidents to Redshift for analysis | Amazon AppFlow |
| React to new Salesforce opportunities in near-real-time | AppFlow (event-based trigger) |
| Complex ETL transformation of raw data | AWS Glue (not AppFlow) |
| Move files from on-prem NFS to S3 | AWS DataSync (not AppFlow) |
| Transfer data between two SaaS applications | Amazon AppFlow |

---

<a name="comparisons"></a>
## Master Comparisons

### SQS vs SNS vs EventBridge — When to Use Each

| Need | Service |
|---|---|
| Decouple producer from consumer; buffer messages | SQS |
| Fan-out one event to multiple consumers simultaneously | SNS |
| Route AWS service events (EC2 state change, S3 create) to targets | EventBridge (default bus) |
| Route events from Salesforce/Zendesk/GitHub to AWS | EventBridge (partner bus) |
| Complex event pattern matching with JSON operators | EventBridge |
| Ordered message delivery | SQS FIFO or SNS FIFO |
| Durable message storage (up to 14 days) | SQS |
| Push email/SMS on event | SNS |
| Schedule recurring tasks (cron) | EventBridge Scheduler |

### Orchestration vs Choreography

| Pattern | How | Services |
|---|---|---|
| **Orchestration** | Central coordinator tells each service what to do | Step Functions |
| **Choreography** | Each service reacts to events independently | SNS + SQS + EventBridge |

```
ORCHESTRATION (Step Functions):
  Order Service → Step Functions State Machine
                    ├─→ calls Payment Service
                    ├─→ calls Inventory Service
                    └─→ calls Notification Service

CHOREOGRAPHY (Event-driven):
  Order placed → EventBridge
    Payment Service listens to "OrderPlaced" → processes
    Inventory Service listens to "OrderPlaced" → updates
    Notification Service listens to "PaymentComplete" → sends email
```

### Message Broker Decision

```
New cloud-native application?
  ├─→ Simple queue, 1 consumer → SQS Standard
  ├─→ Ordered queue, exactly-once → SQS FIFO
  ├─→ Fan-out to multiple services → SNS
  └─→ Event routing with complex patterns → EventBridge

Migrating existing application?
  └─→ Has ActiveMQ/RabbitMQ with AMQP/MQTT/STOMP → Amazon MQ
```

### Step Functions vs Lambda for Workflow

| Scenario | Step Functions | Lambda |
|---|---|---|
| Multi-step workflow with conditional logic | ✅ | Complex/messy code |
| Retry with exponential backoff | ✅ Built-in | Manual code |
| Wait for human approval (days/weeks) | ✅ Task Token | ❌ (15-min limit) |
| Run in parallel | ✅ Parallel state | Manual orchestration |
| Visual debugging | ✅ | ❌ (CloudWatch only) |
| Cost | Per state transition | Per invocation + duration |

---

<a name="architecture-patterns"></a>
## Architecture Patterns to Know

### Pattern 1: Decoupled Order Processing (Fan-Out + Queue)
```
Order Service
    ↓ publishes
SNS Topic: "order.created"
    ├── SQS Queue → Fulfillment Service (processes at own pace)
    ├── SQS Queue → Inventory Service (processes at own pace)
    ├── SQS Queue → Analytics Pipeline (processes at own pace)
    └── Lambda → Fraud Detection (real-time check)

Benefits:
- Fulfillment Service crash → orders buffered in SQS (not lost)
- Add new subscriber → no change to Order Service
- Each service scales independently
```

### Pattern 2: Event-Driven Microservices with EventBridge
```
EC2 Instance stops unexpectedly
    ↓ (automatic AWS event)
EventBridge Default Bus
    ├── Rule 1: → Lambda (attempt restart)
    ├── Rule 2: → SNS → PagerDuty (alert on-call engineer)
    ├── Rule 3: → Step Functions (run recovery workflow)
    └── Rule 4: → CloudWatch Logs (audit trail)

CodePipeline deployment fails
    ↓ (automatic AWS event)
EventBridge Rule → SNS → Slack webhook (notify team)
                → Jira API Destination (create ticket)
```

### Pattern 3: Long-Running Workflow with Human Approval
```
Customer submits large loan application
    ↓
Step Functions Standard Workflow starts
    ↓
State 1: ValidateDocs (Lambda, 30 sec)
    ↓
State 2: AutoCreditCheck (Lambda, 2 min)
    ↓
State 3: WaitForHumanApproval
  ├── SendToLoanOfficer (Lambda sends email with approve/deny links)
  └── WAIT (up to 30 days for loan officer response)
    ↓ (officer clicks Approve → calls SendTaskSuccess)
State 4: DisburseLoansInParallel (Parallel state)
  ├── Branch 1: TransferFunds (Lambda)
  └── Branch 2: SendWelcomeEmail (Lambda + SES)
    ↓
State 5: UpdateCRM (Lambda → Salesforce via AppFlow API)
    ↓
SUCCEED
```

### Pattern 4: SaaS Data Pipeline
```
Salesforce (new opportunities created throughout the day)
    ↓ (AppFlow event-based trigger)
Amazon AppFlow
  ├── Filter: only opportunities > $50,000
  ├── Transform: mask SSN fields
  └── Map: Salesforce fields → Redshift columns
    ↓
Amazon Redshift
    ↓
Amazon QuickSight (Sales dashboard refreshes automatically)
```

### Pattern 5: SQS Buffer for DynamoDB Throttling
```
High-volume API requests (100K/sec)
    ↓
API Gateway → Lambda (validate + enqueue)
    ↓
SQS Standard Queue (buffer up to 14 days)
    ↓ (Lambda polls at sustainable 5K/sec)
Lambda (Event Source Mapping, batch size 10)
    ↓
DynamoDB (stays within provisioned capacity)

Result: API never throttles users; DynamoDB never gets overwhelmed;
        messages safely buffered during traffic spikes
```

---

<a name="exam-tips"></a>
## Exam Tips Summary

### Amazon SQS
- ✅ Standard = unlimited throughput, at-least-once, best-effort order
- ✅ FIFO = 300 TPS (3,000 with batching), exactly-once, strict order; name ends in `.fifo`
- ✅ Visibility timeout (default 30s, max 12h) = message invisible while being processed
- ✅ Long Polling (WaitTimeSeconds=20) = reduce cost + empty responses
- ✅ DLQ = failed messages after maxReceiveCount; same type as source queue
- ✅ Delay Queue = DelaySeconds 0–900s; hold all messages on arrival
- ✅ Max message size = 256 KB; larger → S3 + SQS Extended Client Library
- ✅ Retention: 60 sec – 14 days; default 4 days
- ✅ "Messages processed twice" → increase visibility timeout

### Amazon SNS
- ✅ Pub/Sub: one publish → multiple subscribers simultaneously (fan-out)
- ✅ Protocols: SQS, Lambda, HTTP/S, Email, SMS, Mobile Push, Kinesis Firehose
- ✅ Message filtering = attribute-based filter per subscription
- ✅ FIFO SNS → FIFO SQS only; for ordered fan-out
- ✅ SNS + SQS fan-out = durability (messages wait in SQS if consumer is down)
- ✅ No message persistence in SNS (push-only); messages not stored
- ✅ SNS DLQ = for failed delivery to HTTP/Lambda endpoints

### Amazon EventBridge
- ✅ Default bus = AWS service events; Custom bus = your app events; Partner bus = SaaS (Salesforce, etc.)
- ✅ Rules = event pattern matching (JSON operators) OR schedule (cron/rate)
- ✅ Up to 5 targets per rule
- ✅ API Destinations = send events to external HTTP webhooks (PagerDuty, Jira, Datadog)
- ✅ Archive + Replay = store events and re-run for debugging/testing
- ✅ Pipes = source → filter → enrich → transform → target (point-to-point)
- ✅ Schema Registry = auto-discover event schemas; generate code bindings
- ✅ EventBridge Scheduler = standalone scheduling service for millions of schedules

### AWS Step Functions
- ✅ Standard = exactly-once, 1-year max, full history; Express = at-least-once, 5-min max
- ✅ State types: Task, Choice, Wait, Parallel, Map, Pass, Succeed, Fail
- ✅ Task Token (Wait for Callback) = pause for days/weeks awaiting external signal
- ✅ Retry with backoff and Catch for error handling built-in (no code needed)
- ✅ Map state = process each item in an array
- ✅ Parallel state = run multiple branches simultaneously
- ✅ Direct SDK integrations = call DynamoDB, ECS, SageMaker, etc. without Lambda

### Amazon MQ
- ✅ Managed ActiveMQ or RabbitMQ — for migrating existing broker applications
- ✅ Protocols: AMQP, MQTT, OpenWire, STOMP (ActiveMQ); AMQP 0-9-1 (RabbitMQ)
- ✅ "Migrate ActiveMQ/RabbitMQ to AWS" → Amazon MQ (zero code changes)
- ✅ "New cloud-native app" → SQS or SNS (not MQ)
- ✅ HA: Active/Standby (ActiveMQ) or Cluster (RabbitMQ)
- ✅ Deployed in VPC; not public internet

### AWS AppSync
- ✅ Managed GraphQL API; client requests exactly the data it needs
- ✅ Real-time subscriptions via WebSocket (MQTT over WebSocket)
- ✅ Pipeline resolvers = chain multiple data sources in one query
- ✅ 5 auth modes: API Key, IAM, Cognito, OIDC, Lambda
- ✅ Works with: DynamoDB, Lambda, RDS (Aurora Serverless), HTTP, OpenSearch, EventBridge
- ✅ Amplify DataStore = offline-first sync (AppSync + DynamoDB)
- ✅ AppSync vs API Gateway: AppSync = GraphQL + real-time + multiple sources; APIGW = REST/HTTP

### Amazon AppFlow
- ✅ No-code SaaS ↔ AWS data integration (50+ connectors)
- ✅ Triggers: scheduled, event-based (new record in Salesforce), on-demand
- ✅ Transformations: masking, truncation, validation, mapping, concatenation
- ✅ Destinations: S3, Redshift, DynamoDB, OpenSearch, Salesforce, Snowflake
- ✅ AppFlow vs Glue: AppFlow = SaaS connectors, no-code; Glue = complex ETL, any data
- ✅ AppFlow vs DataSync: AppFlow = SaaS apps; DataSync = file systems/S3/EFS

### Critical Cross-Service Rules
- ✅ "Decouple" → SQS (buffer), SNS (fan-out), or EventBridge (routing)
- ✅ "Exactly-once" + "ordered" → SQS FIFO
- ✅ "Fan-out + durability" → SNS + SQS (not SNS alone)
- ✅ "Migrate ActiveMQ/RabbitMQ" → Amazon MQ
- ✅ "Orchestrate workflow with state" → Step Functions
- ✅ "Route AWS events" → EventBridge default bus
- ✅ "Route SaaS events" → EventBridge partner bus
- ✅ "GraphQL + real-time" → AppSync
- ✅ "SaaS to S3/Redshift, no code" → AppFlow

---
---

<a name="analytics"></a>
# PART 8 — Analytics: Athena, Glue, EMR, Kinesis, MSK, Lake Formation, OpenSearch, QuickSight, Redshift, Data Pipeline & Data Exchange

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

---
---

<a name="ml"></a>
# PART 9 — Machine Learning: Comprehend, Forecast, Fraud Detector, Kendra, Lex, Polly, Rekognition, SageMaker, Textract, Transcribe & Translate

# AWS Exam Study Guide: Machine Learning Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Comprehend · Forecast · Fraud Detector · Kendra · Lex · Polly · Rekognition · SageMaker · Textract · Transcribe · Translate

---

## ML Services Quick-Reference Matrix

| Service | Input | Output | Exam Trigger Words |
|---|---|---|---|
| **Comprehend** | Text | Entities, sentiment, key phrases, topics | "NLP", "sentiment analysis", "entity extraction", "text classification" |
| **Forecast** | Time-series data | Future predictions | "demand forecasting", "predict future values", "time-series" |
| **Fraud Detector** | Events/transactions | Fraud probability score | "fraud detection", "account takeover", "online transactions" |
| **Kendra** | Documents/data sources | Search results (intelligent) | "enterprise search", "natural language search", "find answers in docs" |
| **Lex** | Voice/text | Intent + slots (conversational AI) | "chatbot", "voice bot", "conversational interface", "Alexa" |
| **Polly** | Text | Audio (speech) | "text-to-speech", "voice", "convert text to audio" |
| **Rekognition** | Images/Video | Labels, faces, text, moderation | "image analysis", "face detection", "content moderation", "video analysis" |
| **SageMaker** | Data + code | Trained ML models, endpoints | "build ML model", "train", "deploy", "custom ML", "notebooks" |
| **Textract** | Documents (images/PDF) | Structured text + forms + tables | "extract text from PDF", "form extraction", "OCR", "document analysis" |
| **Transcribe** | Audio | Text transcript | "speech-to-text", "transcribe audio", "subtitles", "call center audio" |
| **Translate** | Text | Translated text | "language translation", "multilingual", "translate content" |

---

## ML Service Categories (for Exam Context)

```
PRE-BUILT AI SERVICES (no ML expertise needed — just call the API):
  Vision:     Rekognition, Textract
  Speech:     Transcribe (speech→text), Polly (text→speech)
  Language:   Comprehend, Translate, Lex
  Search:     Kendra
  Forecasting: Forecast
  Fraud:      Fraud Detector

CUSTOM ML PLATFORM (ML expertise needed):
  SageMaker — build, train, tune, deploy your own models
```

---

---

## 1. Amazon Comprehend

### What Problem Does It Solve?
Understanding the meaning inside large amounts of unstructured text — customer reviews, support tickets, social media, legal documents — is impossible to do manually at scale. Comprehend applies **Natural Language Processing (NLP)** to automatically extract meaning, sentiment, entities, and topics from text without any ML expertise.

**Real-World Example:**
> An airline processes 50,000 customer support emails daily. Comprehend automatically detects that 60% have negative sentiment, identifies entities like "flight numbers" and "cities", classifies tickets into categories (delay, lost baggage, refund), and routes them to the right team — reducing manual triage from 5 hours to seconds.

---

### Comprehend Capabilities

| Capability | Description | Example Output |
|---|---|---|
| **Sentiment Analysis** | Positive, Negative, Neutral, Mixed | `"Your service was terrible"` → NEGATIVE (0.97) |
| **Entity Recognition** | Identifies named entities | `"Amazon in Seattle"` → ORG: Amazon, LOC: Seattle |
| **Key Phrase Extraction** | Identifies important phrases | `"machine learning platform"`, `"cost reduction"` |
| **Language Detection** | Identifies language of text | `"Bonjour"` → French (0.99) |
| **Syntax Analysis** | Parts of speech tagging | Nouns, verbs, adjectives per word |
| **Topic Modeling** | Groups documents by topic (unsupervised) | Document cluster: "product quality", "shipping" |
| **Custom Classification** | Train custom text classifier on your categories | "billing", "technical support", "sales" |
| **Custom Entity Recognition** | Train to recognize your domain-specific entities | Product codes, internal IDs, medical terms |
| **PII Detection** | Identify/redact Personally Identifiable Information | SSNs, credit card numbers, emails in text |
| **Targeted Sentiment** | Sentiment about specific entities | "The camera is great but battery is terrible" → camera=POSITIVE, battery=NEGATIVE |

---

### Comprehend Medical

- A specialized version for **healthcare and medical text**.
- Extracts: medical conditions, medications, dosages, treatments, anatomy, test results.
- Use case: Analyze clinical notes, medical records, drug adverse event reports.

---

### Comprehend vs Kendra vs Lex

| Feature | Comprehend | Kendra | Lex |
|---|---|---|---|
| **Input** | Raw text | Documents / data sources | Conversational input (text/voice) |
| **Output** | Entities, sentiment, topics | Search results and answers | Intent + slots (structured) |
| **Use case** | Analyze text content | Find answers in documents | Build chatbots/voice bots |
| **User interaction** | ❌ Batch analysis | ❌ Search queries | ✅ Multi-turn conversation |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Analyze 10,000 product reviews to find common complaints | Comprehend (sentiment + key phrases) |
| Detect PII in documents before storing in S3 | Comprehend PII detection |
| Route support tickets to correct team based on content | Comprehend custom classification |
| Extract medical conditions from clinical notes | Comprehend Medical |
| Build a chatbot for customer service | Amazon Lex (not Comprehend) |

---

---

## 2. Amazon Forecast

### What Problem Does It Solve?
Predicting future values — demand, sales, web traffic, inventory, energy consumption — requires sophisticated time-series ML models. Traditionally this required data scientists and weeks of model tuning. Amazon Forecast applies the **same ML technology used by Amazon.com** to automatically build accurate forecasting models from your historical data.

**Real-World Example:**
> A retail chain uploads 3 years of weekly product sales data. Forecast automatically trains multiple models (DeepAR, ARIMA, Prophet, ETS), selects the best one, and generates 52-week demand predictions per product per store — enabling optimized inventory ordering that reduces stockouts by 30% and overstock costs by 25%.

---

### How Forecast Works

```
Input Data (upload to S3):
  ├── Target time series: historical values you want to predict
  │   (e.g., weekly sales per product per store)
  ├── Related time series (optional): factors that affect target
  │   (e.g., promotions, price changes, holidays)
  └── Item metadata (optional): attributes about items
      (e.g., product category, brand, color)
            ↓
Dataset Group: combines all datasets
            ↓
Predictor (AutoML or choose algorithm):
  ├── ARIMA
  ├── ETS (Exponential Smoothing)
  ├── NPTS (Non-Parametric Time Series)
  ├── DeepAR+ (deep learning — Amazon's proprietary)
  ├── Prophet (Facebook open source)
  └── AutoML (Forecast selects best automatically)
            ↓
Forecast (generated predictions):
  ├── Point forecast (single most likely value)
  └── Probabilistic forecast (P10, P50, P90 quantiles)
            ↓
Query via API or export to S3
```

---

### Key Forecast Concepts

| Concept | Description |
|---|---|
| **Dataset Group** | Container for all related datasets |
| **Target Time Series** | Historical values being predicted (required) |
| **Related Time Series** | External factors (e.g., promotions) that affect predictions |
| **Predictor** | Trained ML model; Forecast trains multiple and picks best |
| **Forecast** | Generated predictions from a predictor |
| **Forecast Horizon** | How far into the future to predict |
| **Forecast Frequency** | Resolution of predictions (daily, weekly, monthly) |
| **Quantile Loss** | Accuracy metric; P50 = median, P90 = 90th percentile |

---

### Forecast vs SageMaker for Time Series

| Feature | Amazon Forecast | SageMaker (Custom) |
|---|---|---|
| **ML expertise needed** | ❌ None | ✅ Data scientist required |
| **Setup** | Upload data, click create | Build, train, tune model code |
| **Algorithms** | Pre-built (ARIMA, DeepAR+, Prophet) | Any algorithm (TensorFlow, XGBoost, etc.) |
| **Use case** | Time-series forecasting specifically | Any ML problem |
| **Custom features** | Limited | Unlimited |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Predict next quarter's product demand without data scientists | Amazon Forecast |
| Forecast energy consumption for next 30 days | Amazon Forecast |
| Predict website traffic for capacity planning | Amazon Forecast |
| Build a custom recommendation engine | Amazon SageMaker |

---

---

## 3. Amazon Fraud Detector

### What Problem Does It Solve?
Online fraud — fake account creation, payment fraud, account takeover, promotion abuse — costs businesses billions annually. Building fraud detection models requires labeled fraud data and ML expertise. Fraud Detector uses **Amazon's own fraud detection experience** and ML to automatically build, train, and deploy fraud detection models.

**Real-World Example:**
> An e-commerce company loses $2M annually to fraudulent orders. Fraud Detector analyzes patterns in 2 years of order history (marked as fraudulent or legitimate), builds a real-time scoring model, and flags new orders with a fraud probability score in milliseconds. Orders above 0.8 score are blocked; 0.5–0.8 go to manual review; below 0.5 are auto-approved — reducing fraud losses by 80%.

---

### Fraud Detector Key Concepts

| Concept | Description |
|---|---|
| **Event** | A business action to evaluate (order, login, account registration) |
| **Entity** | Who is performing the event (customer, IP address, device) |
| **Variable** | Input attribute for the model (IP, email, purchase amount, device ID) |
| **Model** | ML model trained on your labeled fraud data |
| **Detector** | Evaluates events in real time using models + rules |
| **Rules** | Business logic to interpret model output (if score > 0.8, block) |
| **Outcome** | Result of evaluation (approve, review, block) |

---

### Fraud Detector Model Types

| Model Type | Use Case |
|---|---|
| **Online Fraud Insights (OFI)** | New account fraud, online payment fraud |
| **Transaction Fraud Insights (TFI)** | Credit card / payment transaction fraud |
| **Account Takeover Insights (ATI)** | Login anomaly detection, credential stuffing |

---

### How Fraud Detector Works

```
Historical Events + Labels (fraud/not-fraud) → stored in S3
            ↓
Create Detector + Variables
            ↓
Train Model (Fraud Detector auto-trains ML model)
            ↓
Define Rules (e.g., if model_score > 0.80 → BLOCK)
            ↓
Deploy Detector
            ↓
Real-time API call with event data:
  {
    "email": "user@example.com",
    "ip_address": "192.168.1.1",
    "order_amount": 1500,
    "billing_zip": "10001"
  }
            ↓
Response: { "outcome": "BLOCK", "model_score": 0.94 }
```

---

### Fraud Detector vs SageMaker for Fraud

| Feature | Fraud Detector | SageMaker |
|---|---|---|
| **Domain expertise** | Built-in fraud detection expertise | General ML; you build fraud logic |
| **Setup** | Point to data, train, deploy | Build data pipeline + model code |
| **Real-time API** | ✅ Native | ✅ (SageMaker endpoint) |
| **Interpretability** | ✅ (explains which variables drove score) | Manual implementation |
| **Use case** | Online fraud specifically | Any classification problem |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Detect fraudulent online orders in real time | Amazon Fraud Detector |
| Detect account takeover attempts at login | Fraud Detector (ATI model) |
| Score new user registrations for fraud risk | Fraud Detector (OFI model) |
| Build custom fraud model with domain-specific features | SageMaker |

---

---

## 4. Amazon Kendra

### What Problem Does It Solve?
Traditional keyword-based enterprise search returns results containing matching words but cannot understand meaning. Employees waste hours searching SharePoint, Confluence, S3, databases for answers. Kendra provides **intelligent enterprise search** using ML to understand natural language questions and return precise answers from across your organization's documents.

**Real-World Example:**
> An insurance company has 500,000 documents in SharePoint, S3, and a Salesforce knowledge base. An agent asks: *"What is the maximum liability coverage for commercial trucks in California?"* Traditional search returns 200 documents containing those words. Kendra returns the exact paragraph from the correct policy document with the answer highlighted — in under 1 second.

---

### Kendra Data Source Connectors

Kendra can index content from:

| Data Source | Examples |
|---|---|
| **Amazon S3** | PDFs, Word, PowerPoint, HTML, text files |
| **SharePoint** | Microsoft SharePoint Online and Server |
| **Salesforce** | Knowledge articles, cases |
| **ServiceNow** | Knowledge base, incidents |
| **RDS / Aurora** | Database content |
| **OneDrive** | Microsoft OneDrive |
| **Confluence** | Atlassian Confluence pages |
| **Google Drive** | Documents, sheets |
| **Custom data source** | Via API |

---

### Kendra Features

| Feature | Description |
|---|---|
| **Semantic search** | Understands meaning, not just keywords |
| **FAQ matching** | Match questions to pre-defined Q&A pairs |
| **Document ranking** | Relevance tuning — boost specific documents |
| **Access control** | ACL filtering; users only see authorized content |
| **Incremental sync** | Automatically re-index changed documents |
| **Query suggestions** | Auto-complete and suggested queries |
| **Experience Builder** | Build search UI without coding |
| **Custom attributes** | Add metadata for filtering results |

---

### Kendra vs OpenSearch vs Comprehend

| Feature | Kendra | OpenSearch | Comprehend |
|---|---|---|---|
| **Search type** | Semantic / intelligent | Keyword + full-text | NLP analysis (not search) |
| **Input** | Documents, data sources | Any data | Raw text only |
| **Natural language** | ✅ Understands questions | Limited | ✅ (analysis) |
| **Setup** | Managed, connectors | Manual index setup | API call |
| **Use case** | Enterprise document search | Log search, e-commerce | Text analysis pipeline |
| **Cost** | High (Enterprise) | Moderate | Per API call |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Employees search across SharePoint, S3, Confluence with natural language | Amazon Kendra |
| "Find all HR policies about parental leave" from 10,000 documents | Amazon Kendra |
| Build a customer-facing FAQ bot backed by a knowledge base | Kendra (FAQ feature) + Lex (chatbot interface) |
| Log search and analytics dashboard | OpenSearch (not Kendra) |

---

---

## 5. Amazon Lex

### What Problem Does It Solve?
Building conversational chatbots and voice interfaces requires understanding user intent from natural language, managing multi-turn conversations, and integrating with backend systems. Lex (the same technology behind **Amazon Alexa**) provides a **managed service** to build conversational interfaces for any application using voice or text.

**Real-World Example:**
> A bank builds a customer service bot: customers say or type *"I want to check my account balance"* → Lex understands the intent (CheckBalance), asks for the account number (slot), validates it, calls a Lambda function to query the database, and responds *"Your checking account balance is $1,247.50"* — handling thousands of simultaneous conversations without any human agents.

---

### Core Lex Concepts

| Concept | Description | Example |
|---|---|---|
| **Intent** | What the user wants to do | `CheckBalance`, `BookFlight`, `OrderPizza` |
| **Utterances** | Sample phrases that trigger the intent | "What's my balance", "Show me my account", "Check balance" |
| **Slot** | A piece of information needed to fulfill the intent | AccountType (checking/savings), Date, City |
| **Slot Type** | Data type for a slot | Built-in (date, number, city) or custom (pizza size: small/medium/large) |
| **Fulfillment** | What happens when all slots are collected | Lambda function invocation |
| **Confirmation Prompt** | Ask user to confirm before fulfilling | "You want to transfer $500 to savings. Is that right?" |
| **Elicitation** | Asking for a missing slot | "Which account do you want to check — checking or savings?" |
| **Session Attributes** | Data passed between turns | User ID, conversation context |
| **Bot alias** | Version pointer for the bot | `PROD`, `DEV` pointing to different bot versions |

---

### Lex V2 vs Lex V1

| Feature | Lex V2 (Current) | Lex V1 (Legacy) |
|---|---|---|
| **Languages** | Multi-language in one bot | One language per bot |
| **Streaming** | ✅ Streaming conversations | ❌ |
| **Conversation flow** | Visual builder | JSON config |
| **Intent confirmation** | Improved | Basic |
| **Use** | All new bots | Legacy only |

---

### Lex Integration Points

| Service | Role |
|---|---|
| **Lambda** | Fulfillment (execute business logic, query DB) |
| **API Gateway** | Expose Lex bot via REST API for web chat |
| **Amazon Connect** | Build IVR (Interactive Voice Response) for call centers |
| **Kendra** | Use Kendra as knowledge base for FAQ answers in Lex |
| **Polly** | Lex uses Polly for text-to-speech responses |
| **Transcribe** | Converts voice input to text for Lex to process |
| **CloudWatch** | Monitor conversation metrics |
| **S3** | Store conversation logs |

---

### Lex Multi-Turn Conversation Flow

```
User: "I want to book a flight"
        ↓
Lex identifies Intent: BookFlight
        ↓
Lex checks required Slots:
  - Origin city: ❌ missing → "Where are you flying from?"
User: "New York"
  - Origin city: ✅ JFK
  - Destination: ❌ missing → "Where would you like to go?"
User: "Los Angeles"
  - Destination: ✅ LAX
  - Travel date: ❌ missing → "When do you want to travel?"
User: "Next Friday"
  - Date: ✅ 2026-05-29
        ↓
Confirmation: "Book flight from JFK to LAX on May 29. Confirm?"
User: "Yes"
        ↓
Fulfillment: Lambda function → search flights → return options
        ↓
Response: "I found 3 flights. The cheapest is $299 departing at 8am."
```

---

### Lex vs Comprehend vs Kendra (Conversational Context)

| Need | Service |
|---|---|
| "Build a multi-turn chatbot" | Lex |
| "Analyze sentiment of chatbot conversations" | Comprehend |
| "Answer questions from documents in a chatbot" | Lex + Kendra integration |
| "Classify customer support emails" | Comprehend |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Build a customer service chatbot for a banking app | Amazon Lex |
| Build an Alexa Skill for ordering products | Amazon Lex (same technology) |
| Call center IVR that understands natural language | Amazon Lex + Amazon Connect |
| Chatbot that answers questions from HR policy documents | Amazon Lex + Amazon Kendra |
| Voice-enabled bot responses as audio | Lex + Polly (text-to-speech) |

---

---

## 6. Amazon Polly

### What Problem Does It Solve?
Applications need to speak to users — accessibility tools, e-learning platforms, navigation systems, IVR systems. Recording human voice for every possible response is expensive and inflexible. Polly converts **text to lifelike speech** using deep learning, supporting dozens of languages and voices.

**Real-World Example:**
> An e-learning platform automatically converts course text into audio using Polly's neural voice, creating a podcast-style audio track for every lesson. Students can listen while commuting — and when course text is updated, the audio is automatically regenerated without re-recording.

---

### Polly Voice Types

| Type | Description | Quality |
|---|---|---|
| **Standard voices** | Concatenative TTS; older technology | Good |
| **Neural voices (NTTS)** | Deep learning; more natural and human-like | Excellent |
| **Long-form** | Optimized for reading long-form content (news articles, books) | Best for extended content |
| **Brand voices** | Custom voice trained on your brand's recordings | Enterprise (contact AWS) |
| **Generative voices** | Most expressive; highest quality; newest | Best overall expressiveness |

---

### Polly Key Features

| Feature | Description |
|---|---|
| **Languages** | 30+ languages, 60+ voices |
| **SSML support** | Speech Synthesis Markup Language — control pronunciation, rate, pitch, pauses |
| **Lexicons** | Custom pronunciation dictionary (e.g., "AWS" → "Amazon Web Services") |
| **Speech marks** | Returns metadata about word/phoneme timing (for lip sync, karaoke) |
| **Output format** | MP3, OGG, PCM |
| **Streaming** | Returns audio stream synchronously (real-time playback) |
| **Storage** | Save output as MP3 in S3 for reuse |

---

### SSML Example

```xml
<speak>
  Welcome to <emphasis level="strong">AWS Polly</emphasis>.
  <break time="1s"/>
  Your account balance is
  <say-as interpret-as="currency">$1,247.50</say-as>.
  <prosody rate="slow">Please review your statement carefully.</prosody>
</speak>
```

---

### Polly vs Transcribe (Common Confusion)

| Service | Direction | Use Case |
|---|---|---|
| **Polly** | Text → Speech (audio) | "Make the app talk" |
| **Transcribe** | Speech (audio) → Text | "Transcribe what was said" |

> **Mnemonic:** Polly **speaks** (like a parrot). Transcribe **listens and writes**.

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Convert news articles to audio podcast format | Amazon Polly |
| Generate voice prompts for an IVR system | Amazon Polly |
| Accessibility feature: read website text aloud | Amazon Polly |
| Convert call center recordings to text | Amazon Transcribe (not Polly) |
| Generate speech with custom pronunciation for brand terms | Polly with custom Lexicons |

---

---

## 7. Amazon Rekognition

### What Problem Does It Solve?
Analyzing images and videos manually — detecting objects, faces, inappropriate content, or reading text — is impossible at scale. Rekognition provides **pre-trained computer vision** models that analyze images and videos via a simple API call, with no ML expertise required.

**Real-World Example:**
> A social media platform receives 10 million photo uploads per day. Rekognition automatically: (1) Detects and blurs nudity/violence before any human sees it, (2) Groups photos by person (face recognition), (3) Extracts searchable tags ("beach", "dog", "sunset") from each photo, (4) Detects and indexes text in signs and memes — all in real time.

---

### Rekognition Capabilities (Images)

| Capability | Description | Example |
|---|---|---|
| **Label Detection** | Detect objects, scenes, activities | "Dog", "Beach", "Swimming", "Person" |
| **Face Detection** | Detect faces and attributes | Age range, gender, emotions, glasses, smile |
| **Face Comparison** | Compare two faces for similarity | "Is this the same person?" (0.0–100.0 similarity) |
| **Face Search (Collections)** | Search a face against a collection | Security: match against employee database |
| **Text Detection** | Read text in images | Signs, license plates, book covers |
| **Content Moderation** | Detect inappropriate content | Nudity, violence, drugs, alcohol — with confidence score |
| **Celebrity Recognition** | Identify famous people | "This is Elon Musk (99.8% confidence)" |
| **Custom Labels** | Train on your own images | Detect your specific products, defects, logos |
| **PPE Detection** | Detect Personal Protective Equipment | Hardhats, vests, masks on construction site |

---

### Rekognition Capabilities (Video)

| Capability | Description |
|---|---|
| **Label Detection** | Objects and scenes in video frames |
| **Face Detection + Recognition** | Faces across video frames over time |
| **Content Moderation** | Detect inappropriate content in video |
| **Person Tracking** | Track a person's path through video |
| **Celebrity Recognition** | Identify celebrities in video |
| **Technical Cue Detection** | Detect scene changes, black frames, credits |
| **Text Detection** | Read text appearing in video |
| **Activity Detection** | Detect activities (e.g., "drinking", "hugging") |

---

### Rekognition Face Collections

- A **collection** is a database of indexed faces.
- Add faces with `IndexFaces` → search with `SearchFacesByImage`.
- Use case: Employee badge verification, unauthorized person detection, finding a person in hours of surveillance video.

```
Add employee photos to collection (IndexFaces)
        ↓
Security camera captures frame
        ↓
SearchFacesByImage → match against collection
        ↓
MATCH: John Smith (similarity: 98.2%) → grant access
NO MATCH → trigger alert
```

---

### Rekognition Video Analysis (Async)

For videos stored in S3:
1. Start analysis job (StartLabelDetection, StartFaceDetection, etc.)
2. Job runs asynchronously
3. Get notified via **SNS** when complete
4. Retrieve results via GetLabelDetection, GetFaceDetection, etc.

> **Exam Tip:** Rekognition video analysis is **asynchronous** (SNS notification pattern). Image analysis is **synchronous** (immediate response).

---

### Rekognition Custom Labels

- Train Rekognition to detect **your own custom objects** using your labeled images.
- Process: Upload training images → Label with bounding boxes → Train model → Deploy → Call API.
- Use case: Quality control (detect defects on assembly line), logo detection, medical imaging.

---

### Rekognition vs Textract (Common Confusion)

| Service | Focus | Input | Output |
|---|---|---|---|
| **Rekognition** | Visual content analysis | Images/Video | Labels, faces, text detection |
| **Textract** | Document text extraction | Documents (forms, tables, PDFs) | Structured text, form fields, table data |

> **Key Difference:** Rekognition reads text **visible in images** (signs, photos). Textract reads text from **documents** with structure (forms, tables, handwriting).

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Automatically moderate uploaded images for nudity | Rekognition Content Moderation |
| Verify employee identity from webcam photo vs ID photo | Rekognition Face Comparison |
| Find all videos in an archive containing a specific person | Rekognition Face Collections + video analysis |
| Detect hardhat usage on construction site | Rekognition PPE Detection |
| Read text on a license plate in a parking lot photo | Rekognition Text Detection |
| Detect your product logo in social media images | Rekognition Custom Labels |
| Extract data from an insurance form PDF | Amazon Textract (not Rekognition) |

---

---

## 8. Amazon SageMaker

### What Problem Does It Solve?
Building, training, and deploying ML models involves: setting up compute environments, managing training clusters, storing model artifacts, optimizing hyperparameters, deploying endpoints, and monitoring production models. SageMaker is a **fully managed end-to-end ML platform** that handles all of this, letting data scientists focus on models rather than infrastructure.

**Real-World Example:**
> A streaming service wants to build a custom recommendation engine. Data scientists use SageMaker Studio to explore data, write training code in a managed notebook, train a TensorFlow model on a 100-GPU cluster, automatically tune hyperparameters, deploy the trained model to a scalable endpoint, and monitor prediction quality in production — all from one unified platform.

---

### SageMaker Architecture Overview

```
DATA PREPARATION:
  SageMaker Studio → JupyterLab notebooks → S3 (data storage)
  SageMaker Data Wrangler → visual data preparation + transformation
  SageMaker Feature Store → centralized feature repository

MODEL TRAINING:
  SageMaker Training Jobs → managed EC2 clusters (any instance type)
  Built-in algorithms → XGBoost, Linear Learner, DeepAR, BlazingText, etc.
  Custom training → bring your own TensorFlow, PyTorch, Scikit-learn code
  SageMaker Experiments → track and compare training runs
  SageMaker Debugger → detect training issues in real time

HYPERPARAMETER TUNING:
  SageMaker Hyperparameter Tuning → automated HPO (Bayesian optimization)

MODEL OPTIMIZATION:
  SageMaker Clarify → bias detection + model explainability
  SageMaker Neo → compile models for edge devices (Jetson, Raspberry Pi)

DEPLOYMENT:
  SageMaker Endpoint → real-time inference (auto-scaling)
  SageMaker Batch Transform → offline batch inference
  SageMaker Serverless Inference → pay-per-request (spiky workloads)
  SageMaker Async Inference → async for large inputs

MONITORING:
  SageMaker Model Monitor → detect data drift, model quality degradation

MLOPS:
  SageMaker Pipelines → CI/CD for ML (automated retraining pipeline)
  SageMaker Model Registry → version and approve models
  SageMaker Projects → templates for MLOps best practices
```

---

### SageMaker Key Components

#### SageMaker Studio
- **Unified IDE** for ML: notebooks, training, debugging, experiments, pipelines.
- Web-based; runs on managed infrastructure.
- Replaces local Jupyter notebooks for team collaboration.

#### SageMaker Notebook Instances (Legacy)
- Managed EC2 with Jupyter pre-installed.
- Being replaced by Studio, but still used.

#### SageMaker Training Jobs
- Launch managed training clusters of any size.
- Bring your own code (Python script) with any framework.
- Frameworks supported: TensorFlow, PyTorch, MXNet, Scikit-learn, Spark, Hugging Face.
- Distributed training: data parallelism, model parallelism.
- Spot Training: use EC2 Spot Instances to reduce training cost by up to **90%**.

#### SageMaker Built-In Algorithms

| Algorithm | Use Case |
|---|---|
| **XGBoost** | Classification, regression (tabular data) |
| **Linear Learner** | Linear/logistic regression |
| **DeepAR** | Time-series forecasting |
| **BlazingText** | Text classification, word embeddings |
| **Object Detection** | Detect objects in images |
| **Image Classification** | Classify images |
| **Semantic Segmentation** | Pixel-level image analysis |
| **K-Means** | Clustering |
| **PCA** | Dimensionality reduction |
| **Factorization Machines** | Recommendation, click prediction |
| **Random Cut Forest** | Anomaly detection |

#### SageMaker Inference Options

| Type | Latency | Cost | Use Case |
|---|---|---|---|
| **Real-time Endpoint** | Milliseconds | Per hour | Continuous low-latency predictions |
| **Serverless Inference** | Seconds (cold start) | Per request | Infrequent/spiky traffic |
| **Async Inference** | Minutes | Per inference | Large payloads (>6MB), long processing |
| **Batch Transform** | Minutes to hours | Per instance-hour | Offline bulk inference on S3 data |

---

### SageMaker Ground Truth

- Managed **data labeling** service.
- Human labelers (private team, Mechanical Turk, or vendor) label your training data.
- Auto-labeling: ML model labels easy examples; humans label only uncertain ones.
- Use case: Label images for object detection, transcribe audio, classify text.

---

### SageMaker Feature Store

- Centralized repository for **ML features** (pre-computed input variables).
- **Online store**: low-latency lookup for real-time inference.
- **Offline store**: S3-based for training data retrieval.
- Prevents duplicate feature engineering across teams.

---

### SageMaker JumpStart

- Pre-built **ML solutions and foundation models** (one-click deploy).
- Includes: Llama, Stable Diffusion, BERT, ResNet, and 300+ other models.
- Fine-tune foundation models on your own data.
- Use case: Quick prototyping, starting point for custom models.

---

### SageMaker vs Pre-Built AI Services

| Scenario | SageMaker | Pre-Built Service |
|---|---|---|
| Detect faces in images | SageMaker (custom model) | **Rekognition** ✅ |
| Build custom defect detector | **SageMaker** ✅ | Rekognition Custom Labels |
| Transcribe audio | SageMaker | **Transcribe** ✅ |
| Forecast next quarter demand | SageMaker | **Forecast** ✅ |
| Need full control over model | **SageMaker** ✅ | Limited |
| Domain-specific ML (medical, legal) | **SageMaker** ✅ | Limited |

> **Exam Rule:** Use pre-built AI services (Rekognition, Comprehend, etc.) for common tasks with no ML expertise. Use **SageMaker** when you need a **custom model**, full control, or the pre-built services don't meet your needs.

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Build a custom ML model with your own training data | SageMaker |
| Reduce training cost by 90% using spot capacity | SageMaker Spot Training |
| Deploy ML model that auto-scales based on traffic | SageMaker Real-time Endpoint with Auto Scaling |
| Label 100,000 images for training dataset | SageMaker Ground Truth |
| Detect data drift in production ML model | SageMaker Model Monitor |
| One-click deploy of Llama LLM | SageMaker JumpStart |
| Batch predict on 10 million records overnight | SageMaker Batch Transform |
| MLOps pipeline for automated retraining | SageMaker Pipelines + Model Registry |

---

---

## 9. Amazon Textract

### What Problem Does It Solve?
Organizations deal with millions of paper and digital documents — tax forms, invoices, medical records, contracts, loan applications. Manual data entry is slow, expensive, and error-prone. OCR (Optical Character Recognition) tools extract raw text but cannot understand **document structure** (which text is in which form field, which data is in which table cell). Textract automatically extracts **structured text, form fields, and tables** from any document.

**Real-World Example:**
> A mortgage lender receives 500 loan applications per day as scanned PDFs. Textract automatically extracts: applicant name, address, income (from form fields), employer history (from a table), and signature (from a handwritten field) — populating the loan management system in seconds instead of 10 minutes of manual data entry per application.

---

### Textract Feature Types

| Feature | Description | Output |
|---|---|---|
| **DetectDocumentText** | Extract all raw text (OCR) | Lines and words with bounding boxes |
| **AnalyzeDocument (FORMS)** | Extract key-value pairs from forms | `"Name:" → "John Smith"`, `"DOB:" → "01/01/1990"` |
| **AnalyzeDocument (TABLES)** | Extract table structure with rows/columns | Structured table data |
| **AnalyzeDocument (SIGNATURES)** | Detect handwritten signatures | Signature location + confidence |
| **AnalyzeExpense** | Extract data from invoices/receipts | Vendor, total, line items, taxes |
| **AnalyzeID** | Extract data from identity documents | Driver's license, passport fields |
| **StartDocumentAnalysis** | Async analysis for multi-page PDFs | Job ID → poll or SNS notification |

---

### Textract in a Document Processing Pipeline

```
Document arrives (S3 via email, scan, upload)
        ↓
S3 Event → Lambda
        ↓
Textract AnalyzeDocument (FORMS + TABLES)
        ↓
Structured output (JSON):
  ├── Key-value pairs (form fields)
  ├── Table data (rows + columns)
  └── Raw text blocks
        ↓
Lambda processes JSON → populate database
        ↓
Human review (A2I) for low-confidence extractions
        ↓
Data stored in DynamoDB / RDS
```

---

### Amazon A2I (Augmented AI) Integration

- **Amazon Augmented AI (A2I)** adds human review for Textract low-confidence results.
- Define confidence threshold: if Textract confidence < 80%, route to human reviewer.
- Reviewers correct mistakes → model improves over time.
- Integrated with: Textract, Rekognition, SageMaker.

---

### Textract vs Rekognition Text Detection

| Feature | Textract | Rekognition (Text Detection) |
|---|---|---|
| **Purpose** | Extract structured data from documents | Read text in natural scene images |
| **Output** | Form fields, tables, key-value pairs | Text blocks + bounding boxes |
| **Document structure** | ✅ Understands forms and tables | ❌ Raw text only |
| **Use case** | Process forms, invoices, contracts | Read license plates, signs, memes |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Extract all fields from a scanned W-2 tax form | Amazon Textract (FORMS) |
| Extract data from a table in a PDF financial statement | Amazon Textract (TABLES) |
| Extract vendor, amount, date from an invoice | Textract AnalyzeExpense |
| Verify passport information during identity check | Textract AnalyzeID |
| Read a license plate number from a security camera photo | Rekognition Text Detection (not Textract) |
| Add human review for low-confidence document extractions | Textract + Amazon A2I |

---

---

## 10. Amazon Transcribe

### What Problem Does It Solve?
Businesses generate enormous amounts of audio — call center recordings, medical dictations, video content, meetings. Transcribing audio manually is slow, expensive, and unscalable. Amazon Transcribe provides **automatic speech recognition (ASR)** that converts audio to accurate text, with additional features like speaker identification, custom vocabulary, and real-time streaming.

**Real-World Example:**
> A call center records 10,000 customer service calls daily. Transcribe automatically converts each recording to text. The transcripts are then analyzed by Comprehend for sentiment and entity extraction — identifying calls where customers are angry (negative sentiment) or mention competitor names — providing insights that previously required listening to thousands of hours of audio.

---

### Transcribe Features

| Feature | Description | Use Case |
|---|---|---|
| **Batch transcription** | Upload audio to S3, get transcript | Call recordings, podcast episodes |
| **Real-time streaming** | Stream audio, get transcript live | Live captioning, real-time assistant |
| **Speaker diarization** | Identify who is speaking (Speaker 1, 2, 3) | Call center: separate agent vs customer |
| **Custom vocabulary** | Add domain-specific words | Medical terms, company names, jargon |
| **Vocabulary filters** | Mask/remove profanity or specific words | Content moderation |
| **Custom language model** | Fine-tune on your domain's language | Medical, legal, financial terminology |
| **PII redaction** | Remove PII from transcripts | HIPAA compliance, privacy |
| **Language detection** | Auto-detect spoken language | Multilingual call centers |
| **Content redaction** | Redact sensitive info in audio + transcript | Credit card numbers in call recordings |
| **Subtitles** | Generate SRT/VTT subtitle files | Video captioning |
| **Medical Transcribe** | Specialized for medical dictation | Clinical notes, doctor-patient conversations |

---

### Transcribe Medical

- Optimized for **medical terminology** and clinical contexts.
- Supports two scenarios:
  - **Conversation**: Doctor-patient dialogue
  - **Dictation**: Doctor dictating notes
- Output includes specialized medical entities (body parts, medical conditions, medications).
- HIPAA eligible.

---

### Transcribe + Other Services Pipeline

```
Audio file in S3 (call recording)
        ↓
Amazon Transcribe (batch transcription)
  ├── Speaker diarization: Agent vs Customer separated
  ├── PII redaction: removes SSNs, credit cards
  └── Output: transcript JSON in S3
        ↓
Amazon Comprehend (NLP on transcript)
  ├── Sentiment analysis: Customer was NEGATIVE
  ├── Entity extraction: mentioned competitor "Acme Corp"
  └── Key phrases: "cancel subscription", "worst service"
        ↓
Results stored in DynamoDB → dashboard for supervisors
```

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Convert recorded customer service calls to searchable text | Amazon Transcribe |
| Generate subtitles for video content automatically | Amazon Transcribe (subtitle output) |
| Identify which person is speaking in a conference recording | Transcribe speaker diarization |
| Remove profanity from podcast transcripts | Transcribe vocabulary filter |
| Transcribe doctor's dictations with medical terminology | Amazon Transcribe Medical |
| Real-time captions for a live video stream | Amazon Transcribe real-time streaming |

---

---

## 11. Amazon Translate

### What Problem Does It Solve?
Building multilingual applications requires translating content into dozens of languages. Hiring human translators for every piece of content is expensive and slow. Amazon Translate provides **neural machine translation** that produces fluent, accurate translations at scale, in real time.

**Real-World Example:**
> A global e-commerce platform has 5 million product listings in English. Instead of hiring translators for 20 languages (100 million translations), they automatically translate all listings using Translate, making products searchable and purchasable by customers worldwide — in their native language — overnight.

---

### Translate Features

| Feature | Description |
|---|---|
| **Real-time translation** | Synchronous API for on-the-fly translation |
| **Batch translation** | Translate large volumes of documents asynchronously via S3 |
| **Custom terminology** | Ensure specific terms translate consistently (brand names, technical terms) |
| **Formality** | Control formal vs informal tone (available in select language pairs) |
| **Profanity masking** | Mask profanity in translated output |
| **Active Custom Translation** | Fine-tune translation style using parallel translation examples |
| **Supported languages** | 75+ language pairs |

---

### Custom Terminology

- Define how specific terms should always be translated.
- Example: "Amazon" should never be translated to another word; "AWS" stays "AWS".
- Prevents brand names, technical terms, product names from being incorrectly localized.

---

### Translate in a Multilingual App Pipeline

```
User submits content in any language
        ↓
Amazon Transcribe (if audio) or direct text
        ↓
Amazon Comprehend (detect source language)
        ↓
Amazon Translate (→ target language)
        ↓
Amazon Polly (convert translated text to speech)
        ↓
Multilingual audio response delivered to user
```

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Translate customer support chats from 20 languages to English | Amazon Translate real-time |
| Translate 1 million documents in S3 overnight | Amazon Translate batch translation |
| Ensure brand name "Zephyr" is never translated | Translate custom terminology |
| Build a real-time multilingual chat app | Translate API + WebSocket |
| Translate foreign language call recording | Transcribe → Translate pipeline |

---

---

## Master Comparison: All ML Services

### Speech Pipeline (Input → Output)

```
Human speaks
    ↓
Amazon Transcribe (Speech → Text)
    ↓
Amazon Comprehend (Understand text)
    ↓
Amazon Translate (Text → Another language)
    ↓
Amazon Polly (Text → Speech)
    ↓
Human hears in their language
```

### Document Pipeline

```
Document (image/PDF)
    ↓
Amazon Textract (Extract text + structure)
    ↓
Amazon Comprehend (Analyze meaning)
    ↓
Results stored + acted upon
```

### Vision Pipeline

```
Image/Video
    ↓
Amazon Rekognition (Labels, faces, text, moderation)
    ↓
Results: content tags, face matches, inappropriate content flags
```

### Conversation Pipeline

```
User speaks → Amazon Transcribe (speech to text)
    ↓
Amazon Lex (understand intent + manage dialog)
    ↓
AWS Lambda (business logic)
    ↓
Response text → Amazon Polly (text to speech)
    ↓
User hears response
```

---

## "Which ML Service?" Decision Tree

```
Text input → want to ANALYZE it?
    ├── Find entities, sentiment, key phrases → Comprehend
    ├── Build a chatbot / conversation → Lex
    ├── Search documents by question → Kendra
    └── Detect PII → Comprehend

Audio input?
    ├── Convert speech to text → Transcribe
    └── Want speech output? → Polly

Text → convert to speech?
    └── Polly

Image/Video input?
    ├── Detect objects, faces, labels → Rekognition
    └── Extract text from document/form → Textract

Numbers/time-series → predict future?
    └── Forecast

Transaction/event → fraud risk?
    └── Fraud Detector

Need custom ML model?
    └── SageMaker

Need language translation?
    └── Translate
```

---

## Architecture Patterns to Know

### Pattern 1: Intelligent Document Processing
```
S3 (uploaded documents: invoices, forms, contracts)
        ↓
Lambda trigger → Textract (extract form data + tables)
        ↓
Comprehend (detect entities + sentiment in extracted text)
        ↓
A2I (route low-confidence extractions to human review)
        ↓
DynamoDB (store structured results)
        ↓
QuickSight (analytics dashboard)
```

### Pattern 2: Call Center Analytics
```
Amazon Connect (call center platform)
        ↓
Audio recordings → S3
        ↓
Transcribe (speech → text, speaker diarization, PII redaction)
        ↓
Comprehend (sentiment per call, entity extraction)
        ↓
OpenSearch (full-text search on transcripts)
        ↓
QuickSight Dashboard:
  ├── % negative sentiment calls by product
  ├── Top complaint phrases
  └── Agent performance scores
```

### Pattern 3: Real-Time Content Moderation
```
User uploads image to S3
        ↓
S3 Event → Lambda
        ↓
Rekognition: Content Moderation API
        ↓
If inappropriate (confidence > 90%):
    ├── Delete from S3
    ├── Notify user via SNS/email
    └── Log to DynamoDB for audit

If borderline (50–90%):
    └── Route to human reviewer (Amazon A2I)

If clean (<50%):
    └── Publish image to CDN
```

### Pattern 4: Multilingual Voice Assistant
```
User speaks question (in any language)
        ↓
Kinesis Video Streams (audio capture)
        ↓
Amazon Transcribe (speech → text, language detection)
        ↓
Amazon Translate (→ English)
        ↓
Amazon Lex (understand intent)
        ↓
Lambda (fetch answer from Kendra knowledge base)
        ↓
Amazon Translate (English → user's language)
        ↓
Amazon Polly (text → speech in user's language)
        ↓
Audio response to user
```

### Pattern 5: MLOps Pipeline with SageMaker
```
New training data arrives in S3
        ↓
EventBridge → triggers SageMaker Pipeline
        ↓
Step 1: Data preprocessing (SageMaker Processing Job)
Step 2: Model training (SageMaker Training Job with Spot)
Step 3: Model evaluation (accuracy > threshold?)
Step 4: Register model (SageMaker Model Registry)
Step 5: Human approval gate
Step 6: Deploy to endpoint (SageMaker Endpoint update)
        ↓
SageMaker Model Monitor (detect drift)
        ↓
CloudWatch Alarm → trigger retraining pipeline if drift detected
```

---

## Exam Tips Summary

### Amazon Comprehend
- ✅ NLP service: sentiment, entities, key phrases, language detection, PII, topic modeling
- ✅ Custom classification + custom entity recognition for domain-specific needs
- ✅ Comprehend Medical for healthcare text (clinical notes, medical records)
- ✅ Targeted sentiment = sentiment about specific entities within text

### Amazon Forecast
- ✅ Time-series forecasting — no ML expertise required
- ✅ Supports: ARIMA, ETS, DeepAR+, Prophet, AutoML
- ✅ Related time series + item metadata improve accuracy
- ✅ P10/P50/P90 quantile forecasts (not just single point)

### Amazon Fraud Detector
- ✅ Real-time fraud scoring using your historical labeled data
- ✅ OFI (online/payment fraud), TFI (transaction fraud), ATI (account takeover)
- ✅ Rules layer on top of model score → business outcomes (BLOCK/REVIEW/ALLOW)

### Amazon Kendra
- ✅ Intelligent semantic search across enterprise documents
- ✅ Connectors: S3, SharePoint, Salesforce, ServiceNow, Confluence, RDS, Google Drive
- ✅ Kendra + Lex = FAQ chatbot backed by document knowledge base
- ✅ Access control filtering — users only see authorized content

### Amazon Lex
- ✅ Intents (what user wants) + Slots (info needed) + Utterances (how they say it)
- ✅ Multi-turn conversation management built-in
- ✅ Amazon Connect integration for IVR call centers
- ✅ Fulfillment via Lambda; conversation flows = build once, deploy everywhere

### Amazon Polly
- ✅ Text → Speech; Transcribe = Speech → Text (opposite directions!)
- ✅ Neural voices = most natural; SSML = control pronunciation/rate/pitch
- ✅ Custom Lexicons for brand-specific pronunciation
- ✅ Output formats: MP3, OGG, PCM; can store in S3 for reuse

### Amazon Rekognition
- ✅ Images: Labels, faces, face comparison, text, content moderation, PPE, celebrities
- ✅ Video: Async (SNS notification) for stored videos; person tracking, activity detection
- ✅ Face Collections = searchable face database (IndexFaces → SearchFacesByImage)
- ✅ Custom Labels = train on your own images for domain-specific detection
- ✅ Rekognition ≠ Textract: Rekognition = visual scene analysis; Textract = document structure

### Amazon SageMaker
- ✅ Full ML lifecycle: data prep → training → tuning → deployment → monitoring
- ✅ Spot Training = up to 90% cost reduction for training jobs
- ✅ Inference types: Real-time (low latency), Serverless (spiky/infrequent), Async (large payloads), Batch (offline)
- ✅ Ground Truth = managed data labeling with auto-labeling
- ✅ Feature Store = centralized feature repo (online=real-time, offline=training)
- ✅ JumpStart = pre-built models and solutions (LLMs, vision models)
- ✅ Model Monitor = detect data drift in production

### Amazon Textract
- ✅ Extracts structured data from documents: text, forms (key-value), tables
- ✅ AnalyzeExpense = invoices/receipts; AnalyzeID = passports/driver's licenses
- ✅ Async for multi-page PDFs (SNS notification when done)
- ✅ A2I integration for human review of low-confidence extractions
- ✅ Textract ≠ Rekognition text: Textract = document structure; Rekognition = scene text

### Amazon Transcribe
- ✅ Speech → Text (ASR); Polly = opposite direction
- ✅ Speaker diarization = who said what (call center agent vs customer)
- ✅ PII/content redaction in both transcript and audio
- ✅ Custom vocabulary for domain terms; custom language model for fine-tuning
- ✅ Transcribe Medical = HIPAA-eligible, clinical terminology
- ✅ Real-time streaming for live captioning

### Amazon Translate
- ✅ Neural machine translation; 75+ languages
- ✅ Custom terminology = ensure brand names/terms translate consistently
- ✅ Batch translation for large S3 document volumes
- ✅ Often combined with Transcribe + Polly for multilingual voice pipeline
- ✅ Active Custom Translation = fine-tune with parallel examples

---
---

<a name="mgmt"></a>
# PART 10 — Management & Governance: CloudFormation, CloudTrail, CloudWatch, Config, Control Tower, Organizations, Systems Manager, Trusted Advisor & more

# AWS Exam Study Guide: Management & Governance
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Auto Scaling · CloudFormation · CloudTrail · CloudWatch · CLI · Compute Optimizer · Config · Control Tower · Health Dashboard · License Manager · Managed Grafana · Managed Prometheus · Management Console · Organizations · Proton · Service Catalog · Systems Manager · Trusted Advisor · Well-Architected Tool

---

## Management & Governance Quick-Reference Matrix

| Service | Category | What It Does | Exam Trigger Words |
|---|---|---|---|
| **Auto Scaling** | Elasticity | Scale EC2/ECS/DynamoDB/Aurora automatically | "scale in/out", "capacity", "target tracking", "step scaling" |
| **CloudFormation** | IaC | Provision AWS resources via templates | "infrastructure as code", "stack", "template", "drift detection" |
| **CloudTrail** | Audit | Log all API calls across AWS account | "who did what", "API audit", "compliance logging", "governance" |
| **CloudWatch** | Observability | Metrics, logs, alarms, dashboards | "monitor", "alarm", "log group", "metric filter" |
| **CLI** | Access | Command-line AWS access | "automate", "script", "programmatic" |
| **Compute Optimizer** | Cost/Perf | Right-size recommendations for EC2/Lambda/EBS | "right-sizing", "over-provisioned", "EC2 recommendations" |
| **Config** | Compliance | Track resource configuration changes + rules | "configuration history", "compliance rule", "drift", "who changed" |
| **Control Tower** | Governance | Multi-account setup with guardrails | "landing zone", "guardrails", "multi-account governance" |
| **Health Dashboard** | Awareness | AWS service health + your account events | "service outage", "maintenance", "my account impact" |
| **License Manager** | Licensing | Track and manage software licenses | "BYOL", "license tracking", "Windows Server license" |
| **Managed Grafana** | Visualization | Managed Grafana dashboards for AWS data | "Grafana", "dashboards", "visualization" |
| **Managed Prometheus** | Monitoring | Managed Prometheus for container metrics | "Prometheus", "EKS metrics", "PromQL" |
| **Management Console** | Access | Web-based AWS management UI | "console access", "web UI" |
| **Organizations** | Governance | Multi-account management + SCPs | "multiple accounts", "SCP", "consolidated billing" |
| **Proton** | DevOps | Managed deployment templates for microservices | "self-service deployment", "platform team", "microservice templates" |
| **Service Catalog** | Governance | Approved product portfolio for self-service | "approved products", "self-service", "compliant provisioning" |
| **Systems Manager** | Operations | Manage EC2/on-prem ops at scale | "patch", "run command", "parameter store", "session manager" |
| **Trusted Advisor** | Best Practices | Checks across cost, security, performance, limits | "best practice check", "savings", "security recommendations" |
| **Well-Architected Tool** | Review | Review workloads against 6 pillars | "architectural review", "best practice assessment", "pillars" |

---

---

## 1. AWS Auto Scaling

### What Problem Does It Solve?
Auto Scaling automatically adjusts capacity to maintain performance and minimize cost — scaling **out** (add capacity) during high demand and **in** (remove capacity) during low demand.

**Real-World Example:**
> An e-commerce site normally runs 4 EC2 instances. During Black Friday, traffic spikes 10x. Auto Scaling automatically launches 36 more instances, then terminates them after the sale — paying only for what was used.

---

### Auto Scaling Components

#### EC2 Auto Scaling Group (ASG)
- **Launch Template** (or Launch Configuration — legacy): defines AMI, instance type, SGs, user data.
- **Min/Max/Desired Capacity**: hard limits and target count.
- **Scaling policies**: when and how to scale.
- **Health checks**: EC2 status checks or ELB health checks.

#### Auto Scaling Policies

| Policy Type | How It Works | Use Case |
|---|---|---|
| **Target Tracking** | Maintain a target metric value (e.g., 50% CPU) | Simple, most common |
| **Step Scaling** | Scale by defined increments at metric thresholds | Need different response at different alarm levels |
| **Simple Scaling** | Scale once per alarm; wait for cooldown | Legacy; replaced by step scaling |
| **Scheduled Scaling** | Scale at defined times (cron) | Predictable traffic patterns |
| **Predictive Scaling** | ML-based forecast; scales proactively | Recurring patterns (daily/weekly peaks) |

> **Exam Tip:** Target Tracking is the simplest and recommended default. "Predictive scaling" key phrase = use Predictive Scaling policy.

---

### Scaling Metrics (Target Tracking)

- **ASGAverageCPUUtilization** — average CPU across all instances
- **ASGAverageNetworkIn/Out** — network throughput
- **ALBRequestCountPerTarget** — requests per ALB target
- **Custom metrics** — any CloudWatch metric

---

### Cooldown Period

- Time after a scaling activity during which no further scaling occurs.
- Default: **300 seconds** for simple scaling.
- **Target tracking and step scaling** have their own cooldown (separate from simple).
- Purpose: Allow new instances to start and metrics to stabilize before another action.

---

### Launch Template vs Launch Configuration

| Feature | Launch Template | Launch Configuration |
|---|---|---|
| **Status** | ✅ Current, recommended | Legacy (no new features) |
| **Versioning** | ✅ Yes | ❌ No |
| **Spot + On-Demand mix** | ✅ Yes | ❌ No |
| **Multiple instance types** | ✅ Yes | ❌ No |
| **T2/T3 unlimited** | ✅ Yes | ❌ No |

---

### ASG Instance Lifecycle Hooks

- **Pending:Wait** → Custom action on new instance before it's InService (e.g., install software, warm up cache)
- **Terminating:Wait** → Custom action before termination (e.g., drain connections, save state)
- Timeout: default **3600 seconds** (1 hour)
- Use with: SNS, SQS, Lambda, EventBridge

---

### Multi-AZ Auto Scaling

- ASG distributes instances **evenly across AZs**.
- **Rebalancing**: If AZ becomes unbalanced (e.g., instance failure), ASG launches in under-represented AZ and terminates in over-represented.
- ALB cross-zone load balancing works with ASG for even traffic distribution.

---

### ASG Termination Policy

Default order:
1. AZ with most instances
2. Oldest Launch Configuration/Template
3. Instance closest to next billing hour
4. Random

Custom policies: **OldestInstance, NewestInstance, OldestLaunchTemplate, ClosestToNextInstanceHour**

---

### Other Auto Scaling Targets

| Service | Scaling Dimension |
|---|---|
| **ECS** | Task count per service |
| **DynamoDB** | Read/Write Capacity Units |
| **Aurora** | Read replica count |
| **Spot Fleet** | Instance count |
| **Lambda** | Provisioned concurrency |
| **AppStream 2.0** | Fleet capacity |

---

### Key Exam Scenarios

- **"Scale EC2 based on CPU usage"**: ASG with Target Tracking (CPUUtilization = 50%).
- **"Scale at 7am, scale down at 8pm"**: Scheduled scaling policy.
- **"Pre-scale before Monday morning traffic spike"**: Predictive scaling.
- **"Custom startup script before instance serves traffic"**: Lifecycle hook (Pending:Wait).
- **"Blue/green deployment with ASG"**: Two ASGs — shift traffic on ALB.
- **"DynamoDB table auto-scales during peak"**: Application Auto Scaling on DynamoDB.

---

---

## 2. AWS CloudFormation

### What Problem Does It Solve?
Manually creating AWS resources through the console is slow, error-prone, and not repeatable. CloudFormation lets you define your **entire infrastructure as code** (IaC) in JSON or YAML templates — and provisions all resources automatically, in the right order, with dependency management.

**Real-World Example:**
> A startup defines their entire environment (VPC, subnets, RDS, EC2 ASG, ALB, S3, CloudFront) in a single 300-line YAML template. They can spin up identical environments for dev, staging, and production with one command — guaranteed consistency.

---

### CloudFormation Template Structure

```yaml
AWSTemplateFormatVersion: "2010-09-09"
Description: "Web application stack"

Parameters:          # Input values at deploy time (e.g., env name, instance type)
  EnvironmentName:
    Type: String
    Default: prod

Mappings:            # Static lookup tables (e.g., AMI ID per region)
  RegionAMI:
    us-east-1:
      AMI: ami-0abcdef1234567890

Conditions:          # Conditional resource creation (e.g., only create bastion in prod)
  IsProd: !Equals [!Ref EnvironmentName, prod]

Resources:           # REQUIRED — actual AWS resources to create
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t3.micro
      ImageId: !FindInMap [RegionAMI, !Ref AWS::Region, AMI]

Outputs:             # Values to export (e.g., ALB DNS name, VPC ID)
  LoadBalancerDNS:
    Value: !GetAtt MyALB.DNSName
    Export:
      Name: !Sub "${EnvironmentName}-ALB-DNS"
```

---

### Key CloudFormation Concepts

#### Stack
- A collection of AWS resources managed as a single unit.
- **Create/Update/Delete** the stack → all resources are created/updated/deleted together.
- Stack status: `CREATE_COMPLETE`, `UPDATE_IN_PROGRESS`, `ROLLBACK_COMPLETE`, etc.

#### StackSet
- Deploy **one CloudFormation template across multiple AWS accounts and regions** simultaneously.
- Managed by an administrator account.
- Uses: multi-account baseline security (CloudTrail, Config, IAM roles) across an Organization.

#### Change Set
- **Preview** what changes will be made before executing an update.
- Shows: resources to add, modify, or delete.
- Critical for production — review before applying.

#### Drift Detection
- Detects when actual resource configuration **differs from the template** (manual changes made outside CloudFormation).
- Status: `IN_SYNC` or `DRIFTED`.

#### Nested Stacks
- Stacks that reference other stacks as resources (using `AWS::CloudFormation::Stack`).
- Break large templates into reusable components (networking stack, security stack, app stack).
- Parent stack manages child stacks.

#### Stack Outputs + Cross-Stack References
- Export values from one stack → import in another using `Fn::ImportValue`.
- Allows loose coupling between stacks (networking stack exports VPC ID → app stack imports it).

---

### CloudFormation Rollback

- If a stack creation fails → **automatic rollback** deletes all created resources.
- If a stack update fails → **rolls back to previous working state**.
- `--disable-rollback` flag: keep partially created resources for debugging.

---

### CloudFormation Helper Scripts (cfn-init, cfn-signal)

| Script | Purpose |
|---|---|
| **cfn-init** | Reads `AWS::CloudFormation::Init` metadata; installs packages, creates files, starts services |
| **cfn-signal** | Signals CloudFormation that the EC2 instance is ready (for `CreationPolicy`) |
| **cfn-hup** | Daemon that detects metadata changes and re-runs cfn-init |
| **cfn-get-metadata** | Retrieves metadata from CloudFormation |

---

### CloudFormation vs Terraform vs CDK

| Feature | CloudFormation | Terraform | AWS CDK |
|---|---|---|---|
| **Language** | JSON/YAML | HCL | Python, TypeScript, Java, .NET |
| **Provider** | AWS only | Multi-cloud | AWS only (compiles to CFN) |
| **State management** | AWS manages | Terraform state file | AWS manages |
| **Drift detection** | ✅ | ✅ | ✅ (via CFN) |
| **Cost** | Free | Free (open source) | Free |

---

### Key Exam Scenarios

- **"Deploy identical infrastructure across dev/staging/prod"**: CloudFormation templates.
- **"Deploy baseline security setup across 50 accounts"**: CloudFormation StackSets.
- **"Preview changes before updating production stack"**: Change Set.
- **"Someone manually changed an S3 bucket policy; detect it"**: CloudFormation Drift Detection.
- **"Reusable networking template used by multiple app stacks"**: Nested Stacks + Cross-Stack References.
- **"EC2 instance must signal CloudFormation when app is ready"**: cfn-signal + CreationPolicy.

---

---

## 3. AWS CloudTrail

### What Problem Does It Solve?
Every action in AWS — via Console, CLI, SDK, or service — is an API call. CloudTrail records **every API call** as an event log, answering: *"Who did what, from where, at what time, on which resource?"*

**Real-World Example:**
> A security team discovers an S3 bucket was made public at 2:47am. CloudTrail shows: User `john.doe@company.com` called `PutBucketAcl` from IP `203.0.113.45` at `2026-01-15T02:47:23Z` — immediately identifying the culprit.

---

### CloudTrail Event Types

| Type | What It Logs | Default Enabled? | Cost |
|---|---|---|---|
| **Management Events** | Control-plane API calls: CreateEC2, DeleteS3Bucket, AttachIAMPolicy | ✅ Yes (free for 1 trail) | Free for first copy |
| **Data Events** | Data-plane operations: S3 GetObject/PutObject, Lambda Invoke, DynamoDB PutItem | ❌ No (must enable) | Additional cost |
| **Insight Events** | Detects unusual API call rates (anomaly detection) | ❌ No | Additional cost |
| **Network Activity Events** | VPC endpoint API activity | ❌ No | Additional cost |

---

### CloudTrail Trail Types

| Trail | Coverage | Notes |
|---|---|---|
| **Single-region trail** | One region only | Created in one region |
| **All-region trail** | All regions (recommended) | Single trail covers everything; sent to one S3 bucket |
| **Organization trail** | All accounts in AWS Organization | Created in management account; member accounts cannot modify |

---

### CloudTrail Log Storage + Analysis

| Destination | Use Case |
|---|---|
| **S3 bucket** | Default; long-term storage; query with Athena |
| **CloudWatch Logs** | Real-time monitoring and alerting via metric filters |
| **CloudTrail Lake** | Managed data lake; SQL query without S3/Athena setup |

---

### CloudTrail + CloudWatch Logs Integration

```
CloudTrail Events → CloudWatch Logs Group
                         ↓ (Metric Filter)
                    CloudWatch Metric
                         ↓ (Alarm)
                    CloudWatch Alarm
                         ↓ (Action)
                    SNS → Email / Lambda
```

**Example Use Cases:**
- Alert when root account is used (filter on `userIdentity.type = Root`)
- Alert when IAM policies change
- Alert when security group rules are modified
- Alert when CloudTrail itself is stopped

---

### CloudTrail Log Integrity

- **Log file validation**: CloudTrail creates a **digest file** every hour with hashes of all log files.
- Validates that logs have not been modified or deleted since delivery.
- Ensures tamper-proof audit trail for compliance.

---

### CloudTrail vs AWS Config

| Feature | CloudTrail | AWS Config |
|---|---|---|
| **What it records** | API calls / actions | Resource configurations |
| **Question it answers** | "Who did what?" | "What changed in resource config?" |
| **History** | Event history (API calls) | Configuration timeline |
| **Compliance rules** | ❌ | ✅ Config Rules |
| **Use case** | Security forensics, audit | Compliance, drift detection |

> **Exam Rule:** "Who called the API?" = CloudTrail. "What changed in the resource configuration?" = AWS Config. They complement each other.

---

### Key Exam Scenarios

- **"Determine who deleted an S3 bucket"**: CloudTrail Data Events on S3.
- **"Detect unusual number of API calls (potential attack)"**: CloudTrail Insights.
- **"Ensure CloudTrail logs are not tampered with"**: Log file validation (digest files).
- **"Apply CloudTrail to all accounts in the org"**: Organization trail.
- **"Alert when root account logs in"**: CloudTrail → CloudWatch Logs → Metric Filter → Alarm → SNS.
- **"Query CloudTrail logs with SQL"**: CloudTrail Lake or Athena on S3.

---

---

## 4. Amazon CloudWatch

### What Problem Does It Solve?
CloudWatch is the **central observability service** for AWS — collecting metrics, logs, and events from virtually every AWS service and custom application, enabling monitoring, alerting, and automated responses.

**Real-World Example:**
> A web app's CloudWatch alarm detects CPU > 80% for 5 minutes → triggers ASG scale-out → new instances launch → SNS sends alert to the ops team. All automatically, without human intervention.

---

### CloudWatch Components

#### Metrics
- **Numerical time-series data** sent by AWS services and applications.
- Namespace: groups related metrics (e.g., `AWS/EC2`, `AWS/RDS`).
- **Dimensions**: Key-value attributes to filter metrics (e.g., InstanceId, AutoScalingGroupName).
- **Resolution**:
  - **Standard**: 1-minute granularity (default for most services)
  - **High Resolution**: 1-second granularity (custom metrics; `--storage-resolution 1`)
- **Retention**: 1-minute metrics stored 15 days; 5-minute stored 63 days; 1-hour stored 15 months.

#### Key EC2 Default Metrics (Available Without Agent)
- CPUUtilization, NetworkIn/Out, DiskReadOps/WriteOps, StatusCheckFailed
- **NOT available by default**: Memory usage, disk space — requires **CloudWatch Agent**

---

### CloudWatch Agent

- Installed on EC2 or on-premises servers.
- Collects: **memory, disk usage, custom application metrics, logs**.
- Config stored in **SSM Parameter Store** for centralized management.
- Required for: memory metrics, custom OS-level metrics.

---

### CloudWatch Alarms

| State | Description |
|---|---|
| **OK** | Metric within threshold |
| **ALARM** | Metric breaches threshold |
| **INSUFFICIENT_DATA** | Not enough data to evaluate |

**Alarm Actions:**
- EC2 actions: Stop, Terminate, Reboot, Recover
- Auto Scaling actions: Scale in/out
- SNS notification
- Systems Manager OpsCenter (create OpsItem)

**Composite Alarms:**
- Combine multiple alarms with AND/OR logic.
- Reduce alarm noise: only alert when both CPU > 80% AND memory > 80%.

---

### CloudWatch Logs

| Concept | Description |
|---|---|
| **Log Group** | Container for log streams (e.g., `/aws/lambda/my-function`) |
| **Log Stream** | Sequence of events from one source (e.g., one Lambda instance) |
| **Retention** | 1 day to 10 years (default: never expire) |
| **Metric Filter** | Extract metrics from log patterns (e.g., count ERROR occurrences) |
| **Insights** | Interactive SQL-like query language for log analysis |
| **Subscription Filter** | Stream logs to Lambda, Kinesis, OpenSearch in real time |

---

### CloudWatch Logs Insights Query Example

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| stats count(*) as errorCount by bin(5m)
| sort errorCount desc
| limit 20
```

---

### CloudWatch Container Insights

- Collects metrics and logs from **ECS, EKS, Kubernetes, Fargate**.
- Provides CPU, memory, network, disk metrics at cluster/service/pod level.
- Requires CloudWatch agent as a DaemonSet on EKS nodes.

---

### CloudWatch Application Insights

- Automated monitoring setup for **.NET and SQL Server applications**.
- Detects anomalies and creates OpsItems in Systems Manager.

---

### CloudWatch Dashboards

- Custom views of metrics and alarms.
- Can include data from **multiple regions and accounts**.
- Supports: line charts, stacked area charts, numbers, text widgets.
- Can be shared publicly or with specific AWS accounts.

---

### CloudWatch Events / EventBridge

> CloudWatch Events has been renamed **Amazon EventBridge** (same service, enhanced).

- **Rules**: Match events → route to targets.
- **Scheduled rules**: Cron-based triggers (e.g., run Lambda every 5 minutes).
- **Event patterns**: Match on service/event type (e.g., EC2 state change).

---

### CloudWatch vs CloudTrail vs AWS Config

| Feature | CloudWatch | CloudTrail | AWS Config |
|---|---|---|---|
| **Primary use** | Metrics, logs, alarms | API call audit | Resource config compliance |
| **Data type** | Time-series metrics + logs | API event logs | Config snapshots + history |
| **Real-time alerting** | ✅ | Via CW integration | ✅ (via SNS) |
| **Question** | "Is the system healthy?" | "Who did what?" | "What changed?" |

---

### Key Exam Scenarios

- **"Alert when EC2 CPU > 80% for 5 minutes"**: CloudWatch Alarm on CPUUtilization.
- **"Monitor memory on EC2 instances"**: CloudWatch Agent (memory not in default metrics).
- **"Query Lambda logs for errors in last 1 hour"**: CloudWatch Logs Insights.
- **"Stream application logs to OpenSearch for analysis"**: CloudWatch Logs Subscription Filter → Kinesis → OpenSearch.
- **"Single dashboard showing metrics from us-east-1 and eu-west-1"**: CloudWatch Cross-Region Dashboard.
- **"Alert only when both CPU and memory are high (reduce noise)"**: CloudWatch Composite Alarm.
- **"Trigger Lambda every 15 minutes"**: EventBridge Scheduled Rule.

---

---

## 5. AWS Organizations

### What Problem Does It Solve?
As companies grow, they need multiple AWS accounts for security isolation, billing separation, and workload separation. Organizations provides **centralized management** of multiple accounts — with policy enforcement, consolidated billing, and shared services.

**Real-World Example:**
> Netflix has hundreds of AWS accounts — one per team, service, or environment. Organizations groups them under a single management account, applies SCPs to prevent accidental production changes, and consolidates all billing into one invoice with volume discounts.

---

### Organizations Structure

```
Root (Management Account)
├── Organizational Unit (OU): Security
│   ├── Account: Audit
│   └── Account: Log Archive
├── Organizational Unit (OU): Production
│   ├── Account: Prod-App-1
│   └── Account: Prod-App-2
├── Organizational Unit (OU): Development
│   ├── Account: Dev-Team-A
│   └── Account: Dev-Team-B
└── Organizational Unit (OU): Sandbox
    └── Account: Experiments
```

---

### Service Control Policies (SCPs)

- **JSON policies** attached to Root, OUs, or individual accounts.
- Define the **maximum permissions** available in an account — they do NOT grant permissions.
- Even the account root user is restricted by SCPs.
- **Evaluated with IAM**: SCP must allow AND IAM must allow for an action to succeed.

**Example SCP — Prevent Disabling CloudTrail:**
```json
{
  "Effect": "Deny",
  "Action": ["cloudtrail:DeleteTrail", "cloudtrail:StopLogging"],
  "Resource": "*"
}
```

**Example SCP — Restrict to Specific Regions:**
```json
{
  "Effect": "Deny",
  "Action": "*",
  "Resource": "*",
  "Condition": {
    "StringNotEquals": {
      "aws:RequestedRegion": ["us-east-1", "eu-west-1"]
    }
  }
}
```

---

### Organizations Features

| Feature | Description |
|---|---|
| **Consolidated Billing** | Single payment method; volume discounts across all accounts |
| **Reserved Instance sharing** | RI/Savings Plan benefits shared across org accounts |
| **AWS RAM integration** | Share resources across accounts in the org |
| **Trusted Access** | Enable AWS services (Config, CloudTrail, Backup) to work org-wide |
| **Delegated Administrator** | Designate a member account (not management) to administer specific services |

---

### Organizations vs Control Tower

| Feature | Organizations | Control Tower |
|---|---|---|
| **Accounts** | Manual account creation | Automated via Account Factory |
| **Guardrails** | Manual SCP writing | Pre-built preventive + detective guardrails |
| **Setup** | Manual multi-account structure | Automated landing zone setup |
| **Logging** | Manual setup | Automatic CloudTrail + Config in every account |
| **Use case** | Existing multi-account | New multi-account from scratch |

---

### Key Exam Scenarios

- **"Prevent any account from disabling CloudTrail"**: SCP on Root OU.
- **"Restrict all accounts to us-east-1 and eu-west-1 only"**: SCP with `aws:RequestedRegion` condition.
- **"Share Reserved Instance benefits across 20 accounts"**: AWS Organizations consolidated billing.
- **"Central account manages GuardDuty for all org accounts"**: Organizations + Delegated Administrator.
- **"New account automatically gets security baseline"**: Control Tower (not plain Organizations).

---

---

## 6. AWS Control Tower

### What Problem Does It Solve?
Setting up a secure, compliant multi-account AWS environment from scratch is complex — requiring manual SCP writing, account vending, logging setup, and baseline security. Control Tower **automates this entire process** using a pre-configured **landing zone**.

---

### Key Concepts

| Concept | Description |
|---|---|
| **Landing Zone** | Pre-configured multi-account environment with security baseline |
| **Account Factory** | Self-service portal (or API) to provision new accounts with approved baseline |
| **Guardrails** | Pre-built policies enforced across the landing zone |
| **Log Archive account** | Centralized S3 bucket for CloudTrail and Config logs from all accounts |
| **Audit account** | Read-only access to all accounts for security review |

---

### Guardrail Types

| Type | Mechanism | Description |
|---|---|---|
| **Preventive** | SCP | Block actions before they happen (e.g., "Disallow public S3 buckets") |
| **Detective** | AWS Config rule | Detect non-compliant configs after the fact (e.g., "Detect EC2 without encryption") |
| **Proactive** | CloudFormation hooks | Check resources before provisioning (newer feature) |

---

### Guardrail Status

| Status | Description |
|---|---|
| **Mandatory** | Always enabled; cannot be disabled (e.g., "Disallow changes to landing zone IAM roles") |
| **Strongly Recommended** | Best practice; optional |
| **Elective** | Optional; for specific use cases |

---

### Key Exam Scenarios

- **"Automatically provision new AWS accounts with security baseline"**: Control Tower Account Factory.
- **"Prevent developers from creating public S3 buckets in any account"**: Control Tower Preventive Guardrail.
- **"Detect unencrypted EBS volumes across all accounts"**: Control Tower Detective Guardrail.
- **"Centralized audit logging across all accounts"**: Control Tower Log Archive account.
- **"New multi-account setup from scratch with governance"**: Control Tower (not raw Organizations).

---

---

## 7. AWS Config

### What Problem Does It Solve?
AWS Config continuously records the **configuration state of every AWS resource** and evaluates it against compliance rules. It answers: *"What does my resource look like right now? What did it look like last Tuesday? Is it compliant with my policies?"*

**Real-World Example:**
> A compliance audit requires proof that all S3 buckets had versioning enabled for the past 12 months. AWS Config's configuration history shows the exact state of every bucket at every point in time — meeting the audit requirement with a few clicks.

---

### AWS Config vs CloudTrail (Detailed Comparison)

| Dimension | AWS Config | CloudTrail |
|---|---|---|
| **Records** | Resource configuration state | API calls (who called what) |
| **Timeline** | Configuration timeline per resource | Event log timeline |
| **Example question** | "Was this SG open to 0.0.0.0/0 last week?" | "Who opened this SG to 0.0.0.0/0?" |
| **Compliance rules** | ✅ Config Rules | ❌ |
| **Remediation** | ✅ Automatic (SSM Automation) | ❌ |

---

### Config Rules

| Type | Description | Examples |
|---|---|---|
| **AWS Managed Rules** | Pre-built by AWS (170+ rules) | `s3-bucket-public-read-prohibited`, `encrypted-volumes`, `required-tags` |
| **Custom Rules** | Lambda function evaluates resources | Any custom compliance requirement |
| **Service-Linked Rules** | Created by other AWS services | Security Hub, Control Tower |

---

### Conformance Packs

- Bundle of Config rules + remediation actions deployed as a single unit.
- Pre-built packs: **CIS Benchmark, PCI DSS, HIPAA, NIST**.
- Deployed across org accounts via Organizations.

---

### Automatic Remediation

```
Config Rule violation detected
          ↓
Config triggers remediation action
          ↓
SSM Automation document runs
(e.g., enable S3 versioning, close SG port 22)
          ↓
Resource brought back into compliance
          ↓
SNS notification sent
```

---

### Key Exam Scenarios

- **"Prove S3 buckets were encrypted 6 months ago"**: AWS Config configuration history.
- **"Automatically fix non-compliant Security Groups"**: Config Rule + SSM Automation remediation.
- **"Enforce required tags on all EC2 instances"**: Config Rule (`required-tags`).
- **"Check PCI DSS compliance across all accounts"**: Config Conformance Pack (PCI DSS pack).
- **"Alert when any EC2 is launched without encryption"**: Config Rule → SNS.

---

---

## 8. AWS Systems Manager (SSM)

### What Problem Does It Solve?
Managing hundreds or thousands of EC2 instances (and on-premises servers) — patching, configuration, secrets, remote access, and automation — is operationally complex. Systems Manager provides a **unified operations platform** to manage all of this without SSH/RDP or direct server access.

**Real-World Example:**
> A company has 500 EC2 instances. Instead of SSHing to each one to apply a security patch, SSM Patch Manager automatically scans all instances, identifies missing patches, and applies them during a defined maintenance window — with zero manual effort.

---

### Key SSM Capabilities

#### Session Manager
- **Secure shell/RDP access** to EC2 and on-premises servers without:
  - Opening port 22/3389
  - Managing SSH keys
  - Bastion hosts
- All sessions logged to CloudWatch Logs and S3.
- Works through SSM Agent + IAM role.

> **Exam Rule:** "Remove bastion host, eliminate port 22" → **Session Manager**.

#### Run Command
- Execute scripts or AWS-managed documents on one or many instances simultaneously.
- No SSH needed; fully audited.
- Use cases: Install software, run health checks, apply configuration.

#### Patch Manager
- Automates OS and application patching.
- **Patch Baseline**: Define which patches are approved/rejected.
- **Maintenance Window**: Schedule when patches are applied.
- Supports: Windows, Amazon Linux, Ubuntu, RHEL, CentOS, SUSE.

#### Parameter Store
- Hierarchical storage for **configuration data and secrets**.
- Two tiers:

| Tier | Max Size | Cost | Features |
|---|---|---|---|
| **Standard** | 4 KB | Free | Basic key-value |
| **Advanced** | 8 KB | $0.05/parameter/month | Parameter policies, larger values |

- **Types**: String, StringList, SecureString (KMS encrypted).
- **Hierarchy**: `/prod/db/password`, `/dev/api/key` — organize by env/service.
- Integrated with: Lambda, EC2, ECS, CloudFormation, CodeBuild, Secrets Manager.

> **Parameter Store vs Secrets Manager:**
> | | Parameter Store | Secrets Manager |
> |---|---|---|
> | **Auto-rotation** | ❌ | ✅ |
> | **Cost** | Free (Standard) | $0.40/secret/month |
> | **DB credential rotation** | ❌ | ✅ |
> | **Use for** | App config, feature flags | Passwords, API keys needing rotation |

#### Automation
- Run **runbooks** (automation documents) to automate common operational tasks.
- Examples: Restart EC2, create AMI, resize EBS, patch OS, remediate Config findings.
- Triggered by: Manual, scheduled, Config remediation, EventBridge.

#### Inventory
- Collect detailed **inventory data** from managed instances.
- Data: installed apps, OS info, network config, services, Windows Registry.
- Stored in S3; queryable via Athena.

#### State Manager
- **Enforce desired state** on instances.
- Associations: define what state (document) should be applied to which instances on a schedule.
- Example: Ensure CloudWatch agent is always installed and running.

#### OpsCenter
- Centralized place to **view, investigate, and resolve operational issues** (OpsItems).
- Aggregates findings from: CloudWatch, Config, CloudTrail, GuardDuty, Trusted Advisor.

---

### SSM Agent

- Must be installed on managed instances (pre-installed on Amazon Linux 2, some Windows AMIs).
- Instances need: SSM Agent + IAM role with `AmazonSSMManagedInstanceCore` policy.
- **No open ports required** — agent polls SSM service outbound.

---

### Key Exam Scenarios

- **"SSH to private EC2 without bastion host or port 22"**: Session Manager.
- **"Run a script on 1,000 EC2 instances simultaneously"**: Run Command.
- **"Automate monthly patching across all EC2 instances"**: Patch Manager + Maintenance Window.
- **"Store RDS password securely for Lambda"**: Parameter Store (SecureString) or Secrets Manager.
- **"Ensure CloudWatch agent is always running on all instances"**: State Manager association.
- **"On-premises servers managed by SSM"**: Install SSM Agent + register as managed instances (Hybrid Activations).

---

---

## 9. AWS CloudFormation vs AWS Service Catalog vs AWS Proton

> These three services all deal with "approved infrastructure deployment" but serve very different purposes.

---

## 10. AWS Service Catalog

### What Problem Does It Solve?
Developers want self-service access to AWS resources, but IT/security teams need to ensure only approved, compliant configurations are deployed. Service Catalog allows admins to **create a portfolio of approved products** that end users can launch themselves — without needing broad IAM permissions.

**Real-World Example:**
> A bank's IT team creates a "Compliant RDS PostgreSQL" product in Service Catalog — pre-configured with encryption, specific instance sizes, and approved VPCs. Developers can launch any number of compliant databases themselves without needing RDS IAM permissions or knowledge of security policies.

---

### Key Concepts

| Concept | Description |
|---|---|
| **Portfolio** | Collection of products; shared with specific IAM users/groups |
| **Product** | A CloudFormation template wrapped as a deployable unit |
| **Provisioned Product** | A running instance of a product (a deployed stack) |
| **Launch Constraint** | IAM role used to launch the product (user needs no underlying service permissions) |
| **Tag Update Constraint** | Control which tags users can change |

---

### Service Catalog vs CloudFormation vs Proton

| Feature | CloudFormation | Service Catalog | Proton |
|---|---|---|---|
| **Who creates templates** | DevOps/Platform team | Admins | Platform engineers |
| **Who deploys** | DevOps team (has IAM perms) | End users (via catalog, no perms needed) | Developers (via self-service) |
| **Use case** | IaC for technical teams | Governed self-service for all users | Microservice deployment at scale |
| **Governance** | Manual | ✅ Built-in (launch constraints) | ✅ Built-in |
| **Product type** | Raw template | CloudFormation-based products | Environment + service templates |

---

### Key Exam Scenarios

- **"Developers can launch compliant RDS without RDS IAM permissions"**: Service Catalog with Launch Constraint role.
- **"Ensure all deployed resources meet security standards"**: Service Catalog (only approved products deployable).
- **"Central IT team controls what devs can deploy"**: Service Catalog portfolio.

---

---

## 11. AWS Proton

### What Problem Does It Solve?
Platform teams define **standardized deployment templates** for microservices (infrastructure + CI/CD pipeline). Development teams self-service deploy their services using these templates — ensuring consistency without needing platform expertise.

---

### Key Concepts

| Concept | Description |
|---|---|
| **Environment Template** | Shared infrastructure (VPC, ECS cluster, RDS) |
| **Service Template** | Application infrastructure (ECS service, Lambda, CI/CD pipeline) |
| **Environment** | Deployed instance of an environment template |
| **Service** | Deployed instance of a service template |

---

### Key Exam Scenarios

- **"Platform team manages infrastructure templates; devs self-deploy microservices"**: AWS Proton.
- **"Standardize container deployments across 100 microservices"**: AWS Proton.

---

---

## 12. AWS Trusted Advisor

### What Problem Does It Solve?
Trusted Advisor continuously analyzes your AWS account and provides **real-time recommendations** across five categories — like having an AWS expert review your account.

---

### Trusted Advisor Check Categories

| Category | Examples |
|---|---|
| **Cost Optimization** | Idle EC2 instances, unassociated Elastic IPs, underutilized RDS, low utilization EC2 |
| **Performance** | High utilization EC2, CloudFront header forwarding, EBS throughput limits |
| **Security** | S3 bucket permissions, MFA on root, SGs with unrestricted ports (0.0.0.0/0), IAM use |
| **Fault Tolerance** | EBS snapshot age, RDS Multi-AZ, Route 53 health checks, no Auto Scaling |
| **Service Limits** | Approaching EC2, ELB, VPC, S3 service quotas |
| **Operational Excellence** | CloudTrail not enabled, exposed access keys |

---

### Trusted Advisor Tiers

| Tier | Checks Available | Plan Required |
|---|---|---|
| **Free** | 7 core checks (MFA on root, S3 public, SG open ports, IAM use, EBS snapshots, RDS public, service limits) | All support plans |
| **Full** | All 500+ checks | **Business, Enterprise On-Ramp, or Enterprise** support plan |

---

### Trusted Advisor vs AWS Compute Optimizer vs AWS Config

| Service | Focus | Recommendations |
|---|---|---|
| **Trusted Advisor** | Broad best-practice checks | Cost, security, performance, fault tolerance |
| **Compute Optimizer** | Right-sizing compute resources | EC2, EBS, Lambda, ECS, ASG — specific recommendations |
| **AWS Config** | Configuration compliance | Rule-based compliance evaluation |

---

### Key Exam Scenarios

- **"Identify idle EC2 instances to reduce costs"**: Trusted Advisor Cost Optimization.
- **"Check if root account has MFA enabled"**: Trusted Advisor Security (free tier check).
- **"Alert when approaching EC2 service limits"**: Trusted Advisor Service Limits + CloudWatch Events.
- **"Get right-sizing recommendations for EC2"**: AWS Compute Optimizer (not Trusted Advisor).

---

---

## 13. AWS Compute Optimizer

### What Problem Does It Solve?
Many AWS customers over-provision EC2 instances, resulting in wasted cost. Compute Optimizer uses **ML to analyze utilization metrics** and recommends the optimal resource configuration.

---

### Supported Resources

| Resource | What It Analyzes |
|---|---|
| **EC2 instances** | CPU, memory (with CloudWatch Agent), network, disk |
| **EC2 ASG** | Group-level optimization |
| **EBS volumes** | IOPS, throughput utilization |
| **Lambda functions** | Memory configuration vs actual usage |
| **ECS on Fargate** | vCPU and memory |

---

### Finding Classifications

| Classification | Meaning |
|---|---|
| **Under-provisioned** | Resource is a bottleneck; recommend scale up |
| **Over-provisioned** | Resource is wasted; recommend scale down (save cost) |
| **Optimized** | Current configuration is appropriate |
| **Insufficient data** | Not enough metrics to make recommendation |

---

### Key Exam Scenarios

- **"Reduce EC2 costs without performance impact"**: Compute Optimizer right-sizing recommendations.
- **"Lambda function memory is too high"**: Compute Optimizer Lambda memory recommendation.
- **"Right-size EBS volumes"**: Compute Optimizer EBS recommendations.

---

---

## 14. AWS Health Dashboard

### What Problem Does It Solve?
Two dashboards provide visibility into AWS service health:

| Dashboard | Scope | Description |
|---|---|---|
| **AWS Service Health Dashboard** | All AWS globally | Public; shows all AWS service issues worldwide |
| **AWS Personal Health Dashboard** (AWS Health) | Your account | Shows events impacting YOUR specific resources |

---

### AWS Health Events

| Type | Description |
|---|---|
| **Issue** | Active service event affecting your resources |
| **Scheduled Change** | Upcoming maintenance (EC2 retirement, RDS maintenance) |
| **Account Notification** | Security advisories, billing notifications |

---

### AWS Health API + EventBridge Integration

```
AWS Health Event (e.g., EC2 scheduled retirement)
          ↓
Amazon EventBridge rule
          ↓
Lambda function (migrate workload to new instance)
          + SNS (notify ops team)
```

---

### Key Exam Scenarios

- **"Get notified when AWS schedules EC2 instance retirement"**: AWS Health Dashboard + EventBridge.
- **"Know which of YOUR resources are affected by an AWS outage"**: Personal Health Dashboard.
- **"Automate response to scheduled maintenance"**: Health EventBridge rule → Lambda.

---

---

## 15. AWS License Manager

### What Problem Does It Solve?
Brings on-premises software licenses (SQL Server, Windows Server, Oracle, SAP) to AWS (BYOL) and tracks license consumption to prevent violations and reduce costs.

---

### Key Features

| Feature | Description |
|---|---|
| **License rules** | Define license limits based on vCPUs, sockets, cores |
| **License tracking** | Monitor consumption across EC2 and on-prem |
| **BYOL** | Bring Your Own License to Dedicated Hosts or Dedicated Instances |
| **Marketplace integration** | Track licenses from AWS Marketplace |
| **Org-level** | Manage licenses across all org accounts |

---

### Dedicated Hosts vs Dedicated Instances (for BYOL)

| Feature | Dedicated Host | Dedicated Instance |
|---|---|---|
| **Physical host** | ✅ You get entire physical server | ❌ Dedicated hardware, but may share with other instances of yours |
| **BYOL** | ✅ Best for per-socket/per-core licenses | Limited |
| **Host affinity** | ✅ Control instance placement | ❌ |
| **Cost** | Higher | Lower |
| **Use case** | Oracle, Windows Server BYOL | Compliance requiring dedicated hardware |

---

### Key Exam Scenarios

- **"Track SQL Server licenses across EC2"**: License Manager.
- **"Bring Windows Server licenses to AWS"**: License Manager + Dedicated Hosts.
- **"Prevent launching EC2 beyond licensed vCPU count"**: License Manager license rules with hard limits.

---

---

## 16. Amazon Managed Grafana

### What Problem Does It Solve?
Grafana is an open-source visualization platform. Running Grafana yourself requires managing servers, updates, plugins, and HA. Amazon Managed Grafana provides **fully managed Grafana workspaces** with built-in AWS data source integrations.

---

### Key Features

| Feature | Description |
|---|---|
| **Data sources** | CloudWatch, Prometheus, X-Ray, Timestream, OpenSearch, Athena, Redshift, IoT SiteWise |
| **Authentication** | SSO via IAM Identity Center, SAML |
| **Pricing** | Per active user per month |
| **HA** | Multi-AZ, managed by AWS |
| **Plugins** | 1000+ Grafana plugins available |

---

### Key Exam Scenarios

- **"Unified dashboard visualizing CloudWatch, Prometheus, and X-Ray data"**: Amazon Managed Grafana.
- **"Operational dashboards without managing Grafana infrastructure"**: Amazon Managed Grafana.

---

---

## 17. Amazon Managed Service for Prometheus

### What Problem Does It Solve?
Prometheus is the de-facto open-source monitoring system for Kubernetes/containers. Running Prometheus yourself at scale requires managing storage, HA, and retention. Amazon Managed Service for Prometheus provides a **fully managed, serverless Prometheus-compatible monitoring** service.

---

### Key Features

| Feature | Description |
|---|---|
| **Compatible** | Drop-in replacement; standard PromQL queries work unchanged |
| **Sources** | EKS, ECS, EC2 (via Prometheus agent/scraper) |
| **Storage** | Managed; scales automatically |
| **Retention** | Up to 150 days |
| **Query** | PromQL (Prometheus Query Language) |
| **Integration** | Amazon Managed Grafana for visualization |

---

### Prometheus + Grafana Pattern

```
EKS Cluster
    ↓ (Prometheus remote write)
Amazon Managed Service for Prometheus
    ↓ (PromQL queries)
Amazon Managed Grafana
    ↓
Operational Dashboards
```

---

### Key Exam Scenarios

- **"Monitor EKS cluster metrics without managing Prometheus"**: Amazon Managed Service for Prometheus.
- **"Use PromQL to query container metrics"**: Amazon Managed Prometheus.
- **"Visualize Prometheus metrics in Grafana dashboards"**: Managed Prometheus + Managed Grafana.

---

---

## 18. AWS Well-Architected Tool

### What Problem Does It Solve?
The Well-Architected Tool provides a **structured way to review workloads** against AWS best practices, identifying risks and improvement areas aligned to the **6 pillars of the Well-Architected Framework**.

---

### The 6 Pillars

| Pillar | Key Focus | Example Best Practice |
|---|---|---|
| **Operational Excellence** | Run and improve operations | IaC, runbooks, frequent small changes, monitoring |
| **Security** | Protect information and systems | Least privilege, encryption, MFA, GuardDuty |
| **Reliability** | Recover from failures | Multi-AZ, Auto Scaling, backup/restore, chaos testing |
| **Performance Efficiency** | Use resources efficiently | Right-sizing, serverless, CDN, caching |
| **Cost Optimization** | Avoid unnecessary costs | Reserved Instances, Spot, right-sizing, delete idle resources |
| **Sustainability** | Minimize environmental impact | Efficient instance types, serverless, reduce footprint |

---

### How It Works

1. Define a **workload** (name, description, environment).
2. Answer **milestone questions** per pillar (e.g., "How do you protect data in transit?").
3. Tool generates a **report** with: high risk issues (HRIs), medium risk issues (MRIs), improvement plan.
4. Track improvement over time with **milestones**.

---

### Well-Architected Lenses

- Extend the framework for specific industry/technology use cases.
- Available lenses: **Serverless, SaaS, Machine Learning, Data Analytics, IoT, Games, Financial Services, Healthcare**.

---

### Key Exam Scenarios

- **"Review workload against AWS best practices"**: Well-Architected Tool.
- **"Identify security risks in existing architecture"**: Well-Architected Tool — Security pillar review.
- **"Serverless-specific architecture review"**: Well-Architected Tool with Serverless Lens.

---

---

## 19. AWS CLI, Management Console, and Access Methods

### Access Methods Summary

| Method | Description | Best For |
|---|---|---|
| **AWS Management Console** | Web-based GUI | Learning, one-off tasks, visual exploration |
| **AWS CLI** | Command-line tool | Scripting, automation, CI/CD |
| **AWS SDKs** | Language-specific libraries | Application integration (Python, Java, Node.js) |
| **CloudFormation/CDK** | IaC | Repeatable infrastructure |
| **AWS API** | Direct HTTP API calls | Custom integrations |

---

### AWS CLI Key Facts

- Configured with `aws configure` (access key, secret key, region, output format).
- Uses **named profiles** for multiple accounts: `aws --profile prod s3 ls`.
- **MFA with CLI**: Use `aws sts get-session-token --serial-number <MFA ARN> --token-code <code>`.
- **Output formats**: json (default), yaml, text, table.
- **Pagination**: `--page-size`, `--max-items`, `--starting-token` for large result sets.

---

---

## Master Comparison: Governance Services

### Multi-Account Governance Stack

```
AWS Organizations (foundation — account structure + SCPs)
        ↓
AWS Control Tower (landing zone — automates setup + guardrails)
        ↓ (deploys into every account)
CloudTrail (API audit)  +  Config (config compliance)
        ↓
Security Hub (aggregate findings)
        ↓
Systems Manager (operational management)
```

### Configuration & Compliance Hierarchy

| Tool | What It Enforces | When |
|---|---|---|
| **SCP (Organizations)** | Account-level permission ceiling | Before any action |
| **IAM Policy** | User/role permissions | Before any action |
| **Config Rule** | Resource configuration compliance | After resource created/modified |
| **CloudFormation** | Infrastructure as declared | At provisioning time |
| **Service Catalog** | Only approved products deployable | At provisioning time |

---

## Architecture Patterns to Know

### Pattern 1: Automated Compliance Remediation
```
New EC2 launched without encryption tag
        ↓
AWS Config Rule (required-tags) → NON_COMPLIANT
        ↓
Config Remediation → SSM Automation document runs
        ↓
Adds tag + SNS notification to ops team
        ↓
Config rechecks → COMPLIANT
```

### Pattern 2: Centralized Multi-Account Operations
```
Management Account
├── Control Tower (landing zone + guardrails)
├── CloudFormation StackSets (baseline to all accounts)
├── Organizations (SCP enforcement)
└── Delegated Admin Account:
    ├── Security Hub (aggregate findings)
    ├── GuardDuty (threat detection org-wide)
    ├── Config (conformance packs)
    └── CloudTrail (organization trail → Log Archive account)
```

### Pattern 3: Complete Observability Stack
```
Applications + AWS Services
        ↓
CloudWatch (metrics + logs + alarms)
        + CloudTrail (API audit)
        + X-Ray (distributed tracing)
        + Config (configuration history)
        ↓
Security Hub (aggregation)
        ↓
Managed Grafana (dashboards)
        ↓
Alarms → EventBridge → Lambda → SNS (alerting)
                              → Systems Manager (remediation)
```

### Pattern 4: Self-Service Developer Platform
```
Platform Team:
  ├── Service Catalog (approved products portfolio)
  ├── AWS Proton (microservice deployment templates)
  └── CloudFormation (underlying IaC)

Developer Self-Service:
  ├── Launch approved DB from Service Catalog (no RDS perms needed)
  ├── Deploy microservice via Proton template
  └── All governed by Control Tower guardrails
```

---

## Exam Tips Summary

### Auto Scaling
- ✅ Target Tracking = simplest, most common; Step = granular thresholds; Scheduled = predictable; Predictive = ML-based proactive
- ✅ Launch Template replaces Launch Configuration (supports Spot + On-Demand mix)
- ✅ Lifecycle Hooks = run custom actions on scale-out (Pending:Wait) or scale-in (Terminating:Wait)
- ✅ Cooldown default = 300 seconds for simple scaling

### CloudFormation
- ✅ StackSets = deploy across multiple accounts + regions
- ✅ Change Set = preview changes before applying (use in production!)
- ✅ Drift Detection = find manual changes made outside CloudFormation
- ✅ Nested Stacks = modular reusable templates
- ✅ cfn-signal + CreationPolicy = wait for EC2 to finish bootstrapping before stack completes

### CloudTrail
- ✅ Management Events = free, default; Data Events = extra cost (S3 GetObject, Lambda Invoke)
- ✅ Organization Trail = covers all accounts; created in management account
- ✅ Log integrity = digest files (detect tampering)
- ✅ Insights = detects anomalous API call rates
- ✅ CloudTrail ≠ Config: CloudTrail = "who did what"; Config = "what changed"

### CloudWatch
- ✅ Memory/disk NOT in default EC2 metrics → need CloudWatch Agent
- ✅ Alarms: OK → ALARM → INSUFFICIENT_DATA
- ✅ Composite Alarms = AND/OR logic across multiple alarms (reduce noise)
- ✅ Logs Insights = SQL-like query for logs
- ✅ Subscription Filters → stream logs to Lambda/Kinesis/OpenSearch in real time
- ✅ High-resolution custom metrics = 1-second granularity

### Organizations + Control Tower
- ✅ SCPs = ceiling, not grant; even root is restricted
- ✅ Control Tower = automated landing zone; Account Factory = vend new accounts
- ✅ Preventive guardrails = SCPs; Detective guardrails = Config rules
- ✅ Log Archive + Audit accounts = always created by Control Tower
- ✅ Consolidated billing = share RI/Savings Plans across org

### AWS Config
- ✅ Records resource configuration history (not API calls)
- ✅ Conformance Packs = bundle of rules for PCI/HIPAA/CIS compliance
- ✅ Automatic remediation via SSM Automation documents
- ✅ Per-region service (must enable in each region)

### Systems Manager
- ✅ Session Manager = no bastion, no port 22, fully audited
- ✅ Parameter Store = free (standard), hierarchical, SecureString encrypted with KMS
- ✅ Parameter Store vs Secrets Manager: Secrets Manager has auto-rotation (costs more)
- ✅ Patch Manager + Maintenance Window = automated patching
- ✅ State Manager = enforce desired state (e.g., always run CW agent)
- ✅ No open inbound ports needed — agent polls outbound

### Trusted Advisor
- ✅ 7 free checks for all; 500+ checks need Business/Enterprise support
- ✅ Covers: Cost, Performance, Security, Fault Tolerance, Service Limits, Operational Excellence
- ✅ Trusted Advisor ≠ Compute Optimizer: TA = broad checks; CO = specific right-sizing ML recommendations

### Well-Architected Tool
- ✅ 6 pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability
- ✅ Lenses extend the framework for specific tech (Serverless, ML, SaaS, etc.)
- ✅ Generates HRI (High Risk Issues) and improvement plans

---
---

<a name="migration"></a>
# PART 11 — Migration & Transfer: Application Discovery, MGN, DMS, DataSync, Migration Hub, Snow Family & Transfer Family

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

---
---

<a name="frontend"></a>
# PART 12 — Front-End Web & Mobile: Amplify, API Gateway, Device Farm & Pinpoint

# AWS Exam Study Guide: Front-End Web & Mobile Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** AWS Amplify · Amazon API Gateway · AWS Device Farm · Amazon Pinpoint

---

## Quick-Reference Matrix

| Service | Category | What It Does | Exam Trigger Words |
|---|---|---|---|
| **AWS Amplify** | Full-stack platform | Build, deploy, host web/mobile apps with backend | "host web app", "CI/CD for frontend", "GraphQL backend", "full-stack serverless" |
| **API Gateway** | API Management | Create, publish, manage REST/HTTP/WebSocket APIs | "REST API", "WebSocket", "API throttling", "Lambda integration", "API versioning" |
| **Device Farm** | Testing | Test mobile/web apps on real devices in AWS cloud | "mobile testing", "real devices", "browser testing", "automated test" |
| **Pinpoint** | Engagement | Multichannel customer communication (push, email, SMS) | "push notification", "SMS campaign", "user segmentation", "targeted messaging" |

---

---

## 1. AWS Amplify

### What Problem Does It Solve?

Building a full-stack web or mobile application requires assembling and configuring many AWS services — Cognito for auth, AppSync for APIs, S3 for storage, Lambda for backend logic, CloudFront for hosting. Doing this manually takes weeks. AWS Amplify provides a **unified platform** that abstracts all of this into a streamlined developer experience for building, deploying, and hosting full-stack apps.

**Real-World Example:**
> A startup wants to build a React e-commerce app with user authentication, a GraphQL product API, image upload to S3, and global CDN hosting. Without Amplify, they'd manually configure Cognito, AppSync, S3, Lambda, CloudFront, and set up CI/CD pipelines. With Amplify, the entire backend is provisioned via CLI commands, the frontend is hosted with automatic deployments from GitHub, and the app is live in hours instead of weeks.

---

### Amplify Has Two Distinct Components

#### Component 1: Amplify Hosting
A fully managed **CI/CD and hosting service** for web applications (similar to Netlify or Vercel, but on AWS).

| Feature | Details |
|---|---|
| **Source control** | Connect to GitHub, GitLab, Bitbucket, AWS CodeCommit |
| **CI/CD** | Auto-builds and deploys on every git push |
| **Framework support** | React, Vue, Angular, Next.js, Nuxt.js, Gatsby, Hugo, Jekyll, plain HTML |
| **Global CDN** | CloudFront-powered; content cached at edge locations worldwide |
| **Custom domains** | Free SSL via ACM; custom domain with Route 53 or external DNS |
| **Branch deployments** | Each git branch gets its own URL (e.g., `feature-login.xxxxx.amplifyapp.com`) |
| **Preview URLs** | Pull request previews — every PR gets a unique URL for review |
| **Password protection** | Protect staging branches with a username/password |
| **SSR support** | Server-side rendering for Next.js (runs on Lambda) |
| **Manual deploys** | Deploy static files directly (drag-and-drop or CLI) |

#### Component 2: Amplify Backend (Amplify Studio + CLI)
A **backend-as-a-service** provisioning platform that creates and manages AWS resources using CloudFormation under the hood.

| Amplify Category | Underlying AWS Service |
|---|---|
| **Authentication** | Amazon Cognito (User Pools + Identity Pools) |
| **API (GraphQL)** | AWS AppSync |
| **API (REST)** | Amazon API Gateway + Lambda |
| **Storage** | Amazon S3 |
| **Functions** | AWS Lambda |
| **DataStore** | AppSync + DynamoDB (offline sync) |
| **Hosting** | CloudFront + S3 (for static) or Lambda (for SSR) |
| **Notifications** | Amazon Pinpoint |
| **Analytics** | Amazon Pinpoint |
| **Predictions** | Amazon Rekognition, Transcribe, Translate |
| **Geo** | Amazon Location Service |

---

### Amplify Deployment Flow

```
Developer writes code
        ↓
git push to GitHub/GitLab/Bitbucket
        ↓
Amplify Hosting detects push
        ↓
Build phase (npm install, npm run build)
        ↓
Deploy phase (static files → S3 + CloudFront invalidation)
    or  (SSR → Lambda@Edge / Lambda)
        ↓
New version live at custom domain
        ↓
(Optional) Run E2E Cypress tests before promoting to production
```

---

### Amplify DataStore

One of Amplify's most powerful features — provides **offline-first data sync** for mobile and web apps:

- Local data stored on device using IndexedDB (web) or SQLite (mobile)
- Automatically syncs with AppSync + DynamoDB when connectivity is restored
- Conflict resolution strategies: Auto Merge, Optimistic Concurrency, Custom Lambda

> **Exam Tip:** "Offline-first mobile app with automatic sync when reconnected" → Amplify DataStore (built on AppSync + DynamoDB).

---

### Amplify vs Manual AWS Setup vs Firebase

| Feature | Amplify | Manual AWS | Firebase (Google) |
|---|---|---|---|
| **Setup speed** | Minutes (CLI) | Hours to days | Minutes |
| **AWS integration** | ✅ Native | ✅ Full | ❌ |
| **Customization** | High (via CloudFormation escape hatch) | Complete | Limited |
| **Offline sync** | ✅ DataStore | Manual | ✅ Firestore |
| **Vendor lock-in** | AWS | AWS | Google |
| **SSR hosting** | ✅ Next.js | Via custom setup | ❌ |
| **CI/CD built-in** | ✅ | Manual (CodePipeline) | ❌ |

---

### Amplify vs AWS Elastic Beanstalk vs S3 Static Hosting

| Scenario | Best Choice |
|---|---|
| Static HTML/CSS/JS site | S3 + CloudFront (or Amplify Hosting) |
| React/Vue/Angular SPA with CI/CD from GitHub | Amplify Hosting |
| Full-stack app with backend (auth, API, DB) | Amplify (full platform) |
| Containerized backend application | Elastic Beanstalk or ECS |
| Enterprise app needing full AWS control | Manual AWS configuration |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Host a React SPA with automatic deploys from GitHub | AWS Amplify Hosting |
| Full-stack mobile app with Cognito auth + GraphQL API + S3 storage | AWS Amplify (CLI/Studio provisions all backend) |
| Offline-first mobile app with automatic cloud sync | Amplify DataStore (AppSync + DynamoDB) |
| Preview each pull request at a unique URL before merging | Amplify Hosting branch previews |
| Next.js SSR app hosted on AWS with no server management | Amplify Hosting (SSR support with Lambda) |
| Migrate from Firebase to AWS with similar developer experience | AWS Amplify |

---

---

## 2. Amazon API Gateway

### What Problem Does It Solve?

APIs are the backbone of modern applications — they connect frontend clients to backend services, expose microservices, and enable third-party integrations. Building and operating an API at scale requires handling authentication, throttling, caching, versioning, SSL, CORS, monitoring, and billing. API Gateway provides a **fully managed service** that handles all of this without managing any servers.

**Real-World Example:**
> A mobile banking app needs to expose account balance, transaction history, and transfer endpoints to millions of users globally. API Gateway sits in front of Lambda functions — handling authentication (Cognito), rate limiting (100 req/sec per user), caching responses (5-minute TTL for account balance), SSL termination (ACM cert), and detailed CloudWatch monitoring — all without a single server to manage.

---

### API Gateway Types

| Type | Protocol | Use Case | Key Feature |
|---|---|---|---|
| **REST API** | HTTP/HTTPS | Full-featured REST APIs | Request/response transformation, caching, custom authorizers, usage plans |
| **HTTP API** | HTTP/HTTPS | Simple, low-latency APIs (70% cheaper than REST) | Faster, cheaper; limited features; native OIDC/JWT auth |
| **WebSocket API** | WebSocket | Real-time two-way communication | Persistent connections; route selections; chat, gaming, live feeds |

---

### REST API vs HTTP API (Most Tested Comparison)

| Feature | REST API | HTTP API |
|---|---|---|
| **Cost** | Higher (~$3.50/million) | Lower (~$1.00/million) — **~70% cheaper** |
| **Latency** | Higher | Lower |
| **Request/response transformation** | ✅ (mapping templates) | ❌ |
| **Caching** | ✅ (built-in, configurable TTL) | ❌ |
| **Usage plans + API keys** | ✅ | ❌ |
| **Custom domain** | ✅ | ✅ |
| **JWT/OIDC authorizer** | ❌ (use Lambda authorizer) | ✅ Native |
| **Lambda authorizer** | ✅ | ✅ |
| **Cognito authorizer** | ✅ | ✅ |
| **AWS WAF integration** | ✅ | ❌ |
| **Edge-optimized endpoint** | ✅ | ❌ |
| **VPC Link** | ✅ | ✅ |
| **Private API** | ✅ | ❌ |
| **Resource policies** | ✅ | ❌ |

> **Exam Rule:** "Need caching, WAF, usage plans, request/response transformation" → **REST API**. "Need low-cost, simple Lambda proxy, JWT auth" → **HTTP API**.

---

### WebSocket API

- Maintains **persistent bidirectional connections** between clients and backend.
- Route messages based on a **route selection expression** (e.g., `$request.body.action`).
- **Three built-in routes**: `$connect`, `$disconnect`, `$default`.
- Backend receives and sends messages via: Lambda, HTTP endpoint, AWS service integration.
- Use the **`@connections` API** to push messages to connected clients from the backend.

**Real-World Example:**
> A live collaborative document editor. When User A types, the WebSocket API routes the message to Lambda, which processes the change and uses the `@connections` API to push the update to all other connected users instantly.

---

### API Gateway Integration Types

| Integration Type | Description | Use Case |
|---|---|---|
| **Lambda Proxy** | API GW passes entire request to Lambda as-is; Lambda returns full response | Most common; no transformation needed |
| **Lambda Custom** | API GW transforms request/response using mapping templates (VTL) | Need to reshape request before Lambda |
| **HTTP Proxy** | Pass-through to HTTP backend | Wrap existing HTTP service |
| **HTTP Custom** | Transform + route to HTTP backend | Legacy backend with custom contract |
| **AWS Service** | Direct integration with AWS services (S3, SQS, DynamoDB, etc.) | Call AWS services without Lambda |
| **Mock** | Return static response without backend | API mocking, CORS preflight |

> **Exam Tip:** "PUT directly to SQS from API Gateway without Lambda" → **AWS Service Integration** (SQS). This avoids the cost and latency of an intermediate Lambda.

---

### API Gateway Endpoint Types

| Type | Description | Use Case |
|---|---|---|
| **Edge-Optimized** | Deployed to CloudFront; globally distributed | API clients distributed globally |
| **Regional** | Deployed in a single region; no CloudFront | Clients in same region; use your own CloudFront if needed |
| **Private** | Accessible only via VPC Interface Endpoint | Internal microservices; never exposed to internet |

---

### Authentication & Authorization

| Method | How It Works | Use Case |
|---|---|---|
| **API Keys** | Client sends `x-api-key` header | Simple auth; not for security; use for usage plans/throttling |
| **IAM Authorization** | Client signs request with SigV4 | AWS services / server-to-server calls |
| **Cognito User Pool Authorizer** | Validates JWT from Cognito User Pool | End-user mobile/web apps |
| **Lambda Authorizer (Token)** | Lambda validates bearer token (JWT, OAuth) | Custom auth logic, third-party IdP |
| **Lambda Authorizer (Request)** | Lambda validates headers, query params, etc. | Complex auth based on multiple inputs |
| **JWT Authorizer (HTTP API)** | Native JWT validation (OIDC/OAuth) | HTTP API only; simple JWT auth without Lambda |

---

### API Gateway Caching

- Cache API responses at the **stage level**.
- Cache capacity: 0.5 GB to 237 GB.
- Default TTL: **300 seconds** (min 0, max 3600).
- Clients can invalidate cache with `Cache-Control: max-age=0` header (requires IAM permission).
- Per-method override: enable/disable or set different TTL per method.

> **Exam Tip:** "Reduce backend calls and improve API performance" → Enable **API Gateway caching**. "Cache should vary per user" → Enable per-user cache key based on Cognito sub claim.

---

### Throttling & Usage Plans

| Concept | Description |
|---|---|
| **Account limit** | 10,000 requests/sec per region (default; adjustable) |
| **Stage-level throttling** | Set default rate + burst limit per stage |
| **Method-level throttling** | Override per specific method (e.g., `/payments` = 100 req/sec) |
| **Usage Plan** | Define rate limit + quota (daily/weekly/monthly) per API key |
| **API Key** | Clients include key in header; associated with a Usage Plan |

**Throttling Behavior:**
- **Rate limit**: Steady-state max requests per second (token bucket algorithm)
- **Burst limit**: Max concurrent requests at a spike (up to 5,000 by default)
- Throttled requests → HTTP **429 Too Many Requests**

---

### API Gateway Stages and Deployments

| Concept | Description |
|---|---|
| **Deployment** | Snapshot of API configuration at a point in time |
| **Stage** | Named reference to a deployment (e.g., `dev`, `staging`, `prod`) |
| **Stage Variables** | Key-value pairs injected at stage level (e.g., point `dev` stage to dev Lambda alias) |
| **Canary Release** | Send % of traffic to new deployment while keeping rest on current |

**Stage Variables Example:**
```
/prod stage → stageName = "prod" → Lambda alias = prod
/dev stage  → stageName = "dev"  → Lambda alias = dev

Lambda ARN in integration: arn:aws:lambda:...:function:MyFunc:${stageVariables.stageName}
```

---

### API Gateway Resource Policy

- **JSON policy** attached to the API controlling which principals (IAM users, accounts, IPs, VPCs) can invoke the API.
- Used for:
  - **Cross-account access**: Allow account 123456789012 to call your API
  - **IP whitelist/blacklist**: Allow only specific CIDR ranges
  - **VPC-only access**: Restrict to a specific VPC endpoint

---

### API Gateway Logging & Monitoring

| Feature | Description |
|---|---|
| **CloudWatch Metrics** | Count, Latency, IntegrationLatency, 4XXError, 5XXError, CacheHitCount |
| **Access Logs** | Log each API call (who called, when, latency, response code) → CloudWatch Logs |
| **Execution Logs** | Detailed request/response logging per stage (helpful for debugging) |
| **X-Ray tracing** | Distributed tracing from API Gateway through Lambda to downstream services |

---

### CORS (Cross-Origin Resource Sharing)

- Browser blocks requests from `app.example.com` to `api.example.com` unless API enables CORS.
- API Gateway handles CORS by:
  - Responding to **OPTIONS preflight** requests
  - Adding `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods` headers
- For REST API: manually enable per resource or use **Mock integration** for OPTIONS.
- For HTTP API: built-in CORS configuration.

---

### API Gateway + VPC Link

- **VPC Link** enables API Gateway to privately connect to **resources inside a VPC** (NLB, ALB, ECS services).
- Traffic stays on AWS network; never traverses the internet.
- Used for: exposing private microservices via a public API without exposing the VPC.

```
Internet → API Gateway (REST/HTTP) → VPC Link → NLB → Private ECS Service
```

---

### API Gateway Exam Patterns

#### Pattern 1: Serverless REST API
```
Client → API Gateway (REST) → Lambda → DynamoDB
         (Cognito auth, throttling, caching)
```

#### Pattern 2: Direct AWS Service Integration (No Lambda)
```
Client → API Gateway → SQS Queue (enqueue message directly)
                     → DynamoDB PutItem (directly)
                     → SNS Publish (directly)
```

#### Pattern 3: Private Internal API
```
Internal Service → VPC Interface Endpoint → Private API Gateway → Lambda
(completely private; never leaves VPC)
```

#### Pattern 4: WebSocket Real-Time App
```
Browser Client → WebSocket API → Lambda ($connect / $disconnect / message routes)
                                      ↓
                             Lambda uses @connections API
                             to push to other connected clients
```

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Simple Lambda proxy API, minimize cost | HTTP API |
| Need WAF, caching, usage plans, request transformation | REST API |
| Real-time bidirectional chat app | WebSocket API |
| API accessible only from within VPC | Private endpoint type |
| Globally distributed API clients, low latency | Edge-optimized endpoint |
| Throttle one user group to 100 req/sec, another to 500 req/sec | Usage Plans with API Keys |
| Prevent 429 throttling from DynamoDB — buffer requests | API Gateway → SQS → Lambda → DynamoDB |
| POST data directly to SQS without Lambda | API Gateway AWS Service Integration (SQS) |
| Route `/v1` traffic to old Lambda, `/v2` to new Lambda | Stage Variables pointing to different Lambda aliases |
| Gradually roll out new API version | Canary release deployment |
| API Gateway + CloudFront for edge caching | Use Regional endpoint with your own CloudFront distribution |

---

### API Gateway vs ALB vs AppSync

| Feature | API Gateway | ALB | AppSync |
|---|---|---|---|
| **Protocol** | REST/HTTP/WebSocket | HTTP/HTTPS | GraphQL |
| **Auth built-in** | ✅ Cognito, IAM, Lambda | Limited | ✅ Cognito, IAM, Lambda, OIDC |
| **Throttling** | ✅ | ❌ (use WAF) | Limited |
| **Caching** | ✅ (REST API) | ❌ | ✅ |
| **WebSocket** | ✅ | ❌ | ✅ (subscriptions) |
| **Best for** | REST/HTTP microservices, Lambda APIs | Container/EC2 load balancing | GraphQL, real-time mobile apps |

---

---

## 3. AWS Device Farm

### What Problem Does It Solve?

Mobile apps must work correctly across **hundreds of different device models, OS versions, screen sizes, and network conditions**. Maintaining a physical device lab is expensive (~$50K+), time-consuming, and impossible to scale. Device Farm provides access to **real physical devices and browsers** in AWS data centers for automated and manual testing — without owning any hardware.

**Real-World Example:**
> A fintech company releases an iOS banking app. Before launch, Device Farm runs their Appium test suite simultaneously on 50 real iPhone and Android models — detecting a crash on Samsung Galaxy S21 running Android 12 before any users encounter it. Total test time: 20 minutes instead of days of manual testing.

---

### Device Farm Test Types

| Test Type | Description |
|---|---|
| **Automated testing** | Run test scripts/frameworks on multiple devices in parallel |
| **Remote access (manual)** | Interact with a real device in real time via browser (manual QA, exploratory testing) |

---

### Supported Testing Frameworks

#### Mobile Apps (iOS & Android)
| Framework | Language | Works With |
|---|---|---|
| **Appium** | Python, Java, Node.js, Ruby | iOS + Android |
| **XCTest** | Swift / Objective-C | iOS only |
| **Espresso** | Java / Kotlin | Android only |
| **Calabash** | Ruby | iOS + Android |
| **Built-in (Fuzz)** | N/A | Android; sends random inputs |
| **Instrumentation** | Java | Android |

#### Web Applications (Desktop Browser Testing)
| Feature | Details |
|---|---|
| **Browsers** | Chrome, Firefox |
| **Framework** | Selenium WebDriver |
| **Desktop Testing** | Test web apps on desktop browsers in real browser environments |

---

### How Device Farm Works

```
Developer uploads:
  ├── App package (.apk for Android, .ipa for iOS, or web URL)
  └── Test package (Appium script, XCTest bundle, Espresso APK, etc.)
            ↓
Select test configuration:
  ├── Device pool (curated sets of devices or custom selection)
  ├── Test framework
  └── Test data / files
            ↓
Device Farm runs tests in parallel on all selected devices simultaneously
            ↓
Results report:
  ├── Pass/fail per device
  ├── Screenshots at each test step
  ├── Video recordings of test execution
  ├── Device logs
  ├── Performance data (CPU, memory, network, battery)
  └── Crash reports
```

---

### Device Farm Pricing Models

| Model | How It Works | Best For |
|---|---|---|
| **Pay-as-you-go (metered)** | Pay per device-minute used | Infrequent testing |
| **Unmetered plan** | Fixed monthly fee; unlimited device minutes | High-volume CI/CD pipelines |

---

### Device Farm vs Real Device Labs vs Simulators/Emulators

| Feature | Device Farm | Physical Device Lab | Simulators/Emulators |
|---|---|---|---|
| **Real hardware** | ✅ Real devices | ✅ Real devices | ❌ Virtual only |
| **Maintenance** | AWS manages | You manage | N/A |
| **Scale** | Hundreds of devices | Limited by budget | Unlimited |
| **Cost** | Pay per minute | High capex | Free |
| **Accuracy** | High | Highest | Lower (no real sensors) |
| **Parallel testing** | ✅ Yes | Limited | ✅ Yes |
| **Network simulation** | ✅ (3G, 4G, WiFi) | Manual | Limited |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Test mobile app on 100 real device models before launch | AWS Device Farm |
| Run Appium tests automatically in CI/CD pipeline on real devices | Device Farm automated testing |
| QA engineer needs to manually interact with a real iOS device remotely | Device Farm remote access session |
| Test web app on real Chrome and Firefox browsers at scale | Device Farm desktop browser testing (Selenium) |
| Detect device-specific crashes before app store release | Device Farm parallel automated testing |

---

---

## 4. Amazon Pinpoint

### What Problem Does It Solve?

Modern apps need to communicate with users across multiple channels — push notifications for re-engagement, email for receipts, SMS for OTP/alerts, and in-app messages for onboarding. Building and scaling multichannel communication infrastructure is complex. Amazon Pinpoint provides a **managed multichannel customer engagement platform** with analytics, segmentation, and campaign orchestration.

**Real-World Example:**
> A ride-sharing app uses Pinpoint to: send an SMS one-time password during signup, send a push notification when a driver is 2 minutes away, email a receipt after each ride, and run a re-engagement campaign targeting users who haven't booked in 30 days with a 20% discount push notification — all from one service.

---

### Pinpoint Channels

| Channel | Description | Use Case |
|---|---|---|
| **Push Notifications** | iOS (APNs), Android (FCM), Baidu, Amazon Device Messaging | App re-engagement, transactional alerts |
| **Email** | SES-powered; bulk + transactional | Receipts, newsletters, marketing |
| **SMS** | Short Message Service (text messages) | OTP, alerts, transactional |
| **Voice** | Text-to-speech voice calls | OTP delivery, automated notifications |
| **In-App Messaging** | Messages displayed within the app | Onboarding, feature announcements |
| **Custom channels** | Lambda function as channel | Any custom delivery (WhatsApp, Slack, webhook) |

---

### Core Pinpoint Concepts

#### Endpoints
- An **endpoint** represents a specific destination for a message.
- Each endpoint has: **channel type** (SMS, email, push), **address** (phone number, email, device token), and **attributes** (user properties).
- One user can have multiple endpoints (email + phone + iOS device + Android device).

#### Segments
- Groups of endpoints with shared attributes.
- **Dynamic segments**: automatically updated based on endpoint attribute filters.
- **Imported segments**: upload a CSV/JSON file of endpoint addresses.

**Example Dynamic Segment:**
```
Users WHERE:
  Platform = iOS
  AND Country = "US"
  AND LastPurchase > 30 days ago
  AND AppVersion >= "3.0"
```

#### Campaigns
- Targeted messages sent to a **segment** at a scheduled time or triggered by an event.
- Can include **A/B testing** (split segment into treatment groups with different message variants).
- **Campaign limits**: control max messages per endpoint per 24 hours/campaign.

#### Journeys
- **Multi-step, event-driven** communication workflows.
- Users move through journey stages based on actions/events (like a drip campaign).

**Example Journey:**
```
User signs up → Wait 1 day → Send welcome email
                    → Did user complete profile? 
                        YES → Send "explore features" push
                        NO → Send "complete your profile" email → Wait 3 days → Send reminder SMS
```

#### Templates
- Reusable message templates with **dynamic attributes** (personalization).
- Support for: Email HTML templates, SMS templates, push notification templates.
- Use `{{User.UserAttributes.FirstName}}` for personalization.

---

### Pinpoint Analytics

Pinpoint has **built-in analytics** for tracking:

| Metric | Description |
|---|---|
| **Daily Active Users (DAU)** | Users who opened app today |
| **Monthly Active Users (MAU)** | Users who opened app this month |
| **Session metrics** | Session count, length, interval |
| **Funnel analytics** | Track conversion through multi-step flows |
| **Campaign metrics** | Sends, deliveries, opens, clicks, opt-outs |
| **Revenue** | Track in-app purchase revenue |
| **Custom events** | Track any app event (add_to_cart, checkout, etc.) |

---

### Pinpoint vs Amazon SNS

> **One of the most important comparisons for the exam.**

| Feature | Amazon Pinpoint | Amazon SNS |
|---|---|---|
| **Primary use** | Marketing + transactional user engagement | System-to-system pub/sub notifications |
| **Targeting** | User segmentation, behavioral targeting | Topic-based (broadcast) |
| **Channels** | Push, Email, SMS, Voice, In-App, Custom | Push, SMS, Email, SQS, Lambda, HTTP |
| **Analytics** | ✅ Built-in user analytics, campaign metrics | ❌ |
| **Campaigns** | ✅ Scheduled + triggered campaigns | ❌ |
| **Journeys** | ✅ Multi-step workflows | ❌ |
| **A/B testing** | ✅ | ❌ |
| **Audience** | End-users (consumers) | Applications/systems |
| **Personalization** | ✅ Per-user attributes | ❌ |
| **Use case** | "Re-engage users who haven't opened app in 7 days" | "Fan out S3 events to multiple Lambda functions" |

> **Exam Rule:**
> - "Targeted push notification to specific users based on behavior" → **Pinpoint**
> - "Fan out a system event to multiple services" → **SNS**
> - "Send bulk marketing email to 1 million users" → **Pinpoint** (with SES under the hood)
> - "Send transactional email from an application" → **SES** directly

---

### Pinpoint vs Amazon SES

| Feature | Pinpoint | Amazon SES |
|---|---|---|
| **Type** | Multichannel engagement platform | Email-only service |
| **Marketing campaigns** | ✅ | ❌ |
| **Analytics** | ✅ Built-in | Limited (via CloudWatch) |
| **Segmentation** | ✅ | ❌ |
| **Transactional email** | ✅ | ✅ (primary use) |
| **A/B testing** | ✅ | ❌ |
| **Under the hood** | Uses SES for email delivery | Is the email service |
| **Best for** | Marketing + engagement campaigns | High-volume transactional email from apps |

---

### Pinpoint Transactional vs Campaign Messaging

| Type | Description | Example |
|---|---|---|
| **Transactional** | Immediate, triggered by user action | OTP SMS, order confirmation email |
| **Campaign** | Scheduled batch to a segment | Weekly newsletter, re-engagement push |

---

### Pinpoint Integration with AWS Services

| Service | Integration |
|---|---|
| **AWS Amplify** | Amplify Notifications category uses Pinpoint for push notifications and analytics |
| **Amazon Kinesis** | Stream Pinpoint event data to Kinesis for custom analytics |
| **Amazon S3** | Import endpoint segments from S3 CSV/JSON files |
| **AWS Lambda** | Custom channel delivery via Lambda; event-based journey triggers |
| **Amazon SES** | Email delivery engine behind Pinpoint email channel |
| **Amazon EventBridge** | Emit Pinpoint events to EventBridge for further automation |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Send targeted push notifications to users who haven't opened app in 7 days | Amazon Pinpoint (dynamic segment + campaign) |
| Send SMS OTP during user registration | Amazon Pinpoint (transactional SMS) |
| A/B test two email subject lines with 10% of users before full campaign send | Amazon Pinpoint A/B campaign |
| Multi-step onboarding email sequence triggered by user events | Amazon Pinpoint Journeys |
| Track daily/monthly active users in a mobile app | Amazon Pinpoint analytics (DAU/MAU) |
| Fan out application event to SQS, Lambda, and HTTP endpoint | Amazon SNS (not Pinpoint) |
| Send marketing email to 5 million subscribers with analytics | Amazon Pinpoint email campaign |
| Send transactional receipt email from backend application code | Amazon SES (not Pinpoint) |
| Mobile app needs analytics + push notifications from one service | Amazon Pinpoint (via Amplify or direct SDK) |

---

---

## Master Comparison: All Four Services

### By Role in a Mobile App Architecture

```
User interacts with mobile/web app
            │
┌───────────┴──────────────────────────────────────────────┐
│                  AWS Amplify                             │
│  (auth → Cognito, API → AppSync/APIGW, storage → S3,    │
│   hosting → CloudFront, analytics/notifications → Pinpoint)│
└───────────────────┬──────────────────────────────────────┘
                    │
           API calls to backend
                    │
┌───────────────────▼──────────────────────────────────────┐
│              Amazon API Gateway                          │
│  (REST/HTTP/WebSocket API → Lambda → DynamoDB/RDS/SQS)   │
└───────────────────┬──────────────────────────────────────┘
                    │
           Before release:
                    │
┌───────────────────▼──────────────────────────────────────┐
│              AWS Device Farm                             │
│  (Test app on 100+ real devices, Appium/XCTest, parallel)│
└───────────────────┬──────────────────────────────────────┘
                    │
           After launch — user engagement:
                    │
┌───────────────────▼──────────────────────────────────────┐
│             Amazon Pinpoint                              │
│  (push notifications, email, SMS, journeys, analytics)   │
└──────────────────────────────────────────────────────────┘
```

---

### Decision Framework for Exam Questions

```
"Build and deploy a frontend web/mobile app with backend"
    → AWS Amplify

"Create a managed API endpoint for Lambda/microservices"
    → API Gateway (REST for full features; HTTP for low-cost; WebSocket for real-time)

"Test mobile app on real physical devices automatically"
    → AWS Device Farm

"Send targeted push/email/SMS to specific user segments"
    → Amazon Pinpoint

"Fan out system events to multiple AWS services"
    → Amazon SNS (not Pinpoint)

"Send transactional email from code"
    → Amazon SES (not Pinpoint)
```

---

## Architecture Patterns to Know

### Pattern 1: Full Serverless Mobile App
```
Mobile App (React Native + Amplify SDK)
        ↓ (auth)
Amazon Cognito (via Amplify Auth)
        ↓ (API calls)
API Gateway (REST) → Lambda → DynamoDB
        ↓ (file uploads)
Amazon S3 (via Amplify Storage)
        ↓ (push notifications + analytics)
Amazon Pinpoint (via Amplify Notifications)
        ↓ (CI/CD for backend)
AWS CodePipeline → CloudFormation
        ↓ (frontend hosting)
Amplify Hosting → CloudFront
```

### Pattern 2: Real-Time Chat Application
```
Browser/App → API Gateway (WebSocket API)
                    ├── $connect → Lambda (store connectionId in DynamoDB)
                    ├── sendMessage → Lambda (read all connections from DynamoDB)
                    │                       → @connections API → push to all clients
                    └── $disconnect → Lambda (remove connectionId from DynamoDB)
```

### Pattern 3: API Gateway Direct SQS Integration
```
Mobile Client → POST /order → API Gateway
                                    ↓ (AWS Service Integration — no Lambda)
                               SQS Queue
                                    ↓ (Lambda reads from SQS)
                               Lambda (process order)
                                    ↓
                               DynamoDB (store order)
                                    ↓
                               Pinpoint (send confirmation push notification)
```

### Pattern 4: Pre-Launch Mobile Testing Pipeline
```
Developer commits code → CodePipeline
        ↓
CodeBuild (build .apk / .ipa)
        ↓
Device Farm (run Appium tests on 50 real devices in parallel)
        ↓
Test results: all pass?
    YES → Deploy to production app stores
    NO  → Notify team via SNS; block deployment
```

### Pattern 5: User Re-Engagement Campaign
```
User hasn't opened app in 7 days
        ↓ (tracked by Pinpoint analytics)
Pinpoint Dynamic Segment: "inactive_7_days"
        ↓
Pinpoint Campaign (triggered at 10am user's timezone):
  └── Push notification: "We miss you! Here's 20% off your next order"
        ↓
User opens app → Pinpoint event tracked: "app_open"
        ↓
Journey: User added to "re-engaged" flow
  ├── Wait 1 hour → Did they purchase?
  │     YES → Send thank you email
  │     NO  → Send SMS reminder with promo code
```

---

## Exam Tips Summary

### AWS Amplify
- ✅ Hosting = CI/CD + CDN for frontend (connects to GitHub/GitLab/Bitbucket)
- ✅ Backend = provisions Cognito, AppSync, API Gateway, Lambda, S3 via CLI/Studio
- ✅ DataStore = offline-first sync built on AppSync + DynamoDB
- ✅ Branch previews = unique URL per git branch/PR
- ✅ SSR support for Next.js (runs on Lambda under the hood)
- ✅ Amplify is an abstraction layer — underlying services are standard AWS services

### API Gateway
- ✅ REST API = full features (caching, WAF, usage plans, request transformation) — higher cost
- ✅ HTTP API = lightweight, fast, cheap (~70% less) — JWT auth native, no caching
- ✅ WebSocket API = persistent bidirectional; $connect / $disconnect / $default routes
- ✅ Edge-optimized = CloudFront backed; Regional = single region; Private = VPC only
- ✅ Lambda Proxy = pass everything to Lambda as-is (most common)
- ✅ AWS Service Integration = call SQS/DynamoDB/SNS directly without Lambda
- ✅ Usage Plans + API Keys = throttle and quota per consumer (not per IAM)
- ✅ VPC Link = API Gateway → private NLB → private backend in VPC
- ✅ 429 = throttled; use SQS buffer pattern to absorb traffic spikes
- ✅ Canary deployments = % traffic to new stage; stage variables = point to env-specific Lambda aliases

### AWS Device Farm
- ✅ Real physical devices (not simulators/emulators) in AWS data centers
- ✅ Parallel testing = run on many devices simultaneously = fast
- ✅ Supports: Appium, XCTest, Espresso, Calabash, Selenium (web)
- ✅ Remote access = manual interactive testing on real device via browser
- ✅ Captures: screenshots, videos, logs, performance data per device per test
- ✅ Metered (pay per minute) vs Unmetered (fixed monthly fee for CI/CD)

### Amazon Pinpoint
- ✅ Channels: Push (APNs/FCM), Email (SES), SMS, Voice, In-App, Custom (Lambda)
- ✅ Pinpoint ≠ SNS: Pinpoint = user engagement/marketing; SNS = system pub/sub
- ✅ Pinpoint ≠ SES: Pinpoint = multichannel campaigns; SES = transactional email API
- ✅ Endpoints = individual destinations (one user → multiple endpoints)
- ✅ Segments = filtered groups of endpoints (dynamic or imported)
- ✅ Campaigns = scheduled or event-triggered message to a segment
- ✅ Journeys = multi-step event-driven workflows (drip campaigns)
- ✅ A/B testing built into campaigns
- ✅ Built-in analytics: DAU, MAU, session data, funnel, revenue tracking
- ✅ Amplify Notifications category uses Pinpoint under the hood

---
---

<a name="media"></a>
# PART 13 — Media Services: Elastic Transcoder & Kinesis Video Streams

# AWS Exam Study Guide: Media Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Amazon Elastic Transcoder · Amazon Kinesis Video Streams

---

## Media Services at a Glance

| Service | What It Does | Direction | Latency | Exam Trigger Words |
|---|---|---|---|---|
| **Elastic Transcoder** | Convert/transcode stored media files to different formats | Stored (VOD) | Batch (minutes) | "convert video format", "MP4 to HLS", "video on demand", "transcode" |
| **Kinesis Video Streams** | Ingest, store, and process live video streams from devices | Live streaming | Real-time / near-real-time | "live video", "IP camera", "WebRTC", "real-time video analytics", "dashcam" |

---

---

## 1. Amazon Elastic Transcoder

### What Problem Does It Solve?

Video files come in dozens of formats (MP4, MOV, AVI, MKV), codecs (H.264, HEVC, VP9), and resolutions (4K, 1080p, 720p, 480p). Different devices — iPhones, Android phones, Smart TVs, web browsers — require different formats and bitrates. Without transcoding, a 4K MOV file shot on a camera cannot be streamed on a mobile phone over a slow connection.

**Elastic Transcoder solves this by:**
- Taking an input video file stored in **Amazon S3**
- Converting it to one or more output formats optimized for different devices
- Storing the output files back in **Amazon S3**
- All without managing any servers or encoding infrastructure

**Real-World Example:**
> A media company uploads raw TV episodes (4K ProRes files, ~50GB each) to S3 after filming. Elastic Transcoder automatically converts each episode into:
> - 1080p H.264 MP4 for Smart TVs
> - 720p HLS for iOS/macOS streaming
> - 480p DASH for Android streaming
> - 360p WebM for older browsers
>
> All four outputs are stored in S3 and served via CloudFront to global viewers.

---

### Core Concepts

#### Pipeline
- A **queue** that processes transcoding jobs.
- Has an input S3 bucket and one or more output S3 buckets.
- Controls throughput — multiple pipelines can run in parallel.
- **Status**: Active or Paused.

> **Analogy:** A pipeline is like a factory assembly line. You feed raw video in one end; finished videos come out the other.

#### Job
- A single transcoding request.
- Specifies: input file (S3 key), output preset(s), output S3 location, thumbnail settings.
- One job can produce **multiple outputs** simultaneously (e.g., 1080p + 720p + 480p in one job).
- Jobs are submitted to a pipeline and processed in order.

#### Preset
- A **template** defining the output format, codec, bitrate, resolution, and audio settings.
- Two types:
  - **System Presets**: Pre-built by AWS (e.g., "Generic 1080p", "iPhone 6", "Web: Facebook HD 720p 24fps")
  - **Custom Presets**: You define codec, bitrate, resolution, frame rate, audio channels
- You do NOT need to know codec settings manually — apply the right preset.

> **Example Presets:**
> - `1351620000001-000001` → Generic 1080p
> - `1351620000001-100070` → iPhone 5 (1080p)
> - `1351620000001-200045` → Web: 720p
> - `1351620000001-500050` → Audio MP3 → 128k

#### Thumbnail
- Elastic Transcoder can extract **thumbnail images** from the video at defined intervals.
- Stored as JPEG or PNG in the output S3 bucket.
- Used for video preview images in streaming platforms.

---

### Supported Input Formats

| Category | Formats |
|---|---|
| **Video containers** | MP4, MOV, MXF, FLV, AVI, WMV, 3GP, TS, MKV |
| **Video codecs** | H.264, H.265 (HEVC), MPEG-2, VP8, VP9 |
| **Audio** | AAC, MP3, PCM, WAV, FLAC, Vorbis |

---

### Supported Output Formats

| Format | Use Case |
|---|---|
| **MP4 (H.264/AAC)** | Universal; works on all devices |
| **HLS (HTTP Live Streaming)** | iOS, macOS, Smart TVs; adaptive bitrate |
| **MPEG-DASH** | Android, web; adaptive bitrate |
| **WebM (VP8/VP9)** | Web browsers (Chrome, Firefox) |
| **Smooth Streaming** | Microsoft/Silverlight (legacy) |
| **MP3/AAC** | Audio-only output |
| **GIF** | Animated GIF from video clip |

---

### HLS and Adaptive Bitrate Streaming

**HLS (HTTP Live Streaming)** is the most common output for video streaming:

- Video is split into small **segments** (e.g., 10-second .ts files).
- A **manifest file** (.m3u8) lists all segments and available bitrate variants.
- The player automatically switches quality based on network speed.
- Elastic Transcoder creates the HLS segments and manifest file automatically.

```
S3 Input: movie.mp4 (4K, 8 Gbps raw)
    ↓ Elastic Transcoder Job (HLS preset)
S3 Output:
    ├── movie_1080p/  → 10-second .ts segments
    ├── movie_720p/   → 10-second .ts segments
    ├── movie_360p/   → 10-second .ts segments
    └── master.m3u8   → Master playlist referencing all quality levels
```

---

### Elastic Transcoder Architecture

```
Raw Video File
(uploaded to S3)
      ↓
S3 Input Bucket
      ↓
Elastic Transcoder Pipeline
      ↓ (Job submitted via API/Lambda/Console)
Elastic Transcoder Job
  ├── Output 1: 1080p MP4 (System Preset)
  ├── Output 2: 720p HLS Segments
  ├── Output 3: 480p DASH
  └── Thumbnails: every 60 seconds → JPEG
      ↓
S3 Output Bucket
      ↓
CloudFront Distribution
      ↓
End Users (TV, Mobile, Web)
```

---

### Notifications

Elastic Transcoder sends **SNS notifications** on:
- Job **Progressing** (started)
- Job **Complete** (success)
- Job **Warning** (non-fatal issues)
- Job **Error** (failed)

This allows event-driven workflows: SNS → Lambda → update database, trigger CDN invalidation, send user email.

---

### Encryption

| Type | Description |
|---|---|
| **S3 server-side encryption** | Input and output files encrypted at rest in S3 (SSE-S3 or SSE-KMS) |
| **HLS content protection** | AES-128 encryption of HLS segments; key served via HTTPS |
| **DRM** | Not natively supported; use third-party DRM with Elastic Transcoder |

---

### Elastic Transcoder vs AWS Elemental MediaConvert

> **This is the most important comparison for the exam.**

| Feature | Elastic Transcoder | AWS Elemental MediaConvert |
|---|---|---|
| **Status** | Legacy (still available) | **Current, recommended service** |
| **Pricing model** | Per minute of output video | Per minute + feature-based |
| **Broadcast features** | Limited | ✅ Full broadcast-grade (captions, HDR, Dolby) |
| **Ad insertion** | ❌ | ✅ SCTE-35 ad markers |
| **Captions/Subtitles** | Basic | ✅ SRT, TTML, SCC, WebVTT |
| **HDR** | ❌ | ✅ HDR10, Dolby Vision |
| **Reserved queues** | ❌ | ✅ (predictable cost for high volume) |
| **Accelerated transcoding** | ❌ | ✅ (parallel processing) |
| **Use case** | Simple web/mobile transcoding | Broadcast, OTT, professional media |

> **Exam Rule:** For new workloads, AWS recommends **MediaConvert** over Elastic Transcoder. If the exam asks about "simple transcoding from S3 to S3 for web/mobile" → Elastic Transcoder. If it asks about "broadcast-quality", "captions", "HDR", "ad insertion" → MediaConvert.

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Convert MP4 uploaded to S3 into multiple HLS resolutions | Elastic Transcoder |
| Auto-transcode when video uploaded to S3 | S3 Event → Lambda → Elastic Transcoder Job |
| Extract thumbnail images from video | Elastic Transcoder thumbnail feature |
| Get notified when transcoding job completes | Elastic Transcoder SNS notification |
| Broadcast-quality transcoding with captions and HDR | AWS Elemental MediaConvert (not Elastic Transcoder) |
| Serverless video processing pipeline | S3 → Lambda → Elastic Transcoder → S3 → CloudFront |
| Protect HLS video stream from unauthorized access | CloudFront Signed URLs + HLS AES-128 encryption |

---

---

## 2. Amazon Kinesis Video Streams

### What Problem Does It Solve?

Millions of IoT devices, IP cameras, smartphones, drones, and dashcams generate continuous live video. Traditionally, building infrastructure to ingest, store, and process this video at scale requires significant engineering effort — managing servers, handling network interruptions, dealing with device reconnects, storing video durably, and enabling real-time ML inference.

**Kinesis Video Streams solves this by:**
- Providing a managed, scalable service to **ingest live video** from any number of devices
- **Durably storing** video (seconds to years) with time-indexed access
- Enabling **real-time and batch video analytics** using ML services (Rekognition, SageMaker)
- Supporting **WebRTC** for two-way, ultra-low latency video communication

**Real-World Examples:**
> 1. **Smart Home Security:** 500 home cameras stream live video to Kinesis Video Streams. Amazon Rekognition Video analyzes each stream for motion, faces, and package detection in real time. Alerts are sent via SNS when a person is detected at the front door.
>
> 2. **Autonomous Vehicles:** Dashcams on 10,000 fleet vehicles continuously stream video to Kinesis Video Streams. ML models analyze video for near-miss incidents, driver behavior, and road conditions.
>
> 3. **Telehealth Video:** A hospital uses Kinesis Video Streams with WebRTC for real-time two-way video between doctors and patients — with the video durably stored for compliance.

---

### Core Concepts

#### Stream
- The fundamental unit — represents a **single video source** (one camera, one device).
- Named resource; stores video fragments with time-based indexing.
- **Retention period**: Configurable from 0 hours (no storage) to 87,600 hours (10 years).
- Unlimited streams per account.

#### Producer
- A **device or application** that sends video data to Kinesis Video Streams.
- Uses the **Kinesis Video Streams Producer SDK** (C++, Java, Android, iOS).
- Handles: connection, retry logic, buffering, codec negotiation.
- Examples: IP camera, Raspberry Pi, smartphone, drone, IoT device.

#### Consumer
- An **application** that reads and processes video from a stream.
- Types:
  - **Real-time consumer**: Reads live fragments as they arrive
  - **Historical consumer**: Reads stored video from a past timestamp
- Examples: ML inference app, recording application, monitoring dashboard.

#### Fragment
- A self-contained unit of video (typically 2–10 seconds of video + audio).
- Each fragment has a **producer timestamp** and **server timestamp**.
- Fragments are the smallest unit of consumption.
- Stored durably and retrievable by timestamp range.

#### MKV (Matroska) Format
- Kinesis Video Streams uses **MKV** as the streaming container format.
- The Producer SDK handles encoding to MKV automatically.
- Consumers receive MKV fragments.

---

### Kinesis Video Streams Ingestion Flow

```
Physical Devices / Sources
│
├── IP Cameras (RTSP)
├── Smartphones (Android/iOS SDK)
├── Drones / Dashcams
├── Raspberry Pi / IoT devices
└── WebRTC peers
         │
         ↓ (Producer SDK — handles TLS, auth, retry)
Amazon Kinesis Video Streams
         │
    Stream Storage
    (time-indexed, durable, S3-backed internally)
         │
    ┌────┴──────────────────────────┐
    ↓                               ↓
Real-Time Processing          Historical Playback
(Rekognition, Lambda,         (playback at any past
 SageMaker, custom app)        timestamp range)
```

---

### Playback APIs

Kinesis Video Streams provides multiple APIs for reading video:

| API | Description | Use Case |
|---|---|---|
| **GetMedia** | Reads live or historical MKV fragments continuously | Real-time processing apps |
| **GetMediaForFragmentList** | Retrieves specific fragments by fragment number | Replay specific events |
| **ListFragments** | Lists fragments in a time range | Find relevant video segments |
| **GetHLSStreamingSessionURL** | Generates an HLS URL for playback | In-browser video playback |
| **GetDASHStreamingSessionURL** | Generates a DASH URL for playback | Adaptive bitrate playback |

> **Exam Tip:** `GetHLSStreamingSessionURL` is the key API for playing back Kinesis Video Streams in a web browser without writing a custom player.

---

### Kinesis Video Streams + Amazon Rekognition Video

**The most important integration for the exam.**

```
IP Camera Stream
      ↓
Kinesis Video Streams (ingest + store)
      ↓
Amazon Rekognition Video
      ↓ (real-time analysis per fragment)
Kinesis Data Streams (output stream of detections)
      ↓
Lambda Function
      ↓
SNS / DynamoDB / EventBridge
```

**What Rekognition Video can detect:**
- Faces (detection, recognition, comparison)
- Objects and scenes
- Text in video
- People and activity
- Inappropriate content (content moderation)
- PPE (Personal Protective Equipment) detection

**Example:** A retail store with 50 cameras streams to Kinesis Video Streams. Rekognition Video detects when a customer is at an empty checkout lane and alerts staff via SNS in real time.

---

### WebRTC Support

Kinesis Video Streams supports **WebRTC** for ultra-low latency (sub-second) two-way video:

| Feature | Details |
|---|---|
| **Latency** | Sub-second (< 500ms) |
| **Protocol** | WebRTC (ICE, DTLS, SRTP) |
| **Use case** | Two-way video communication (telehealth, remote monitoring, video doorbells) |
| **Signaling** | Kinesis Video Streams provides the WebRTC signaling service |
| **TURN/STUN** | AWS provides TURN and STUN servers for NAT traversal |
| **Peer types** | Browser-to-device, device-to-device, device-to-browser |

**WebRTC vs Standard Streaming:**

| Feature | Standard Streaming | WebRTC |
|---|---|---|
| **Latency** | 2–5 seconds | < 500ms |
| **Direction** | One-way (device → cloud) | Two-way |
| **Storage** | Yes (in stream) | Optional |
| **Use case** | Surveillance, analytics | Live interaction, doorbells |

---

### Kinesis Video Streams vs Kinesis Data Streams

> **Very commonly confused on the exam.**

| Feature | Kinesis Video Streams | Kinesis Data Streams |
|---|---|---|
| **Data type** | Video, audio, RADAR, LIDAR | Any data records (JSON, CSV, logs, events) |
| **Record unit** | Video fragment (MKV, seconds long) | Data record (up to 1 MB) |
| **Ingestion** | Producer SDK on device | Kinesis Producer Library (KPL) or API |
| **Consumers** | Rekognition Video, custom apps | Lambda, KCL, Firehose, Analytics |
| **Storage** | Up to 10 years | 24 hours to 365 days |
| **Shard concept** | ❌ No shards (per-stream) | ✅ Shards (partition key based) |
| **Ordering** | Per stream, time-ordered | Per shard, sequence-ordered |
| **Use case** | Video surveillance, IoT cameras | Event streaming, clickstreams, logs |

---

### Kinesis Video Streams vs Amazon IVS (Interactive Video Service)

| Feature | Kinesis Video Streams | Amazon IVS |
|---|---|---|
| **Use case** | IoT/device video ingestion + analytics | Live streaming to large audiences |
| **Protocol** | SDK-based (Producer SDK), WebRTC | RTMPS ingest |
| **Latency** | Configurable (near-real-time to batch) | < 3 seconds (ultra-low latency) |
| **Audience** | Machines / analytics systems | Human viewers (Twitch-like) |
| **Rekognition integration** | ✅ Native | ❌ |
| **Interactive features** | ❌ | ✅ Timed metadata, chat |
| **Use case example** | Security camera analytics | Live game streaming, virtual events |

---

### Storage and Retention

| Retention Setting | Use Case |
|---|---|
| **0 hours** | No storage; real-time processing only; lowest cost |
| **1–24 hours** | Short-term buffer for real-time processing with recent replay |
| **7 days** | Typical for security camera footage review |
| **30 days** | Insurance/compliance review windows |
| **87,600 hours (10 years)** | Maximum; long-term compliance or forensic use |

> **Cost Consideration:** You pay for: (1) volume of video ingested (per GB), (2) video stored (per GB/month), (3) video consumed/read (per GB). Setting retention to 0 eliminates storage costs when only real-time processing is needed.

---

### Kinesis Video Streams Security

| Feature | Details |
|---|---|
| **Encryption in transit** | TLS 1.2 always enforced |
| **Encryption at rest** | KMS server-side encryption |
| **IAM** | IAM policies control PutMedia, GetMedia per stream |
| **VPC** | Interface endpoint (PrivateLink) for private VPC access |
| **Stream-level IAM** | Fine-grained: different roles for producers vs consumers |

---

### Producer SDK Platforms

| Platform | SDK |
|---|---|
| **C++** | Linux, macOS, Windows, Raspberry Pi, embedded |
| **Java** | JVM-based applications |
| **Android** | Native Android apps |
| **iOS/macOS** | Swift/Objective-C |
| **GStreamer plugin** | IP cameras with RTSP using GStreamer pipeline |

> **Exam Tip:** IP cameras with RTSP feeds connect to Kinesis Video Streams using the **GStreamer plugin** — they don't natively run the SDK but GStreamer acts as the bridge.

---

### Kinesis Video Streams Architecture Patterns

#### Pattern 1: Smart Building Security
```
100 IP Cameras (RTSP)
      ↓ (GStreamer plugin)
Kinesis Video Streams (100 streams, 7-day retention)
      ↓
Rekognition Video (face detection, PPE detection)
      ↓
Kinesis Data Streams (detection events)
      ↓
Lambda → DynamoDB (log events) + SNS (real-time alerts)
```

#### Pattern 2: Video Doorbell (WebRTC)
```
Doorbell Device (camera + mic)
      ↓ WebRTC (sub-second latency)
Kinesis Video Streams (WebRTC signaling + TURN)
      ↓ WebRTC
Mobile App (homeowner sees and speaks to visitor)
      ↓
Optional: Store video clip to stream (motion events)
```

#### Pattern 3: ML Training Data Collection
```
Fleet Vehicles (dashcams, 10,000 trucks)
      ↓ (Producer SDK)
Kinesis Video Streams (30-day retention)
      ↓ (GetMediaForFragmentList — extract interesting clips)
S3 (raw video clips for labeling)
      ↓
SageMaker Ground Truth (human labeling)
      ↓
SageMaker Training Job (train object detection model)
```

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Ingest live video from 1,000 IoT cameras | Kinesis Video Streams |
| Real-time face detection from security cameras | KVS + Rekognition Video + Kinesis Data Streams |
| Play back security camera footage from 3 days ago in browser | KVS GetHLSStreamingSessionURL (historical playback) |
| Two-way video for telehealth with sub-second latency | KVS WebRTC |
| IP camera (RTSP) to Kinesis Video Streams | GStreamer plugin on a local server |
| Store video for exactly 30 days then auto-expire | KVS 30-day retention period |
| Real-time video analytics without storing video | KVS with 0-hour retention + GetMedia consumer |
| Live streaming a gaming event to thousands of viewers | Amazon IVS (not KVS) |

---

---

## Master Comparison: Elastic Transcoder vs Kinesis Video Streams

| Dimension | Elastic Transcoder | Kinesis Video Streams |
|---|---|---|
| **Data source** | Stored files in S3 | Live streams from devices |
| **Processing model** | Batch (job-based) | Real-time / continuous |
| **Latency** | Minutes (offline) | Sub-second to seconds |
| **Output** | Transcoded files in S3 | Live stream / stored fragments |
| **ML integration** | ❌ | ✅ Rekognition Video |
| **WebRTC** | ❌ | ✅ |
| **Primary use** | Video on demand (VOD) | Live video surveillance, IoT, video calls |
| **Consumers** | CloudFront viewers | Real-time analytics apps, humans |
| **Serverless** | ✅ (fully managed) | ✅ (fully managed) |

---

## Full End-to-End Streaming Architecture (Exam Pattern)

### VOD (Video On Demand) Pipeline
```
Content Creator → S3 Upload (raw video, any format)
                        ↓ S3 Event Notification
                   Lambda (submit Elastic Transcoder job)
                        ↓
                  Elastic Transcoder Pipeline
                        ↓ (produces multiple outputs)
                  S3 Output Bucket
                  ├── 1080p HLS segments + manifest
                  ├── 720p HLS segments + manifest
                  ├── 480p MP4
                  └── Thumbnails (JPEG)
                        ↓
                  CloudFront CDN (global distribution)
                        ↓ SNS notification on job complete
                   Lambda → DynamoDB (update video status)
                        ↓
                  End Users (TV / Mobile / Web)
```

### Live Video Analytics Pipeline
```
IoT Cameras / Devices
      ↓ (Producer SDK)
Kinesis Video Streams
      ↓                      ↓
Real-Time Path         Historical Path
(GetMedia)             (GetHLSStreamingSessionURL)
      ↓                      ↓
Rekognition Video     Browser Playback
      ↓
Kinesis Data Streams
      ↓
Lambda / Analytics / Alerts
```

---

## Exam Tips Summary

### Amazon Elastic Transcoder
- ✅ Converts stored S3 video files → multiple output formats in S3
- ✅ Pipeline = job queue; Job = single transcode request; Preset = output format template
- ✅ One job → multiple outputs (1080p + 720p + 480p + thumbnails simultaneously)
- ✅ HLS output = segments (.ts files) + manifest (.m3u8) for adaptive streaming
- ✅ SNS notifications for job completion → trigger Lambda for event-driven workflows
- ✅ System presets (pre-built) vs Custom presets (user-defined)
- ✅ **Legacy service** — MediaConvert is the modern replacement for broadcast/professional
- ✅ Use Elastic Transcoder for: simple web/mobile transcoding, S3-to-S3 pipelines
- ✅ Use MediaConvert for: captions, HDR, ad markers, broadcast-quality, Dolby audio

### Amazon Kinesis Video Streams
- ✅ Ingest live video from IoT devices, cameras, mobile apps
- ✅ Fragment = self-contained MKV unit of video (2–10 seconds)
- ✅ Producer SDK handles connection, retry, buffering automatically
- ✅ Retention 0 hours (real-time only) to 10 years (compliance)
- ✅ GetHLSStreamingSessionURL = play back video in browser (key API!)
- ✅ WebRTC = two-way, sub-second latency video communication
- ✅ KVS + Rekognition Video = real-time video analytics (most exam pattern)
- ✅ IP cameras (RTSP) → KVS via GStreamer plugin
- ✅ KVS ≠ Kinesis Data Streams: KVS = video/audio; KDS = general data records
- ✅ KVS ≠ Amazon IVS: KVS = IoT/analytics; IVS = broadcast to human viewers
- ✅ VPC interface endpoint for private stream access (no internet)
- ✅ KMS encryption at rest; TLS always in transit

---
---

<a name="xray"></a>
# PART 14 — Developer Tools: AWS X-Ray

# AWS Exam Study Guide: AWS X-Ray
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Category:** Developer Tools — Observability & Distributed Tracing

---

## What Is AWS X-Ray?

AWS X-Ray is a **distributed tracing service** that helps developers analyze, debug, and optimize the performance of **distributed applications** — especially microservices and serverless architectures. It collects data about the requests flowing through your application, then provides tools to **visualize the entire request journey** across every service, identify bottlenecks, and find errors.

> **Core Question X-Ray Answers:**
> *"A user says my app is slow — but I have 12 microservices. Which one is the problem, and why?"*
>
> X-Ray shows you the **entire request path** with **timing for every hop** — so you know exactly where the latency or error occurred.

---

## The Problem X-Ray Solves

### Traditional Monolith Debugging
```
User request → Single application → Database
                     ↓
              One set of logs
              One place to look
```
Easy to debug — everything is in one place.

### Modern Microservices / Serverless Reality
```
User request → API Gateway → Lambda A → SQS → Lambda B → DynamoDB
                                    ↓                ↓
                               Cognito          ElastiCache
                                    ↓
                              External API (Stripe)
```

**Problems without X-Ray:**
- Request passes through 8 services — which one is slow?
- Error occurs in Lambda B — caused by Lambda A's bad data or DynamoDB timeout?
- P99 latency is 3 seconds — where are those 3 seconds spent?
- How often does the external Stripe API fail?

**X-Ray solves this by tracking a single request across all services end-to-end.**

---

## Core X-Ray Concepts

### 1. Trace
- A **trace** represents the **end-to-end journey** of a single request through your application.
- It is composed of multiple **segments**.
- Every trace has a unique **Trace ID** (injected as an HTTP header: `X-Amzn-Trace-Id`).

**Example Trace:**
```
Trace ID: 1-5f84e3e3-xxxxxxxxxxxxxxxxxxxxxxxx
Total duration: 1.24 seconds

├── API Gateway (50ms)
├── Lambda: OrderService (380ms)
│   ├── DynamoDB: GetItem (45ms)
│   ├── DynamoDB: PutItem (62ms)
│   └── SNS: Publish (28ms)
├── Lambda: PaymentService (710ms)  ← BOTTLENECK
│   ├── ElastiCache: Get (8ms)
│   └── External: Stripe API (695ms)  ← ROOT CAUSE
└── Lambda: NotificationService (100ms)
```

---

### 2. Segment
- A **segment** is the data recorded by a **single service or application** for its portion of processing a request.
- Each service (Lambda, EC2, ECS task) sends one segment per request.
- Contains: service name, host, start/end times, HTTP details, error flags.

---

### 3. Subsegment
- A **subsegment** is a more **granular breakdown within a segment** — representing a specific operation.
- Automatically created for: DynamoDB calls, S3 calls, HTTP downstream calls, SQL queries.
- You can also create **custom subsegments** in your code for any logical block.

```
Lambda Segment (380ms total)
  ├── Subsegment: DynamoDB GetItem (45ms)
  ├── Subsegment: Business logic (200ms)  ← custom subsegment
  ├── Subsegment: DynamoDB PutItem (62ms)
  └── Subsegment: SNS Publish (28ms)
```

---

### 4. Service Map (Service Graph)
- A **visual diagram** of all services involved in your application and how they connect.
- Nodes = services (Lambda, DynamoDB, external APIs, etc.)
- Edges = connections between services with latency and error rates.
- Color-coded: **green** (healthy), **yellow** (slow), **red** (errors).

**Real-World Example:**
> Opening the service map shows: Lambda A → DynamoDB (green, 45ms avg) → Lambda B → Stripe API (red, 30% error rate, 2.1 sec avg). You immediately know Stripe integration is causing failures.

---

### 5. Annotations
- **Key-value pairs** that you add to traces in your application code.
- **Indexed** — can be used to **filter and search** traces in the console.
- Use for: user ID, order ID, feature flags, environment name.

```python
# Python example
subsegment.put_annotation("user_id", "usr_12345")
subsegment.put_annotation("order_id", "ord_99876")
subsegment.put_annotation("payment_method", "credit_card")
```

> **Exam Tip:** Annotations are **indexed** → searchable. Metadata is **not indexed** → for debugging only.

---

### 6. Metadata
- **Arbitrary data** attached to a segment/subsegment for debugging context.
- **Not indexed** — cannot be used to filter traces (unlike annotations).
- Use for: full request payloads, stack traces, detailed debugging info.

```python
subsegment.put_metadata("order_details", {
    "items": ["laptop", "mouse"],
    "total": 1299.99
})
```

---

### 7. Groups
- **Filter expressions** saved as named groups to analyze specific subsets of traces.
- Example: create a group for `"error = true AND responsetime > 1"` — traces with errors AND slow response.
- Used to set up **CloudWatch alarms** on trace group metrics.

---

### 8. Sampling
- X-Ray does NOT record **every single request** by default — that would be too expensive and generate too much data.
- **Sampling rules** define what percentage of requests to record.
- Default rule: **5% of requests** (minimum 1 req/sec guaranteed).
- You can configure custom rules: e.g., record 100% of `/checkout` requests but only 1% of `/health` requests.

**Sampling Rule Configuration:**
```json
{
  "RuleName": "CheckoutHighPriority",
  "ResourceARN": "*",
  "Priority": 1,
  "FixedRate": 1.0,         ← 100% of matching requests
  "ReservoirSize": 60,       ← first 60 req/sec guaranteed
  "ServiceName": "OrderService",
  "URLPath": "/checkout",
  "HTTPMethod": "POST"
}
```

> **Exam Tip:** If traces are **missing** from X-Ray, the most likely cause is the **sampling rate** — increase it. Never set to 100% in high-traffic production (cost and performance impact).

---

## X-Ray Components

### X-Ray SDK
- **Library installed in your application code** (Java, Python, Go, Node.js, Ruby, .NET).
- Instruments your code to capture segments and subsegments.
- Automatically instruments: AWS SDK calls (DynamoDB, S3, SQS, SNS), HTTP requests, SQL queries.
- You add custom segments/subsegments for business logic.

### X-Ray Daemon
- A **background process** (agent) that runs on EC2, ECS, Elastic Beanstalk, etc.
- Collects raw segment data from the SDK over **UDP port 2000**.
- Buffers and sends segment data to the X-Ray service (API) in batches.
- Required for: EC2, on-premises, ECS containers.
- **NOT required for Lambda** — Lambda has X-Ray built-in (just enable active tracing).

```
Application Code (SDK) → UDP:2000 → X-Ray Daemon → X-Ray API (HTTPS)
```

### X-Ray API (Service)
- Receives segment data from the daemon.
- Stores and indexes traces.
- Provides APIs for querying traces, service maps, and statistics.

---

## X-Ray Integration by Service

| AWS Service | How X-Ray Works | What You Need |
|---|---|---|
| **Lambda** | Active tracing built-in | Enable "Active Tracing" in function config; add SDK for subsegments |
| **API Gateway** | Native integration | Enable X-Ray tracing per stage |
| **EC2** | X-Ray Daemon + SDK | Install daemon + instrument code |
| **ECS / Fargate** | X-Ray Daemon as sidecar | Add X-Ray Daemon container to task definition |
| **Elastic Beanstalk** | Daemon pre-installed | Enable in `.ebextensions` config or console |
| **App Runner** | Native integration | Enable in App Runner service config |
| **SNS** | Pass-through tracing | Trace ID propagated in message attributes |
| **SQS** | Pass-through tracing | Trace ID propagated in message attributes |
| **DynamoDB** | Auto-instrumented by SDK | Use X-Ray SDK — DynamoDB calls appear as subsegments |
| **RDS / Aurora** | Auto-instrumented by SDK | SQL queries appear as subsegments |
| **S3** | SDK instrumentation | S3 calls appear as subsegments |

---

## X-Ray on Lambda — Deep Dive

Lambda X-Ray has **two modes**:

| Mode | Description |
|---|---|
| **Passive tracing** | Lambda records segments only if an upstream service (e.g., API Gateway) started a trace |
| **Active tracing** | Lambda always starts a new trace regardless of upstream |

**To enable Active Tracing on Lambda:**
- Console: Configuration → Monitoring → Enable X-Ray active tracing
- CloudFormation: `Tracing: Active`
- Lambda automatically sends data to X-Ray — **no daemon needed**

**Lambda IAM permission required:**
```
AWSXRayDaemonWriteAccess policy on the Lambda execution role
```

**Lambda + X-Ray Segments:**
- Lambda automatically creates a **facade segment** (the Lambda invocation metadata).
- Your code using the X-Ray SDK adds **subsegments** within the facade segment.

> **Exam Tip:** Lambda does NOT require the X-Ray Daemon — it has a built-in mechanism. ECS and EC2 DO require the daemon as a sidecar container or installed agent.

---

## X-Ray on ECS — Sidecar Pattern

For ECS (EC2 or Fargate), the X-Ray daemon runs as a **sidecar container** in the same task definition:

```json
{
  "containerDefinitions": [
    {
      "name": "my-app",
      "image": "my-app:latest",
      "environment": [
        { "name": "AWS_XRAY_DAEMON_ADDRESS", "value": "127.0.0.1:2000" }
      ]
    },
    {
      "name": "xray-daemon",
      "image": "amazon/aws-xray-daemon",
      "portMappings": [
        { "hostPort": 2000, "containerPort": 2000, "protocol": "udp" }
      ]
    }
  ]
}
```

- App container sends trace data to `127.0.0.1:2000` (localhost within task).
- X-Ray daemon container forwards to X-Ray service.

---

## X-Ray Analytics & Console Features

### Trace Map
- Visual graph of all services + their connections.
- Shows avg response time and error/fault/throttle rates per node and edge.
- Click on a node → filter traces through that service.
- Click on an edge → see details of calls between those two services.

### Trace List
- Searchable list of all recorded traces.
- Filter by: time range, status code, annotation values, URL, response time.
- Example filter: `annotation.user_id = "usr_12345" AND responsetime > 2`

### Analytics Dashboard
- **Compare** two groups of traces side by side.
- Metrics: response time distribution, error rate, fault rate, throttle rate.
- Identify outliers: "What's different about requests that take > 3 seconds vs normal?"

---

## X-Ray Errors, Faults, and Throttles

X-Ray classifies issues into three types:

| Type | HTTP Status | Description |
|---|---|---|
| **Error** | 4xx | Client error (bad request, not found, unauthorized) |
| **Fault** | 5xx | Server error (crash, timeout, internal error) |
| **Throttle** | 429 | Too Many Requests (throttling) |

These are tracked per service in the service map — making it easy to see which service is experiencing which type of issue and at what rate.

---

## X-Ray vs CloudWatch vs CloudTrail

> **This three-way comparison is frequently tested.**

| Dimension | X-Ray | CloudWatch | CloudTrail |
|---|---|---|---|
| **Primary purpose** | Distributed tracing | Metrics, logs, alarms | API call audit |
| **What it records** | Request journey across services | Time-series metrics + log data | Who called which AWS API |
| **Question answered** | "Why is my app slow/failing?" | "Is my app healthy right now?" | "Who changed my infrastructure?" |
| **Visualization** | Service map (graph) | Dashboards, charts | Event history table |
| **Granularity** | Per-request traces | Per-minute metrics | Per API call |
| **Best for** | Debugging distributed systems | Monitoring, alerting | Security audit, compliance |
| **Latency analysis** | ✅ Per-service, per-call | Aggregate only | ❌ |
| **Real-time** | Near real-time | ✅ Real-time | Near real-time |

> **Exam Rule:**
> - **"Why is this specific request slow?"** → X-Ray
> - **"Alert me when CPU > 80%"** → CloudWatch
> - **"Who deleted that S3 bucket?"** → CloudTrail

---

## X-Ray vs AWS Distro for OpenTelemetry (ADOT)

| Feature | X-Ray (native SDK) | AWS Distro for OpenTelemetry (ADOT) |
|---|---|---|
| **Standard** | AWS proprietary | OpenTelemetry (vendor-neutral open standard) |
| **Vendor lock-in** | AWS only | Any backend (X-Ray, Prometheus, Jaeger, Datadog) |
| **Language support** | 6 languages | Many languages + auto-instrumentation |
| **Setup complexity** | Simpler | More flexible but more config |
| **Recommendation** | AWS-native apps | Multi-cloud or existing OTel investment |

> **Exam Tip:** For SAP-C02, know that **ADOT** (AWS Distro for OpenTelemetry) can send traces to **X-Ray** — it's the AWS-recommended path for new applications, especially containers.

---

## X-Ray Pricing

| Dimension | Free Tier | Paid |
|---|---|---|
| **Traces recorded** | 100,000 traces/month | $5.00 per million traces |
| **Traces scanned** | 1,000,000 traces/month | $0.50 per million traces scanned |
| **Trace retrieval** | Included | Included |
| **Retention** | 30 days | 30 days |

> **Exam Tip:** X-Ray uses sampling (not 100% recording) partly to control costs. Traces are retained for **30 days**.

---

## Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Identify which microservice is causing latency in a distributed app | AWS X-Ray (service map + traces) |
| Debug why some user requests fail while others succeed | X-Ray trace filtering with annotations |
| Monitor overall application health with alarms | CloudWatch (not X-Ray) |
| Find out who called DeleteTable on DynamoDB at 2am | CloudTrail (not X-Ray) |
| Trace requests through API Gateway → Lambda → DynamoDB | X-Ray with active tracing on all services |
| X-Ray traces are missing for some requests | Check/increase sampling rate |
| Reduce X-Ray data collection for high-volume `/health` endpoint | Custom sampling rule with low fixed rate |
| Add searchable business context to traces (user ID, order ID) | X-Ray annotations |
| Add debugging payload data to traces (not searchable) | X-Ray metadata |
| X-Ray on ECS without modifying app container image | Add X-Ray daemon as sidecar container |
| Enable X-Ray on Lambda | Enable "Active Tracing" in Lambda config (no daemon needed) |
| Build vendor-agnostic tracing for multi-cloud app | AWS Distro for OpenTelemetry (ADOT) → X-Ray |
| Alert when error rate in a specific service exceeds 5% | X-Ray Groups → CloudWatch alarms on group metrics |

---

## Architecture Patterns

### Pattern 1: End-to-End Serverless Tracing
```
User Request (HTTP)
        ↓
API Gateway (X-Ray enabled at stage level)
        ↓ [Trace ID propagated in header]
Lambda: AuthService (Active Tracing ON)
  └── Subsegment: Cognito token validation
        ↓
Lambda: OrderService (Active Tracing ON)
  ├── Subsegment: DynamoDB GetItem (check inventory)
  ├── Subsegment: Custom: "calculate discount" (business logic)
  └── Subsegment: SQS SendMessage (queue order)
        ↓
Lambda: PaymentService (Active Tracing ON)
  ├── Subsegment: ElastiCache GET (check cached card)
  └── Subsegment: HTTP POST to Stripe API
        ↓
X-Ray Service Map shows:
  API GW → AuthService → OrderService → PaymentService
                              └──→ DynamoDB (green, fast)
                         PaymentService → Stripe (RED: 3 sec avg)
```

### Pattern 2: ECS Microservices Tracing
```
ECS Task: ProductService
  ├── App Container (ProductService code + X-Ray SDK)
  │   └── SDK sends UDP to 127.0.0.1:2000
  └── Sidecar Container (X-Ray Daemon)
      └── Forwards to X-Ray API over HTTPS

ECS Task: InventoryService
  ├── App Container (InventoryService code + X-Ray SDK)
  └── Sidecar Container (X-Ray Daemon)

Both services trace connected requests:
ProductService → HTTP call → InventoryService
                                   ↓
                            X-Ray shows complete
                            cross-service trace
```

### Pattern 3: EC2-Based Application Tracing
```
EC2 Instance (Elastic Beanstalk or plain EC2):
  ├── Application code with X-Ray SDK (Python/Java/Node.js)
  ├── X-Ray Daemon (installed, listens on UDP 2000)
  └── IAM Role with AWSXRayDaemonWriteAccess

Flow:
  Request comes in → App SDK captures segment
  SDK sends to Daemon (UDP:2000)
  Daemon buffers + sends to X-Ray API (HTTPS)
  Trace appears in X-Ray console within seconds
```

### Pattern 4: Sampling Strategy for Cost Control
```
High-traffic API (/products/list):
  Rule: FixedRate = 0.01 (1%), ReservoirSize = 5
  → Records 5 req/sec guaranteed + 1% of the rest
  → Low cost; statistical visibility

Critical path (/checkout):
  Rule: FixedRate = 1.0 (100%), ReservoirSize = 50
  → Records every checkout request
  → Full visibility for revenue-critical path

Health check (/health):
  Rule: FixedRate = 0.0 (0%)
  → Never recorded (not useful, saves cost)
```

---

## Exam Tips Summary

### Core Concepts
- ✅ **Trace** = end-to-end request journey; **Segment** = one service's contribution; **Subsegment** = granular operation within a segment
- ✅ **Service Map** = visual graph of all services with latency + error rates (colored: green/yellow/red)
- ✅ **Annotations** = indexed key-value pairs → searchable in trace filter; **Metadata** = non-indexed → debug only
- ✅ **Sampling** = not all requests recorded by default; default = 5% + 1 req/sec guaranteed
- ✅ Traces retained for **30 days**

### Setup & Integration
- ✅ **Lambda**: Enable "Active Tracing" → NO daemon needed; built-in mechanism
- ✅ **EC2**: Requires X-Ray SDK in code + X-Ray Daemon installed + IAM role (`AWSXRayDaemonWriteAccess`)
- ✅ **ECS/Fargate**: X-Ray Daemon as **sidecar container** in same task definition; app sends to `localhost:2000`
- ✅ **API Gateway**: Enable X-Ray tracing per stage in console
- ✅ **Elastic Beanstalk**: Daemon pre-installed; enable in `.ebextensions` or console config
- ✅ Daemon receives on **UDP port 2000**; sends to X-Ray API over **HTTPS**

### Error Classification
- ✅ **Error** = 4xx (client errors); **Fault** = 5xx (server errors); **Throttle** = 429
- ✅ All three are tracked per service in the service map with rate percentages

### X-Ray vs Others
- ✅ X-Ray = "**Why** is it slow/broken?" (distributed tracing per request)
- ✅ CloudWatch = "**Is it healthy?**" (metrics, alarms, dashboards)
- ✅ CloudTrail = "**Who** did what to infrastructure?" (API audit)
- ✅ ADOT = vendor-neutral OpenTelemetry; can send to X-Ray as backend

### Cost & Sampling
- ✅ Missing traces → check sampling rate (increase if needed)
- ✅ High-traffic endpoints → use low sampling rate (1%) or 0% for health checks
- ✅ Revenue-critical paths → 100% sampling regardless of cost
- ✅ Free tier: 100K traces recorded/month, 1M traces scanned/month

---
---

<a name="cost"></a>
# PART 15 — Cost Management: Budgets, Cost and Usage Report, Cost Explorer & Savings Plans

# AWS Exam Study Guide: Cost Management
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** AWS Budgets · AWS Cost and Usage Report (CUR) · AWS Cost Explorer · Savings Plans

---

## 📋 Table of Contents

1. [Cost Management Quick-Reference Matrix](#quick-reference)
2. [Cost Management Service Relationships](#relationships)
3. [AWS Budgets](#budgets)
4. [AWS Cost and Usage Report (CUR)](#cur)
5. [AWS Cost Explorer](#cost-explorer)
6. [Savings Plans](#savings-plans)
7. [Related Cost Optimization Services](#related)
8. [Master Comparisons](#comparisons)
9. [Architecture Patterns](#patterns)
10. [Exam Tips Summary](#exam-tips)

---

<a name="quick-reference"></a>
## Cost Management Quick-Reference Matrix

| Service | Primary Purpose | Action | Exam Trigger Words |
|---|---|---|---|
| **AWS Budgets** | Set spending thresholds + alerts | **Alert** when cost/usage exceeds limits | "budget alert", "notify when spending exceeds", "cost threshold", "forecast alert" |
| **Cost and Usage Report (CUR)** | Granular raw billing data export | **Export** every AWS charge to S3 | "detailed billing", "raw cost data", "S3 export", "Athena cost analysis", "per-resource cost" |
| **Cost Explorer** | Visualize + analyze past + forecast costs | **Analyze** and explore spending patterns | "visualize costs", "cost breakdown", "spending trends", "RI recommendations" |
| **Savings Plans** | Flexible committed-use discounts | **Save** by committing to usage | "flexible discount", "commit to spend", "savings plan", "reduce EC2/Lambda/Fargate cost" |

---

<a name="relationships"></a>
## Cost Management Service Relationships

```
Raw Data Layer:
  AWS Cost and Usage Report (CUR)
  └── Exports every single charge to S3 (most granular)
      └── Query with Athena, visualize with QuickSight

Analysis Layer:
  AWS Cost Explorer
  └── Visual UI for trends, breakdowns, forecasts (up to 13 months)
      └── Also provides RI and Savings Plan purchase recommendations

Action Layer:
  AWS Budgets
  └── Set thresholds → alerts (email/SNS) → automated actions

Optimization Layer:
  Savings Plans
  └── Commit to $ spend/hour → get discounts on EC2, Lambda, Fargate, SageMaker
```

**How They Work Together:**
> Step 1 — Use **Cost Explorer** to understand your spending patterns over the last 3 months.
> Step 2 — Use **CUR + Athena** for deep-dive per-resource cost analysis across all accounts.
> Step 3 — Set **Budgets** to alert when monthly spend exceeds a defined threshold.
> Step 4 — Purchase **Savings Plans** based on Cost Explorer's recommendations to lock in discounts.

---

<a name="budgets"></a>
## 1. AWS Budgets

### What Problem Does It Solve?
AWS bills you for what you use — there are no automatic spending caps. Without alerts, a misconfigured resource, a runaway Lambda, or an unexpected traffic spike can generate a massive bill before anyone notices. AWS Budgets lets you **set spending thresholds and receive proactive alerts** before costs exceed acceptable limits — and even take **automated corrective actions**.

**Real-World Example:**
> A startup gives developers sandbox accounts with a strict $200/month budget. They set up a Budget that: alerts the developer at 80% ($160) via email, alerts the manager at 100% ($200) via SNS, and automatically applies an SCP at 110% ($220) that restricts new resource creation — preventing runaway spending before the month ends.

---

### Budget Types

| Budget Type | What It Tracks | Example |
|---|---|---|
| **Cost Budget** | Dollar amount spent | Alert when monthly spend > $500 |
| **Usage Budget** | Service usage quantity | Alert when EC2 hours > 1,000 hr/month |
| **Reservation Budget** | RI/Savings Plan utilization or coverage | Alert when RI coverage drops below 80% |
| **Savings Plans Budget** | Savings Plan utilization or coverage | Alert when SP utilization drops below 70% |

---

### Budget Filters

Budgets can be scoped to:
- **Service**: EC2 only, RDS only, all services
- **Linked account**: specific accounts within an AWS Organization
- **Tag**: cost allocation tags (e.g., `Environment = Production`)
- **Region**: us-east-1 only
- **Instance type**: m5.large only
- **Purchase type**: On-Demand, Reserved, Spot

---

### Budget Thresholds and Alerts

| Alert Type | Trigger | Notification |
|---|---|---|
| **Actual cost** | Real spend exceeds X% or $X of budget | Email, SNS |
| **Forecasted cost** | Projected spend will exceed budget by month end | Email, SNS |

- Up to **5 alerts per budget**.
- Alert recipients: up to **10 email addresses** per alert OR **1 SNS topic**.

**Example Alert Setup:**
```
Monthly Budget: $1,000

Alert 1: Actual cost > 50% ($500) → Email developers
Alert 2: Actual cost > 80% ($800) → Email finance team
Alert 3: Forecasted cost > 100% ($1,000) → Email + SNS
Alert 4: Actual cost > 100% ($1,000) → SNS → Lambda (take action)
Alert 5: Actual cost > 120% ($1,200) → SNS → Lambda (hard stop)
```

---

### Budget Actions (Automated Responses)

> **This is the most exam-tested AWS Budgets feature.**

When a budget threshold is breached, Budgets can **automatically execute actions** without human intervention:

| Action Type | Description | Example |
|---|---|---|
| **IAM Policy** | Attach/detach IAM policy to users/roles/groups | Attach DenyAllEC2 policy to developers when over budget |
| **SCP (Organizations)** | Attach SCP to OU or account | Apply `DenyNewResources` SCP to over-budget account |
| **EC2/RDS Actions** | Stop targeted EC2 or RDS instances | Stop non-production EC2 instances to halt costs |

**Budget Action Workflow:**
```
Budget threshold breached (e.g., 90% of $500 budget)
        ↓
Budget Action triggered automatically OR requires approval
        ↓
Action executed:
  ├── Attach IAM Policy: DenyEC2Launch to dev group
  ├── Apply SCP: RestrictServices to dev OU
  └── Stop EC2 instances tagged: Environment=Dev
        ↓
Notification sent via SNS/email
```

**Action Execution Options:**
- **Automatic**: Action runs immediately when threshold breached.
- **Manual approval**: Action queued; requires human to approve via console/API.

---

### AWS Budgets Pricing

| Feature | Cost |
|---|---|
| First 2 budgets | **Free** |
| Additional budgets | $0.02/day per budget (~$0.62/month) |
| Budget Actions | $0.10 per action execution |

> **Exam Tip:** The first **2 budgets are free** — a common exam question about which services have free tiers.

---

### Key Budgets Exam Scenarios

| Scenario | Answer |
|---|---|
| Alert when monthly EC2 spend exceeds $500 | AWS Budgets (Cost Budget) |
| Alert when forecasted spend will exceed budget before month end | AWS Budgets (Forecasted cost alert) |
| Automatically stop dev EC2 instances when budget is exceeded | AWS Budgets Actions (EC2 stop action) |
| Restrict new resource creation in dev account when over budget | AWS Budgets Actions (SCP attachment) |
| Alert when Reserved Instance coverage drops below 70% | AWS Budgets (Reservation Budget — coverage alert) |
| Alert when Savings Plan utilization is below 80% | AWS Budgets (Savings Plans Budget) |
| Notify finance team AND engineering team at different thresholds | AWS Budgets with multiple alert thresholds (up to 5 per budget) |

---

<a name="cur"></a>
## 2. AWS Cost and Usage Report (CUR)

### What Problem Does It Solve?
AWS Cost Explorer provides good visualizations but cannot answer highly specific questions like: "Show me the exact cost of each Lambda function invocation by tag across all accounts last quarter" or "Give me the cost per S3 bucket per day for the last 6 months." CUR provides the **most granular, complete billing data available** — every AWS charge broken down to the **hourly resource level** — exported to S3 for custom analysis.

**Real-World Example:**
> A Fortune 500 company has 150 AWS accounts. The finance team needs to allocate costs by business unit (tagged with `CostCenter`). CUR exports every charge from every account to a central S3 bucket, Glue crawls the data, and Athena queries like:
> ```sql
> SELECT resource_tags_user_costcenter, SUM(line_item_blended_cost) AS total_cost
> FROM cur_data.cur_report
> WHERE billing_period = '2026-01'
> GROUP BY resource_tags_user_costcenter
> ORDER BY total_cost DESC;
> ```
> This breaks down $2.3M in monthly spend by business unit in seconds.

---

### CUR Data Granularity Options

| Granularity | Rows Per Month (estimate) | Best For |
|---|---|---|
| **Monthly** | One row per service | High-level monthly summaries |
| **Daily** | One row per service per day | Daily trend analysis |
| **Hourly** | One row per resource per hour | Detailed cost attribution, anomaly detection |

> **Exam Tip:** Hourly granularity = most data, most cost to store/query, but enables per-resource per-hour cost attribution.

---

### CUR Data Format Options

| Format | Description | Best With |
|---|---|---|
| **CSV (GZIP/ZIP)** | Text-based; easy to inspect | Excel, basic analysis |
| **Apache Parquet** | Columnar format; highly compressed | Athena (10x cheaper to query), Redshift Spectrum |

> **Exam Tip:** Use **Parquet** format for CUR when querying with Athena — significantly reduces query cost and time.

---

### CUR Key Data Fields

| Field Category | Examples |
|---|---|
| **Identity** | `bill_payer_account_id`, `line_item_usage_account_id` |
| **Time** | `bill_billing_period_start_date`, `line_item_usage_start_date` |
| **Resource** | `line_item_resource_id` (specific EC2 instance, Lambda function, S3 bucket) |
| **Cost** | `line_item_unblended_cost`, `line_item_blended_cost`, `line_item_net_unblended_cost` |
| **Usage** | `line_item_usage_type`, `line_item_usage_amount` |
| **Tags** | `resource_tags_user_environment`, `resource_tags_user_project` |
| **Discounts** | `reservation_amortized_upfront_fee`, `savings_plan_savings_plan_rate` |
| **Service** | `product_service_code`, `product_region`, `product_instance_type` |

---

### Blended vs Unblended vs Amortized Cost

| Cost Type | Description | Use Case |
|---|---|---|
| **Unblended** | Actual charge rate per resource (actual invoice amount) | Billing reconciliation |
| **Blended** | Average rate across all accounts in org (blended RI/SP benefits) | Internal chargeback in orgs |
| **Amortized** | Spreads RI/SP upfront costs evenly over commitment period | True cost allocation with upfront commitments |
| **Net Unblended** | After applying all discounts including EDP | Enterprise discount programs |

---

### CUR Analysis Stack

```
AWS Services generate charges
        ↓
CUR Report (hourly, Parquet format)
        ↓
Delivered to S3 bucket (any account — central for multi-account)
        ↓
AWS Glue Crawler → updates Glue Data Catalog (table schema)
        ↓
Amazon Athena → SQL queries on CUR data
        ↓
Amazon QuickSight → visual dashboards
    OR
Custom BI tool (Tableau, PowerBI) → connect to Athena/Redshift
```

---

### CUR vs Cost Explorer vs Budgets

| Feature | CUR | Cost Explorer | Budgets |
|---|---|---|---|
| **Data granularity** | ✅ Per-resource per-hour | Daily/monthly aggregates | Aggregated |
| **Historical data** | Unlimited (you store in S3) | 13 months | Budget period only |
| **Custom analysis** | ✅ Any SQL query via Athena | Pre-built filters | Threshold-based only |
| **Cost tags analysis** | ✅ Any tag | ✅ (cost allocation tags) | ✅ (limited) |
| **Real-time alerts** | ❌ | ❌ | ✅ |
| **Forecasting** | ❌ | ✅ | ✅ |
| **Setup required** | ✅ S3 + Glue + Athena | None (built-in) | Minimal |
| **Cost** | S3 + Athena/Glue costs | Free (limited) / paid (advanced features) | Free (first 2) |

---

### CUR in Multi-Account Organizations

```
Management Account → enables CUR with "Include linked accounts"
        ↓
CUR exports all linked account charges to central S3 bucket
        ↓
Single query across ALL accounts:
  SELECT account_id, SUM(cost) FROM cur GROUP BY account_id
```

> **Exam Tip:** CUR must be **enabled in the management (payer) account** to get consolidated billing data across all linked accounts.

---

### Key CUR Exam Scenarios

| Scenario | Answer |
|---|---|
| Most granular billing data per resource per hour | AWS Cost and Usage Report (CUR) |
| Query billing data with SQL across all accounts | CUR + S3 + Athena |
| Visualize cost breakdown by custom tag (CostCenter) | CUR + Athena + QuickSight |
| Build a cost chargeback system for business units | CUR with resource tags + Athena |
| Cheapest way to query CUR data | Parquet format + Athena |
| Include all linked account charges in one report | CUR enabled in management account with linked accounts |

---

<a name="cost-explorer"></a>
## 3. AWS Cost Explorer

### What Problem Does It Solve?
Understanding where your AWS money is going — which services, which accounts, which regions, which teams — is essential for cost optimization. Cost Explorer provides an **interactive visual interface** to explore, analyze, and forecast AWS spending without any data pipeline setup.

**Real-World Example:**
> A team notices their AWS bill jumped 40% last month. They open Cost Explorer, filter to EC2 in the last 30 days, group by instance type, and immediately see that `p3.8xlarge` instances (GPU for ML) are running 24/7 instead of just during business hours — accounting for $15,000 of unexpected spend. They set up a Budget alert and schedule automatic stop during nights/weekends.

---

### Cost Explorer Key Features

| Feature | Description |
|---|---|
| **Time range** | Last 1, 3, 6, 12 months or custom range (up to **13 months** of historical data) |
| **Granularity** | Monthly, daily, hourly (hourly requires extra cost) |
| **Grouping** | Service, account, region, AZ, instance type, tag, purchase option, API operation |
| **Filtering** | Any combination of service, account, region, tag, usage type, etc. |
| **Forecasting** | Project future costs up to **12 months** ahead based on historical patterns |
| **RI Recommendations** | Suggests optimal Reserved Instance purchases based on usage |
| **Savings Plan Recommendations** | Suggests Savings Plan type and commitment level |
| **Cost Anomaly Detection** | ML-based detection of unusual spending |

---

### Cost Explorer Views

#### Saved Reports (Built-In)
| Report | Description |
|---|---|
| **Monthly costs by service** | High-level service breakdown |
| **Monthly costs by linked account** | Per-account spend in organization |
| **Daily costs by service** | Daily granularity service view |
| **Monthly EC2 running hours** | Instance hours consumed |
| **RI utilization** | How much of purchased RIs are being used |
| **RI coverage** | % of hours covered by Reserved Instances |
| **Savings Plans utilization** | How much committed spend is being used |
| **Savings Plans coverage** | % of usage covered by Savings Plans |

---

### Cost Explorer Forecasting

- Uses **machine learning** based on your actual historical usage patterns.
- Forecast horizon: up to **12 months** forward.
- Includes **confidence interval** (upper/lower bounds).
- Used in Budgets for "forecasted cost" alerts.

```
Historical spend (last 12 months): $5,000 → $5,500 → $6,000 (growing trend)
        ↓
Cost Explorer Forecast:
  Next month: $6,500 (±$400)
  3 months: $7,500 (±$800)
  12 months: $11,000 (±$2,000)
```

---

### Cost Anomaly Detection

- **ML-powered** service that monitors spending and alerts on unexpected increases.
- Defines normal baseline from historical patterns.
- Alerts via SNS/email when anomaly detected.
- Can monitor: AWS services, cost categories, member accounts, cost allocation tags.
- **No thresholds to set** — fully automated ML detection.

**Example:**
> Normal DynamoDB spend: $200/month. Cost Anomaly Detection alerts at 3am when DynamoDB spend spikes to $2,000 due to a runaway scan operation — before it becomes $20,000 by month end.

---

### RI and Savings Plan Recommendations

Cost Explorer analyzes your last 7, 30, or 60 days of usage and recommends:

| Recommendation Type | Analysis |
|---|---|
| **RI purchase recommendations** | Which RIs to buy, expected savings, break-even point |
| **Savings Plan purchase recommendations** | Compute vs EC2 SP, commitment amount, estimated savings |
| **Rightsizing recommendations** | Under-utilized EC2 instances that should be downsized (integrates with Compute Optimizer) |

---

### Cost Explorer API

- Query cost data **programmatically** for custom dashboards and automation.
- `GetCostAndUsage` — retrieve cost/usage data with filters and groups.
- `GetForecast` — retrieve cost forecast for a future period.
- `GetReservationUtilization` — RI utilization data.
- **Cost**: $0.01 per API request.

---

### Key Cost Explorer Exam Scenarios

| Scenario | Answer |
|---|---|
| Visualize which service is driving AWS costs | AWS Cost Explorer |
| Forecast next month's AWS spend | Cost Explorer Forecasting |
| Identify EC2 instances to rightsize | Cost Explorer Rightsizing Recommendations (+ Compute Optimizer) |
| Get RI purchase recommendations | Cost Explorer RI recommendations |
| Alert when unexpected cost spike occurs | Cost Anomaly Detection |
| See 6 months of daily cost by linked account | Cost Explorer (filter by account, daily granularity) |
| Programmatically query billing data | Cost Explorer API (GetCostAndUsage) |

---

<a name="savings-plans"></a>
## 4. AWS Savings Plans

### What Problem Does It Solve?
On-Demand pricing is the most flexible but most expensive way to run AWS workloads. Reserved Instances provide discounts but are rigid — tied to specific instance types, regions, and OS. Savings Plans provide **significant discounts (up to 72%)** by committing to a **minimum dollar amount of spend per hour** over 1 or 3 years — with far more flexibility than Reserved Instances.

**Real-World Example:**
> A company consistently spends $5,000/month on EC2 On-Demand. By committing to a $3.00/hour Compute Savings Plan (1-year, no upfront) = $2,190/month commitment. AWS automatically applies up to 66% discount on their EC2, Lambda, and Fargate usage until the $3.00/hour is consumed — saving ~$25,000 per year with minimal flexibility loss.

---

### Savings Plans Types

#### 1. Compute Savings Plans
| Property | Details |
|---|---|
| **Discount** | Up to **66%** off On-Demand |
| **Flexibility** | Applies to **any** EC2 instance family, size, region, OS, tenancy **AND** Lambda **AND** Fargate |
| **Automatic application** | AWS automatically applies to cheapest eligible usage first |
| **Commitment** | $/hour for 1 or 3 years |

> **Most flexible Savings Plan** — works across instance families, regions, and even changes from EC2 to Fargate or Lambda.

#### 2. EC2 Instance Savings Plans
| Property | Details |
|---|---|
| **Discount** | Up to **72%** off On-Demand (deepest discount) |
| **Flexibility** | Locked to **specific instance family** in a **specific region** (e.g., M5 in us-east-1) |
| **Flexibility within** | Any size (m5.large, m5.xlarge), any OS, any tenancy within that family |
| **No Lambda/Fargate** | ❌ Only EC2 |

> **Highest discount** but least flexible — still more flexible than Standard RIs.

#### 3. SageMaker Savings Plans
| Property | Details |
|---|---|
| **Discount** | Up to **64%** off On-Demand |
| **Covers** | SageMaker instance usage (training, hosting, notebooks) |
| **Flexibility** | Any SageMaker instance type, region, component |

---

### Savings Plans vs Reserved Instances (Critical Comparison)

| Feature | Savings Plans | Reserved Instances |
|---|---|---|
| **Commitment unit** | $/hour spend | Specific instance type |
| **Instance flexibility** | ✅ Any family (Compute SP) | ❌ Locked to instance type (Standard RI) |
| **Region flexibility** | ✅ Any region (Compute SP) | ❌ Region-specific |
| **OS flexibility** | ✅ Any OS | ❌ Locked to OS |
| **Covers Lambda/Fargate** | ✅ (Compute SP) | ❌ |
| **Covers SageMaker** | ✅ (SageMaker SP) | ❌ |
| **Max discount** | 72% (EC2 Instance SP) | 72% (Standard RI, 3-year all-upfront) |
| **Marketplace sellable** | ❌ | ✅ Standard RIs |
| **Convertible option** | N/A | ✅ Convertible RIs (change attributes) |
| **Recommended by** | Cost Explorer | Cost Explorer |

---

### Payment Options

| Option | Payment | Discount Level |
|---|---|---|
| **All Upfront** | Pay entire 1 or 3 years now | Highest discount |
| **Partial Upfront** | Pay part now, rest monthly | Medium discount |
| **No Upfront** | Monthly payments only | Lowest discount (but still much less than On-Demand) |

**Term Options:** 1-year (lower discount) or 3-year (higher discount).

---

### How Savings Plans Work (Application Logic)

```
Savings Plan purchased: $3.00/hour Compute SP (1-year, no-upfront)

Each hour, AWS calculates usage at SP rate:
  EC2 m5.large (On-Demand: $0.096/hr, SP rate: $0.061/hr) → 30 instances × $0.061 = $1.83
  Lambda invocations at SP rate: $0.52
  Fargate tasks at SP rate: $0.50
  Total applied: $2.85/hr → within $3.00 commitment ✓

Any usage above $3.00/hr falls back to On-Demand pricing.
```

> **Exam Tip:** AWS automatically applies Savings Plans to **your most expensive eligible usage first** to maximize your savings.

---

### Savings Plans for Lambda

- Compute Savings Plans cover Lambda **duration charges**.
- Lambda at Savings Plan rate: up to **17% discount** vs On-Demand.
- Applied automatically — no code changes or configuration needed.

---

### Savings Plans Utilization and Coverage

| Metric | Description | Monitored In |
|---|---|---|
| **Utilization** | % of your committed $/hr actually consumed | Cost Explorer, Budgets |
| **Coverage** | % of eligible spend covered by Savings Plans | Cost Explorer, Budgets |

**Utilization example:**
> Committed $3.00/hr. Actual eligible spend = $2.50/hr. Utilization = 83%. (Wasted $0.50/hr on unused commitment.)

**Coverage example:**
> Total EC2 spend = $5.00/hr. Savings Plan covers $3.00/hr. Coverage = 60%. (40% still at On-Demand rates.)

---

### Purchasing Savings Plans

1. **Review** Cost Explorer Savings Plan recommendations.
2. **Choose** plan type (Compute, EC2 Instance, SageMaker).
3. **Set** commitment amount ($/hour) based on recommendation.
4. **Choose** term (1 or 3 years) and payment option.
5. AWS immediately **starts applying** discounts.
6. **Monitor** utilization and coverage in Cost Explorer.

---

### Key Savings Plans Exam Scenarios

| Scenario | Answer |
|---|---|
| Maximize discount while keeping flexibility to change instance types | Compute Savings Plans |
| Highest discount for a specific instance family you'll always use | EC2 Instance Savings Plans (72% discount) |
| Discount on Lambda and Fargate as well as EC2 | Compute Savings Plans |
| Discount on SageMaker training jobs | SageMaker Savings Plans |
| Receive savings plan purchase recommendations | AWS Cost Explorer |
| Alert when Savings Plan utilization drops below 70% | AWS Budgets (Savings Plans Budget) |
| Flexible commitment covering any instance family, any region | Compute Savings Plans |

---

<a name="related"></a>
## 5. Related Cost Optimization Services

### Reserved Instances (RIs)

Although not one of the listed services, RIs are heavily tested and closely related to Savings Plans:

| RI Type | Flexibility | Discount |
|---|---|---|
| **Standard RI** | Locked to instance family, region, OS, tenancy | Up to 72% |
| **Convertible RI** | Can exchange for different RI attributes (family, OS, tenancy) | Up to 66% |

**RI Scope:**
- **Regional**: applies to any AZ, any size in the family within the region.
- **Zonal**: applies to specific AZ and specific instance size (capacity reservation).

**RI Payment Options:** All Upfront, Partial Upfront, No Upfront (same as Savings Plans).

---

### AWS Compute Optimizer

- **ML-based rightsizing** recommendations for EC2, EBS, Lambda, ECS, ASG.
- Identifies over-provisioned and under-provisioned resources.
- Separate from Cost Explorer rightsizing (uses CloudWatch metrics for deeper analysis).
- Free to enable; enhanced recommendations require CloudWatch agent for memory metrics.

---

### AWS Trusted Advisor

- **Best practice checks** including cost optimization.
- Key cost checks:
  - Idle EC2 instances (< 10% CPU for 14 days)
  - Underutilized EBS volumes
  - Unassociated Elastic IPs
  - Idle RDS instances
  - Low utilization EC2 instances
- Full checks require **Business or Enterprise Support plan**.

---

### Cost Allocation Tags

- **Key-value tags** applied to AWS resources for cost tracking.
- Must be **activated** in the billing console to appear in CUR and Cost Explorer.
- Types:
  - **AWS-generated tags**: `aws:createdBy`, `aws:cloudformation:stack-name`
  - **User-defined tags**: any custom tags you apply (e.g., `Environment`, `Project`, `CostCenter`)
- Best practice: enforce tagging via SCP or Config rules.

---

<a name="comparisons"></a>
## Master Comparisons

### All Four Services: When to Use Each

| If the Question Is About... | Use This Service |
|---|---|
| "Alert me when I'm about to exceed my budget" | AWS Budgets |
| "Automatically stop resources when budget exceeded" | AWS Budgets Actions |
| "Show me where my money is going" | Cost Explorer |
| "Forecast my costs for the next 6 months" | Cost Explorer |
| "Detect unexpected cost spikes automatically" | Cost Anomaly Detection (via Cost Explorer) |
| "Get the most granular billing data for custom analysis" | CUR (Cost and Usage Report) |
| "Build a cost chargeback system by business unit" | CUR + Athena + QuickSight |
| "Query billing data with SQL" | CUR + Athena |
| "Reduce costs with a commitment discount" | Savings Plans |
| "Recommend which Savings Plans to buy" | Cost Explorer |
| "Monitor my Savings Plan is being used efficiently" | AWS Budgets (SP Budget) + Cost Explorer |

---

### Cost Explorer vs CUR vs Budgets

| Dimension | Cost Explorer | CUR | Budgets |
|---|---|---|---|
| **Purpose** | Analyze + visualize | Export raw data | Alert + act |
| **Data detail** | Aggregated | Per-resource per-hour | Aggregated |
| **History** | 13 months | Unlimited (S3) | Current budget period |
| **Setup** | Zero | S3 + Glue + Athena | Minimal |
| **Forecasting** | ✅ | ❌ | ✅ (based on CE) |
| **Alerting** | ❌ (Cost Anomaly) | ❌ | ✅ |
| **Automation** | ❌ | ❌ | ✅ (Budget Actions) |
| **Cost allocation tags** | ✅ | ✅ (richest) | ✅ (limited) |
| **API access** | ✅ ($0.01/req) | Via Athena/S3 | ✅ |

---

### Savings Plans vs Reserved Instances

| Dimension | Savings Plans | Reserved Instances |
|---|---|---|
| **Commitment unit** | $/hour | Specific instance config |
| **Covers Lambda** | ✅ (Compute SP) | ❌ |
| **Covers Fargate** | ✅ (Compute SP) | ❌ |
| **Cross-region** | ✅ (Compute SP) | ❌ |
| **Marketplace** | ❌ Cannot sell | ✅ Standard RI can be sold |
| **Capacity reservation** | ❌ | ✅ Zonal RI reserves capacity |
| **Convertible option** | N/A | ✅ Convertible RI |
| **Recommendation** | Cost Explorer | Cost Explorer |
| **Best for** | Modern workloads, Lambda, flexible EC2 | Dedicated capacity reservation, Oracle BYOL |

> **Exam Rule:** For most new workloads → **Savings Plans** (more flexible). For guaranteed capacity reservation → **Zonal Reserved Instances**. For Oracle BYOL requiring Dedicated Host → **Dedicated Host Reservations**.

---

<a name="patterns"></a>
## Architecture Patterns to Know

### Pattern 1: Enterprise Cost Governance Framework
```
AWS Organizations (100 accounts)
        │
        ├── CUR enabled in Management Account
        │   └── Exports to Central S3 Bucket (all accounts)
        │
        ├── AWS Budgets per account/OU
        │   ├── Alert at 80% → Email finance + engineering
        │   ├── Alert at 100% → SNS → Lambda → Notify Slack
        │   └── Action at 110% → Attach DenyResourceCreation IAM policy
        │
        ├── Cost Explorer (management account)
        │   ├── Monitor trends across all accounts
        │   └── Cost Anomaly Detection per service + account
        │
        └── Savings Plans (purchased centrally)
            └── Benefits shared across all org accounts automatically
```

### Pattern 2: CUR Analytics Pipeline
```
AWS charges generated hourly
        ↓
CUR Report (Parquet, hourly, all linked accounts)
        ↓
Amazon S3 (central cost data bucket)
        ↓
AWS Glue Crawler (runs nightly → updates table schema)
        ↓
Amazon Athena (SQL queries)
  ├── Cost by team: SELECT tag_costcenter, SUM(cost) ...
  ├── Anomaly: WHERE date = today AND cost > avg * 3
  └── Forecast: trend analysis queries
        ↓
Amazon QuickSight Dashboard
  ├── Executive: Monthly spend by BU
  ├── Engineering: Daily spend by service
  └── Finance: Chargeback report by cost center
```

### Pattern 3: FinOps Optimization Loop
```
Week 1 — Discover:
  Cost Explorer → identify top 5 cost drivers
  CUR + Athena → find wasteful per-resource spend

Week 2 — Analyze:
  Compute Optimizer → rightsizing recommendations
  Cost Explorer → RI/SP recommendations
  Trusted Advisor → idle resource alerts

Week 3 — Optimize:
  Purchase Savings Plans (based on CE recommendations)
  Stop/terminate idle resources
  Rightsize oversized instances

Week 4 — Govern:
  AWS Budgets → set alerts for each team
  Budget Actions → auto-stop dev resources at night
  Cost Allocation Tags → enforce via Config rules
  Monitor SP utilization/coverage via Budgets + CE
```

### Pattern 4: Automated Cost Control for Dev Accounts
```
Developer starts expensive GPU instance (p3.8xlarge)
        ↓
CUR + CloudWatch (EC2 running + daily cost tracked)
        ↓
AWS Budget: "Dev account > $100/day"
        ↓
Budget threshold breached
        ↓
Budget Action 1: Email developer + manager
Budget Action 2: SNS → Lambda → tag instance for review
Budget Action 3 (at 120%): EC2 Stop action on tagged instances
        ↓
Instance stopped; developer notified to use smaller instance
```

### Pattern 5: Multi-Account Savings Plan Strategy
```
Management Account purchases Savings Plans:
  └── $100/hr Compute Savings Plan (3-year, all-upfront)

Automatic benefit sharing:
  ├── Account A (EC2 heavy): $45/hr covered at SP rate
  ├── Account B (Fargate): $30/hr covered at SP rate
  ├── Account C (Lambda): $15/hr covered at SP rate
  └── Remaining $10/hr unused → Cost Explorer alerts
            ↓
AWS Organizations consolidated billing shares benefits automatically
(No configuration needed — works by default within organization)
```

---

<a name="exam-tips"></a>
## Exam Tips Summary

### AWS Budgets
- ✅ 4 budget types: Cost, Usage, Reservation, Savings Plans
- ✅ 2 alert types: Actual (real spend) and Forecasted (projected spend)
- ✅ Up to 5 alerts per budget; up to 10 email recipients per alert + 1 SNS topic
- ✅ **Budget Actions** = automated response: IAM policy attach, SCP apply, EC2/RDS stop
- ✅ Budget Actions: Automatic (instant) or Manual (requires approval)
- ✅ **First 2 budgets are FREE**; $0.02/day per additional budget
- ✅ "Alert when forecasted spend will exceed budget" → Budgets forecasted alert
- ✅ "Automatically stop dev resources when over budget" → Budget Actions (EC2 stop)

### AWS Cost and Usage Report (CUR)
- ✅ Most granular billing data: per-resource, per-hour
- ✅ Export destination: **Amazon S3 only**
- ✅ Formats: CSV (GZIP/ZIP) or **Parquet** (use Parquet for Athena — cheaper + faster)
- ✅ Analysis stack: CUR → S3 → Glue Crawler → Athena → QuickSight
- ✅ Enable in management account to include all linked accounts
- ✅ Blended vs Unblended vs Amortized cost: know the difference
- ✅ CUR = raw data; Cost Explorer = visual analysis; Budgets = alerts

### AWS Cost Explorer
- ✅ Historical data: up to **13 months**
- ✅ Forecast: up to **12 months** ahead (ML-based)
- ✅ Granularity: Monthly, Daily (free), Hourly (extra cost)
- ✅ **Cost Anomaly Detection** = ML-based automatic spending spike detection (no thresholds)
- ✅ RI and Savings Plan purchase recommendations built-in
- ✅ Rightsizing recommendations (integrates with Compute Optimizer)
- ✅ API: $0.01 per GetCostAndUsage request
- ✅ "Visualize costs, identify trends, get SP/RI recommendations" → Cost Explorer

### Savings Plans
- ✅ 3 types: **Compute SP** (most flexible, 66%), **EC2 Instance SP** (most discount, 72%), **SageMaker SP** (64%)
- ✅ Compute SP covers: EC2 (any family/region), Lambda, Fargate
- ✅ EC2 Instance SP: locked to instance family + region (not size/OS/tenancy)
- ✅ Savings Plans ≠ RIs: SP = $/hour commitment; RI = specific instance type commitment
- ✅ Savings Plans **cannot be sold** on Marketplace (RIs can)
- ✅ Savings Plans do **NOT** reserve capacity (Zonal RIs do)
- ✅ Payment: All Upfront > Partial > No Upfront (discount order)
- ✅ Term: 3-year > 1-year (discount order)
- ✅ AWS applies SP to **most expensive eligible usage first** automatically
- ✅ Monitor via Cost Explorer (utilization/coverage) + Budgets (SP budget type)
- ✅ "Need to discount Lambda AND EC2 AND Fargate" → Compute Savings Plans
- ✅ "Maximum discount, always using same instance family" → EC2 Instance Savings Plans

### Cost Allocation Tags
- ✅ Must be **activated** in Billing console before appearing in CUR and Cost Explorer
- ✅ User-defined tags (custom) + AWS-generated tags
- ✅ Enforce tagging via SCPs or AWS Config rules
- ✅ Cost allocation tags are the foundation of chargeback/showback systems

### General Cost Management Rules
- ✅ "Alert me" → AWS Budgets
- ✅ "Show me / analyze" → Cost Explorer
- ✅ "Per-resource data / SQL queries" → CUR + Athena
- ✅ "Reduce cost with commitment" → Savings Plans (or RIs)
- ✅ "Rightsize instances" → Compute Optimizer or Cost Explorer
- ✅ "Best practice checks (idle resources)" → Trusted Advisor
- ✅ SP benefits shared across org automatically via consolidated billing
- ✅ Cost Anomaly Detection requires no thresholds — fully automated ML

---
---

<a name="transform"></a>
# PART 16 — AWS Transform: Assessments, Windows, Mainframe, VMware & Custom Code

# AWS Exam Study Guide: AWS Transform
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Source:** https://aws.amazon.com/transform/ (verified May 2026)

---

## ⚠️ Important Exam Note

AWS Transform is a **relatively new service** (launched ~2024–2025) and is an **emerging topic** on AWS certification exams. It is more likely to appear on the **SAP-C02 (Professional)** level given its enterprise modernization focus. It may appear in scenario-based questions involving:
- Migration strategy selection (the 7 Rs)
- Mainframe or Windows/.NET modernization at scale
- VMware to AWS migration with AI-assisted planning
- Code modernization and tech debt reduction

---

## What Is AWS Transform?

AWS Transform is an **agentic AI-powered enterprise IT transformation workbench** that accelerates cloud migration, legacy application modernization, and technical debt reduction using specialized AI agents. It combines:

- **Multi-agent orchestration** — dozens of specialized AI agents working together
- **Agentic AI** — goal-driven automation that adapts plans dynamically
- **Continuous learning** — agents improve from every execution and human feedback
- **Collaborative workspaces** — cross-functional teams track and manage transformation in real time

**Real-World Impact (as of 2026):**
- Migrated tens of thousands of VMs
- Processed over **4.5 billion lines of code**
- Saved **1.69 million hours** of manual effort (equivalent to ~810 developer years)
- Up to **5x faster** full-stack Windows modernization

---

## The Problem It Solves

### Traditional Modernization Pain Points

| Pain Point | How AWS Transform Addresses It |
|---|---|
| Manual assessment takes weeks/months | Automated discovery and analysis in minutes/hours |
| Dependency mapping is complex and error-prone | AI-powered dependency analysis and wave planning |
| COBOL/mainframe expertise is scarce | AI agents encode 20 years of migration expertise |
| .NET Framework migration requires specialized skills | Automated porting with AI-led error remediation |
| Inconsistent results across large codebases | Repeatable, continually improving transformations |
| Migration projects run years behind schedule | Parallel transformation of hundreds of apps simultaneously |
| Legacy licensing costs remain high | Eliminates Windows/SQL Server licensing by moving to Linux/Aurora |

---

## AWS Transform Use Cases / Types

AWS Transform has **five main transformation areas**, each with a dedicated specialized agent:

---

### 1. AWS Transform for Assessments

#### What It Does
Analyzes your IT environment to generate a **data-driven business case** for migration — including TCO comparisons, EC2 right-sizing recommendations, and cost projections — without needing a detailed manual inventory process.

#### How It Works
```
Upload server inventory (RVTools, AWS Transform discovery tool,
Migration Evaluator collector, or third-party tools)
            ↓
Specify target AWS Region
            ↓
AWS Transform analyzes inventory → maps to optimal EC2 instances
            ↓
Business case generated:
  ├── TCO comparison (on-prem vs AWS)
  ├── Multiple purchase commitment scenarios (On-Demand, Reserved)
  ├── OS licensing options (BYOL vs license-included)
  ├── Tenancy options (dedicated vs shared)
  └── Next step recommendations
            ↓
Download PDF or ask follow-up questions via built-in AI chat
```

#### Supported Data Sources
- **RVTools** exports (CSV/Excel with VMware configuration data)
- **AWS Transform discovery tool** (OVA appliance deployed in vCenter)
- **AWS Migration Evaluator** agentless collector
- **AWS Migration Portfolio Assessment (MPA)** exports
- Third-party tools: Cloudamize, modelizeIT

#### AWS Transform Assessments vs AWS Migration Evaluator

| Feature | AWS Transform Assessments | AWS Migration Evaluator |
|---|---|---|
| **Type** | Self-service, fast | Expert-led, comprehensive |
| **Focus** | x86 server migration to EC2 | Broad (storage, sustainability, SQL Server, etc.) |
| **Guidance** | AI chat assistant | AWS Solutions Architects |
| **Time to result** | Minutes to hours | Days to weeks |
| **Best for** | Quick TCO estimates, initial planning | Thorough enterprise migration planning |

---

### 2. AWS Transform for Windows (Full-Stack Windows Modernization)

#### What It Does
Accelerates full-stack Windows modernization — converting .NET Framework applications to cross-platform .NET (Linux) and migrating Microsoft SQL Server databases to Amazon Aurora PostgreSQL — up to **5x faster** than manual methods, reducing operating costs by **up to 70%**.

#### Two Main Components

##### Component A: .NET Framework → Cross-Platform .NET
```
Source code in GitHub / GitLab / Azure Repos / Bitbucket
            ↓ (connects via AWS CodeConnections — requires IT admin approval)
Analysis Phase:
  ├── Identify supported project types
  ├── Map repository dependencies
  ├── Identify private packages and third-party libraries
  └── Generate transformation plan (ordered by dependency + modification date)
            ↓
Human Review → Approve plan
            ↓
Transformation (in network-isolated environment):
  ├── Convert .NET Framework 3.5+ / Core 3.1 / .NET 5/6/7 → .NET 8 (LTS) or .NET 10
  ├── Port MVC, Web API, WCF, Class Libraries, Console Apps
  ├── Convert Razor Views, Web Forms → Blazor on ASP.NET Core
  ├── Port WinForms, WPF, Xamarin → cross-platform .NET
  ├── Entity Framework + ADO.NET → Aurora PostgreSQL compatible
  └── AI-led auto-remediation loop for build errors
            ↓
Transformed code committed to new branch in your repository
            ↓
Deploy as container on Amazon EC2 or Amazon ECS
```

##### Component B: SQL Server → Aurora PostgreSQL
```
SQL Server on EC2 or RDS
            ↓
Discovery: schemas, stored procedures, database dependencies in source code
            ↓
Wave planning: group databases + apps by dependency relationships
            ↓
3-Step Transformation:
  Step 1 — Schema Conversion:
    SQL Server schema → Aurora PostgreSQL schema
    AI auto-remediation for conversion failures
    Stored procedures converted to PostgreSQL-compatible

  Step 2 — Data Migration:
    Data migrated from SQL Server to Aurora PostgreSQL clusters
    Uses AWS DMS under the hood
    Migration issues reported for manual resolution

  Step 3 — Code Transformation:
    Connection strings updated → PostgreSQL target
    Embedded SQL ported for PostgreSQL compatibility
    Entity Framework / ADO.NET updated
    Changes committed to new repository branch
```

#### Supported .NET Project Types
Console apps, Class libraries, Web APIs, WCF Services, MVC, SPA, ASP.NET Web Forms, WinForms, WPF, Xamarin, VB.NET, xUnit/NUnit/MSTest unit tests

#### Key Properties
- **Code ownership**: You own all transformed code; AWS Transform does not retain code after job completion
- **Security**: Customer-managed KMS keys for encrypting code in the transformation environment
- **Human-in-the-loop (HITL)**: Developers can review and refine in Visual Studio IDE extension
- **Progress tracking**: Interactive AI chat + detailed worklogs in the web console
- **Notifications**: Email with deep links to transformed repositories on completion

---

### 3. AWS Transform for Mainframe

#### What It Does
Accelerates mainframe application modernization from **years to months** using an AI agent that orchestrates analysis, documentation generation, business logic extraction, monolith decomposition, code transformation, and automated testing.

#### Supported Modernization Patterns

| Pattern | Description | Output |
|---|---|---|
| **Refactor** | Convert COBOL to modern Java | Functional-equivalent Java apps on AWS |
| **Reimagine** | Transform to cloud-native architecture | Modern cloud-native microservices |

#### Mainframe Modernization Flow (Refactor Pattern)
```
COBOL Mainframe Codebase (uploaded as zip to S3)
  ├── COBOL programs
  ├── Copybooks
  ├── JCL (Job Control Language)
  ├── Procedures + parameter cards
  ├── DB2 definitions
  └── CICS / IMS TM / CSD files (optional)
            ↓
Analyze Step (required):
  ├── Classify file types
  ├── Metrics: LOC, effective LOC, cyclomatic complexity
  ├── Identify missing/duplicate files
  └── Generate dependency mapping
            ↓
Decompose Step:
  ├── Break monolith into business-aligned domains
  ├── Identify low-utilization components (candidates for retirement)
  └── Generate prioritized modernization waves
            ↓
Documentation Step:
  ├── Extract business rules and logic from COBOL
  ├── Generate functional specifications
  └── Make knowledge accessible to non-mainframe developers
            ↓
Transform Step (COBOL → Java):
  └── Automated refactoring with functional equivalence preservation
            ↓
Test Step:
  ├── Automated test plan generation
  ├── Test data collection scripts
  └── Continuous regression testing environment
            ↓
Post-Transformation:
  └── IaC templates (CloudFormation, CDK, Terraform) for deployment
```

#### Key Capabilities
- **Business logic extraction**: Extracts business rules from COBOL — makes mainframe knowledge accessible to business stakeholders without mainframe expertise
- **Modular approach**: Use any individual phase (analyze, decompose, document, transform, test) independently
- **Cyclomatic complexity analysis**: Understand code complexity before committing to transformation
- **Reforge step**: Improve Java code readability and maintainability post-transformation

---

### 4. AWS Transform for VMware

#### What It Does
AI-powered end-to-end VMware to Amazon EC2 migration — automating the entire lifecycle from discovery through wave planning, network conversion, and cutover. Migrations that previously took **years now complete in months**.

#### Supported Migration Job Plans

| Job Plan | Scope |
|---|---|
| **End-to-end** | Discovery → wave planning → VPC network generation → server rehost |
| **Network migration only** | Generate and deploy VPC configurations only |
| **Network + server migration** | Skip discovery; configure VPC + migrate servers |
| **Discovery + server migration** | Discover + plan + migrate; skip network configuration |

#### VMware Migration Flow
```
Source Environment (VMware, bare metal, Hyper-V)
            ↓
Discovery (via one of):
  ├── AWS Transform discovery tool (OVA in vCenter)
  ├── RVTools CSV/Excel exports
  ├── Cloudamize / Matilda Cloud / ModelizeIT exports
  └── Manual inventory upload
            ↓
Analysis + Planning:
  ├── Application dependency mapping
  ├── Logical application group identification (servers that must migrate together)
  ├── Right-sized EC2 instance recommendations
  └── Business case with TCO projections
            ↓
Network Conversion:
  ├── Convert on-prem network topology → VPC configuration
  ├── Supports Cisco ACI, Palo Alto, Fortigate as network sources
  ├── Generate IaC (CloudFormation / Terraform)
  └── Automated VPC deployment
            ↓
Rehost (migrate servers to EC2):
  └── Uses AWS Application Migration Service (MGN) under the hood
            ↓
Target: Amazon EC2 in your chosen AWS Region
```

#### Key Properties
- Handles **complex multi-tier applications** with 500+ VMs
- Supports **multiple target accounts** in one migration
- Private connectivity supported: **Direct Connect** or **Site-to-Site VPN** (no public internet for data replication)
- Supports rehost to **EC2** (NOT automated migration to VMware Cloud on AWS / Amazon EVS)
- Encryption: TLS 1.2+ in transit; KMS at rest (customer-managed keys supported)

#### Available Workspace Regions
US East (N. Virginia), Asia Pacific (Mumbai, Seoul, Sydney, Tokyo), Canada (Central), Europe (Frankfurt, London)

---

### 5. AWS Transform for Custom Code

#### What It Does
Performs **large-scale custom code modernization** across any codebase — from version upgrades and runtime migrations to complex framework changes and organization-specific transformations — using agentic AI that learns and improves with every execution.

#### Four Core Components

| Component | Description |
|---|---|
| **Natural language transformation definition** | Define transformations via chat + documentation + code samples |
| **Transformation execution** | Apply transformations consistently across hundreds of codebases in parallel |
| **Continual learning** | Automatically improves from execution data and developer feedback |
| **AWS-managed transformations** | Pre-built, vetted transformations for common scenarios |

#### AWS-Managed (Out-of-the-Box) Transformations

| Transformation | Details |
|---|---|
| **Java 8 → 17** | Gradle and Maven support |
| **Node.js 12 → 22** | Including Lambda environments |
| **Python → 3.11/3.12/3.13** | Standard and Lambda runtimes |
| **AWS SDK v1 → v2** | Migrate legacy SDK usage |

#### Custom Transformation Examples
- Java 8 → 17 with organization-specific internal library rules
- x86 → AWS Graviton (ARM) runtime migrations
- Spring Boot version upgrades
- Angular → React framework migration
- Observability instrumentation (add structured logging, tracing)
- Complex language translations (e.g., COBOL → Python)
- API framework migrations

#### Custom Transformation Lifecycle

```
Phase 1 — Define (optional for custom; skip for AWS-managed):
  Provide: natural language prompt + reference docs + code samples
  AI generates transformation definition
  Iteratively refine via chat
  Test on sample codebases

Phase 2 — Pilot:
  Apply transformation to a subset of codebases
  Validate results and estimate cost/time
  Continual learning improves quality during pilot

Phase 3 — Scaled Execution:
  Bulk execution across all target codebases via CLI or web console
  Track progress in real time

Phase 4 — Monitor + Review:
  Review AI-generated "knowledge items" from continual learning
  Approve or disable specific learnings
  Enable or disable improvements for future runs
```

#### Access Methods
- **AWS Transform CLI**: Local development, CI/CD pipeline integration, scriptable
- **AWS Transform Web App**: Large-scale campaigns, real-time progress tracking
- **IDE integration**: VS Code, Kiro IDE (via Kiro plugin for human-in-the-loop work)
- **MCP (Model Context Protocol)**: Invoke from orchestrators, IDEs, coding environments (Claude, Cursor, Codex, etc.)

---

## AWS Transform Architecture (How It All Fits Together)

```
Access Layer:
  ├── Web Console (console.aws.amazon.com/transform)
  ├── AWS Transform CLI
  ├── IDE Plugins (VS Code, Kiro)
  └── MCP Integration (Claude, Cursor, Codex)
            ↓
AWS Transform Workbench:
  ├── Shared workspace (cross-functional team collaboration)
  ├── Natural language chat interface
  ├── Worklogs (audit trail of all agent actions)
  └── Progress tracking dashboard
            ↓
Agent Orchestration Layer:
  ├── Specialized task agents (network generation, COBOL analysis, .NET porting, etc.)
  ├── Dynamic job plan generation (adapts to your specific goals)
  └── Human-in-the-loop (HITL) checkpoints
            ↓
Underlying AWS Services:
  ├── AWS Application Migration Service (MGN) — for VMware rehost
  ├── AWS DMS — for SQL Server → Aurora PostgreSQL data migration
  ├── AWS CodeConnections — for secure source code access
  ├── Amazon Bedrock — foundation models powering agents
  ├── Amazon S3 — temporary code storage during transformation
  └── KMS — encryption of code and data at rest
```

---

## Key Differentiators vs Traditional Migration Tools

| Dimension | Traditional Tools | AWS Transform |
|---|---|---|
| **Planning** | Manual spreadsheets, weeks of work | AI-generated wave plans in minutes |
| **Dependency mapping** | Manual interviews + diagrams | Automated from code analysis and network data |
| **Code conversion** | Manual developer effort (COBOL → Java) | Automated with AI agents, HITL oversight |
| **Learning** | None — same quality regardless of iterations | Continual learning improves each run |
| **Scale** | One application at a time | Hundreds in parallel |
| **Expertise required** | Deep mainframe/COBOL expertise | AI agents encode the expertise |
| **Error handling** | Manual debugging | AI auto-remediation loop |
| **Progress visibility** | Status meetings and emails | Real-time chat + worklogs |

---

## AWS Transform vs Related AWS Migration Services

| Service | Relationship to AWS Transform |
|---|---|
| **AWS Application Migration Service (MGN)** | AWS Transform for VMware uses MGN under the hood for the actual server rehost |
| **AWS DMS** | AWS Transform for Windows uses DMS for SQL Server → Aurora PostgreSQL data migration |
| **AWS Application Discovery Service** | AWS Transform includes its own discovery tool; ADS is a standalone alternative |
| **AWS Migration Hub** | Separate tracking service; AWS Transform has its own built-in progress tracking |
| **AWS Schema Conversion Tool (SCT)** | AWS Transform for Windows automates what SCT does manually — schema conversion at scale |
| **AWS Migration Evaluator** | Complementary; Evaluator is expert-guided comprehensive assessment; Transform Assessment is self-service fast |

---

## Security and Data Privacy

| Area | Details |
|---|---|
| **Code storage** | Stored only for job duration; purged after completion |
| **Encryption in transit** | TLS 1.2+ for all communications |
| **Encryption at rest** | AWS-managed or customer-managed KMS keys |
| **Source code access** | Via AWS CodeConnections — requires IT admin approval |
| **Database access** | Via explicit admin-approved credentials + AWS Secrets Manager |
| **Code ownership** | Customer owns all transformed code; AWS retains no copies after job completion |
| **Model training** | Content may be used for service improvement/FM training unless you opt out |
| **Opt-out** | Via AWS Organizations AI services opt-out policy |
| **Schema data** | Deleted after job completion; not retained |

---

## Exam Tips Summary

### Core Identity
- ✅ AWS Transform = **agentic AI** workbench for enterprise IT modernization
- ✅ NOT a traditional migration tool — uses AI agents with continuous learning
- ✅ Covers: Assessment, Windows/.NET, Mainframe, VMware, Custom Code
- ✅ Leverages MGN (for VM rehost), DMS (for DB migration), Bedrock (for AI agents)
- ✅ Launched ~2024–2025; **newer service** — more likely SAP-C02 than SAA-C03

### Assessment
- ✅ Self-service fast TCO analysis vs Migration Evaluator (expert-led, comprehensive)
- ✅ Accepts RVTools, Cloudamize, MPA exports as data sources
- ✅ Output: business case PDF + AI chat for follow-up questions

### Windows Modernization
- ✅ .NET Framework → .NET 8/10 (cross-platform Linux-ready)
- ✅ SQL Server → Aurora PostgreSQL (3 steps: schema → data → code)
- ✅ Up to 5x faster than manual porting; up to 70% operating cost reduction
- ✅ Code committed to your repo branch; you own it; AWS does not retain it
- ✅ Customer-managed KMS keys for code encryption in transformation environment
- ✅ Human-in-the-loop review at key checkpoints; Visual Studio IDE extension for developers

### Mainframe Modernization
- ✅ Refactor = COBOL → Java (functional equivalence); Reimagine = cloud-native rewrite
- ✅ Analyze step is always required first
- ✅ Key capability: business logic extraction from COBOL for non-mainframe stakeholders
- ✅ Cyclomatic complexity analysis helps assess modernization effort before committing
- ✅ Modular — use any individual phase independently

### VMware Migration
- ✅ End-to-end migration lifecycle: discovery → wave planning → network → rehost
- ✅ Uses MGN under the hood for server rehost to EC2
- ✅ Supports Cisco ACI, Palo Alto, Fortigate for network configuration conversion
- ✅ Private connectivity: Direct Connect or Site-to-Site VPN (keep data off public internet)
- ✅ Does NOT support automated migration to VMware Cloud on AWS / Amazon EVS

### Custom Transformations
- ✅ AWS-managed prebuilt: Java 8→17, Node.js 12→22, Python →3.11/12/13, AWS SDK v1→v2
- ✅ Define custom: natural language + docs + code samples → AI generates transformation definition
- ✅ Continual learning = "knowledge items" extracted per transformation; private per customer
- ✅ CLI for CI/CD integration; Web app for large-scale campaigns
- ✅ MCP integration enables use from Claude, Cursor, Codex, Kiro

### Architecture / Migration Strategy Questions
- ✅ AWS Transform maps to **Rehost** (VMware → EC2) and **Replatform** (.NET → Linux containers, SQL Server → Aurora) in the 7 Rs framework
- ✅ When exam asks "modernize 200 .NET Framework apps with minimal manual effort" → AWS Transform for Windows
- ✅ When exam asks "COBOL mainframe to Java with AI assistance" → AWS Transform for Mainframe
- ✅ When exam asks "migrate 1,000 VMware VMs with AI wave planning" → AWS Transform for VMware
- ✅ When exam asks "upgrade Java 8 to Java 17 across 500 microservices automatically" → AWS Transform Custom (AWS-managed Java transformation)

---

## Architecture Pattern: End-to-End Enterprise Modernization

```
DISCOVERY & ASSESSMENT
────────────────────────
On-prem inventory (RVTools/OVA collector)
        ↓
AWS Transform Assessment
        ↓
Business case (TCO, EC2 right-sizing, cost projections)
        ↓

MIGRATION PLANNING (AWS Transform for VMware)
────────────────────────────────────────────
Dependency mapping → logical app groups → migration waves
        ↓
Network topology → VPC IaC (CloudFormation/Terraform)
        ↓

INFRASTRUCTURE MIGRATION
────────────────────────
VMware VMs → AWS MGN → Amazon EC2
(rehost with minimal downtime)
        ↓

DATABASE MODERNIZATION (AWS Transform for Windows)
────────────────────────────────────────────────
SQL Server → (Schema conversion) → (Data migration via DMS) → Aurora PostgreSQL
        ↓

APPLICATION MODERNIZATION (AWS Transform for Windows / Custom)
──────────────────────────────────────────────────────────────
.NET Framework → .NET 8 cross-platform
Legacy Java 8 → Java 17 (continual learning)
        ↓
Deploy as containers → Amazon ECS / EC2
        ↓

ONGOING OPTIMIZATION
────────────────────
Cost savings: remove Windows Server + SQL Server licenses
Performance: Aurora > SQL Server at scale; Linux cheaper than Windows
Modern architecture: containers + managed DB + cloud-native services
```

---
---

# End of AWS Complete Study Guide

> **Good luck on your AWS Solutions Architect exams!**
> This guide covers SAA-C03 (Associate) and SAP-C02 (Professional) exam objectives.
> Review architecture patterns and exam tips sections before your exam date.
