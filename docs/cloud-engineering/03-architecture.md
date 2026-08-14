# 03. Architecture

## 1. Architecture Overview

এই ডকুমেন্টে **Student Portal Platform**-এর Cloud Architecture বিস্তারিত বর্ণনা করা হয়েছে।  
Architecture-টি **Microsoft Azure**-এ ডিজাইন করা হয়েছে এবং **High Availability, Auto Scaling, Zero Downtime Deployment, Security, Monitoring এবং Cost Optimization** — এই সবগুলো Business Requirement পূরণের জন্য তৈরি।

Architecture-টি সম্পূর্ণ **Infrastructure as Code (Terraform)** দিয়ে Provision করা হয় এবং Demo-র সময় চালু রেখে বাকি সময় বন্ধ রাখা হয় (Cost Control)।

---

## 2. Architecture Diagram

নিচে Architecture Diagram-টি দেখানো হলো:

🧩 Mermaid Diagram Code Blocks

1. High-Level Architecture (Student → App → DB)

flowchart TD
    A[Student Browser] -->|HTTPS TLS 1.2| B[Internet]
    B --> C[Azure App Service]
    
    subgraph C[Azure App Service - Standard S1]
        direction TB
        C1[Production Slot - 2+ Instances]
        C2[Staging Slot - Deployment]
        C3[Auto Scaling Rules]
        C4[Stateless Application]
        C5[Managed Identity]
    end
    
    C --> D[Virtual Network - Subnet + Service Endpoint]
    D --> E[Azure SQL Database Serverless]
    
    C --> F[Azure Key Vault]
    F -->|Secrets| C
    
    C --> G[Application Insights + Log Analytics]
    G -->|Metrics & Logs| H[Azure Monitor / Alerts]

2. CI/CD Pipeline (GitHub Actions + Deployment Slots)

flowchart LR
    A[Developer Push to GitHub] --> B[GitHub Actions Trigger]
    B --> C[Build & Unit Test]
    C --> D[Publish Artifact]
    D --> E[Deploy to Staging Slot]
    E --> F[Smoke Test / Health Check]
    F -->|Pass| G[Swap Staging ↔ Production]
    F -->|Fail| H[Pipeline Stop + Alert]
    G --> I[Zero Downtime Deployment Complete]


3. Auto Scaling Flow

flowchart TD
    A[Traffic Load Increases] --> B{CPU > 70% for 5 min?}
    B -->|Yes| C[Add 1 Instance]
    C --> D[Load Balancer Distributes Traffic]
    D --> E[More Capacity]
    B -->|No| F[Keep Current Instances]
    
    G[Traffic Load Decreases] --> H{CPU < 40% for 10 min?}
    H -->|Yes| I[Remove 1 Instance]
    I --> J[Cost Optimization]


4. Disaster Recovery (Lab + Production Design)

flowchart TD
    subgraph Lab_Implementation["Lab Implementation"]
        A[Azure SQL Database - Serverless] --> B[Automated Backups]
        B --> C[Point-in-Time Restore]
        C --> D[Recover Database]
    end
    
    subgraph Production_Design["Production Design"]
        E[Primary Region - Southeast Asia] -->|Auto-Failover Group| F[Secondary Region - East Asia]
        F --> G[Readable Secondary Database]
        E --> H[Traffic Manager]
        H -->|Failover| F
    end

5. Cost Management On/Off Strategy

flowchart LR
    A[Demo Start] --> B[terraform apply]
    B --> C[Resources Running]
    C --> D[Demo Presentation]
    D --> E[Demo End]
    E --> F[terraform destroy]
    F --> G[Resources Deleted - Cost Zero]

```
                        +-------------------+
                        |    Student (Internet) |
                        +-------------------+
                                |
                                | HTTPS (TLS 1.2+)
                                v
                        +-------------------+
                        |  Azure Traffic Manager? (Production Design)
                        |  Lab: Direct to App Service
                        +-------------------+
                                |
                                v
                +-----------------------------------+
                | Azure App Service (Standard S1)   |
                | - Production Slot (Min 2 Instances)|
                | - Staging Slot (Deployment)        |
                | - Auto Scaling Rules               |
                | - Stateless Application            |
                | - Managed Identity (System)        |
                +-----------------------------------+
                        |                |
                        | Regional VNet Integration
                        | (Subnet with Service Endpoint)
                        v
                +-----------------------------------+
                | Virtual Network (VNet)            |
                | - Subnet: app-subnet              |
                | - Service Endpoint: Microsoft.Sql |
                +-----------------------------------+
                        |
                        v
                +-----------------------------------+
                | Azure SQL Database (Serverless)   |
                | - Auto-Pause                       |
                | - Built-in HA (99.99%)            |
                | - Automatic Backups + PITR        |
                | - Connection Pooling (App Side)   |
                +-----------------------------------+
                        |
                        | Secrets
                        v
                +-------------------+
                | Azure Key Vault  |
                | - DB Connection String |
                | - JWT Signing Key      |
                +-------------------+
                        |
                        | Monitoring
                        v
                +-----------------------------------+
                | Application Insights + Log Analytics |
                | - Live Metrics                      |
                | - Availability Ping Test            |
                | - Alert Rules                        |
                | - Dashboard                          |
                +-----------------------------------+

        Infrastructure as Code (Terraform)
        Deployment / DevOps (GitHub Actions)
        Cost Management (Budget, Policies, Tags)
```

---

## 3. Components and Services

| Azure Service | Purpose | Decision / Rationale |
|---|---|---|
| Azure App Service | Hosting Student Portal Application | PaaS, Auto Scaling, Deployment Slots, Built-in Load Balancing |
| App Service Plan (Standard S1) | Compute Tier for App Service | Auto Scaling supported, 99.95% SLA, cost-effective for Lab |
| Deployment Slots (Staging + Production) | Zero Downtime Deployment | Swap mechanism ensures no downtime during release |
| Azure SQL Database (Serverless) | Application Database | True Auto-Scale compute, Auto-Pause, Built-in HA, cost-efficient |
| Azure Key Vault | Secrets Management | Store DB credentials, JWT secret; accessed via Managed Identity |
| Virtual Network + Service Endpoint | Secure Database Connectivity | Block Public Internet access to DB, allow only VNet subnet |
| Application Insights | Application Performance Monitoring (APM) | Real-time metrics, response time, failed requests, availability |
| Log Analytics Workspace | Central Log Storage | Store logs, enable alert queries and dashboards |
| Azure Cost Management | Budget & Cost Tracking | Set budget $80, alert thresholds, cost analysis |
| Azure Policy | Governance & SKU Restriction | Restrict expensive SKUs, enforce resource tagging |
| Terraform | Infrastructure as Code | Repeatable, version-controlled infrastructure provisioning |

---

## 4. Architecture Decision Log (ADR)

প্রতিটি গুরুত্বপূর্ণ Decision সংক্ষেপে নিচে উল্লেখ করা হলো:

| Decision | Context | Decision | Reason | Status |
|---|---|---|---|---|
| Application Hosting | Hosting Platform | Azure App Service | PaaS, Auto Scaling, Managed Infrastructure | Final |
| App Service Tier | Scaling & HA | Standard S1 | Auto Scaling + Deployment Slots supported | Final |
| Initial Instance Count | HA Starting Point | Min 2 Instances | 2+ Instance required for 99.95% SLA | Final |
| Max Instance Count (Lab) | Cost vs Scale | 5 Instances | Demo-র জন্য যথেষ্ট, খরচ নিয়ন্ত্রণ | Final |
| Database Service | Database Hosting | Azure SQL Database (Serverless) | True Auto-Scale, Auto-Pause, HA built-in | Final |
| Database Connectivity | Security | VNet Integration + Service Endpoint | Public Internet থেকে DB Block, সহজ Lab Setup | Final |
| Authentication Type | Session Management | Stateless Authentication (JWT) | Horizontal Scaling-এ Session সমস্যা এড়ানো | Final |
| Secrets Management | Security | Azure Key Vault + Managed Identity | Credential Rotation, Code-এ Secret নেই | Final |
| CI/CD Tool | Automation | GitHub Actions | Free, GitHub integration, easy | Final |
| Zero Downtime Deployment | Release Strategy | Deployment Slots + Swap | Atomic Swap, Rollback সহজ | Final |
| Infrastructure as Code | Provisioning | Terraform | Multi-cloud, team familiarity, State Management | Final |
| Monitoring | Observability | Application Insights + Log Analytics | Real-time metrics, alerting, availability test | Final |
| DR (Lab) | Disaster Recovery | Automated Backups + PITR | Cost-effective, sufficient for Lab | Final |
| DR (Production Design) | Disaster Recovery | Auto-Failover Group + Geo-Replication | RTO<5min, RPO<1min Target | Future |

---

## 5. Detailed Design

### 5.1 User Access

- Studentরা Internet-এর মাধ্যমে HTTPS ব্যবহার করে Portal-এ Access করবে।
- HTTP Traffic полностью Block থাকবে; শুধু HTTPS Allow।
- Minimum TLS Version 1.2 enforce করা হবে।

### 5.2 Application Layer (Azure App Service)

- **Service:** Azure App Service (Standard S1 Plan)।
- **Instances:** Minimum 2, Maximum 5 (Lab)।
- **Auto Scaling Rules:**
  - Scale-Out: CPU > 70% (5 min average) → +1 Instance
  - Scale-In: CPU < 40% (10 min average) → -1 Instance
  - Backup Metric: HTTP Queue Length (Scale-Out > 100, Scale-In < 10)
- **Stateless Design:** Application কোনো Instance-এর Local Memory-তে State রাখবে না।
- **ARR Affinity:** Off (Sticky Session বন্ধ, Load Balancing evenly)।
- **Deployment Slots:** Production Slot (Live) + Staging Slot (Deploy)।

### 5.3 Database Layer (Azure SQL Database)

- **Service:** Azure SQL Database (Serverless Tier)।
- **Compute:** Auto-Scale vCore (Lab: Min 0.5, Max 4/8; Production Design: Max 40)।
- **Auto-Pause:** Idle থাকলে 1 ঘণ্টা পর Compute বন্ধ, Storage চালু থাকে।
- **High Availability:** Built-in 3 Replica (Primary + Secondary) — 99.99% SLA।
- **Backup:** Automatic Full/Differential/Transaction Log, Point-in-Time Restore (PITR)।
- **Connection Management:** প্রতিটি App Instance-এ Connection Pooling (Max 50-100), Retry Logic, Command Timeout।
- **Network Security:** Database Firewall শুধু VNet Subnet থেকে Access Allow করবে।

### 5.4 Networking & Security

- **Virtual Network:** App Service Regional VNet Integration-এর মাধ্যমে VNet-এ যুক্ত।
- **Subnet:** `app-subnet`-এ Service Endpoint `Microsoft.Sql` enabled।
- **Key Vault:** Database Connection String, JWT Secret ইত্যাদি Secure রাখা হয়।
- **Managed Identity:** App Service-এর System Assigned Managed Identity Key Vault থেকে Secret পড়ে।
- **App Service Security:**
  - HTTPS Only
  - Minimum TLS 1.2
  - FTP/FTPS Disabled
  - SCM Access IP Restriction (Developer IP)
- **Database Authentication:** SQL Authentication (Credential in Key Vault); Production Design-এ Azure AD Passwordless।

### 5.5 Monitoring & Alerting

- **Application Insights:** App Service-এ Enable, Live Metrics Stream।
- **Log Analytics Workspace:** সব Logs centralized।
- **Availability Test:** URL Ping Test প্রতি 5 মিনিটে `/health` Endpoint-এ।
- **Alert Rules:**
  - CPU > 85% (5 min) → Email
  - Memory > 80% (5 min) → Email
  - Failed Requests > 20 (5 min) → Email
  - Response Time > 200 ms (5 min) → Email
  - Availability Down (2 consecutive failures) → Email
  - Database CPU > 80% (5 min) → Email
  - Database Storage > 80% → Email
- **Dashboard:** Azure Dashboard-এ সব Metrics এক জায়গায়।

### 5.6 Disaster Recovery (DR)

**Lab Implementation:**
- Database: Automated Backups + Point-in-Time Restore (PITR)।
- RTO: 15–60 minutes, RPO: 5–10 minutes (Backup log interval)।
- কোনো Active Geo-Replication বা Failover Group Lab-এ ব্যবহার করা হয়নি (Cost)।

**Production Design (Documentation Only):**
- Database: Active Geo-Replication + Auto-Failover Group।
- Secondary Region: East Asia।
- App Service DR: Pre-provisioned Secondary + Traffic Manager (Priority Mode)।
- RTO < 5 min, RPO < 1 min Achievable।

### 5.7 Deployment / DevOps (CI/CD)

- **Source Control:** GitHub (main branch)।
- **CI/CD Tool:** GitHub Actions।
- **Pipeline Stages:**
  1. Checkout Code
  2. Setup Runtime
  3. Build
  4. Unit Test
  5. Publish Artifact
  6. Deploy to Staging Slot
  7. Smoke Test (Health Check + DB Connectivity)
  8. Swap (Staging → Production)
  9. Post-Swap Verification
- **Zero Downtime:** Deployment Slots + Swap ensures no interruption।
- **Rollback:** Swap Back (আগের Production Slot এখন Staging-এ)।
- **Secrets:** GitHub Secrets-এ Azure Login (OIDC), অন্য Secrets Key Vault-এ।

### 5.8 Infrastructure as Code (IaC)

- **Tool:** Terraform (HCL)।
- **State Management:** Local State (Lab), পরে Remote State (Azure Blob Storage)।
- **Project Structure:**
  ```
  terraform/
  ├── main.tf
  ├── variables.tf
  ├── outputs.tf
  ├── providers.tf
  ├── modules/
  │   ├── appservice/
  │   ├── sql/
  │   ├── keyvault/
  │   ├── network/
  │   └── monitoring/
  └── environments/
      ├── lab.tfvars
      └── production.tfvars
  ```
- **Deployment Commands:**
  - `terraform init`
  - `terraform plan -var-file="environments/lab.tfvars"`
  - `terraform apply -var-file="environments/lab.tfvars"`
  - `terraform destroy -var-file="environments/lab.tfvars"`

---

## 6. Requirement Compliance Mapping

| Requirement | Architecture Support |
|---|---|
| High Availability | App Service 2+ Instance + SQL Built-in HA (99.99%) |
| Auto Scaling | App Service Rules + SQL Serverless Auto-Scale |
| Zero Downtime Deployment | Deployment Slots + Swap |
| Disaster Recovery | Lab: PITR, Production: Failover Group |
| Secure Authentication | Stateless JWT, Key Vault, Managed Identity, VNet |
| Fast Response Time | Scaling + Connection Pooling + Monitoring |
| Real-Time Monitoring | Application Insights + Alerts + Dashboard |
| Infrastructure as Code | Terraform |
| Cost Optimization | Resource Group On/Off, Budget, Policies, Auto-Pause |

---