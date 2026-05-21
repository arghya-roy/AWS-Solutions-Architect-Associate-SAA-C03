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
