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
