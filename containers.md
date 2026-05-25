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
