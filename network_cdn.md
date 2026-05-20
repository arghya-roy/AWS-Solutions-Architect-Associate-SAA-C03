# AWS Exam Study Guide: Networking & Content Delivery
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Client VPN · CloudFront · Direct Connect · ELB · Global Accelerator · PrivateLink · Route 53 · Site-to-Site VPN · Transit Gateway · Amazon VPC

---
# OSI Model Layers

| Layer No. | Layer Name          | Main Function                                      | Examples |
|------------|--------------------|---------------------------------------------------|-----------|
| 7 | Application Layer | Provides network services to end-user applications | HTTP, HTTPS, FTP, SMTP, DNS |
| 6 | Presentation Layer | Data formatting, encryption, compression | SSL/TLS, JPEG, ASCII |
| 5 | Session Layer | Establishes, manages, and terminates sessions | NetBIOS, RPC |
| 4 | Transport Layer | Reliable data transfer, flow control, error handling | TCP, UDP |
| 3 | Network Layer | Routing and logical addressing | IP, ICMP, Routers |
| 2 | Data Link Layer | Node-to-node delivery, MAC addressing | Ethernet, Switches, MAC |
| 1 | Physical Layer | Transmission of raw bits over physical medium | Cables, Hubs, Signals |

---

# Mnemonic

## Bottom → Top (1 → 7)
**P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

## Top → Bottom (7 → 1)
**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing

## Networking Quick-Reference Matrix

| Service | Layer | Scope | Primary Purpose | Exam Trigger Words |
|---|---|---|---|---|
| **VPC** | L3/L4 | Regional | Isolated network environment | "private network", "subnets", "routing" |
| **ELB** | L4/L7 | Regional/AZ | Distribute traffic across targets | "load balance", "high availability", "health check" |
| **Route 53** | L7 (DNS) | Global | DNS, routing, health checks | "DNS", "failover", "latency routing", "domain" |
| **CloudFront** | L7 | Global (Edge) | CDN, cache, edge delivery | "cache", "low latency global", "edge", "static content" |
| **Global Accelerator** | L4 | Global (Edge) | TCP/UDP acceleration via AWS backbone | "non-HTTP", "static IP", "global TCP/UDP acceleration" |
| **Direct Connect** | L2 | Hybrid | Dedicated private line to AWS | "dedicated bandwidth", "consistent latency", "on-prem to AWS" |
| **Site-to-Site VPN** | L3 | Hybrid | Encrypted tunnel over internet | "IPsec VPN", "quick setup", "on-prem to VPC" |
| **Client VPN** | L3 | Hybrid | Remote user VPN to AWS | "remote employees", "OpenVPN", "user VPN" |
| **Transit Gateway** | L3 | Regional/Global | Hub-and-spoke network hub | "many VPCs", "transitive routing", "centralized routing" |
| **PrivateLink** | L4 | Regional | Private service access without IGW | "private endpoint", "expose SaaS service", "no public internet" |

---

---

## 1. Amazon VPC (Virtual Private Cloud)

### What Problem Does It Solve?
VPC lets you provision a **logically isolated section of the AWS cloud** where you define your own IP address range, subnets, route tables, and network gateways — essentially your own private data center in AWS.

---

### VPC Core Components

| Component | Description |
|---|---|
| **VPC** | Logical network boundary; spans all AZs in a region |
| **Subnet** | Range of IPs within an AZ; can be public or private |
| **Route Table** | Rules that determine where network traffic is directed |
| **Internet Gateway (IGW)** | Enables internet access for public subnets; horizontally scaled, HA |
| **NAT Gateway** | Enables outbound-only internet for private subnets; managed; per-AZ |
| **NAT Instance** | Self-managed EC2 acting as NAT; legacy; not HA by default |
| **Security Group** | Stateful, instance-level firewall; allow-only rules |
| **Network ACL (NACL)** | Stateless, subnet-level firewall; allow + deny rules |
| **VPC Peering** | Private connection between two VPCs |
| **VPC Endpoints** | Private connectivity to AWS services without internet |
| **Elastic IP (EIP)** | Static public IPv4 address |

---

### Subnet Types

| Type | Route Table Has | Use Case |
|---|---|---|
| **Public subnet** | Route to IGW (`0.0.0.0/0 → igw-xxx`) | Web servers, ALBs, NAT Gateways |
| **Private subnet** | Route to NAT Gateway | App servers, databases |
| **Isolated subnet** | No internet route at all | Sensitive databases, internal services |

---

### Security Groups vs Network ACLs *(Heavily Tested)*

| Feature | Security Group | Network ACL |
|---|---|---|
| **Level** | Instance (ENI) | Subnet |
| **State** | Stateful (return traffic auto-allowed) | Stateless (must allow both directions) |
| **Rules** | Allow only | Allow + Deny |
| **Rule evaluation** | All rules evaluated together | Rules evaluated in number order; first match wins |
| **Default** | Deny all inbound, allow all outbound | Allow all in/out |
| **Applies to** | EC2, RDS, Lambda in VPC, etc. | All traffic to/from subnet |

> **Exam Tip:** Security Groups are stateful — if you allow inbound 443, the response is automatically allowed outbound. NACLs require explicit rules for both directions.

---

### NAT Gateway vs NAT Instance

| Feature | NAT Gateway | NAT Instance |
|---|---|---|
| **Management** | Fully managed by AWS | You manage EC2 |
| **HA** | Redundant within AZ | Single point of failure |
| **Bandwidth** | Up to 100 Gbps | Depends on instance type |
| **Cost** | Per hour + per GB | EC2 instance cost |
| **Security Groups** | Cannot apply | Can apply |
| **Placement** | Public subnet | Public subnet |
| **Multi-AZ HA** | Deploy one per AZ | Requires Auto Scaling + scripting |
| **Port forwarding** | ❌ | ✅ |

> **Exam Rule:** Always use NAT Gateway in production. NAT Instance is legacy and only appears in trick questions.

---

### VPC Peering

- Private, direct connection between two VPCs (same or different accounts/regions).
- Uses AWS backbone — no internet transit.
- **Not transitive**: A↔B and B↔C does NOT mean A↔C.
- No overlapping CIDR blocks allowed.
- Must update route tables in both VPCs.

---

### VPC Endpoints

| Type | Description | Use Case |
|---|---|---|
| **Gateway Endpoint** | Route-table-based; free | **S3 and DynamoDB only** |
| **Interface Endpoint (PrivateLink)** | ENI in subnet; hourly + data charge | All other AWS services, SaaS |

> **Exam Tip:** "EC2 in private subnet needs to access S3 without internet" → **VPC Gateway Endpoint** (free, route-table-based).

---

### VPC Flow Logs

- Capture metadata (not payload) of IP traffic to/from ENIs, subnets, or VPCs.
- Destination: CloudWatch Logs, S3, Kinesis Data Firehose.
- Fields: source IP, dest IP, port, protocol, bytes, action (ACCEPT/REJECT).
- Does NOT capture: DNS queries (use Route 53 Resolver logs), NTP, DHCP, instance metadata.

---

### CIDR and IP Planning

- VPC CIDR: `/16` to `/28`.
- AWS **reserves 5 IPs per subnet**: network, VPC router, DNS, future use, broadcast.
- Example: `/24` subnet = 256 − 5 = **251 usable IPs**.
- IPv6: `/56` for VPC, `/64` for subnets; all IPv6 are public by default.

---

### Key Exam Scenarios

- **Private EC2 → internet (outbound only)**: NAT Gateway in public subnet.
- **Private EC2 → S3 without internet**: VPC Gateway Endpoint for S3.
- **Private EC2 → any AWS service privately**: Interface Endpoint (PrivateLink).
- **Connect two VPCs (no transitive routing needed)**: VPC Peering.
- **Connect many VPCs (transitive routing needed)**: Transit Gateway.
- **Block specific IP at subnet level**: NACL deny rule.
- **Capture traffic for security analysis**: VPC Flow Logs.
- **EC2 needs static public IP**: Elastic IP.

---

---

## 2. Elastic Load Balancing (ELB)

### What Problem Does It Solve?
ELB distributes **incoming application traffic** across multiple targets (EC2, ECS, Lambda, IPs) in multiple AZs, providing fault tolerance, automatic scaling, and health checking.

---

### ELB Types — The Big Four

#### Application Load Balancer (ALB) — Layer 7
- HTTP/HTTPS/gRPC traffic.
- **Content-based routing**: path (`/api/*`), host (`api.example.com`), headers, query strings, IP.
- Supports **WebSockets**, HTTP/2.
- Targets: EC2, ECS, Lambda, IP addresses.
- **Sticky sessions**: via cookies (AWSALBCOOKIE or custom).
- **SSL termination**: offloads TLS from backend.
- Returns **fixed response** or **redirects** (e.g., HTTP → HTTPS).

#### Network Load Balancer (NLB) — Layer 4
- TCP, UDP, TLS traffic.
- Handles **millions of requests per second** with ultra-low latency.
- **Static IP per AZ** (or use Elastic IP) — critical for whitelisting.
- Preserves **client source IP** (unlike ALB which uses X-Forwarded-For).
- Targets: EC2, IP addresses, ALB (ALB behind NLB).
- Supports **PrivateLink** (expose services privately).
- **No security groups** on NLB itself (but target SGs must allow NLB traffic from VPC CIDR).

#### Gateway Load Balancer (GWLB) — Layer 3
- For deploying, scaling, and managing **third-party virtual network appliances** (firewalls, IDS/IPS, deep packet inspection).
- Uses **GENEVE protocol** (port 6081).
- Traffic flow: user → GWLB → appliance (inspect/modify) → GWLB → destination.
- Operates at VPC boundary using **Gateway Load Balancer Endpoints**.

#### Classic Load Balancer (CLB) — Layer 4/7 (Legacy)
- Original ELB; supports HTTP/HTTPS/TCP/SSL.
- Does not support content-based routing.
- Being phased out; avoid in new designs.

---

### ALB vs NLB vs GWLB Comparison

| Feature | ALB | NLB | GWLB |
|---|---|---|---|
| **Layer** | 7 (HTTP) | 4 (TCP/UDP) | 3 (IP) |
| **Protocol** | HTTP, HTTPS, gRPC | TCP, UDP, TLS | IP (GENEVE) |
| **Routing** | Content-based | Connection-based | Transparent pass-through |
| **Static IP** | ❌ (use Global Accelerator) | ✅ (per AZ) | N/A |
| **Source IP preserve** | Via header | ✅ Native | ✅ |
| **PrivateLink** | ❌ | ✅ | ✅ |
| **Lambda target** | ✅ | ❌ | ❌ |
| **Use case** | Web apps, microservices | High-perf, gaming, IoT, VPN | Security appliances |

---

### ELB Health Checks

- ALB: HTTP/HTTPS health check (must return 2xx/3xx).
- NLB: TCP, HTTP, HTTPS health checks.
- Unhealthy targets are deregistered from routing until healthy.
- **Deregistration delay** (connection draining): time to complete in-flight requests when deregistering target (default 300s, range 0–3600s).

---

### Cross-Zone Load Balancing

| ELB Type | Default | Cost |
|---|---|---|
| **ALB** | Enabled | Free |
| **NLB** | Disabled | Charged if enabled |
| **GWLB** | Disabled | Charged if enabled |

> **Exam Tip:** With cross-zone LB disabled, each LB node only distributes traffic among targets in its own AZ. With it enabled, traffic distributes evenly across all AZs.

---

### ALB Listener Rules

Priority-ordered rules evaluated top-down:
1. **Conditions**: host header, path, HTTP method, query string, source IP, header value
2. **Actions**: forward to target group, redirect, fixed response, authenticate (Cognito / OIDC)

---

### ELB Access Logs & Monitoring

- **Access logs**: Stored in S3; detailed per-request logging (source IP, latency, response code).
- **CloudWatch metrics**: RequestCount, TargetResponseTime, HTTPCode_ELB_5XX, HealthyHostCount.
- **Request tracing**: ALB adds `X-Amzn-Trace-Id` header; integrate with X-Ray.

---

### Key Exam Scenarios

- **Route `/api` to one service, `/web` to another**: ALB path-based routing.
- **Whitelist load balancer IP in firewall**: NLB (static IP per AZ).
- **Expose internal service to another VPC via PrivateLink**: NLB as PrivateLink service.
- **Third-party firewall appliance in traffic path**: GWLB.
- **Lambda as backend for HTTP API**: ALB with Lambda target group.
- **Redirect HTTP to HTTPS**: ALB listener rule (redirect action, port 443).
- **Blue/green deployment with traffic shift**: ALB weighted target groups.
- **Sticky sessions for stateful app**: ALB with cookie-based stickiness.

---

---

## 3. Amazon Route 53

### What Problem Does It Solve?
Route 53 is AWS's **highly available, scalable DNS service** that also provides domain registration, DNS-based health checks, and intelligent traffic routing.

---

### Route 53 Record Types

| Record | Description |
|---|---|
| **A** | IPv4 address |
| **AAAA** | IPv6 address |
| **CNAME** | Alias to another hostname; **cannot be used for zone apex (root domain)** |
| **Alias** | AWS-specific; points to AWS resources (ALB, CloudFront, S3, etc.); **works at zone apex**; free of charge |
| **MX** | Mail server |
| **TXT** | Text records; domain verification |
| **NS** | Name servers for hosted zone |
| **PTR** | Reverse DNS lookup |
| **SRV** | Service location |

> **Critical Exam Rule:** `CNAME` cannot be at the zone apex (`example.com`). Use **Alias** record instead. Alias records are free; CNAME queries are charged.

---

### Hosted Zones

| Type | Description |
|---|---|
| **Public Hosted Zone** | Resolves queries from the internet |
| **Private Hosted Zone** | Resolves queries within VPCs; must associate VPC |

---

### Route 53 Routing Policies *(Most Tested Topic)*

#### 1. Simple Routing
- One record → one or more IP values.
- If multiple values: **client randomly picks one**.
- No health checks on individual values.

#### 2. Weighted Routing
- Multiple records with **weight values** (0–255).
- Traffic proportional to weight: weight of record / total weights.
- Use for: A/B testing, gradual deployments, blue/green.
- Health check supported.

#### 3. Latency-Based Routing
- Routes to the **AWS region with lowest network latency** for the user.
- Not geographic — based on measured latency.
- Health check supported.

#### 4. Failover Routing
- **Active-passive** failover.
- Primary record used when healthy; Route 53 fails over to secondary if primary health check fails.
- Requires health check on primary.

#### 5. Geolocation Routing
- Routes based on **user's geographic location** (continent, country, state).
- Must define a **default record** for unmatched locations.
- Use for: localized content, compliance (GDPR), language routing.
- Different from latency routing — based on location, not latency.

#### 6. Geoproximity Routing (Traffic Flow Only)
- Routes based on geographic location of **users AND resources**.
- **Bias value**: Expand (+) or shrink (−) geographic area served by a resource.
- Requires **Route 53 Traffic Flow**.

#### 7. Multi-Value Answer Routing
- Returns **multiple healthy IP values** (up to 8) — clients randomly pick.
- Health checks supported; unhealthy records excluded.
- NOT a replacement for a load balancer; client-side "load balancing".

#### 8. IP-Based Routing
- Route based on the **client's IP CIDR** (you provide CIDR → endpoint mapping).
- Use for: ISP-based routing, known office IP routing.

---

### Route 53 Health Checks

| Type | Description |
|---|---|
| **Endpoint health check** | HTTP/HTTPS/TCP check against IP or hostname |
| **Calculated health check** | Combines results of multiple child health checks (AND/OR logic) |
| **CloudWatch alarm health check** | Healthy = alarm state is OK |

- Health checkers are globally distributed; **18% threshold** by default must report healthy.
- Private endpoints require **CloudWatch alarm** health check (health checkers can't reach private IPs).

---

### Route 53 Resolver

| Feature | Description |
|---|---|
| **Inbound Endpoint** | On-prem DNS can query Route 53 private hosted zones |
| **Outbound Endpoint** | Route 53 forwards queries to on-prem DNS |
| **Resolver Rules** | Forwarding rules: which domains go to on-prem DNS |

---

### Route 53 vs CloudFront for Routing

| Scenario | Service |
|---|---|
| DNS-level failover between regions | Route 53 Failover routing |
| Serve cached content from edge | CloudFront |
| Low-latency API for global users | Route 53 Latency + Global Accelerator |

---

### Key Exam Scenarios

- **Root domain pointing to ALB**: Alias A record (not CNAME).
- **Blue/green with 10% traffic to new version**: Weighted routing.
- **Serve users from nearest AWS region**: Latency-based routing.
- **Failover to S3 static website if primary down**: Failover routing with health check.
- **Route EU users to EU endpoint for GDPR**: Geolocation routing.
- **On-prem DNS resolve AWS private hosted zone**: Route 53 Resolver inbound endpoint.
- **Route 53 forward on-prem domain resolution**: Route 53 Resolver outbound endpoint.
- **Health check for private ALB**: CloudWatch alarm → Route 53 CloudWatch alarm health check.

---

---

## 4. Amazon CloudFront

### What Problem Does It Solve?
CloudFront is AWS's **Content Delivery Network (CDN)** that caches content at **450+ edge locations** globally, reducing latency for end users and offloading origin servers.

---

### CloudFront Components

| Component | Description |
|---|---|
| **Distribution** | The CloudFront deployment (one per origin/behavior config) |
| **Origin** | Source of content: S3, ALB, EC2, API Gateway, custom HTTP |
| **Edge Location** | Cache node closest to the user |
| **Regional Edge Cache** | Larger cache between edge locations and origin (fewer origin fetches) |
| **Cache Behavior** | Rules per path pattern: which origin, TTL, headers, cookies |

---

### CloudFront Origins

| Origin Type | Notes |
|---|---|
| **S3 bucket** | Use Origin Access Control (OAC) to restrict S3 to CloudFront only |
| **S3 static website endpoint** | Used for website hosting with custom error pages |
| **ALB** | Must allow CloudFront IP ranges in SG |
| **EC2** | Must be publicly accessible |
| **API Gateway** | Common for serverless APIs |
| **Custom HTTP origin** | Any HTTP/HTTPS endpoint |

---

### Origin Access Control (OAC) vs Origin Access Identity (OAI)

| Feature | OAC (New) | OAI (Legacy) |
|---|---|---|
| **S3 SSE-KMS** | ✅ Supported | ❌ |
| **All S3 operations** | ✅ | Limited |
| **Recommended** | ✅ | Deprecated |

> **Exam Tip:** Use **OAC** (not OAI) for restricting S3 bucket access to CloudFront only. Set bucket policy to allow CloudFront service principal.

---

### CloudFront Cache Behavior

- **TTL settings**: min, max, default TTL.
- **Cache key**: URL path + selected headers, cookies, query strings.
- **Cache invalidation**: `/*` invalidates everything (first 1,000 paths/month free).
- **Cache-Control / Expires headers** from origin override default TTL.

---

### CloudFront Security

| Feature | Description |
|---|---|
| **HTTPS** | Enforce HTTPS between viewer-CloudFront and CloudFront-origin |
| **WAF** | Attach WAF Web ACL to distribution |
| **Geo-restriction** | Allow/block by country (allowlist or blocklist) |
| **Signed URLs** | Grant time-limited access to a single file |
| **Signed Cookies** | Grant time-limited access to multiple files |
| **Field-Level Encryption** | Encrypt specific POST fields at edge using public key |

#### Signed URL vs Signed Cookie

| Feature | Signed URL | Signed Cookie |
|---|---|---|
| **Scope** | One file | Multiple files |
| **Client support** | Any client | Must support cookies |
| **Use case** | Individual file download | Premium video streaming, content library |

---

### CloudFront Functions vs Lambda@Edge

| Feature | CloudFront Functions | Lambda@Edge |
|---|---|---|
| **Runtime** | JavaScript (ES5.1) | Node.js, Python |
| **Max execution** | < 1 ms | 5 sec (viewer) / 30 sec (origin) |
| **Triggers** | Viewer request/response only | Viewer + origin request/response |
| **Memory** | 2 MB | 128 MB – 10 GB |
| **Network access** | ❌ | ✅ |
| **Cost** | ~1/6th of Lambda@Edge | Higher |
| **Use case** | URL rewrites, header manipulation, simple redirects | Auth, A/B testing, complex routing, API calls |

---

### CloudFront Pricing Classes

| Class | Edge Locations | Cost |
|---|---|---|
| **Price Class All** | All regions | Highest |
| **Price Class 200** | Most regions (excl. most expensive) | Medium |
| **Price Class 100** | Cheapest regions (US, EU, Asia) | Lowest |

---

### CloudFront vs S3 Transfer Acceleration

| Feature | CloudFront | S3 Transfer Acceleration |
|---|---|---|
| **Purpose** | Cache + deliver content | Accelerate S3 uploads |
| **Direction** | Download (mostly) | Upload to S3 |
| **Caching** | ✅ | ❌ |
| **Use case** | Serve static/dynamic content | Upload large files to S3 globally |

---

### Key Exam Scenarios

- **Serve S3 static website globally with low latency**: CloudFront + S3 OAC.
- **Prevent direct S3 access; only via CloudFront**: S3 bucket policy + OAC.
- **Serve premium video only to subscribers**: CloudFront Signed Cookies.
- **Block users from specific countries**: CloudFront Geo-restriction.
- **Add security headers to all responses**: CloudFront Functions (viewer response trigger).
- **A/B test new origin**: CloudFront weighted origin groups.
- **Force HTTPS**: Redirect HTTP to HTTPS in CloudFront viewer protocol policy.
- **Real-time logs for monitoring**: CloudFront Real-time Logs → Kinesis.

---

---

## 5. AWS Global Accelerator

### What Problem Does It Solve?
Global Accelerator improves performance of **global TCP and UDP applications** by routing traffic through the **AWS global backbone network** instead of the public internet, reducing latency and packet loss.

---

### Key Properties

| Property | Details |
|---|---|
| **Entry points** | **2 static Anycast IPs** (globally announced from all edge locations) |
| **Protocol** | TCP, UDP (not HTTP-aware) |
| **Backend targets** | ALB, NLB, EC2, Elastic IP |
| **Health checks** | Built-in; automatic failover in < 30 seconds |
| **Client affinity** | Route same client to same endpoint |

---

### How It Works

```
User → Nearest AWS Edge Location (Anycast IP)
              ↓
    AWS Global Backbone Network (private, fast)
              ↓
    Endpoint (ALB/NLB/EC2 in target region)
```

---

### Global Accelerator vs CloudFront *(Most Tested Comparison)*

| Feature | Global Accelerator | CloudFront |
|---|---|---|
| **Layer** | L4 (TCP/UDP) | L7 (HTTP/HTTPS) |
| **Caching** | ❌ No caching | ✅ Caches content |
| **Static IP** | ✅ 2 static Anycast IPs | ❌ (uses CloudFront domain) |
| **Protocol** | TCP, UDP | HTTP, HTTPS |
| **Use case** | Gaming, IoT, VoIP, non-HTTP | Web, API, media delivery |
| **Routing logic** | Latency + health-based | Cache + behavior rules |
| **Origin** | ALB, NLB, EC2, EIP | S3, ALB, API GW, custom |
| **DDoS** | Shield Standard included | Shield Standard included |

> **Key Exam Rule:** If the question mentions **HTTP/HTTPS with caching** → CloudFront. If it mentions **static IPs**, **non-HTTP (gaming/IoT/VoIP)**, or **TCP/UDP global acceleration** → Global Accelerator.

---

### Global Accelerator Endpoint Groups

- One endpoint group per AWS region.
- Traffic dial: control % of traffic to each region (0–100%).
- Useful for gradual region migrations and blue/green deployments.

---

### Key Exam Scenarios

- **Static IPs for global application (firewall whitelisting)**: Global Accelerator.
- **Multiplayer gaming with low latency globally**: Global Accelerator.
- **Instant failover between regions for TCP app**: Global Accelerator (< 30 sec).
- **HTTP REST API with caching globally**: CloudFront.
- **Gradual traffic shift between regions**: Global Accelerator traffic dial.

---

---

## 6. AWS Direct Connect

### What Problem Does It Solve?
Direct Connect provides a **dedicated private network connection** from your on-premises data center to AWS — bypassing the public internet for **consistent bandwidth, lower latency, and reduced data transfer costs**.

---

### Connection Types

| Type | Speed | Description |
|---|---|---|
| **Dedicated Connection** | 1, 10, 100 Gbps | Physical port at AWS Direct Connect location |
| **Hosted Connection** | 50 Mbps – 10 Gbps | Provisioned by AWS Partner; sub-1G speeds available |

---

### Virtual Interfaces (VIFs)

| VIF Type | Description | Connects To |
|---|---|---|
| **Private VIF** | Access resources in your VPC via private IPs | VPC (via VGW or TGW) |
| **Public VIF** | Access AWS public services (S3, DynamoDB) via AWS IPs (not internet) | AWS public endpoints |
| **Transit VIF** | Connect to Transit Gateway | Transit Gateway (for many VPCs) |

---

### Direct Connect Gateway

- Connect **one or more VPCs across multiple regions** using a single Direct Connect connection.
- Does NOT allow VPC-to-VPC communication through the gateway.
- Use with: Private VIF or Transit VIF.

```
On-Premises ──DX──→ Direct Connect Gateway → VPC (us-east-1)
                                            → VPC (eu-west-1)
                                            → Transit Gateway
```

---

### Direct Connect Resilience

| Pattern | Description | SLA |
|---|---|---|
| **Single connection** | One DX connection | No redundancy |
| **Dual connections (same location)** | Redundant at device level | Device failure protection |
| **Dual locations** | Two DX locations | Location failure protection |
| **Maximum resilience** | 2 connections at 2 locations | Highest availability |

> **Exam Tip:** For high availability, **always pair Direct Connect with a Site-to-Site VPN as backup**.

---

### Direct Connect + VPN (Backup)

- DX as primary; Site-to-Site VPN as failover.
- Use BGP routing: DX has lower cost metric (preferred); VPN activates when DX fails.

---

### MACsec

- Layer 2 encryption on Direct Connect.
- Encrypts data in transit at the link layer (10G and 100G dedicated connections).
- For compliance requirements needing encryption at L2.

---

### Direct Connect vs Site-to-Site VPN

| Feature | Direct Connect | Site-to-Site VPN |
|---|---|---|
| **Connection** | Dedicated private line | Encrypted tunnel over internet |
| **Bandwidth** | Up to 100 Gbps | Up to 1.25 Gbps per tunnel |
| **Latency** | Consistent, low | Variable (internet) |
| **Setup time** | Weeks to months | Minutes to hours |
| **Cost** | Higher | Lower |
| **Encryption** | Not by default (MACsec optional) | Always encrypted (IPsec) |
| **Use case** | Consistent, high-volume, compliance | Quick setup, backup, lower cost |

---

### Key Exam Scenarios

- **Dedicated high-bandwidth to AWS**: Direct Connect (dedicated).
- **Sub-1 Gbps connection via partner**: Direct Connect (hosted).
- **Access multiple VPCs across regions from on-prem**: Direct Connect Gateway.
- **Access S3/DynamoDB without internet from on-prem**: Direct Connect Public VIF.
- **Maximum resiliency**: Two connections at two DX locations.
- **DX backup**: Site-to-Site VPN as failover.
- **Consistent latency + compliance**: Direct Connect (over VPN).

---

---

## 7. AWS Site-to-Site VPN

### What Problem Does It Solve?
Site-to-Site VPN creates an **encrypted IPsec tunnel over the public internet** connecting your on-premises network to an AWS VPC — quickly and at low cost.

---

### Components

| Component | Description |
|---|---|
| **Virtual Private Gateway (VGW)** | AWS side of VPN; attached to a VPC |
| **Transit Gateway (TGW)** | Alternative to VGW for connecting many VPCs |
| **Customer Gateway (CGW)** | Represents your on-prem VPN device (IP + BGP/static config) |
| **VPN Connection** | Two IPsec tunnels (HA); one per AZ |

---

### VPN Tunnel Details

- **2 tunnels per VPN connection** for redundancy (different AZs on AWS side).
- Each tunnel: up to **1.25 Gbps**.
- Protocols: IKEv1 / IKEv2, AES-128/256, SHA-1/2.
- Routing: **Static** or **BGP (dynamic)**.

---

### Accelerated Site-to-Site VPN

- Routes VPN traffic through **Global Accelerator** edge locations.
- Reduces internet hops → lower latency for VPN tunnels.
- Requires Transit Gateway (not VGW).

---

### VPN CloudHub

- Connect **multiple on-premises sites** to a single VGW (hub-and-spoke).
- On-prem sites can communicate with each other via AWS hub.
- Each site needs its own Customer Gateway.
- Low-cost WAN alternative.

---

### Key Exam Scenarios

- **Quick encrypted connection to VPC**: Site-to-Site VPN.
- **Multiple branch offices to AWS**: VPN CloudHub.
- **VPN for multiple VPCs**: TGW + VPN attachment.
- **Reduce VPN latency globally**: Accelerated Site-to-Site VPN + TGW.
- **Backup for Direct Connect**: Site-to-Site VPN (automatic BGP failover).

---

---

## 8. AWS Client VPN

### What Problem Does It Solve?
Client VPN provides **remote access VPN for individual users** (employees, developers) to securely connect to AWS VPCs and on-premises networks from any location using an **OpenVPN-based client**.

---

### Key Properties

| Property | Details |
|---|---|
| **Protocol** | OpenVPN (TLS) |
| **Authentication** | Active Directory, SAML-based IdP, certificate-based (mutual TLS) |
| **Authorization** | Network-based authorization rules (CIDR ranges) |
| **Split tunneling** | Route only AWS-bound traffic through VPN (reduces bandwidth) |
| **Logging** | CloudWatch Logs |
| **Scaling** | Auto-scales based on connected users |

---

### Client VPN vs Site-to-Site VPN

| Feature | Client VPN | Site-to-Site VPN |
|---|---|---|
| **Connects** | Individual user device | Entire on-prem network |
| **Protocol** | OpenVPN | IPsec |
| **Use case** | Remote workers | Branch office/DC to AWS |
| **Auth** | AD, SAML, certificates | Shared key or certificates |
| **Scale** | Per user | Per site |

---

### Key Exam Scenarios

- **Remote employees access VPC resources**: Client VPN.
- **Developer accessing private RDS from home**: Client VPN.
- **Authenticate VPN users with corporate AD**: Client VPN + Active Directory authentication.
- **Reduce client VPN bandwidth usage**: Enable split tunneling.

---

---

## 9. AWS Transit Gateway

### What Problem Does It Solve?
Transit Gateway acts as a **regional network hub** that simplifies complex VPC-to-VPC and VPC-to-on-premises connectivity — replacing the unmanageable mesh of VPC peering connections with a single hub-and-spoke model.

---

### What Can Connect to TGW

| Attachment Type | Description |
|---|---|
| **VPC** | Attach VPCs in same or different accounts (via RAM) |
| **Site-to-Site VPN** | Connect on-prem via IPsec VPN |
| **Direct Connect Gateway** | Connect on-prem via DX |
| **TGW Peering** | Connect TGWs in different regions |
| **SD-WAN appliance** | Via VPN attachment |

---

### Transit Gateway Route Tables

- Each TGW has one or more **route tables**.
- Attachments are associated with one route table (for inbound route lookup).
- Attachments can propagate routes to a route table.
- **Route isolation**: Create separate route tables to prevent certain VPCs from communicating.

---

### TGW vs VPC Peering

| Feature | Transit Gateway | VPC Peering |
|---|---|---|
| **Transitive routing** | ✅ | ❌ |
| **Scale** | Thousands of VPCs | Point-to-point (N²) |
| **Cross-region** | ✅ (TGW peering) | ✅ (inter-region peering) |
| **Cross-account** | ✅ (RAM) | ✅ |
| **Bandwidth** | Up to 50 Gbps per attachment | Effectively unlimited |
| **Centralized control** | ✅ | ❌ |
| **Cost** | Per attachment + data | Per GB (no hourly) |

---

### TGW Network Topologies

#### Hub-and-Spoke (Default)
```
Spoke VPC A ──┐
Spoke VPC B ──┤── Transit Gateway ── On-Premises (VPN/DX)
Spoke VPC C ──┘
```

#### Isolated VPCs (Multiple Route Tables)
```
Prod VPCs ──→ Prod Route Table ──→ Shared Services VPC
Dev VPCs ──→ Dev Route Table ──→ Shared Services VPC
(Prod ↔ Dev communication blocked by separate route tables)
```

#### Centralized Inspection
```
All VPCs → TGW → Inspection VPC (Network Firewall / GWLB)
                       ↓
                  Internet / On-Prem
```

---

### TGW Multicast

- Supports **IP multicast** across VPCs — for video streaming, financial tick data.
- Only TGW supports multicast (VPC Peering does not).

---

### Key Exam Scenarios

- **Connect 10+ VPCs without N² peering**: Transit Gateway.
- **On-prem to multiple VPCs via one DX**: DX Gateway + TGW + Transit VIF.
- **Isolate prod and dev VPCs but share services**: TGW with separate route tables.
- **Centralized firewall inspection for all VPC traffic**: TGW + Inspection VPC + Network Firewall.
- **Share TGW across accounts in org**: TGW + RAM.
- **Connect TGWs in us-east-1 and eu-west-1**: TGW peering.

---

---

## 10. AWS PrivateLink

### What Problem Does It Solve?
PrivateLink provides **private connectivity between VPCs, AWS services, and on-premises** using **Interface Endpoints** — without exposing traffic to the public internet, VPC peering, NAT, or IGW.

---

### PrivateLink Architecture

```
Consumer VPC ──Interface Endpoint (ENI)──→ PrivateLink ──→ Provider Service
                        (private IP)                         (behind NLB)
```

---

### Use Cases

| Use Case | Description |
|---|---|
| **AWS service access** | Access S3, DynamoDB, SSM, Secrets Manager, etc. privately |
| **Expose your own service** | Put NLB in front of your service → share as PrivateLink endpoint service |
| **SaaS integration** | Third-party SaaS accessible via PrivateLink (no internet) |
| **Cross-account service** | Share service between accounts without VPC peering |

---

### PrivateLink vs VPC Peering vs VPC Gateway Endpoint

| Feature | PrivateLink (Interface Endpoint) | VPC Peering | Gateway Endpoint |
|---|---|---|---|
| **Services** | Any AWS/custom service | VPCs only | S3 and DynamoDB only |
| **Overlapping CIDR** | Allowed | ❌ Not allowed | N/A |
| **Transitive** | No (point-to-point) | ❌ | N/A |
| **Cost** | Hourly + data | Per GB | **Free** |
| **On-prem access** | ✅ (via DX/VPN) | ❌ | ❌ |
| **Security** | ENI + Security Group | Route table | Route table |

---

### PrivateLink for On-Prem

- On-prem can access services via Interface Endpoint **through Direct Connect or VPN** → no internet needed.

---

### Key Exam Scenarios

- **Access SSM/Secrets Manager from private EC2 without NAT**: Interface Endpoint (PrivateLink).
- **Expose internal microservice to partner account without peering**: PrivateLink endpoint service (NLB).
- **SaaS provider expose service to customers privately**: PrivateLink.
- **Overlapping CIDRs between consumer and provider**: PrivateLink (works; peering doesn't).
- **On-prem access to S3 privately via DX**: Interface Endpoint (in VPC) + Direct Connect.

---

---

## Master Topology: All Networking Services Together

```
                        INTERNET
                            │
                    Route 53 (DNS)
                            │
                 CloudFront (CDN/WAF)
                            │
              Global Accelerator (Static IPs)
                            │
                   ┌────────┴────────┐
              ALB (L7)          NLB (L4)
                   │                │
            ┌──────┴──────┐         │
          EC2/ECS      Lambda    EC2/ECS
            │
         ──── VPC ────────────────────────────────────
         │  Public Subnet                             │
         │  [ALB][NAT GW][Bastion]                   │
         │                                            │
         │  Private Subnet                            │
         │  [EC2][RDS][ECS]                          │
         │     │                                      │
         │  VPC Endpoint (S3/DynamoDB)               │
         │     │                                      │
         └─────┼──────────────────────────────────────┘
               │
         ┌─────┴─────────────────────────────────┐
         │          Transit Gateway               │
         ├─── VPC (Prod)                          │
         ├─── VPC (Dev)                           │
         ├─── Inspection VPC (Network Firewall)   │
         ├─── Site-to-Site VPN                    │
         └─── Direct Connect Gateway              │
                            │                     │
                     On-Premises DC               │
                     [Client VPN users] ──────────┘
```

---

## Critical Comparisons for the Exam

### Connectivity: On-Premises to AWS

| Scenario | Solution |
|---|---|
| Quick encrypted connection | Site-to-Site VPN |
| High bandwidth, consistent latency | Direct Connect |
| Highly available hybrid | Direct Connect + VPN backup |
| Multiple branch offices | VPN CloudHub |
| Remote users | Client VPN |
| On-prem → many VPCs | Direct Connect + TGW + Transit VIF |

### VPC-to-VPC Connectivity

| Scenario | Solution |
|---|---|
| Two VPCs, simple | VPC Peering |
| Many VPCs, transitive | Transit Gateway |
| Expose service (no peering) | PrivateLink |
| Overlapping CIDRs | PrivateLink |
| Share subnets across accounts | RAM |

### Global Traffic Distribution

| Scenario | Solution |
|---|---|
| DNS-level routing | Route 53 (latency / failover / weighted) |
| Content caching at edge | CloudFront |
| Static IPs, non-HTTP global | Global Accelerator |
| HTTP API globally | CloudFront or Route 53 + ALB |

### Load Balancer Selection

| Scenario | Load Balancer |
|---|---|
| HTTP/HTTPS routing by path/host | ALB |
| High-performance, static IP, TCP/UDP | NLB |
| Third-party firewall appliance | GWLB |
| Lambda as backend | ALB |
| PrivateLink exposure | NLB |

---

## Architecture Patterns to Know

### Pattern 1: Multi-Region Active-Active
```
Route 53 (Latency routing)
    ├── us-east-1: CloudFront → ALB → EC2 → RDS
    └── eu-west-1: CloudFront → ALB → EC2 → RDS (replica)
```

### Pattern 2: Hybrid Cloud Network
```
On-Prem DC ──Direct Connect (primary)──┐
            ──Site-to-Site VPN (backup)─┤
                                        └→ Transit Gateway
                                              ├── Prod VPC
                                              ├── Dev VPC
                                              └── Shared Services VPC
```

### Pattern 3: Centralized Ingress/Egress
```
Internet → ALB (in Ingress VPC)
                → Transit Gateway
                      ├── App VPC A
                      ├── App VPC B
                      └── Egress VPC (NAT Gateway → Internet)
```

### Pattern 4: Private SaaS Service
```
Consumer Account VPC
  └── Interface Endpoint (ENI, private IP)
            ↓ PrivateLink
Provider Account VPC
  └── NLB → EC2 (SaaS Service)
```

### Pattern 5: Serverless Global API
```
Users globally → Route 53 (latency routing)
                      ↓
                 CloudFront (cache GET responses)
                      ↓
                 API Gateway → Lambda → DynamoDB Global Tables
```

---

## Exam Tips Summary

### VPC
- ✅ SGs = stateful, instance-level, allow-only; NACLs = stateless, subnet-level, allow+deny
- ✅ NAT Gateway in public subnet; private subnet instances route to it
- ✅ Gateway Endpoint = free; S3/DynamoDB only; route-table-based
- ✅ Interface Endpoint = PrivateLink; all services; hourly cost
- ✅ VPC peering = no transitive routing; no overlapping CIDRs
- ✅ AWS reserves 5 IPs per subnet
- ✅ Flow Logs = metadata only; not payload; goes to S3/CloudWatch/Firehose

### ELB
- ✅ ALB = L7; path/host routing; Lambda target; Cognito auth
- ✅ NLB = L4; static IP; source IP preserve; PrivateLink; ultra-low latency
- ✅ GWLB = security appliances; GENEVE protocol
- ✅ ALB: cross-zone LB free; NLB/GWLB: cross-zone LB charged
- ✅ Deregistration delay = drain in-flight requests (0–3600s, default 300s)

### Route 53
- ✅ Alias = works at zone apex (root domain); free; AWS resources only
- ✅ CNAME = cannot be at zone apex; charged
- ✅ Latency routing ≠ geolocation routing
- ✅ Failover = active-passive; requires health check on primary
- ✅ Private hosted zone health checks → must use CloudWatch alarm
- ✅ Resolver inbound = on-prem queries Route 53; outbound = Route 53 queries on-prem

### CloudFront
- ✅ OAC (not OAI) for restricting S3 access
- ✅ Signed URL = single file; Signed Cookie = multiple files
- ✅ CloudFront Functions = < 1ms, viewer triggers only
- ✅ Lambda@Edge = heavier, all 4 trigger types, network access
- ✅ Price Class 100 = cheapest, US/EU/Asia only
- ✅ ACM cert for CloudFront must be in us-east-1

### Global Accelerator
- ✅ 2 static Anycast IPs; TCP/UDP; no caching
- ✅ Use when: non-HTTP, static IPs needed, gaming/IoT/VoIP
- ✅ CloudFront = HTTP caching; Global Accelerator = TCP/UDP acceleration

### Direct Connect
- ✅ Dedicated = 1/10/100 Gbps; Hosted = 50 Mbps–10 Gbps (via partner)
- ✅ Private VIF → VPC; Public VIF → AWS public services; Transit VIF → TGW
- ✅ DX Gateway = multi-region multi-VPC from one DX
- ✅ Not encrypted by default; use MACsec or IPsec VPN over DX
- ✅ Maximum resiliency = 2 connections × 2 locations

### Site-to-Site VPN
- ✅ 2 tunnels per connection; 1.25 Gbps each
- ✅ VPN CloudHub = multiple on-prem sites via single VGW
- ✅ Accelerated VPN = Global Accelerator + TGW
- ✅ Always use VPN as DX backup

### Transit Gateway
- ✅ Hub-and-spoke; transitive routing; replaces mesh peering
- ✅ Multiple route tables = traffic isolation between VPCs
- ✅ Share via RAM across accounts
- ✅ TGW peering = connect TGWs across regions
- ✅ Only TGW supports multicast

### PrivateLink
- ✅ Interface Endpoint = ENI in your subnet; PrivateLink under the hood
- ✅ Works with overlapping CIDRs (unlike peering)
- ✅ Expose own service: put NLB in front → create endpoint service
- ✅ On-prem access via DX/VPN through Interface Endpoint
- ✅ Gateway Endpoint (S3/DynamoDB) is free; Interface Endpoint costs money
