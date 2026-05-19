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
