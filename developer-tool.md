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
