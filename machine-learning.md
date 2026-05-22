# AWS Exam Study Guide: Machine Learning Services
> **Target Exams:** AWS Solutions Architect Associate (SAA-C03) & Professional (SAP-C02)
> **Services Covered:** Comprehend · Forecast · Fraud Detector · Kendra · Lex · Polly · Rekognition · SageMaker · Textract · Transcribe · Translate

---

## ML Services Quick-Reference Matrix

| Service | Input | Output | Exam Trigger Words |
|---|---|---|---|
| **Comprehend** | Text | Entities, sentiment, key phrases, topics | "NLP", "sentiment analysis", "entity extraction", "text classification" |
| **Forecast** | Time-series data | Future predictions | "demand forecasting", "predict future values", "time-series" |
| **Fraud Detector** | Events/transactions | Fraud probability score | "fraud detection", "account takeover", "online transactions" |
| **Kendra** | Documents/data sources | Search results (intelligent) | "enterprise search", "natural language search", "find answers in docs" |
| **Lex** | Voice/text | Intent + slots (conversational AI) | "chatbot", "voice bot", "conversational interface", "Alexa" |
| **Polly** | Text | Audio (speech) | "text-to-speech", "voice", "convert text to audio" |
| **Rekognition** | Images/Video | Labels, faces, text, moderation | "image analysis", "face detection", "content moderation", "video analysis" |
| **SageMaker** | Data + code | Trained ML models, endpoints | "build ML model", "train", "deploy", "custom ML", "notebooks" |
| **Textract** | Documents (images/PDF) | Structured text + forms + tables | "extract text from PDF", "form extraction", "OCR", "document analysis" |
| **Transcribe** | Audio | Text transcript | "speech-to-text", "transcribe audio", "subtitles", "call center audio" |
| **Translate** | Text | Translated text | "language translation", "multilingual", "translate content" |

---

## ML Service Categories (for Exam Context)

```
PRE-BUILT AI SERVICES (no ML expertise needed — just call the API):
  Vision:     Rekognition, Textract
  Speech:     Transcribe (speech→text), Polly (text→speech)
  Language:   Comprehend, Translate, Lex
  Search:     Kendra
  Forecasting: Forecast
  Fraud:      Fraud Detector

CUSTOM ML PLATFORM (ML expertise needed):
  SageMaker — build, train, tune, deploy your own models
```

---

---

## 1. Amazon Comprehend

### What Problem Does It Solve?
Understanding the meaning inside large amounts of unstructured text — customer reviews, support tickets, social media, legal documents — is impossible to do manually at scale. Comprehend applies **Natural Language Processing (NLP)** to automatically extract meaning, sentiment, entities, and topics from text without any ML expertise.

**Real-World Example:**
> An airline processes 50,000 customer support emails daily. Comprehend automatically detects that 60% have negative sentiment, identifies entities like "flight numbers" and "cities", classifies tickets into categories (delay, lost baggage, refund), and routes them to the right team — reducing manual triage from 5 hours to seconds.

---

### Comprehend Capabilities

| Capability | Description | Example Output |
|---|---|---|
| **Sentiment Analysis** | Positive, Negative, Neutral, Mixed | `"Your service was terrible"` → NEGATIVE (0.97) |
| **Entity Recognition** | Identifies named entities | `"Amazon in Seattle"` → ORG: Amazon, LOC: Seattle |
| **Key Phrase Extraction** | Identifies important phrases | `"machine learning platform"`, `"cost reduction"` |
| **Language Detection** | Identifies language of text | `"Bonjour"` → French (0.99) |
| **Syntax Analysis** | Parts of speech tagging | Nouns, verbs, adjectives per word |
| **Topic Modeling** | Groups documents by topic (unsupervised) | Document cluster: "product quality", "shipping" |
| **Custom Classification** | Train custom text classifier on your categories | "billing", "technical support", "sales" |
| **Custom Entity Recognition** | Train to recognize your domain-specific entities | Product codes, internal IDs, medical terms |
| **PII Detection** | Identify/redact Personally Identifiable Information | SSNs, credit card numbers, emails in text |
| **Targeted Sentiment** | Sentiment about specific entities | "The camera is great but battery is terrible" → camera=POSITIVE, battery=NEGATIVE |

---

### Comprehend Medical

- A specialized version for **healthcare and medical text**.
- Extracts: medical conditions, medications, dosages, treatments, anatomy, test results.
- Use case: Analyze clinical notes, medical records, drug adverse event reports.

---

### Comprehend vs Kendra vs Lex

| Feature | Comprehend | Kendra | Lex |
|---|---|---|---|
| **Input** | Raw text | Documents / data sources | Conversational input (text/voice) |
| **Output** | Entities, sentiment, topics | Search results and answers | Intent + slots (structured) |
| **Use case** | Analyze text content | Find answers in documents | Build chatbots/voice bots |
| **User interaction** | ❌ Batch analysis | ❌ Search queries | ✅ Multi-turn conversation |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Analyze 10,000 product reviews to find common complaints | Comprehend (sentiment + key phrases) |
| Detect PII in documents before storing in S3 | Comprehend PII detection |
| Route support tickets to correct team based on content | Comprehend custom classification |
| Extract medical conditions from clinical notes | Comprehend Medical |
| Build a chatbot for customer service | Amazon Lex (not Comprehend) |

---

---

## 2. Amazon Forecast

### What Problem Does It Solve?
Predicting future values — demand, sales, web traffic, inventory, energy consumption — requires sophisticated time-series ML models. Traditionally this required data scientists and weeks of model tuning. Amazon Forecast applies the **same ML technology used by Amazon.com** to automatically build accurate forecasting models from your historical data.

**Real-World Example:**
> A retail chain uploads 3 years of weekly product sales data. Forecast automatically trains multiple models (DeepAR, ARIMA, Prophet, ETS), selects the best one, and generates 52-week demand predictions per product per store — enabling optimized inventory ordering that reduces stockouts by 30% and overstock costs by 25%.

---

### How Forecast Works

```
Input Data (upload to S3):
  ├── Target time series: historical values you want to predict
  │   (e.g., weekly sales per product per store)
  ├── Related time series (optional): factors that affect target
  │   (e.g., promotions, price changes, holidays)
  └── Item metadata (optional): attributes about items
      (e.g., product category, brand, color)
            ↓
Dataset Group: combines all datasets
            ↓
Predictor (AutoML or choose algorithm):
  ├── ARIMA
  ├── ETS (Exponential Smoothing)
  ├── NPTS (Non-Parametric Time Series)
  ├── DeepAR+ (deep learning — Amazon's proprietary)
  ├── Prophet (Facebook open source)
  └── AutoML (Forecast selects best automatically)
            ↓
Forecast (generated predictions):
  ├── Point forecast (single most likely value)
  └── Probabilistic forecast (P10, P50, P90 quantiles)
            ↓
Query via API or export to S3
```

---

### Key Forecast Concepts

| Concept | Description |
|---|---|
| **Dataset Group** | Container for all related datasets |
| **Target Time Series** | Historical values being predicted (required) |
| **Related Time Series** | External factors (e.g., promotions) that affect predictions |
| **Predictor** | Trained ML model; Forecast trains multiple and picks best |
| **Forecast** | Generated predictions from a predictor |
| **Forecast Horizon** | How far into the future to predict |
| **Forecast Frequency** | Resolution of predictions (daily, weekly, monthly) |
| **Quantile Loss** | Accuracy metric; P50 = median, P90 = 90th percentile |

---

### Forecast vs SageMaker for Time Series

| Feature | Amazon Forecast | SageMaker (Custom) |
|---|---|---|
| **ML expertise needed** | ❌ None | ✅ Data scientist required |
| **Setup** | Upload data, click create | Build, train, tune model code |
| **Algorithms** | Pre-built (ARIMA, DeepAR+, Prophet) | Any algorithm (TensorFlow, XGBoost, etc.) |
| **Use case** | Time-series forecasting specifically | Any ML problem |
| **Custom features** | Limited | Unlimited |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Predict next quarter's product demand without data scientists | Amazon Forecast |
| Forecast energy consumption for next 30 days | Amazon Forecast |
| Predict website traffic for capacity planning | Amazon Forecast |
| Build a custom recommendation engine | Amazon SageMaker |

---

---

## 3. Amazon Fraud Detector

### What Problem Does It Solve?
Online fraud — fake account creation, payment fraud, account takeover, promotion abuse — costs businesses billions annually. Building fraud detection models requires labeled fraud data and ML expertise. Fraud Detector uses **Amazon's own fraud detection experience** and ML to automatically build, train, and deploy fraud detection models.

**Real-World Example:**
> An e-commerce company loses $2M annually to fraudulent orders. Fraud Detector analyzes patterns in 2 years of order history (marked as fraudulent or legitimate), builds a real-time scoring model, and flags new orders with a fraud probability score in milliseconds. Orders above 0.8 score are blocked; 0.5–0.8 go to manual review; below 0.5 are auto-approved — reducing fraud losses by 80%.

---

### Fraud Detector Key Concepts

| Concept | Description |
|---|---|
| **Event** | A business action to evaluate (order, login, account registration) |
| **Entity** | Who is performing the event (customer, IP address, device) |
| **Variable** | Input attribute for the model (IP, email, purchase amount, device ID) |
| **Model** | ML model trained on your labeled fraud data |
| **Detector** | Evaluates events in real time using models + rules |
| **Rules** | Business logic to interpret model output (if score > 0.8, block) |
| **Outcome** | Result of evaluation (approve, review, block) |

---

### Fraud Detector Model Types

| Model Type | Use Case |
|---|---|
| **Online Fraud Insights (OFI)** | New account fraud, online payment fraud |
| **Transaction Fraud Insights (TFI)** | Credit card / payment transaction fraud |
| **Account Takeover Insights (ATI)** | Login anomaly detection, credential stuffing |

---

### How Fraud Detector Works

```
Historical Events + Labels (fraud/not-fraud) → stored in S3
            ↓
Create Detector + Variables
            ↓
Train Model (Fraud Detector auto-trains ML model)
            ↓
Define Rules (e.g., if model_score > 0.80 → BLOCK)
            ↓
Deploy Detector
            ↓
Real-time API call with event data:
  {
    "email": "user@example.com",
    "ip_address": "192.168.1.1",
    "order_amount": 1500,
    "billing_zip": "10001"
  }
            ↓
Response: { "outcome": "BLOCK", "model_score": 0.94 }
```

---

### Fraud Detector vs SageMaker for Fraud

| Feature | Fraud Detector | SageMaker |
|---|---|---|
| **Domain expertise** | Built-in fraud detection expertise | General ML; you build fraud logic |
| **Setup** | Point to data, train, deploy | Build data pipeline + model code |
| **Real-time API** | ✅ Native | ✅ (SageMaker endpoint) |
| **Interpretability** | ✅ (explains which variables drove score) | Manual implementation |
| **Use case** | Online fraud specifically | Any classification problem |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Detect fraudulent online orders in real time | Amazon Fraud Detector |
| Detect account takeover attempts at login | Fraud Detector (ATI model) |
| Score new user registrations for fraud risk | Fraud Detector (OFI model) |
| Build custom fraud model with domain-specific features | SageMaker |

---

---

## 4. Amazon Kendra

### What Problem Does It Solve?
Traditional keyword-based enterprise search returns results containing matching words but cannot understand meaning. Employees waste hours searching SharePoint, Confluence, S3, databases for answers. Kendra provides **intelligent enterprise search** using ML to understand natural language questions and return precise answers from across your organization's documents.

**Real-World Example:**
> An insurance company has 500,000 documents in SharePoint, S3, and a Salesforce knowledge base. An agent asks: *"What is the maximum liability coverage for commercial trucks in California?"* Traditional search returns 200 documents containing those words. Kendra returns the exact paragraph from the correct policy document with the answer highlighted — in under 1 second.

---

### Kendra Data Source Connectors

Kendra can index content from:

| Data Source | Examples |
|---|---|
| **Amazon S3** | PDFs, Word, PowerPoint, HTML, text files |
| **SharePoint** | Microsoft SharePoint Online and Server |
| **Salesforce** | Knowledge articles, cases |
| **ServiceNow** | Knowledge base, incidents |
| **RDS / Aurora** | Database content |
| **OneDrive** | Microsoft OneDrive |
| **Confluence** | Atlassian Confluence pages |
| **Google Drive** | Documents, sheets |
| **Custom data source** | Via API |

---

### Kendra Features

| Feature | Description |
|---|---|
| **Semantic search** | Understands meaning, not just keywords |
| **FAQ matching** | Match questions to pre-defined Q&A pairs |
| **Document ranking** | Relevance tuning — boost specific documents |
| **Access control** | ACL filtering; users only see authorized content |
| **Incremental sync** | Automatically re-index changed documents |
| **Query suggestions** | Auto-complete and suggested queries |
| **Experience Builder** | Build search UI without coding |
| **Custom attributes** | Add metadata for filtering results |

---

### Kendra vs OpenSearch vs Comprehend

| Feature | Kendra | OpenSearch | Comprehend |
|---|---|---|---|
| **Search type** | Semantic / intelligent | Keyword + full-text | NLP analysis (not search) |
| **Input** | Documents, data sources | Any data | Raw text only |
| **Natural language** | ✅ Understands questions | Limited | ✅ (analysis) |
| **Setup** | Managed, connectors | Manual index setup | API call |
| **Use case** | Enterprise document search | Log search, e-commerce | Text analysis pipeline |
| **Cost** | High (Enterprise) | Moderate | Per API call |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Employees search across SharePoint, S3, Confluence with natural language | Amazon Kendra |
| "Find all HR policies about parental leave" from 10,000 documents | Amazon Kendra |
| Build a customer-facing FAQ bot backed by a knowledge base | Kendra (FAQ feature) + Lex (chatbot interface) |
| Log search and analytics dashboard | OpenSearch (not Kendra) |

---

---

## 5. Amazon Lex

### What Problem Does It Solve?
Building conversational chatbots and voice interfaces requires understanding user intent from natural language, managing multi-turn conversations, and integrating with backend systems. Lex (the same technology behind **Amazon Alexa**) provides a **managed service** to build conversational interfaces for any application using voice or text.

**Real-World Example:**
> A bank builds a customer service bot: customers say or type *"I want to check my account balance"* → Lex understands the intent (CheckBalance), asks for the account number (slot), validates it, calls a Lambda function to query the database, and responds *"Your checking account balance is $1,247.50"* — handling thousands of simultaneous conversations without any human agents.

---

### Core Lex Concepts

| Concept | Description | Example |
|---|---|---|
| **Intent** | What the user wants to do | `CheckBalance`, `BookFlight`, `OrderPizza` |
| **Utterances** | Sample phrases that trigger the intent | "What's my balance", "Show me my account", "Check balance" |
| **Slot** | A piece of information needed to fulfill the intent | AccountType (checking/savings), Date, City |
| **Slot Type** | Data type for a slot | Built-in (date, number, city) or custom (pizza size: small/medium/large) |
| **Fulfillment** | What happens when all slots are collected | Lambda function invocation |
| **Confirmation Prompt** | Ask user to confirm before fulfilling | "You want to transfer $500 to savings. Is that right?" |
| **Elicitation** | Asking for a missing slot | "Which account do you want to check — checking or savings?" |
| **Session Attributes** | Data passed between turns | User ID, conversation context |
| **Bot alias** | Version pointer for the bot | `PROD`, `DEV` pointing to different bot versions |

---

### Lex V2 vs Lex V1

| Feature | Lex V2 (Current) | Lex V1 (Legacy) |
|---|---|---|
| **Languages** | Multi-language in one bot | One language per bot |
| **Streaming** | ✅ Streaming conversations | ❌ |
| **Conversation flow** | Visual builder | JSON config |
| **Intent confirmation** | Improved | Basic |
| **Use** | All new bots | Legacy only |

---

### Lex Integration Points

| Service | Role |
|---|---|
| **Lambda** | Fulfillment (execute business logic, query DB) |
| **API Gateway** | Expose Lex bot via REST API for web chat |
| **Amazon Connect** | Build IVR (Interactive Voice Response) for call centers |
| **Kendra** | Use Kendra as knowledge base for FAQ answers in Lex |
| **Polly** | Lex uses Polly for text-to-speech responses |
| **Transcribe** | Converts voice input to text for Lex to process |
| **CloudWatch** | Monitor conversation metrics |
| **S3** | Store conversation logs |

---

### Lex Multi-Turn Conversation Flow

```
User: "I want to book a flight"
        ↓
Lex identifies Intent: BookFlight
        ↓
Lex checks required Slots:
  - Origin city: ❌ missing → "Where are you flying from?"
User: "New York"
  - Origin city: ✅ JFK
  - Destination: ❌ missing → "Where would you like to go?"
User: "Los Angeles"
  - Destination: ✅ LAX
  - Travel date: ❌ missing → "When do you want to travel?"
User: "Next Friday"
  - Date: ✅ 2026-05-29
        ↓
Confirmation: "Book flight from JFK to LAX on May 29. Confirm?"
User: "Yes"
        ↓
Fulfillment: Lambda function → search flights → return options
        ↓
Response: "I found 3 flights. The cheapest is $299 departing at 8am."
```

---

### Lex vs Comprehend vs Kendra (Conversational Context)

| Need | Service |
|---|---|
| "Build a multi-turn chatbot" | Lex |
| "Analyze sentiment of chatbot conversations" | Comprehend |
| "Answer questions from documents in a chatbot" | Lex + Kendra integration |
| "Classify customer support emails" | Comprehend |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Build a customer service chatbot for a banking app | Amazon Lex |
| Build an Alexa Skill for ordering products | Amazon Lex (same technology) |
| Call center IVR that understands natural language | Amazon Lex + Amazon Connect |
| Chatbot that answers questions from HR policy documents | Amazon Lex + Amazon Kendra |
| Voice-enabled bot responses as audio | Lex + Polly (text-to-speech) |

---

---

## 6. Amazon Polly

### What Problem Does It Solve?
Applications need to speak to users — accessibility tools, e-learning platforms, navigation systems, IVR systems. Recording human voice for every possible response is expensive and inflexible. Polly converts **text to lifelike speech** using deep learning, supporting dozens of languages and voices.

**Real-World Example:**
> An e-learning platform automatically converts course text into audio using Polly's neural voice, creating a podcast-style audio track for every lesson. Students can listen while commuting — and when course text is updated, the audio is automatically regenerated without re-recording.

---

### Polly Voice Types

| Type | Description | Quality |
|---|---|---|
| **Standard voices** | Concatenative TTS; older technology | Good |
| **Neural voices (NTTS)** | Deep learning; more natural and human-like | Excellent |
| **Long-form** | Optimized for reading long-form content (news articles, books) | Best for extended content |
| **Brand voices** | Custom voice trained on your brand's recordings | Enterprise (contact AWS) |
| **Generative voices** | Most expressive; highest quality; newest | Best overall expressiveness |

---

### Polly Key Features

| Feature | Description |
|---|---|
| **Languages** | 30+ languages, 60+ voices |
| **SSML support** | Speech Synthesis Markup Language — control pronunciation, rate, pitch, pauses |
| **Lexicons** | Custom pronunciation dictionary (e.g., "AWS" → "Amazon Web Services") |
| **Speech marks** | Returns metadata about word/phoneme timing (for lip sync, karaoke) |
| **Output format** | MP3, OGG, PCM |
| **Streaming** | Returns audio stream synchronously (real-time playback) |
| **Storage** | Save output as MP3 in S3 for reuse |

---

### SSML Example

```xml
<speak>
  Welcome to <emphasis level="strong">AWS Polly</emphasis>.
  <break time="1s"/>
  Your account balance is
  <say-as interpret-as="currency">$1,247.50</say-as>.
  <prosody rate="slow">Please review your statement carefully.</prosody>
</speak>
```

---

### Polly vs Transcribe (Common Confusion)

| Service | Direction | Use Case |
|---|---|---|
| **Polly** | Text → Speech (audio) | "Make the app talk" |
| **Transcribe** | Speech (audio) → Text | "Transcribe what was said" |

> **Mnemonic:** Polly **speaks** (like a parrot). Transcribe **listens and writes**.

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Convert news articles to audio podcast format | Amazon Polly |
| Generate voice prompts for an IVR system | Amazon Polly |
| Accessibility feature: read website text aloud | Amazon Polly |
| Convert call center recordings to text | Amazon Transcribe (not Polly) |
| Generate speech with custom pronunciation for brand terms | Polly with custom Lexicons |

---

---

## 7. Amazon Rekognition

### What Problem Does It Solve?
Analyzing images and videos manually — detecting objects, faces, inappropriate content, or reading text — is impossible at scale. Rekognition provides **pre-trained computer vision** models that analyze images and videos via a simple API call, with no ML expertise required.

**Real-World Example:**
> A social media platform receives 10 million photo uploads per day. Rekognition automatically: (1) Detects and blurs nudity/violence before any human sees it, (2) Groups photos by person (face recognition), (3) Extracts searchable tags ("beach", "dog", "sunset") from each photo, (4) Detects and indexes text in signs and memes — all in real time.

---

### Rekognition Capabilities (Images)

| Capability | Description | Example |
|---|---|---|
| **Label Detection** | Detect objects, scenes, activities | "Dog", "Beach", "Swimming", "Person" |
| **Face Detection** | Detect faces and attributes | Age range, gender, emotions, glasses, smile |
| **Face Comparison** | Compare two faces for similarity | "Is this the same person?" (0.0–100.0 similarity) |
| **Face Search (Collections)** | Search a face against a collection | Security: match against employee database |
| **Text Detection** | Read text in images | Signs, license plates, book covers |
| **Content Moderation** | Detect inappropriate content | Nudity, violence, drugs, alcohol — with confidence score |
| **Celebrity Recognition** | Identify famous people | "This is Elon Musk (99.8% confidence)" |
| **Custom Labels** | Train on your own images | Detect your specific products, defects, logos |
| **PPE Detection** | Detect Personal Protective Equipment | Hardhats, vests, masks on construction site |

---

### Rekognition Capabilities (Video)

| Capability | Description |
|---|---|
| **Label Detection** | Objects and scenes in video frames |
| **Face Detection + Recognition** | Faces across video frames over time |
| **Content Moderation** | Detect inappropriate content in video |
| **Person Tracking** | Track a person's path through video |
| **Celebrity Recognition** | Identify celebrities in video |
| **Technical Cue Detection** | Detect scene changes, black frames, credits |
| **Text Detection** | Read text appearing in video |
| **Activity Detection** | Detect activities (e.g., "drinking", "hugging") |

---

### Rekognition Face Collections

- A **collection** is a database of indexed faces.
- Add faces with `IndexFaces` → search with `SearchFacesByImage`.
- Use case: Employee badge verification, unauthorized person detection, finding a person in hours of surveillance video.

```
Add employee photos to collection (IndexFaces)
        ↓
Security camera captures frame
        ↓
SearchFacesByImage → match against collection
        ↓
MATCH: John Smith (similarity: 98.2%) → grant access
NO MATCH → trigger alert
```

---

### Rekognition Video Analysis (Async)

For videos stored in S3:
1. Start analysis job (StartLabelDetection, StartFaceDetection, etc.)
2. Job runs asynchronously
3. Get notified via **SNS** when complete
4. Retrieve results via GetLabelDetection, GetFaceDetection, etc.

> **Exam Tip:** Rekognition video analysis is **asynchronous** (SNS notification pattern). Image analysis is **synchronous** (immediate response).

---

### Rekognition Custom Labels

- Train Rekognition to detect **your own custom objects** using your labeled images.
- Process: Upload training images → Label with bounding boxes → Train model → Deploy → Call API.
- Use case: Quality control (detect defects on assembly line), logo detection, medical imaging.

---

### Rekognition vs Textract (Common Confusion)

| Service | Focus | Input | Output |
|---|---|---|---|
| **Rekognition** | Visual content analysis | Images/Video | Labels, faces, text detection |
| **Textract** | Document text extraction | Documents (forms, tables, PDFs) | Structured text, form fields, table data |

> **Key Difference:** Rekognition reads text **visible in images** (signs, photos). Textract reads text from **documents** with structure (forms, tables, handwriting).

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Automatically moderate uploaded images for nudity | Rekognition Content Moderation |
| Verify employee identity from webcam photo vs ID photo | Rekognition Face Comparison |
| Find all videos in an archive containing a specific person | Rekognition Face Collections + video analysis |
| Detect hardhat usage on construction site | Rekognition PPE Detection |
| Read text on a license plate in a parking lot photo | Rekognition Text Detection |
| Detect your product logo in social media images | Rekognition Custom Labels |
| Extract data from an insurance form PDF | Amazon Textract (not Rekognition) |

---

---

## 8. Amazon SageMaker

### What Problem Does It Solve?
Building, training, and deploying ML models involves: setting up compute environments, managing training clusters, storing model artifacts, optimizing hyperparameters, deploying endpoints, and monitoring production models. SageMaker is a **fully managed end-to-end ML platform** that handles all of this, letting data scientists focus on models rather than infrastructure.

**Real-World Example:**
> A streaming service wants to build a custom recommendation engine. Data scientists use SageMaker Studio to explore data, write training code in a managed notebook, train a TensorFlow model on a 100-GPU cluster, automatically tune hyperparameters, deploy the trained model to a scalable endpoint, and monitor prediction quality in production — all from one unified platform.

---

### SageMaker Architecture Overview

```
DATA PREPARATION:
  SageMaker Studio → JupyterLab notebooks → S3 (data storage)
  SageMaker Data Wrangler → visual data preparation + transformation
  SageMaker Feature Store → centralized feature repository

MODEL TRAINING:
  SageMaker Training Jobs → managed EC2 clusters (any instance type)
  Built-in algorithms → XGBoost, Linear Learner, DeepAR, BlazingText, etc.
  Custom training → bring your own TensorFlow, PyTorch, Scikit-learn code
  SageMaker Experiments → track and compare training runs
  SageMaker Debugger → detect training issues in real time

HYPERPARAMETER TUNING:
  SageMaker Hyperparameter Tuning → automated HPO (Bayesian optimization)

MODEL OPTIMIZATION:
  SageMaker Clarify → bias detection + model explainability
  SageMaker Neo → compile models for edge devices (Jetson, Raspberry Pi)

DEPLOYMENT:
  SageMaker Endpoint → real-time inference (auto-scaling)
  SageMaker Batch Transform → offline batch inference
  SageMaker Serverless Inference → pay-per-request (spiky workloads)
  SageMaker Async Inference → async for large inputs

MONITORING:
  SageMaker Model Monitor → detect data drift, model quality degradation

MLOPS:
  SageMaker Pipelines → CI/CD for ML (automated retraining pipeline)
  SageMaker Model Registry → version and approve models
  SageMaker Projects → templates for MLOps best practices
```

---

### SageMaker Key Components

#### SageMaker Studio
- **Unified IDE** for ML: notebooks, training, debugging, experiments, pipelines.
- Web-based; runs on managed infrastructure.
- Replaces local Jupyter notebooks for team collaboration.

#### SageMaker Notebook Instances (Legacy)
- Managed EC2 with Jupyter pre-installed.
- Being replaced by Studio, but still used.

#### SageMaker Training Jobs
- Launch managed training clusters of any size.
- Bring your own code (Python script) with any framework.
- Frameworks supported: TensorFlow, PyTorch, MXNet, Scikit-learn, Spark, Hugging Face.
- Distributed training: data parallelism, model parallelism.
- Spot Training: use EC2 Spot Instances to reduce training cost by up to **90%**.

#### SageMaker Built-In Algorithms

| Algorithm | Use Case |
|---|---|
| **XGBoost** | Classification, regression (tabular data) |
| **Linear Learner** | Linear/logistic regression |
| **DeepAR** | Time-series forecasting |
| **BlazingText** | Text classification, word embeddings |
| **Object Detection** | Detect objects in images |
| **Image Classification** | Classify images |
| **Semantic Segmentation** | Pixel-level image analysis |
| **K-Means** | Clustering |
| **PCA** | Dimensionality reduction |
| **Factorization Machines** | Recommendation, click prediction |
| **Random Cut Forest** | Anomaly detection |

#### SageMaker Inference Options

| Type | Latency | Cost | Use Case |
|---|---|---|---|
| **Real-time Endpoint** | Milliseconds | Per hour | Continuous low-latency predictions |
| **Serverless Inference** | Seconds (cold start) | Per request | Infrequent/spiky traffic |
| **Async Inference** | Minutes | Per inference | Large payloads (>6MB), long processing |
| **Batch Transform** | Minutes to hours | Per instance-hour | Offline bulk inference on S3 data |

---

### SageMaker Ground Truth

- Managed **data labeling** service.
- Human labelers (private team, Mechanical Turk, or vendor) label your training data.
- Auto-labeling: ML model labels easy examples; humans label only uncertain ones.
- Use case: Label images for object detection, transcribe audio, classify text.

---

### SageMaker Feature Store

- Centralized repository for **ML features** (pre-computed input variables).
- **Online store**: low-latency lookup for real-time inference.
- **Offline store**: S3-based for training data retrieval.
- Prevents duplicate feature engineering across teams.

---

### SageMaker JumpStart

- Pre-built **ML solutions and foundation models** (one-click deploy).
- Includes: Llama, Stable Diffusion, BERT, ResNet, and 300+ other models.
- Fine-tune foundation models on your own data.
- Use case: Quick prototyping, starting point for custom models.

---

### SageMaker vs Pre-Built AI Services

| Scenario | SageMaker | Pre-Built Service |
|---|---|---|
| Detect faces in images | SageMaker (custom model) | **Rekognition** ✅ |
| Build custom defect detector | **SageMaker** ✅ | Rekognition Custom Labels |
| Transcribe audio | SageMaker | **Transcribe** ✅ |
| Forecast next quarter demand | SageMaker | **Forecast** ✅ |
| Need full control over model | **SageMaker** ✅ | Limited |
| Domain-specific ML (medical, legal) | **SageMaker** ✅ | Limited |

> **Exam Rule:** Use pre-built AI services (Rekognition, Comprehend, etc.) for common tasks with no ML expertise. Use **SageMaker** when you need a **custom model**, full control, or the pre-built services don't meet your needs.

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Build a custom ML model with your own training data | SageMaker |
| Reduce training cost by 90% using spot capacity | SageMaker Spot Training |
| Deploy ML model that auto-scales based on traffic | SageMaker Real-time Endpoint with Auto Scaling |
| Label 100,000 images for training dataset | SageMaker Ground Truth |
| Detect data drift in production ML model | SageMaker Model Monitor |
| One-click deploy of Llama LLM | SageMaker JumpStart |
| Batch predict on 10 million records overnight | SageMaker Batch Transform |
| MLOps pipeline for automated retraining | SageMaker Pipelines + Model Registry |

---

---

## 9. Amazon Textract

### What Problem Does It Solve?
Organizations deal with millions of paper and digital documents — tax forms, invoices, medical records, contracts, loan applications. Manual data entry is slow, expensive, and error-prone. OCR (Optical Character Recognition) tools extract raw text but cannot understand **document structure** (which text is in which form field, which data is in which table cell). Textract automatically extracts **structured text, form fields, and tables** from any document.

**Real-World Example:**
> A mortgage lender receives 500 loan applications per day as scanned PDFs. Textract automatically extracts: applicant name, address, income (from form fields), employer history (from a table), and signature (from a handwritten field) — populating the loan management system in seconds instead of 10 minutes of manual data entry per application.

---

### Textract Feature Types

| Feature | Description | Output |
|---|---|---|
| **DetectDocumentText** | Extract all raw text (OCR) | Lines and words with bounding boxes |
| **AnalyzeDocument (FORMS)** | Extract key-value pairs from forms | `"Name:" → "John Smith"`, `"DOB:" → "01/01/1990"` |
| **AnalyzeDocument (TABLES)** | Extract table structure with rows/columns | Structured table data |
| **AnalyzeDocument (SIGNATURES)** | Detect handwritten signatures | Signature location + confidence |
| **AnalyzeExpense** | Extract data from invoices/receipts | Vendor, total, line items, taxes |
| **AnalyzeID** | Extract data from identity documents | Driver's license, passport fields |
| **StartDocumentAnalysis** | Async analysis for multi-page PDFs | Job ID → poll or SNS notification |

---

### Textract in a Document Processing Pipeline

```
Document arrives (S3 via email, scan, upload)
        ↓
S3 Event → Lambda
        ↓
Textract AnalyzeDocument (FORMS + TABLES)
        ↓
Structured output (JSON):
  ├── Key-value pairs (form fields)
  ├── Table data (rows + columns)
  └── Raw text blocks
        ↓
Lambda processes JSON → populate database
        ↓
Human review (A2I) for low-confidence extractions
        ↓
Data stored in DynamoDB / RDS
```

---

### Amazon A2I (Augmented AI) Integration

- **Amazon Augmented AI (A2I)** adds human review for Textract low-confidence results.
- Define confidence threshold: if Textract confidence < 80%, route to human reviewer.
- Reviewers correct mistakes → model improves over time.
- Integrated with: Textract, Rekognition, SageMaker.

---

### Textract vs Rekognition Text Detection

| Feature | Textract | Rekognition (Text Detection) |
|---|---|---|
| **Purpose** | Extract structured data from documents | Read text in natural scene images |
| **Output** | Form fields, tables, key-value pairs | Text blocks + bounding boxes |
| **Document structure** | ✅ Understands forms and tables | ❌ Raw text only |
| **Use case** | Process forms, invoices, contracts | Read license plates, signs, memes |

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Extract all fields from a scanned W-2 tax form | Amazon Textract (FORMS) |
| Extract data from a table in a PDF financial statement | Amazon Textract (TABLES) |
| Extract vendor, amount, date from an invoice | Textract AnalyzeExpense |
| Verify passport information during identity check | Textract AnalyzeID |
| Read a license plate number from a security camera photo | Rekognition Text Detection (not Textract) |
| Add human review for low-confidence document extractions | Textract + Amazon A2I |

---

---

## 10. Amazon Transcribe

### What Problem Does It Solve?
Businesses generate enormous amounts of audio — call center recordings, medical dictations, video content, meetings. Transcribing audio manually is slow, expensive, and unscalable. Amazon Transcribe provides **automatic speech recognition (ASR)** that converts audio to accurate text, with additional features like speaker identification, custom vocabulary, and real-time streaming.

**Real-World Example:**
> A call center records 10,000 customer service calls daily. Transcribe automatically converts each recording to text. The transcripts are then analyzed by Comprehend for sentiment and entity extraction — identifying calls where customers are angry (negative sentiment) or mention competitor names — providing insights that previously required listening to thousands of hours of audio.

---

### Transcribe Features

| Feature | Description | Use Case |
|---|---|---|
| **Batch transcription** | Upload audio to S3, get transcript | Call recordings, podcast episodes |
| **Real-time streaming** | Stream audio, get transcript live | Live captioning, real-time assistant |
| **Speaker diarization** | Identify who is speaking (Speaker 1, 2, 3) | Call center: separate agent vs customer |
| **Custom vocabulary** | Add domain-specific words | Medical terms, company names, jargon |
| **Vocabulary filters** | Mask/remove profanity or specific words | Content moderation |
| **Custom language model** | Fine-tune on your domain's language | Medical, legal, financial terminology |
| **PII redaction** | Remove PII from transcripts | HIPAA compliance, privacy |
| **Language detection** | Auto-detect spoken language | Multilingual call centers |
| **Content redaction** | Redact sensitive info in audio + transcript | Credit card numbers in call recordings |
| **Subtitles** | Generate SRT/VTT subtitle files | Video captioning |
| **Medical Transcribe** | Specialized for medical dictation | Clinical notes, doctor-patient conversations |

---

### Transcribe Medical

- Optimized for **medical terminology** and clinical contexts.
- Supports two scenarios:
  - **Conversation**: Doctor-patient dialogue
  - **Dictation**: Doctor dictating notes
- Output includes specialized medical entities (body parts, medical conditions, medications).
- HIPAA eligible.

---

### Transcribe + Other Services Pipeline

```
Audio file in S3 (call recording)
        ↓
Amazon Transcribe (batch transcription)
  ├── Speaker diarization: Agent vs Customer separated
  ├── PII redaction: removes SSNs, credit cards
  └── Output: transcript JSON in S3
        ↓
Amazon Comprehend (NLP on transcript)
  ├── Sentiment analysis: Customer was NEGATIVE
  ├── Entity extraction: mentioned competitor "Acme Corp"
  └── Key phrases: "cancel subscription", "worst service"
        ↓
Results stored in DynamoDB → dashboard for supervisors
```

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Convert recorded customer service calls to searchable text | Amazon Transcribe |
| Generate subtitles for video content automatically | Amazon Transcribe (subtitle output) |
| Identify which person is speaking in a conference recording | Transcribe speaker diarization |
| Remove profanity from podcast transcripts | Transcribe vocabulary filter |
| Transcribe doctor's dictations with medical terminology | Amazon Transcribe Medical |
| Real-time captions for a live video stream | Amazon Transcribe real-time streaming |

---

---

## 11. Amazon Translate

### What Problem Does It Solve?
Building multilingual applications requires translating content into dozens of languages. Hiring human translators for every piece of content is expensive and slow. Amazon Translate provides **neural machine translation** that produces fluent, accurate translations at scale, in real time.

**Real-World Example:**
> A global e-commerce platform has 5 million product listings in English. Instead of hiring translators for 20 languages (100 million translations), they automatically translate all listings using Translate, making products searchable and purchasable by customers worldwide — in their native language — overnight.

---

### Translate Features

| Feature | Description |
|---|---|
| **Real-time translation** | Synchronous API for on-the-fly translation |
| **Batch translation** | Translate large volumes of documents asynchronously via S3 |
| **Custom terminology** | Ensure specific terms translate consistently (brand names, technical terms) |
| **Formality** | Control formal vs informal tone (available in select language pairs) |
| **Profanity masking** | Mask profanity in translated output |
| **Active Custom Translation** | Fine-tune translation style using parallel translation examples |
| **Supported languages** | 75+ language pairs |

---

### Custom Terminology

- Define how specific terms should always be translated.
- Example: "Amazon" should never be translated to another word; "AWS" stays "AWS".
- Prevents brand names, technical terms, product names from being incorrectly localized.

---

### Translate in a Multilingual App Pipeline

```
User submits content in any language
        ↓
Amazon Transcribe (if audio) or direct text
        ↓
Amazon Comprehend (detect source language)
        ↓
Amazon Translate (→ target language)
        ↓
Amazon Polly (convert translated text to speech)
        ↓
Multilingual audio response delivered to user
```

---

### Key Exam Scenarios

| Scenario | Answer |
|---|---|
| Translate customer support chats from 20 languages to English | Amazon Translate real-time |
| Translate 1 million documents in S3 overnight | Amazon Translate batch translation |
| Ensure brand name "Zephyr" is never translated | Translate custom terminology |
| Build a real-time multilingual chat app | Translate API + WebSocket |
| Translate foreign language call recording | Transcribe → Translate pipeline |

---

---

## Master Comparison: All ML Services

### Speech Pipeline (Input → Output)

```
Human speaks
    ↓
Amazon Transcribe (Speech → Text)
    ↓
Amazon Comprehend (Understand text)
    ↓
Amazon Translate (Text → Another language)
    ↓
Amazon Polly (Text → Speech)
    ↓
Human hears in their language
```

### Document Pipeline

```
Document (image/PDF)
    ↓
Amazon Textract (Extract text + structure)
    ↓
Amazon Comprehend (Analyze meaning)
    ↓
Results stored + acted upon
```

### Vision Pipeline

```
Image/Video
    ↓
Amazon Rekognition (Labels, faces, text, moderation)
    ↓
Results: content tags, face matches, inappropriate content flags
```

### Conversation Pipeline

```
User speaks → Amazon Transcribe (speech to text)
    ↓
Amazon Lex (understand intent + manage dialog)
    ↓
AWS Lambda (business logic)
    ↓
Response text → Amazon Polly (text to speech)
    ↓
User hears response
```

---

## "Which ML Service?" Decision Tree

```
Text input → want to ANALYZE it?
    ├── Find entities, sentiment, key phrases → Comprehend
    ├── Build a chatbot / conversation → Lex
    ├── Search documents by question → Kendra
    └── Detect PII → Comprehend

Audio input?
    ├── Convert speech to text → Transcribe
    └── Want speech output? → Polly

Text → convert to speech?
    └── Polly

Image/Video input?
    ├── Detect objects, faces, labels → Rekognition
    └── Extract text from document/form → Textract

Numbers/time-series → predict future?
    └── Forecast

Transaction/event → fraud risk?
    └── Fraud Detector

Need custom ML model?
    └── SageMaker

Need language translation?
    └── Translate
```

---

## Architecture Patterns to Know

### Pattern 1: Intelligent Document Processing
```
S3 (uploaded documents: invoices, forms, contracts)
        ↓
Lambda trigger → Textract (extract form data + tables)
        ↓
Comprehend (detect entities + sentiment in extracted text)
        ↓
A2I (route low-confidence extractions to human review)
        ↓
DynamoDB (store structured results)
        ↓
QuickSight (analytics dashboard)
```

### Pattern 2: Call Center Analytics
```
Amazon Connect (call center platform)
        ↓
Audio recordings → S3
        ↓
Transcribe (speech → text, speaker diarization, PII redaction)
        ↓
Comprehend (sentiment per call, entity extraction)
        ↓
OpenSearch (full-text search on transcripts)
        ↓
QuickSight Dashboard:
  ├── % negative sentiment calls by product
  ├── Top complaint phrases
  └── Agent performance scores
```

### Pattern 3: Real-Time Content Moderation
```
User uploads image to S3
        ↓
S3 Event → Lambda
        ↓
Rekognition: Content Moderation API
        ↓
If inappropriate (confidence > 90%):
    ├── Delete from S3
    ├── Notify user via SNS/email
    └── Log to DynamoDB for audit

If borderline (50–90%):
    └── Route to human reviewer (Amazon A2I)

If clean (<50%):
    └── Publish image to CDN
```

### Pattern 4: Multilingual Voice Assistant
```
User speaks question (in any language)
        ↓
Kinesis Video Streams (audio capture)
        ↓
Amazon Transcribe (speech → text, language detection)
        ↓
Amazon Translate (→ English)
        ↓
Amazon Lex (understand intent)
        ↓
Lambda (fetch answer from Kendra knowledge base)
        ↓
Amazon Translate (English → user's language)
        ↓
Amazon Polly (text → speech in user's language)
        ↓
Audio response to user
```

### Pattern 5: MLOps Pipeline with SageMaker
```
New training data arrives in S3
        ↓
EventBridge → triggers SageMaker Pipeline
        ↓
Step 1: Data preprocessing (SageMaker Processing Job)
Step 2: Model training (SageMaker Training Job with Spot)
Step 3: Model evaluation (accuracy > threshold?)
Step 4: Register model (SageMaker Model Registry)
Step 5: Human approval gate
Step 6: Deploy to endpoint (SageMaker Endpoint update)
        ↓
SageMaker Model Monitor (detect drift)
        ↓
CloudWatch Alarm → trigger retraining pipeline if drift detected
```

---

## Exam Tips Summary

### Amazon Comprehend
- ✅ NLP service: sentiment, entities, key phrases, language detection, PII, topic modeling
- ✅ Custom classification + custom entity recognition for domain-specific needs
- ✅ Comprehend Medical for healthcare text (clinical notes, medical records)
- ✅ Targeted sentiment = sentiment about specific entities within text

### Amazon Forecast
- ✅ Time-series forecasting — no ML expertise required
- ✅ Supports: ARIMA, ETS, DeepAR+, Prophet, AutoML
- ✅ Related time series + item metadata improve accuracy
- ✅ P10/P50/P90 quantile forecasts (not just single point)

### Amazon Fraud Detector
- ✅ Real-time fraud scoring using your historical labeled data
- ✅ OFI (online/payment fraud), TFI (transaction fraud), ATI (account takeover)
- ✅ Rules layer on top of model score → business outcomes (BLOCK/REVIEW/ALLOW)

### Amazon Kendra
- ✅ Intelligent semantic search across enterprise documents
- ✅ Connectors: S3, SharePoint, Salesforce, ServiceNow, Confluence, RDS, Google Drive
- ✅ Kendra + Lex = FAQ chatbot backed by document knowledge base
- ✅ Access control filtering — users only see authorized content

### Amazon Lex
- ✅ Intents (what user wants) + Slots (info needed) + Utterances (how they say it)
- ✅ Multi-turn conversation management built-in
- ✅ Amazon Connect integration for IVR call centers
- ✅ Fulfillment via Lambda; conversation flows = build once, deploy everywhere

### Amazon Polly
- ✅ Text → Speech; Transcribe = Speech → Text (opposite directions!)
- ✅ Neural voices = most natural; SSML = control pronunciation/rate/pitch
- ✅ Custom Lexicons for brand-specific pronunciation
- ✅ Output formats: MP3, OGG, PCM; can store in S3 for reuse

### Amazon Rekognition
- ✅ Images: Labels, faces, face comparison, text, content moderation, PPE, celebrities
- ✅ Video: Async (SNS notification) for stored videos; person tracking, activity detection
- ✅ Face Collections = searchable face database (IndexFaces → SearchFacesByImage)
- ✅ Custom Labels = train on your own images for domain-specific detection
- ✅ Rekognition ≠ Textract: Rekognition = visual scene analysis; Textract = document structure

### Amazon SageMaker
- ✅ Full ML lifecycle: data prep → training → tuning → deployment → monitoring
- ✅ Spot Training = up to 90% cost reduction for training jobs
- ✅ Inference types: Real-time (low latency), Serverless (spiky/infrequent), Async (large payloads), Batch (offline)
- ✅ Ground Truth = managed data labeling with auto-labeling
- ✅ Feature Store = centralized feature repo (online=real-time, offline=training)
- ✅ JumpStart = pre-built models and solutions (LLMs, vision models)
- ✅ Model Monitor = detect data drift in production

### Amazon Textract
- ✅ Extracts structured data from documents: text, forms (key-value), tables
- ✅ AnalyzeExpense = invoices/receipts; AnalyzeID = passports/driver's licenses
- ✅ Async for multi-page PDFs (SNS notification when done)
- ✅ A2I integration for human review of low-confidence extractions
- ✅ Textract ≠ Rekognition text: Textract = document structure; Rekognition = scene text

### Amazon Transcribe
- ✅ Speech → Text (ASR); Polly = opposite direction
- ✅ Speaker diarization = who said what (call center agent vs customer)
- ✅ PII/content redaction in both transcript and audio
- ✅ Custom vocabulary for domain terms; custom language model for fine-tuning
- ✅ Transcribe Medical = HIPAA-eligible, clinical terminology
- ✅ Real-time streaming for live captioning

### Amazon Translate
- ✅ Neural machine translation; 75+ languages
- ✅ Custom terminology = ensure brand names/terms translate consistently
- ✅ Batch translation for large S3 document volumes
- ✅ Often combined with Transcribe + Polly for multilingual voice pipeline
- ✅ Active Custom Translation = fine-tune with parallel examples
- 
