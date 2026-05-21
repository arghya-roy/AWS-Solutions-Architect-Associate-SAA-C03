# AWS Exam Study Guide: Front-End Web & Mobile Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** AWS Amplify · Amazon API Gateway · AWS Device Farm · Amazon Pinpoint

---

## Quick-Reference Matrix

| Service | Category | What It Does | Exam Trigger Words |
|---|---|---|---|
| **AWS Amplify** | Full-stack platform | Build, deploy, host web/mobile apps with backend | "host web app", "CI/CD for frontend", "GraphQL backend", "full-stack serverless" |
| **API Gateway** | API Management | Create, publish, manage REST/HTTP/WebSocket APIs | "REST API", "WebSocket", "API throttling", "Lambda integration", "API versioning" |
| **Device Farm** | Testing | Test mobile/web apps on real devices in AWS cloud | "mobile testing", "real devices", "browser testing", "automated test" |
| **Pinpoint** | Engagement | Multichannel customer communication (push, email, SMS) | "push notification", "SMS campaign", "user segmentation", "targeted messaging" |

---

---

## 1. AWS Amplify

### What Problem Does It Solve?

Building a full-stack web or mobile application requires assembling and configuring many AWS services — Cognito for auth, AppSync for APIs, S3 for storage, Lambda for backend logic, CloudFront for hosting. Doing this manually takes weeks. AWS Amplify provides a **unified platform** that abstracts all of this into a streamlined developer experience for building, deploying, and hosting full-stack apps.

**Real-World Example:**
> A startup wants to build a React e-commerce app with user authentication, a GraphQL product API, image upload to S3, and global CDN hosting. Without Amplify, they'd manually configure Cognito, AppSync, S3, Lambda, CloudFront, and set up CI/CD pipelines. With Amplify, the entire backend is provisioned via CLI commands, the frontend is hosted with automatic deployments from GitHub, and the app is live in hours instead of weeks.

---

### Amplify Has Two Distinct Components

#### Component 1: Amplify Hosting
A fully managed **CI/CD and hosting service** for web applications (similar to Netlify or Vercel, but on AWS).

| Feature | Details |
|---|---|
| **Source control** | Connect to GitHub, GitLab, Bitbucket, AWS CodeCommit |
| **CI/CD** | Auto-builds and deploys on every git push |
| **Framework support** | React, Vue, Angular, Next.js, Nuxt.js, Gatsby, Hugo, Jekyll, plain HTML |
| **Global CDN** | CloudFront-powered; content cached at edge locations worldwide |
| **Custom domains** | Free SSL via ACM; custom domain with Route 53 or external DNS |
| **Branch deployments** | Each git branch gets its own URL (e.g., `feature-login.xxxxx.amplifyapp.com`) |
| **Preview URLs** | Pull request previews — every PR gets a unique URL for review |
| **Password protection** | Protect staging branches with a username/password |
| **SSR support** | Server-side rendering for Next.js (runs on Lambda) |
| **Manual deploys** | Deploy static files directly (drag-and-drop or CLI) |

#### Component 2: Amplify Backend (Amplify Studio + CLI)
A **backend-as-a-service** provisioning platform that creates and manages AWS resources using CloudFormation under the hood.

| Amplify Category | Underlying AWS Service |
|---|---|
| **Authentication** | Amazon Cognito (User Pools + Identity Pools) |
| **API (GraphQL)** | AWS AppSync |
| **API (REST)** | Amazon API Gateway + Lambda |
| **Storage** | Amazon S3 |
| **Functions** | AWS Lambda |
| **DataStore** | AppSync + DynamoDB (offline sync) |
| **Hosting** | CloudFront + S3 (for static) or Lambda (for SSR) |
| **Notifications** | Amazon Pinpoint |
| **Analytics** | Amazon Pinpoint |
| **Predictions** | Amazon Rekognition, Transcribe, Translate |
| **Geo** | Amazon Location Service |

---

### Amplify Deployment Flow

```
Developer writes code
        ↓
git push to GitHub/GitLab/Bitbucket
        ↓
Amplify Hosting detects push
        ↓
Build phase (npm install, npm run build)
        ↓
Deploy phase (static files → S3 + CloudFront invalidation)
    or  (SSR → Lambda@Edge / Lambda)
        ↓
New version live at custom domain
        ↓
(Optional) Run E2E Cypress tests before promoting to production
```

---

### Amplify DataStore

One of Amplify's most powerful features — provides **offline-first data sync** for mobile and web apps:

- Local data stored on device using IndexedDB (web) or SQLite (mobile)
- Automatically syncs with AppSync + DynamoDB when connectivity is restored
- Conflict resolution strategies: Auto Merge, Optimistic Concurrency, Custom Lambda

> **Exam Tip:** "Offline-first mobile app with automatic sync when reconnected" → Amplify DataStore (built on AppSync + DynamoDB).

---

### Amplify vs Manual AWS Setup vs Firebase

| Feature | Amplify | Manual AWS | Firebase (Google) |
|---|---|---|---|
| **Setup speed** | Minutes (CLI) | Hours to days | Minutes |
| **AWS integration** | ✅ Native | ✅ Full | ❌ |
| **Customization** | High (via CloudFormation escape hatch) | Complete | Limited |
| **Offline sync** | ✅ DataStore | Manual | ✅ Firestore |
| **Vendor lock-in** | AWS | AWS | Google |
| **SSR hosting** | ✅ Next.js | Via custom setup | ❌ |
| **CI/CD built-in** | ✅ | Manual (CodePipeline) | ❌ |

---

### Amplify vs AWS Elastic Beanstalk vs S3 Static Hosting

| Scenario | Best Choice |
|---|---|
| Static HTML/CSS/JS site | S3 + CloudFront (or Amplify Hosting) |
| React/Vue/Angular SPA with CI/CD from GitHub | Amplify Hosting |
| Full-stack app with backend (auth, API, DB) | Amplify (full platform) |
| Containerized backend application | Elastic Beanstalk or ECS |
| Enterprise app needing full AWS control | Manual AWS configuration |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Host a React SPA with automatic deploys from GitHub | AWS Amplify Hosting |
| Full-stack mobile app with Cognito auth + GraphQL API + S3 storage | AWS Amplify (CLI/Studio provisions all backend) |
| Offline-first mobile app with automatic cloud sync | Amplify DataStore (AppSync + DynamoDB) |
| Preview each pull request at a unique URL before merging | Amplify Hosting branch previews |
| Next.js SSR app hosted on AWS with no server management | Amplify Hosting (SSR support with Lambda) |
| Migrate from Firebase to AWS with similar developer experience | AWS Amplify |

---

---

## 2. Amazon API Gateway

### What Problem Does It Solve?

APIs are the backbone of modern applications — they connect frontend clients to backend services, expose microservices, and enable third-party integrations. Building and operating an API at scale requires handling authentication, throttling, caching, versioning, SSL, CORS, monitoring, and billing. API Gateway provides a **fully managed service** that handles all of this without managing any servers.

**Real-World Example:**
> A mobile banking app needs to expose account balance, transaction history, and transfer endpoints to millions of users globally. API Gateway sits in front of Lambda functions — handling authentication (Cognito), rate limiting (100 req/sec per user), caching responses (5-minute TTL for account balance), SSL termination (ACM cert), and detailed CloudWatch monitoring — all without a single server to manage.

---

### API Gateway Types

| Type | Protocol | Use Case | Key Feature |
|---|---|---|---|
| **REST API** | HTTP/HTTPS | Full-featured REST APIs | Request/response transformation, caching, custom authorizers, usage plans |
| **HTTP API** | HTTP/HTTPS | Simple, low-latency APIs (70% cheaper than REST) | Faster, cheaper; limited features; native OIDC/JWT auth |
| **WebSocket API** | WebSocket | Real-time two-way communication | Persistent connections; route selections; chat, gaming, live feeds |

---

### REST API vs HTTP API (Most Tested Comparison)

| Feature | REST API | HTTP API |
|---|---|---|
| **Cost** | Higher (~$3.50/million) | Lower (~$1.00/million) — **~70% cheaper** |
| **Latency** | Higher | Lower |
| **Request/response transformation** | ✅ (mapping templates) | ❌ |
| **Caching** | ✅ (built-in, configurable TTL) | ❌ |
| **Usage plans + API keys** | ✅ | ❌ |
| **Custom domain** | ✅ | ✅ |
| **JWT/OIDC authorizer** | ❌ (use Lambda authorizer) | ✅ Native |
| **Lambda authorizer** | ✅ | ✅ |
| **Cognito authorizer** | ✅ | ✅ |
| **AWS WAF integration** | ✅ | ❌ |
| **Edge-optimized endpoint** | ✅ | ❌ |
| **VPC Link** | ✅ | ✅ |
| **Private API** | ✅ | ❌ |
| **Resource policies** | ✅ | ❌ |

> **Exam Rule:** "Need caching, WAF, usage plans, request/response transformation" → **REST API**. "Need low-cost, simple Lambda proxy, JWT auth" → **HTTP API**.

---

### WebSocket API

- Maintains **persistent bidirectional connections** between clients and backend.
- Route messages based on a **route selection expression** (e.g., `$request.body.action`).
- **Three built-in routes**: `$connect`, `$disconnect`, `$default`.
- Backend receives and sends messages via: Lambda, HTTP endpoint, AWS service integration.
- Use the **`@connections` API** to push messages to connected clients from the backend.

**Real-World Example:**
> A live collaborative document editor. When User A types, the WebSocket API routes the message to Lambda, which processes the change and uses the `@connections` API to push the update to all other connected users instantly.

---

### API Gateway Integration Types

| Integration Type | Description | Use Case |
|---|---|---|
| **Lambda Proxy** | API GW passes entire request to Lambda as-is; Lambda returns full response | Most common; no transformation needed |
| **Lambda Custom** | API GW transforms request/response using mapping templates (VTL) | Need to reshape request before Lambda |
| **HTTP Proxy** | Pass-through to HTTP backend | Wrap existing HTTP service |
| **HTTP Custom** | Transform + route to HTTP backend | Legacy backend with custom contract |
| **AWS Service** | Direct integration with AWS services (S3, SQS, DynamoDB, etc.) | Call AWS services without Lambda |
| **Mock** | Return static response without backend | API mocking, CORS preflight |

> **Exam Tip:** "PUT directly to SQS from API Gateway without Lambda" → **AWS Service Integration** (SQS). This avoids the cost and latency of an intermediate Lambda.

---

### API Gateway Endpoint Types

| Type | Description | Use Case |
|---|---|---|
| **Edge-Optimized** | Deployed to CloudFront; globally distributed | API clients distributed globally |
| **Regional** | Deployed in a single region; no CloudFront | Clients in same region; use your own CloudFront if needed |
| **Private** | Accessible only via VPC Interface Endpoint | Internal microservices; never exposed to internet |

---

### Authentication & Authorization

| Method | How It Works | Use Case |
|---|---|---|
| **API Keys** | Client sends `x-api-key` header | Simple auth; not for security; use for usage plans/throttling |
| **IAM Authorization** | Client signs request with SigV4 | AWS services / server-to-server calls |
| **Cognito User Pool Authorizer** | Validates JWT from Cognito User Pool | End-user mobile/web apps |
| **Lambda Authorizer (Token)** | Lambda validates bearer token (JWT, OAuth) | Custom auth logic, third-party IdP |
| **Lambda Authorizer (Request)** | Lambda validates headers, query params, etc. | Complex auth based on multiple inputs |
| **JWT Authorizer (HTTP API)** | Native JWT validation (OIDC/OAuth) | HTTP API only; simple JWT auth without Lambda |

---

### API Gateway Caching

- Cache API responses at the **stage level**.
- Cache capacity: 0.5 GB to 237 GB.
- Default TTL: **300 seconds** (min 0, max 3600).
- Clients can invalidate cache with `Cache-Control: max-age=0` header (requires IAM permission).
- Per-method override: enable/disable or set different TTL per method.

> **Exam Tip:** "Reduce backend calls and improve API performance" → Enable **API Gateway caching**. "Cache should vary per user" → Enable per-user cache key based on Cognito sub claim.

---

### Throttling & Usage Plans

| Concept | Description |
|---|---|
| **Account limit** | 10,000 requests/sec per region (default; adjustable) |
| **Stage-level throttling** | Set default rate + burst limit per stage |
| **Method-level throttling** | Override per specific method (e.g., `/payments` = 100 req/sec) |
| **Usage Plan** | Define rate limit + quota (daily/weekly/monthly) per API key |
| **API Key** | Clients include key in header; associated with a Usage Plan |

**Throttling Behavior:**
- **Rate limit**: Steady-state max requests per second (token bucket algorithm)
- **Burst limit**: Max concurrent requests at a spike (up to 5,000 by default)
- Throttled requests → HTTP **429 Too Many Requests**

---

### API Gateway Stages and Deployments

| Concept | Description |
|---|---|
| **Deployment** | Snapshot of API configuration at a point in time |
| **Stage** | Named reference to a deployment (e.g., `dev`, `staging`, `prod`) |
| **Stage Variables** | Key-value pairs injected at stage level (e.g., point `dev` stage to dev Lambda alias) |
| **Canary Release** | Send % of traffic to new deployment while keeping rest on current |

**Stage Variables Example:**
```
/prod stage → stageName = "prod" → Lambda alias = prod
/dev stage  → stageName = "dev"  → Lambda alias = dev

Lambda ARN in integration: arn:aws:lambda:...:function:MyFunc:${stageVariables.stageName}
```

---

### API Gateway Resource Policy

- **JSON policy** attached to the API controlling which principals (IAM users, accounts, IPs, VPCs) can invoke the API.
- Used for:
  - **Cross-account access**: Allow account 123456789012 to call your API
  - **IP whitelist/blacklist**: Allow only specific CIDR ranges
  - **VPC-only access**: Restrict to a specific VPC endpoint

---

### API Gateway Logging & Monitoring

| Feature | Description |
|---|---|
| **CloudWatch Metrics** | Count, Latency, IntegrationLatency, 4XXError, 5XXError, CacheHitCount |
| **Access Logs** | Log each API call (who called, when, latency, response code) → CloudWatch Logs |
| **Execution Logs** | Detailed request/response logging per stage (helpful for debugging) |
| **X-Ray tracing** | Distributed tracing from API Gateway through Lambda to downstream services |

---

### CORS (Cross-Origin Resource Sharing)

- Browser blocks requests from `app.example.com` to `api.example.com` unless API enables CORS.
- API Gateway handles CORS by:
  - Responding to **OPTIONS preflight** requests
  - Adding `Access-Control-Allow-Origin`, `Access-Control-Allow-Methods` headers
- For REST API: manually enable per resource or use **Mock integration** for OPTIONS.
- For HTTP API: built-in CORS configuration.

---

### API Gateway + VPC Link

- **VPC Link** enables API Gateway to privately connect to **resources inside a VPC** (NLB, ALB, ECS services).
- Traffic stays on AWS network; never traverses the internet.
- Used for: exposing private microservices via a public API without exposing the VPC.

```
Internet → API Gateway (REST/HTTP) → VPC Link → NLB → Private ECS Service
```

---

### API Gateway Exam Patterns

#### Pattern 1: Serverless REST API
```
Client → API Gateway (REST) → Lambda → DynamoDB
         (Cognito auth, throttling, caching)
```

#### Pattern 2: Direct AWS Service Integration (No Lambda)
```
Client → API Gateway → SQS Queue (enqueue message directly)
                     → DynamoDB PutItem (directly)
                     → SNS Publish (directly)
```

#### Pattern 3: Private Internal API
```
Internal Service → VPC Interface Endpoint → Private API Gateway → Lambda
(completely private; never leaves VPC)
```

#### Pattern 4: WebSocket Real-Time App
```
Browser Client → WebSocket API → Lambda ($connect / $disconnect / message routes)
                                      ↓
                             Lambda uses @connections API
                             to push to other connected clients
```

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Simple Lambda proxy API, minimize cost | HTTP API |
| Need WAF, caching, usage plans, request transformation | REST API |
| Real-time bidirectional chat app | WebSocket API |
| API accessible only from within VPC | Private endpoint type |
| Globally distributed API clients, low latency | Edge-optimized endpoint |
| Throttle one user group to 100 req/sec, another to 500 req/sec | Usage Plans with API Keys |
| Prevent 429 throttling from DynamoDB — buffer requests | API Gateway → SQS → Lambda → DynamoDB |
| POST data directly to SQS without Lambda | API Gateway AWS Service Integration (SQS) |
| Route `/v1` traffic to old Lambda, `/v2` to new Lambda | Stage Variables pointing to different Lambda aliases |
| Gradually roll out new API version | Canary release deployment |
| API Gateway + CloudFront for edge caching | Use Regional endpoint with your own CloudFront distribution |

---

### API Gateway vs ALB vs AppSync

| Feature | API Gateway | ALB | AppSync |
|---|---|---|---|
| **Protocol** | REST/HTTP/WebSocket | HTTP/HTTPS | GraphQL |
| **Auth built-in** | ✅ Cognito, IAM, Lambda | Limited | ✅ Cognito, IAM, Lambda, OIDC |
| **Throttling** | ✅ | ❌ (use WAF) | Limited |
| **Caching** | ✅ (REST API) | ❌ | ✅ |
| **WebSocket** | ✅ | ❌ | ✅ (subscriptions) |
| **Best for** | REST/HTTP microservices, Lambda APIs | Container/EC2 load balancing | GraphQL, real-time mobile apps |

---

---

## 3. AWS Device Farm

### What Problem Does It Solve?

Mobile apps must work correctly across **hundreds of different device models, OS versions, screen sizes, and network conditions**. Maintaining a physical device lab is expensive (~$50K+), time-consuming, and impossible to scale. Device Farm provides access to **real physical devices and browsers** in AWS data centers for automated and manual testing — without owning any hardware.

**Real-World Example:**
> A fintech company releases an iOS banking app. Before launch, Device Farm runs their Appium test suite simultaneously on 50 real iPhone and Android models — detecting a crash on Samsung Galaxy S21 running Android 12 before any users encounter it. Total test time: 20 minutes instead of days of manual testing.

---

### Device Farm Test Types

| Test Type | Description |
|---|---|
| **Automated testing** | Run test scripts/frameworks on multiple devices in parallel |
| **Remote access (manual)** | Interact with a real device in real time via browser (manual QA, exploratory testing) |

---

### Supported Testing Frameworks

#### Mobile Apps (iOS & Android)
| Framework | Language | Works With |
|---|---|---|
| **Appium** | Python, Java, Node.js, Ruby | iOS + Android |
| **XCTest** | Swift / Objective-C | iOS only |
| **Espresso** | Java / Kotlin | Android only |
| **Calabash** | Ruby | iOS + Android |
| **Built-in (Fuzz)** | N/A | Android; sends random inputs |
| **Instrumentation** | Java | Android |

#### Web Applications (Desktop Browser Testing)
| Feature | Details |
|---|---|
| **Browsers** | Chrome, Firefox |
| **Framework** | Selenium WebDriver |
| **Desktop Testing** | Test web apps on desktop browsers in real browser environments |

---

### How Device Farm Works

```
Developer uploads:
  ├── App package (.apk for Android, .ipa for iOS, or web URL)
  └── Test package (Appium script, XCTest bundle, Espresso APK, etc.)
            ↓
Select test configuration:
  ├── Device pool (curated sets of devices or custom selection)
  ├── Test framework
  └── Test data / files
            ↓
Device Farm runs tests in parallel on all selected devices simultaneously
            ↓
Results report:
  ├── Pass/fail per device
  ├── Screenshots at each test step
  ├── Video recordings of test execution
  ├── Device logs
  ├── Performance data (CPU, memory, network, battery)
  └── Crash reports
```

---

### Device Farm Pricing Models

| Model | How It Works | Best For |
|---|---|---|
| **Pay-as-you-go (metered)** | Pay per device-minute used | Infrequent testing |
| **Unmetered plan** | Fixed monthly fee; unlimited device minutes | High-volume CI/CD pipelines |

---

### Device Farm vs Real Device Labs vs Simulators/Emulators

| Feature | Device Farm | Physical Device Lab | Simulators/Emulators |
|---|---|---|---|
| **Real hardware** | ✅ Real devices | ✅ Real devices | ❌ Virtual only |
| **Maintenance** | AWS manages | You manage | N/A |
| **Scale** | Hundreds of devices | Limited by budget | Unlimited |
| **Cost** | Pay per minute | High capex | Free |
| **Accuracy** | High | Highest | Lower (no real sensors) |
| **Parallel testing** | ✅ Yes | Limited | ✅ Yes |
| **Network simulation** | ✅ (3G, 4G, WiFi) | Manual | Limited |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Test mobile app on 100 real device models before launch | AWS Device Farm |
| Run Appium tests automatically in CI/CD pipeline on real devices | Device Farm automated testing |
| QA engineer needs to manually interact with a real iOS device remotely | Device Farm remote access session |
| Test web app on real Chrome and Firefox browsers at scale | Device Farm desktop browser testing (Selenium) |
| Detect device-specific crashes before app store release | Device Farm parallel automated testing |

---

---

## 4. Amazon Pinpoint

### What Problem Does It Solve?

Modern apps need to communicate with users across multiple channels — push notifications for re-engagement, email for receipts, SMS for OTP/alerts, and in-app messages for onboarding. Building and scaling multichannel communication infrastructure is complex. Amazon Pinpoint provides a **managed multichannel customer engagement platform** with analytics, segmentation, and campaign orchestration.

**Real-World Example:**
> A ride-sharing app uses Pinpoint to: send an SMS one-time password during signup, send a push notification when a driver is 2 minutes away, email a receipt after each ride, and run a re-engagement campaign targeting users who haven't booked in 30 days with a 20% discount push notification — all from one service.

---

### Pinpoint Channels

| Channel | Description | Use Case |
|---|---|---|
| **Push Notifications** | iOS (APNs), Android (FCM), Baidu, Amazon Device Messaging | App re-engagement, transactional alerts |
| **Email** | SES-powered; bulk + transactional | Receipts, newsletters, marketing |
| **SMS** | Short Message Service (text messages) | OTP, alerts, transactional |
| **Voice** | Text-to-speech voice calls | OTP delivery, automated notifications |
| **In-App Messaging** | Messages displayed within the app | Onboarding, feature announcements |
| **Custom channels** | Lambda function as channel | Any custom delivery (WhatsApp, Slack, webhook) |

---

### Core Pinpoint Concepts

#### Endpoints
- An **endpoint** represents a specific destination for a message.
- Each endpoint has: **channel type** (SMS, email, push), **address** (phone number, email, device token), and **attributes** (user properties).
- One user can have multiple endpoints (email + phone + iOS device + Android device).

#### Segments
- Groups of endpoints with shared attributes.
- **Dynamic segments**: automatically updated based on endpoint attribute filters.
- **Imported segments**: upload a CSV/JSON file of endpoint addresses.

**Example Dynamic Segment:**
```
Users WHERE:
  Platform = iOS
  AND Country = "US"
  AND LastPurchase > 30 days ago
  AND AppVersion >= "3.0"
```

#### Campaigns
- Targeted messages sent to a **segment** at a scheduled time or triggered by an event.
- Can include **A/B testing** (split segment into treatment groups with different message variants).
- **Campaign limits**: control max messages per endpoint per 24 hours/campaign.

#### Journeys
- **Multi-step, event-driven** communication workflows.
- Users move through journey stages based on actions/events (like a drip campaign).

**Example Journey:**
```
User signs up → Wait 1 day → Send welcome email
                    → Did user complete profile? 
                        YES → Send "explore features" push
                        NO → Send "complete your profile" email → Wait 3 days → Send reminder SMS
```

#### Templates
- Reusable message templates with **dynamic attributes** (personalization).
- Support for: Email HTML templates, SMS templates, push notification templates.
- Use `{{User.UserAttributes.FirstName}}` for personalization.

---

### Pinpoint Analytics

Pinpoint has **built-in analytics** for tracking:

| Metric | Description |
|---|---|
| **Daily Active Users (DAU)** | Users who opened app today |
| **Monthly Active Users (MAU)** | Users who opened app this month |
| **Session metrics** | Session count, length, interval |
| **Funnel analytics** | Track conversion through multi-step flows |
| **Campaign metrics** | Sends, deliveries, opens, clicks, opt-outs |
| **Revenue** | Track in-app purchase revenue |
| **Custom events** | Track any app event (add_to_cart, checkout, etc.) |

---

### Pinpoint vs Amazon SNS

> **One of the most important comparisons for the exam.**

| Feature | Amazon Pinpoint | Amazon SNS |
|---|---|---|
| **Primary use** | Marketing + transactional user engagement | System-to-system pub/sub notifications |
| **Targeting** | User segmentation, behavioral targeting | Topic-based (broadcast) |
| **Channels** | Push, Email, SMS, Voice, In-App, Custom | Push, SMS, Email, SQS, Lambda, HTTP |
| **Analytics** | ✅ Built-in user analytics, campaign metrics | ❌ |
| **Campaigns** | ✅ Scheduled + triggered campaigns | ❌ |
| **Journeys** | ✅ Multi-step workflows | ❌ |
| **A/B testing** | ✅ | ❌ |
| **Audience** | End-users (consumers) | Applications/systems |
| **Personalization** | ✅ Per-user attributes | ❌ |
| **Use case** | "Re-engage users who haven't opened app in 7 days" | "Fan out S3 events to multiple Lambda functions" |

> **Exam Rule:**
> - "Targeted push notification to specific users based on behavior" → **Pinpoint**
> - "Fan out a system event to multiple services" → **SNS**
> - "Send bulk marketing email to 1 million users" → **Pinpoint** (with SES under the hood)
> - "Send transactional email from an application" → **SES** directly

---

### Pinpoint vs Amazon SES

| Feature | Pinpoint | Amazon SES |
|---|---|---|
| **Type** | Multichannel engagement platform | Email-only service |
| **Marketing campaigns** | ✅ | ❌ |
| **Analytics** | ✅ Built-in | Limited (via CloudWatch) |
| **Segmentation** | ✅ | ❌ |
| **Transactional email** | ✅ | ✅ (primary use) |
| **A/B testing** | ✅ | ❌ |
| **Under the hood** | Uses SES for email delivery | Is the email service |
| **Best for** | Marketing + engagement campaigns | High-volume transactional email from apps |

---

### Pinpoint Transactional vs Campaign Messaging

| Type | Description | Example |
|---|---|---|
| **Transactional** | Immediate, triggered by user action | OTP SMS, order confirmation email |
| **Campaign** | Scheduled batch to a segment | Weekly newsletter, re-engagement push |

---

### Pinpoint Integration with AWS Services

| Service | Integration |
|---|---|
| **AWS Amplify** | Amplify Notifications category uses Pinpoint for push notifications and analytics |
| **Amazon Kinesis** | Stream Pinpoint event data to Kinesis for custom analytics |
| **Amazon S3** | Import endpoint segments from S3 CSV/JSON files |
| **AWS Lambda** | Custom channel delivery via Lambda; event-based journey triggers |
| **Amazon SES** | Email delivery engine behind Pinpoint email channel |
| **Amazon EventBridge** | Emit Pinpoint events to EventBridge for further automation |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Send targeted push notifications to users who haven't opened app in 7 days | Amazon Pinpoint (dynamic segment + campaign) |
| Send SMS OTP during user registration | Amazon Pinpoint (transactional SMS) |
| A/B test two email subject lines with 10% of users before full campaign send | Amazon Pinpoint A/B campaign |
| Multi-step onboarding email sequence triggered by user events | Amazon Pinpoint Journeys |
| Track daily/monthly active users in a mobile app | Amazon Pinpoint analytics (DAU/MAU) |
| Fan out application event to SQS, Lambda, and HTTP endpoint | Amazon SNS (not Pinpoint) |
| Send marketing email to 5 million subscribers with analytics | Amazon Pinpoint email campaign |
| Send transactional receipt email from backend application code | Amazon SES (not Pinpoint) |
| Mobile app needs analytics + push notifications from one service | Amazon Pinpoint (via Amplify or direct SDK) |

---

---

## Master Comparison: All Four Services

### By Role in a Mobile App Architecture

```
User interacts with mobile/web app
            │
┌───────────┴──────────────────────────────────────────────┐
│                  AWS Amplify                             │
│  (auth → Cognito, API → AppSync/APIGW, storage → S3,    │
│   hosting → CloudFront, analytics/notifications → Pinpoint)│
└───────────────────┬──────────────────────────────────────┘
                    │
           API calls to backend
                    │
┌───────────────────▼──────────────────────────────────────┐
│              Amazon API Gateway                          │
│  (REST/HTTP/WebSocket API → Lambda → DynamoDB/RDS/SQS)   │
└───────────────────┬──────────────────────────────────────┘
                    │
           Before release:
                    │
┌───────────────────▼──────────────────────────────────────┐
│              AWS Device Farm                             │
│  (Test app on 100+ real devices, Appium/XCTest, parallel)│
└───────────────────┬──────────────────────────────────────┘
                    │
           After launch — user engagement:
                    │
┌───────────────────▼──────────────────────────────────────┐
│             Amazon Pinpoint                              │
│  (push notifications, email, SMS, journeys, analytics)   │
└──────────────────────────────────────────────────────────┘
```

---

### Decision Framework for Exam Questions

```
"Build and deploy a frontend web/mobile app with backend"
    → AWS Amplify

"Create a managed API endpoint for Lambda/microservices"
    → API Gateway (REST for full features; HTTP for low-cost; WebSocket for real-time)

"Test mobile app on real physical devices automatically"
    → AWS Device Farm

"Send targeted push/email/SMS to specific user segments"
    → Amazon Pinpoint

"Fan out system events to multiple AWS services"
    → Amazon SNS (not Pinpoint)

"Send transactional email from code"
    → Amazon SES (not Pinpoint)
```

---

## Architecture Patterns to Know

### Pattern 1: Full Serverless Mobile App
```
Mobile App (React Native + Amplify SDK)
        ↓ (auth)
Amazon Cognito (via Amplify Auth)
        ↓ (API calls)
API Gateway (REST) → Lambda → DynamoDB
        ↓ (file uploads)
Amazon S3 (via Amplify Storage)
        ↓ (push notifications + analytics)
Amazon Pinpoint (via Amplify Notifications)
        ↓ (CI/CD for backend)
AWS CodePipeline → CloudFormation
        ↓ (frontend hosting)
Amplify Hosting → CloudFront
```

### Pattern 2: Real-Time Chat Application
```
Browser/App → API Gateway (WebSocket API)
                    ├── $connect → Lambda (store connectionId in DynamoDB)
                    ├── sendMessage → Lambda (read all connections from DynamoDB)
                    │                       → @connections API → push to all clients
                    └── $disconnect → Lambda (remove connectionId from DynamoDB)
```

### Pattern 3: API Gateway Direct SQS Integration
```
Mobile Client → POST /order → API Gateway
                                    ↓ (AWS Service Integration — no Lambda)
                               SQS Queue
                                    ↓ (Lambda reads from SQS)
                               Lambda (process order)
                                    ↓
                               DynamoDB (store order)
                                    ↓
                               Pinpoint (send confirmation push notification)
```

### Pattern 4: Pre-Launch Mobile Testing Pipeline
```
Developer commits code → CodePipeline
        ↓
CodeBuild (build .apk / .ipa)
        ↓
Device Farm (run Appium tests on 50 real devices in parallel)
        ↓
Test results: all pass?
    YES → Deploy to production app stores
    NO  → Notify team via SNS; block deployment
```

### Pattern 5: User Re-Engagement Campaign
```
User hasn't opened app in 7 days
        ↓ (tracked by Pinpoint analytics)
Pinpoint Dynamic Segment: "inactive_7_days"
        ↓
Pinpoint Campaign (triggered at 10am user's timezone):
  └── Push notification: "We miss you! Here's 20% off your next order"
        ↓
User opens app → Pinpoint event tracked: "app_open"
        ↓
Journey: User added to "re-engaged" flow
  ├── Wait 1 hour → Did they purchase?
  │     YES → Send thank you email
  │     NO  → Send SMS reminder with promo code
```

---

## Exam Tips Summary

### AWS Amplify
- ✅ Hosting = CI/CD + CDN for frontend (connects to GitHub/GitLab/Bitbucket)
- ✅ Backend = provisions Cognito, AppSync, API Gateway, Lambda, S3 via CLI/Studio
- ✅ DataStore = offline-first sync built on AppSync + DynamoDB
- ✅ Branch previews = unique URL per git branch/PR
- ✅ SSR support for Next.js (runs on Lambda under the hood)
- ✅ Amplify is an abstraction layer — underlying services are standard AWS services

### API Gateway
- ✅ REST API = full features (caching, WAF, usage plans, request transformation) — higher cost
- ✅ HTTP API = lightweight, fast, cheap (~70% less) — JWT auth native, no caching
- ✅ WebSocket API = persistent bidirectional; $connect / $disconnect / $default routes
- ✅ Edge-optimized = CloudFront backed; Regional = single region; Private = VPC only
- ✅ Lambda Proxy = pass everything to Lambda as-is (most common)
- ✅ AWS Service Integration = call SQS/DynamoDB/SNS directly without Lambda
- ✅ Usage Plans + API Keys = throttle and quota per consumer (not per IAM)
- ✅ VPC Link = API Gateway → private NLB → private backend in VPC
- ✅ 429 = throttled; use SQS buffer pattern to absorb traffic spikes
- ✅ Canary deployments = % traffic to new stage; stage variables = point to env-specific Lambda aliases

### AWS Device Farm
- ✅ Real physical devices (not simulators/emulators) in AWS data centers
- ✅ Parallel testing = run on many devices simultaneously = fast
- ✅ Supports: Appium, XCTest, Espresso, Calabash, Selenium (web)
- ✅ Remote access = manual interactive testing on real device via browser
- ✅ Captures: screenshots, videos, logs, performance data per device per test
- ✅ Metered (pay per minute) vs Unmetered (fixed monthly fee for CI/CD)

### Amazon Pinpoint
- ✅ Channels: Push (APNs/FCM), Email (SES), SMS, Voice, In-App, Custom (Lambda)
- ✅ Pinpoint ≠ SNS: Pinpoint = user engagement/marketing; SNS = system pub/sub
- ✅ Pinpoint ≠ SES: Pinpoint = multichannel campaigns; SES = transactional email API
- ✅ Endpoints = individual destinations (one user → multiple endpoints)
- ✅ Segments = filtered groups of endpoints (dynamic or imported)
- ✅ Campaigns = scheduled or event-triggered message to a segment
- ✅ Journeys = multi-step event-driven workflows (drip campaigns)
- ✅ A/B testing built into campaigns
- ✅ Built-in analytics: DAU, MAU, session data, funnel, revenue tracking
- ✅ Amplify Notifications category uses Pinpoint under the hood
