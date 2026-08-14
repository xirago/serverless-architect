# Enterprise Serverless Architecture: AWS vs. GCP Decision Framework

## 1. Enterprise Master Decision Tree

```mermaid
flowchart TD
    Start(["🏢 Start: Enterprise Application Scope"]) --> Q_Ingress{"User Audience & Ingress Boundary?"}

    %% Branch 1: Internal Employee Application
    Q_Ingress -->|"Internal Employees Only (Corporate SSO / Zero Trust)"| Q_Internal_Cloud{"Target Cloud?"}
    
    Q_Internal_Cloud -->|"Google Cloud"| Res_GCP_Internal["<b>GCP Cloud Run + Identity-Aware Proxy (IAP)</b> 🏆<br/>• Enforces Google Workspace / Okta / Azure AD SSO<br/>• Direct VPC Egress to internal enterprise subnets<br/>• Secret Manager runtime injection via Workload Identity<br/>• Scales to 0 in Dev / min-instances=1 in Prod"]
    Q_Internal_Cloud -->|"AWS"| Res_AWS_Internal["<b>AWS App Runner / Lambda behind internal ALB + OIDC</b><br/>• ALB authenticates against corporate IdP (Okta / Cognito)<br/>• Enforces security groups & private subnets<br/>• AWS Secrets Manager + KMS encryption<br/>• Zero public internet exposure"]

    %% Branch 2: Public Customer / Partner Application
    Q_Ingress -->|"External / Public Facing"| Q_Public_Proto{"App Interaction Model?"}
    
    Q_Public_Proto -->|"Interactive Web App / AI Chat (SSE Streaming)"| Q_Public_Cloud{"Target Cloud?"}
    Q_Public_Cloud -->|"Google Cloud"| Res_GCP_Public_Web["<b>GCP Cloud Run + Cloud Armor (WAF)</b><br/>• DDoS & OWASP Top 10 protection via Cloud Armor<br/>• Multi-concurrency handles simultaneous SSE AI streams<br/>• Custom corporate SSL domain with automated cert rotation"]
    Q_Public_Cloud -->|"AWS"| Res_AWS_Public_Web["<b>AWS Lambda + Web Adapter or App Runner + AWS WAF</b><br/>• AWS WAF attached to CloudFront / ALB / App Runner<br/>• Response streaming via Lambda Function URLs<br/>• Auto-scaling with AWS Shield standard DDoS protection"]

    Q_Public_Proto -->|"Static Enterprise Portal / Docs"| Res_Static_Ent["<b>Static Enterprise Frontend</b><br/>• GCP: <code>Cloud Storage + Cloud CDN + Cloud Armor</code><br/>• AWS: <code>S3 + CloudFront + Origin Access Control (OAC) + WAF</code>"]

    %% Branch 3: Background Batch / Data Pipeline
    Q_Ingress -->|"Internal Batch Worker / Scraper / Compliance Job"| Q_Batch_Cloud{"Target Cloud?"}
    Q_Batch_Cloud -->|"Google Cloud"| Res_GCP_Batch["<b>GCP Cloud Run Jobs</b><br/>• Up to 24-hour execution timeout<br/>• Non-root container runtime<br/>• VPC Egress via Cloud NAT for fixed outbound egress IP"]
    Q_Batch_Cloud -->|"AWS"| Res_AWS_Batch["<b>AWS ECS Tasks on Fargate</b><br/>• Triggered via EventBridge / Step Functions<br/>• Strict IAM task execution role<br/>• Egress through AWS NAT Gateway with Elastic IP"]
```

---

## 2. Enterprise Database & VPC Connectivity Tree

```mermaid
flowchart TD
    DB_Start(["🔒 Enterprise Data Connectivity"]) --> DB_Target{"Database Location & Type?"}

    %% Private Internal Database
    DB_Target -->|"Existing Corporate Database in Private VPC (PostgreSQL, Oracle, SQL Server)"| DB_VPC_Cloud{"Target Cloud?"}
    DB_VPC_Cloud -->|"Google Cloud"| Res_GCP_VPC["<b>Direct VPC Egress / Serverless VPC Access</b><br/>• Connects Cloud Run directly to VPC subnet<br/>• Enforces Firewall Rules & Private Service Connect (PSC)<br/>• Uses Cloud SQL Auth Proxy for IAM database authentication"]
    DB_VPC_Cloud -->|"AWS"| Res_AWS_VPC["<b>VPC Subnet Attachment (ENI) + Security Groups</b><br/>• Lambda/App Runner runs inside private subnets<br/>• Uses AWS RDS Proxy to manage connection pooling<br/>• IAM Database Authentication (no DB passwords in code)"]

    %% Cloud-Native Managed Serverless DB
    DB_Target -->|"New Cloud-Native Serverless Database"| DB_Native_Cloud{"Target Cloud?"}
    DB_Native_Cloud -->|"Google Cloud"| Res_GCP_DB["• <b>Cloud SQL (PostgreSQL/MySQL)</b>: Private IP only with automated daily backups & PITR<br/>• <b>Cloud Firestore</b>: Enterprise mode with VPC Service Controls & CMEK"]
    DB_Native_Cloud -->|"AWS"| Res_AWS_DB["• <b>Aurora Serverless v2</b>: Private subnets with automated multi-AZ failover<br/>• <b>Amazon DynamoDB</b>: Point-in-Time Recovery (PITR) + KMS CMEK encryption"]
```

---

## 3. Side-by-Side Enterprise Security & Governance Matrix

| Enterprise Dimension | **Google Cloud Platform (GCP)** | **Amazon Web Services (AWS)** |
| :--- | :--- | :--- |
| **Zero-Trust SSO / IdP** | **Identity-Aware Proxy (IAP)** natively intercepts HTTP before hitting Cloud Run. Works with Google Workspace, Okta, Entra ID. | **Application Load Balancer (ALB) OIDC** or **AWS Verified Access** forwards validated user claims to Lambda/App Runner. |
| **Private VPC Egress** | **Direct VPC Egress** (GA, no connector VM costs) or Serverless VPC Connector into dedicated private subnets. | **VPC Configuration (Hyperplane ENI)** attached to private subnets with Security Groups. |
| **Fixed Outbound Egress IP** | Route VPC traffic through **Cloud NAT** with allocated static external IPs (for whitelisting). | Route VPC traffic through **AWS NAT Gateway** with allocated Elastic IPs. |
| **Secrets Management** | **GCP Secret Manager**: Mounted as environment variables or volume files at runtime via Workload Identity. | **AWS Secrets Manager / SSM Parameter Store**: Resolved via IAM task execution role at runtime. |
| **Data Encryption & CMEK** | **Cloud KMS**: Native Customer-Managed Encryption Keys supported across Cloud Run, GCS, Cloud SQL, and Firestore. | **AWS KMS**: Customer Managed Keys (CMK) enforced across Lambda, S3, Aurora, and DynamoDB. |
| **Audit Logging & Compliance** | **Cloud Audit Logs** (Admin Activity & Data Access) streamed to BigQuery or SIEM (Splunk/Chronicle). | **AWS CloudTrail** + **CloudWatch Logs** streamed to S3 / OpenSearch / SIEM. |
| **CI/CD Deployment Auth** | **Workload Identity Federation** (Keyless OIDC from GitLab CI / GitHub Actions). No static JSON service account keys. | **AWS IAM OpenID Connect (OIDC) Identity Provider** for GitHub/GitLab. No long-lived AWS Access Keys. |
| **Compute Sizing** | Up to **32 GB RAM / 8 vCPUs** per container instance (or GPU instances). | Up to **10,240 MB RAM (10 GB) / 6 vCPUs** per Lambda function; 4 vCPU / 12 GB for App Runner. |
| **Production SLA / Latency** | `min-instances: 1` eliminates cold starts; 99.95% monthly uptime SLA. | Provisioned Concurrency / App Runner minimum 1 instance; 99.95% monthly uptime SLA. |

---

## 4. Hard Limits, Quotas & Scalability Bottlenecks (Verified Accurate)

| Limit / Bottleneck Category | **AWS Lambda / App Runner** | **GCP Cloud Run / Functions** | Scalability Impact & Mitigation |
| :--- | :--- | :--- | :--- |
| **Regional Concurrency Cap** | **1,000 unreserved concurrent executions** per region (account-wide pool). | **100 default max instances** per region (soft limit, easily raised to 1,000+). | *AWS Risk*: One spiking function can exhaust account quota and throttle all other microservices. Set reserved concurrency per function. |
| **Burst Scaling Rate** | Scales up to 1,000 instances immediately, then **500 instances every 10 seconds** until quota is reached. | Rapid scale-out from 0 (sub-second to 2s depending on container image size). | *App Runner Note*: App Runner scales out more slowly (1–3 min per instance), which can cause latency spikes during sudden bursts. |
| **Max Execution Timeout** | **15 minutes (900s)** hard limit for Lambda; 30m for App Runner. | **60 minutes (3,600s)** for Cloud Run HTTP; **24 hours** for Cloud Run Jobs. | *Hard Ceiling*: Multi-step data processing or heavy AI ingestion exceeding 15m must use Cloud Run Jobs or AWS Step Functions / ECS Fargate. |
| **Payload Size Limits** | **6 MB sync** (API Gateway/ALB), **256 KB async** (SQS/EventBridge). | **32 MB HTTP request** body; unconstrained streaming response. | *Impact*: Uploading large files directly to serverless functions will fail. Must use direct S3/GCS Pre-Signed URLs for client uploads. |
| **SNAT Port Exhaustion** | 64,000 concurrent outbound connections per NAT Gateway IP. | 64,000 concurrent outbound connections per Cloud NAT IP. | *Impact*: High concurrency calls to external third-party APIs can exhaust port pool. Use connection reuse / HTTP keep-alive. |
| **RDBMS Connection Exhaustion** | Max connections on RDS instance (100–500). | Max connections on Cloud SQL instance (100–500). | *Impact*: Direct serverless scale bursts saturate database connection pools. Enforce AWS RDS Proxy or Cloud SQL Auth Proxy with pooling (e.g. PgBouncer). |
| **NoSQL Write Contention** | DynamoDB: 1,000 WCU / 3,000 RCU per physical partition. | Firestore: **1 write/sec per document**. | *Impact*: Updating a single global counter or status doc across 200+ users will fail on Firestore. Use distributed counters or sharding. |
| **Downstream SaaS Rate Limits** | External CRM, ticketing, and team messaging API daily/minute limits. | External CRM, ticketing, and team messaging API daily/minute limits. | *Impact*: Unbuffered bursts from 200+ users will trigger 429 Too Many Requests. Mandates asynchronous buffering queues (SQS / Cloud Tasks). |
