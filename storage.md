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
