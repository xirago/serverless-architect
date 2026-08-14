# Enterprise Serverless Architecture: AWS vs. GCP Decision Framework

## 1. Enterprise Master Decision Tree

```mermaid
flowchart TD
    Start(["🏢 Start: Enterprise Application Scope"]) --> Q_Cloud{"Target Cloud Ecosystem?"}

    %% Cloud Footprint Selection
    Q_Cloud -->|"Google Cloud (GCP)"| Q_GCP_Ingress{"User Audience & Ingress Boundary?"}
    Q_Cloud -->|"Amazon Web Services (AWS)"| Q_AWS_Ingress{"User Audience & Ingress Boundary?"}
    Q_Cloud -->|"Multi-Cloud / Greenfield Evaluation"| Q_Multi_Proto{"Application Interaction Model?"}

    %% GCP Ingress Branches
    Q_GCP_Ingress -->|"Internal Employees (SSO / Zero Trust)"| Res_GCP_Internal["<b>GCP Cloud Run behind Application Load Balancer + IAP</b> 🏆<br/>• ALB + Serverless NEG + Identity-Aware Proxy (IAP)<br/>• Enforces Google Workspace / Okta / Entra ID SSO<br/>• Direct VPC Egress to internal enterprise subnets<br/>• Secret Manager injection via Workload Identity<br/>• Ingress locked to internal-and-cloud-load-balancing"]
    Q_GCP_Ingress -->|"Public Web / Customers"| Q_GCP_Proto{"Protocol Model?"}
    Q_GCP_Proto -->|"HTTP / SSE AI Streaming"| Res_GCP_Public_SSE["<b>GCP Cloud Run + Cloud Armor (WAF)</b><br/>• Multi-concurrency (up to 1,000 req/instance) for AI SSE streams<br/>• Startup CPU Boost for fast container cold-starts<br/>• Scale-to-zero in Dev / min-instances=1 in Prod"]
    Q_GCP_Proto -->|"Bi-directional WebSockets / gRPC"| Res_GCP_Public_WS["<b>GCP Cloud Run (Native WebSockets)</b><br/>• Full bi-directional WebSocket support up to 60-min timeout<br/>• Set <code>no-cpu-throttling</code> if maintaining idle socket background state"]
    Q_GCP_Ingress -->|"Batch Job / Scraper (15m+)"| Res_GCP_Batch["<b>GCP Cloud Run Jobs</b><br/>• Up to 24-hour execution timeout<br/>• Egress routed through Cloud NAT with static external IP"]

    %% AWS Ingress Branches
    Q_AWS_Ingress -->|"Internal Employees (SSO / Zero Trust)"| Res_AWS_Internal["<b>AWS ECS Fargate or Lambda behind Internal ALB + OIDC</b> 🏆<br/>• ALB intercepts HTTP and authenticates with Okta / Entra ID<br/>• Hyperplane ENI attachments into private VPC subnets<br/>• AWS Secrets Manager + KMS encryption<br/>• Zero public internet exposure"]
    Q_AWS_Ingress -->|"Public Web / Customers"| Q_AWS_Proto{"Protocol Model?"}
    Q_AWS_Proto -->|"HTTP / SSE AI Streaming"| Res_AWS_Public_SSE["<b>AWS Lambda (Web Adapter) or ECS Fargate + AWS WAF</b><br/>• Response streaming via Lambda Function URLs / ALB<br/>• AWS WAF attached to CloudFront / ALB<br/>• Scale-to-zero in Dev / Provisioned Concurrency in Prod"]
    Q_AWS_Proto -->|"Bi-directional WebSockets"| Res_AWS_Public_WS["<b>AWS API Gateway WebSocket API + Lambda or ECS Fargate</b><br/>• API Gateway manages persistent socket connections<br/>• Translates events ($connect, $disconnect) to stateless Lambda invocations<br/>• Or long-lived stateful containers on ECS Fargate behind ALB"]
    Q_AWS_Ingress -->|"Batch Job / Scraper (15m+)"| Res_AWS_Batch["<b>AWS ECS Tasks on Fargate</b><br/>• Triggered via EventBridge / Step Functions<br/>• Overcomes Lambda 15-min timeout<br/>• Egress through AWS NAT Gateway with Elastic IP"]

    %% Multi-Cloud Static
    Q_Multi_Proto -->|"Static Enterprise Portal"| Res_Static_Ent["<b>Static Enterprise Frontend</b><br/>• GCP: <code>Cloud Storage + Cloud CDN + Cloud Armor</code><br/>• AWS: <code>S3 + CloudFront + Origin Access Control (OAC) + WAF</code>"]
```

---

## 2. Enterprise Database, VPC & Storage Architecture

```mermaid
flowchart TD
    DB_Start(["🔒 Enterprise Data & Storage Connectivity"]) --> DB_Target{"Data Access Pattern?"}

    %% Private Internal Database
    DB_Target -->|"Existing Corporate Database in Private VPC (PostgreSQL, Oracle, SQL Server)"| DB_VPC_Cloud{"Target Cloud?"}
    DB_VPC_Cloud -->|"Google Cloud"| Res_GCP_VPC["<b>Direct VPC Egress + Cloud SQL Auth Proxy</b><br/>• Connects Cloud Run directly to VPC subnet without connector VM costs<br/>• Enforces Private Service Connect (PSC) & IAM Database Authentication<br/>• Mandatory PgBouncer / Connection Pooler"]
    DB_VPC_Cloud -->|"AWS"| Res_AWS_VPC["<b>VPC Subnet Attachment (ENI) + AWS RDS Proxy</b><br/>• Lambda / ECS runs inside private subnets<br/>• Enforces AWS RDS Proxy to manage connection pooling during scale bursts<br/>• IAM Database Authentication (no database passwords in code)"]

    %% Cloud-Native Managed Serverless DB
    DB_Target -->|"Managed Cloud Serverless Database"| DB_Native_Cloud{"Target Cloud?"}
    DB_Native_Cloud -->|"Google Cloud"| Res_GCP_DB["• <b>Cloud SQL (PostgreSQL/MySQL)</b>: Private IP only with automated PITR<br/>• <b>Cloud Firestore</b>: Enforce sharding for counters (1 write/sec per doc limit)"]
    DB_Native_Cloud -->|"AWS"| Res_AWS_DB["• <b>Aurora Serverless v2</b>: Private subnets with auto-scaling ACUs<br/>• <b>Amazon DynamoDB</b>: Single-table design + Point-in-Time Recovery (PITR)"]

    %% Large Payload Direct Storage
    DB_Target -->|"Large File Uploads / Downloads (>10 MB)"| Res_Direct_Storage["<b>Direct Cloud Storage with Signed URLs</b><br/>• GCP: <code>GCS Signed URLs</code> (bypasses Cloud Run 32 MB request cap)<br/>• AWS: <code>S3 Pre-Signed URLs</code> (bypasses Lambda / API Gateway 6 MB cap)<br/>• Enable VPC Gateway Endpoints to eliminate NAT Gateway transfer fees ($0.045/GB)"]
```

---

## 3. Side-by-Side Enterprise Security & Governance Matrix

| Enterprise Dimension | **Google Cloud Platform (GCP)** | **Amazon Web Services (AWS)** |
| :--- | :--- | :--- |
| **Zero-Trust SSO / IdP** | **ALB + Serverless NEG + Identity-Aware Proxy (IAP)** intercepts HTTP before hitting Cloud Run. Supports Google Workspace, Okta, Entra ID. | **Internal Application Load Balancer (ALB) OIDC** or **AWS Verified Access** authenticates against corporate IdP (Okta / Cognito / Entra ID). |
| **Container Serverless Standard** | **Cloud Run**: Scales from 0 to 1,000+ instances; multi-concurrency (up to 1,000 req/instance); Startup CPU Boost. | **ECS Tasks on Fargate**: True container isolation, custom networking, long-running jobs. (App Runner for simpler PaaS setups). |
| **Real-Time Protocols** | **Native WebSockets, HTTP/2, gRPC** supported out-of-the-box up to 60 minutes per connection. | **Amazon API Gateway WebSocket API** (stateless Lambda routing) or **ECS Fargate** (stateful socket server). |
| **CPU Allocation Modes** | • `cpu-throttling=true` (Default, CPU disabled when idle)<br/>• `no-cpu-throttling` (CPU always allocated for background threads / sockets). | • Lambda: CPU tied strictly to active invocation.<br/>• ECS Fargate: Full continuous vCPU allocation. |
| **Private VPC Connectivity** | **Direct VPC Egress** directly into subnet (no connector VM overhead) with Private Service Connect (PSC). | **VPC Configuration (Hyperplane ENI)** attached to private subnets with Security Groups. |
| **NAT Cost Optimization** | Route traffic to GCP APIs via **Private Google Access / PSC**; Cloud NAT for static outbound IP. | Route traffic to S3/DynamoDB via free **VPC Gateway Endpoints**; Interface Endpoints (PrivateLink) to bypass $0.045/GB NAT fees. |
| **Secrets Governance** | **GCP Secret Manager**: Mounted as environment variables via Workload Identity. | **AWS Secrets Manager / SSM Parameter Store**: Resolved via IAM execution role. |
| **AI / GPU Acceleration** | **Cloud Run GPU** (Nvidia L4 GPUs) for serverless model inference with scale-to-zero. | **AWS SageMaker Serverless Inference** or **ECS Fargate with GPU**. |
| **Audit Logging & Compliance** | **Cloud Audit Logs** (Admin & Data Access) streamed to BigQuery / SIEM. | **AWS CloudTrail** + **CloudWatch Logs** streamed to S3 / SIEM. |
| **CI/CD Deployment Auth** | **Workload Identity Federation** (Keyless OIDC from GitLab CI / GitHub Actions). | **AWS IAM OpenID Connect (OIDC)** Provider for GitHub/GitLab CI. |

---

## 4. Hard Limits, Quotas & Scalability Bottlenecks

| Bottleneck Category | **AWS Lambda / ECS Fargate** | **GCP Cloud Run / Functions** | Scalability Impact & Architectural Mitigation |
| :--- | :--- | :--- | :--- |
| **Regional Concurrency Cap** | **1,000 unreserved executions** per region (soft default quota across entire account). | **100 default max instances** per region (soft limit, raised to 1,000+ via quota ticket). | *AWS Risk*: Spiking functions can starve all account workloads. Set dedicated Reserved Concurrency per function. |
| **Burst Scaling Speed** | Lambda scales up to 500–3,000 instances immediately, then **500 instances every 10s** (or 500/min). | Rapid scale-out from 0 (sub-second to 2s with Startup CPU Boost). | *App Runner Warning*: App Runner scale-out takes 1–3 minutes per instance; avoid for highly volatile traffic surges. |
| **Max Execution Timeout** | **15 minutes (900s)** hard limit for Lambda; unlimited for ECS Fargate. | **60 minutes (3,600s)** for Cloud Run HTTP; **24 hours** for Cloud Run Jobs. | *Ceiling*: Multi-step data processing or heavy AI ingestion exceeding 15m must use Cloud Run Jobs or AWS ECS Fargate / Step Functions. |
| **Payload Size Limits** | **6 MB synchronous** (API Gateway/ALB), **256 KB async** (SQS/EventBridge). | **32 MB HTTP request** body; unconstrained streaming response. | *Impact*: Uploading large files directly to serverless runtimes will fail. Use **S3 Pre-Signed URLs** or **GCS Signed URLs** for direct client uploads. |
| **NAT Bandwidth & Port Exhaustion** | 64,000 concurrent outbound connections per NAT IP; **$0.045/GB** NAT data processing fee. | 64,000 concurrent outbound connections per Cloud NAT IP. | *FinOps Impact*: Use **VPC Gateway Endpoints** (free for S3 & DynamoDB) and connection pooling / keep-alive to eliminate NAT costs and port exhaustion. |
| **RDBMS Connection Saturation** | Max database connections (typically 100–500). | Max database connections (typically 100–500). | *Impact*: Sudden serverless scale bursts saturate database connections. Enforce **AWS RDS Proxy** or **Cloud SQL Auth Proxy + PgBouncer**. |
| **NoSQL Write Contention** | DynamoDB: 1,000 WCU / 3,000 RCU per physical partition. | Firestore: **1 write/sec per document**. | *Impact*: Global counters or status updates across multiple users will fail on Firestore. Mandates distributed counters or sharding. |
| **Downstream SaaS Rate Limits** | External CRM, ticketing, and messaging API rate limits. | External CRM, ticketing, and messaging API rate limits. | *Impact*: Unbuffered bursts trigger 429 Too Many Requests. Mandates asynchronous buffering queues (**Amazon SQS** / **GCP Cloud Tasks**). |
| **Serverless Cost Inflection** | High continuous load (>200–500 sustained RPS 24/7). | High continuous load (>200–500 sustained RPS 24/7). | *FinOps Tipping Point*: At steady 24/7 high throughput, evaluate provisioned container clusters (**GKE Autopilot / EKS with Karpenter**) for 30–50% cost savings. |
