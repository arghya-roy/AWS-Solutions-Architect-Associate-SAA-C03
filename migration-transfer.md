# AWS Exam Study Guide: Migration & Transfer Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Application Discovery Service · Application Migration Service · DMS · DataSync · Migration Hub · Snow Family · Transfer Family

---

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
