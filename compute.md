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
