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
