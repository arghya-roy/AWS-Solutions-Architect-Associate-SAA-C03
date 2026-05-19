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
- Lambda in VPC must need NAT to communicate with internet, Internet gateway alone would't work. either need NAT or NAT and IG both.

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
