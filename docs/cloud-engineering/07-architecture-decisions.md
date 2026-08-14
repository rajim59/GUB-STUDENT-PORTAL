# 07. Architecture Decisions (ADR)

## 1. Objective

এই Document-এ Student Portal Platform-এর **প্রতিটি গুরুত্বপূর্ণ Architecture Decision** কেন নেওয়া হয়েছে, কী কী বিকল্প ছিল এবং কী Trade-off ছিল তা বিস্তারিতভাবে বর্ণনা করা হয়েছে।

এটি একটি **Architecture Decision Record (ADR)** হিসেবে কাজ করবে, যাতে ভবিষ্যতে কেউ সহজেই বুঝতে পারে — কেন এই নির্দিষ্ট Technology/Service বেছে নেওয়া হয়েছে।

---

## 2. Decision Format

প্রতিটি Decision নিচের Format-এ লেখা হয়েছে:

| Field | Description |
|---|---|
| **Context** | Decisionটি কেন দরকার |
| **Options** | কী কী বিকল্প ছিল |
| **Decision** | আমরা কোনটি বেছে নিয়েছি |
| **Reason** | কেন বেছে নিয়েছি |
| **Trade-off** | কী সুবিধা, কী অসুবিধা |

---

## 3. Architecture Decision Records

### 3.1 Application Hosting Platform

**Context:**  
Student Portal Application কোথায় Host করা হবে?

**Options:**  
- Azure Virtual Machine (IaaS)
- Azure App Service (PaaS)
- Azure Kubernetes Service (AKS)

**Decision:**  
Azure App Service (Standard S1 Plan)

**Reason:**  
- PaaS হওয়ায় Infrastructure Management (OS Patch, Scaling, Load Balancing) Azure-এর দায়িত্ব।
- Auto Scaling, Deployment Slots, Built-in Monitoring সহজে পাওয়া যায়।
- Lab Project-এর জন্য খরচ কম এবং Implementation দ্রুত।
- VM বা AKS-এর মতো জটিলতা নেই।

**Trade-off:**  
- VM-এর মতো Full Control নেই (তবে আমাদের দরকারও নেই)।
- App Service-এর কিছু সীমাবদ্ধতা আছে (যেমন নির্দিষ্ট কিছু Custom Configuration), তবে আমাদের Use Case-এ প্রযোজ্য নয়।

---

### 3.2 App Service Plan Tier

**Context:**  
App Service Plan-এর কোন Pricing Tier ব্যবহার করা হবে?

**Options:**  
- Basic Tier
- Standard S1
- Premium v3
- Isolated

**Decision:**  
Standard S1

**Reason:**  
- Basic-এ Auto Scaling ও Deployment Slots Support নেই।
- Standard-এ Auto Scaling, Deployment Slots, 2+ Instance-এ 99.95% SLA সব আছে।
- Premium v3 Zone Redundancy দেয়, কিন্তু Lab-এর জন্য খরচ বেশি।
- Isolated Enterprise Private Environment-এর জন্য, Lab-এ Overkill।

**Trade-off:**  
- Standard-এ Zone Redundancy নেই (তবে 99.95% SLA আমাদের Requirement পূরণ করে)।
- Premium v3-এর তুলনায় Max Instance Limit কম (Standard-এ 10, Premium-এ 30), কিন্তু Lab-এর জন্য 5 যথেষ্ট।

---

### 3.3 Database Service Selection

**Context:**  
Application Database-এর জন্য কোন Azure Database Service ব্যবহার করা হবে?

**Options:**  
- Azure SQL Database (Serverless)
- Azure Database for PostgreSQL Flexible Server
- Azure Database for MySQL Flexible Server
- Azure Cosmos DB

**Decision:**  
Azure SQL Database (Serverless)

**Reason:**  
- Serverless Tier-এ **True Auto-Scale Compute** আছে (Peak Load-এ vCore নিজে বাড়ায়)।
- Auto-Pause থাকায় Idle অবস্থায় Compute খরচ শূন্য।
- Built-in HA (99.99% SLA) এবং Automatic Backups।
- Relational Data Model-এর জন্য SQL Database উপযুক্ত (Student, Course, Payment ইত্যাদি)।
- PostgreSQL/MySQL-এ Auto-Scale নেই, Manual/Scheduled Scaling লাগে।

**Trade-off:**  
- PostgreSQL/MySQL-এর মতো Built-in PgBouncer নেই, তবে App-side Connection Pooling করে সমাধান করা হয়।
- Cosmos DB NoSQL, Relational Transaction-এর জন্য উপযুক্ত নয়।

---

### 3.4 Database Connectivity / Network Security

**Context:**  
App Service থেকে SQL Database-এ কীভাবে Connect হবে?

**Options:**  
- Public Endpoint + IP Firewall
- Service Endpoint + VNet Integration
- Private Endpoint + Private Link

**Decision:**  
Service Endpoint + App Service Regional VNet Integration

**Reason:**  
- Public Endpoint + IP Firewall দুর্বল, “Allow Azure Services” চালু করলে যে কেউ Access পায়।
- Private Endpoint সবচেয়ে নিরাপদ, কিন্তু Lab-এ DNS Configuration জটিল এবং সামান্য খরচ।
- Service Endpoint-এ Database Public Internet থেকে সম্পূর্ণ Block থাকে, কিন্তু Implementation সহজ।

**Trade-off:**  
- Private Endpoint-এর মতো Fully Private IP নেই, তবে আমাদের Security Requirement পূরণ হয়।
- Production Design-এ Private Endpoint-এ Upgrade করা যাবে।

---

### 3.5 Authentication Strategy

**Context:**  
Student Login Session কীভাবে Manage হবে?

**Options:**  
- Server-side Session (Stateful)
- Stateless Authentication (JWT)
- বাহ্যিক Identity Provider (Azure AD B2C)

**Decision:**  
Stateless Authentication (JWT, Database-based Credential Verification)

**Reason:**  
- Horizontal Scaling-এ Server-side Session সমস্যা করে (কোন Instance-এ Session আছে, Load Balancer জানেনা)।
- JWT Stateless, তাই যেকোনো Instance Request Handle করতে পারে।
- আমাদের Requirement-এ External Identity Provider জরুরি নয়, Database-based Authentication যথেষ্ট।
- Redis Session Store-এর অতিরিক্ত খরচ ও জটিলতা এড়ানো যায়।

**Trade-off:**  
- Token Revocation একটু কঠিন (তবে Lab-এ Acceptable)।
- Password Hashing (bcrypt/argon2) Backend-এ implement করতে হবে।

---

### 3.6 Secrets Management

**Context:**  
Database Password, JWT Secret কোথায় রাখা হবে?

**Options:**  
- Code-এ Hardcode
- App Settings-এ Plain Text
- Azure Key Vault + Managed Identity

**Decision:**  
Azure Key Vault + Managed Identity

**Reason:**  
- Code-এ Secret রাখা Security Risk।
- App Settings-এ Plain Text রাখলে Rotation সহজ নয়।
- Key Vault-এ Secret Secure থাকে এবং Managed Identity দিয়ে App Service Access পায়।
- Credential Rotation সহজ।

**Trade-off:**  
- Key Vault-এর জন্য সামান্য Management Overhead আছে, তবে Security Best Practice।

---

### 3.7 CI/CD Tool

**Context:**  
কোন CI/CD Tool ব্যবহার করা হবে?

**Options:**  
- GitHub Actions
- Azure DevOps Pipelines
- Jenkins (Self-hosted)

**Decision:**  
GitHub Actions

**Reason:**  
- Source Code GitHub-এ থাকবে, তাই GitHub Actions Natural Choice।
- Public/Private Repo-তে Free Usage।
- Azure Login Action (OIDC) দিয়ে Security ভালো।
- Jenkins-এ আলাদা Server Manage করতে হয়, খরচ ও জটিলতা।

**Trade-off:**  
- Azure DevOps-এর কিছু Advanced Feature নেই, তবে আমাদের দরকার নেই।
- GitHub Actions-এ Free Tier-এর সীমাবদ্ধতা আছে, তবে Lab-এ সমস্যা নেই।

---

### 3.8 Zero Downtime Deployment

**Context:**  
Deployment-এর সময় Service বন্ধ থাকবে না — কীভাবে নিশ্চিত হবে?

**Options:**  
- Blue-Green Deployment (Two Separate Environments)
- Rolling Deployment
- Deployment Slots + Swap (Azure App Service)

**Decision:**  
Deployment Slots + Swap

**Reason:**  
- App Service-এর Built-in Feature, Standard Plan-এ Support করে।
- Staging Slot-এ Deploy করে Health Check-এর পরে Swap, ফলে Production Down হয় না।
- Rollback সহজ (আবার Swap Back)।
- Blue-Green-এর মতো আলাদা Environment Maintain করতে হয় না।

**Trade-off:**  
- Slot Settings কিছু সময় Alাদা করতে হয় (App Settings সংরক্ষণ)।
- Auto Swap বন্ধ রাখা হয় (Manual Swap after Smoke Test)।

---

### 3.9 Infrastructure as Code (IaC) Tool

**Context:**  
Infrastructure Automation-এর জন্য কোন IaC Tool ব্যবহার হবে?

**Options:**  
- Azure Bicep
- Terraform
- ARM Template (JSON)

**Decision:**  
Terraform

**Reason:**  
- Team-এর পূর্ব অভিজ্ঞতা আছে Terraform-এ।
- Multi-cloud Support (Azure, AWS, GCP) — ভবিষ্যতে অন্য Cloud-এ যাওয়া সহজ।
- State Management এবং Module Ecosystem ভালো।
- ARM Template JSON জটিল, Bicep নতুন শিখতে হতো।

**Trade-off:**  
- Terraform-এ State File Manage করতে হয় (Local/Remote)।
- Bicep-এর মতো Azure Native Integration নেই, তবে AzureRM Provider যথেষ্ট।

---

### 3.10 Disaster Recovery (Lab vs Production)

**Context:**  
Disaster Recovery Strategy কী হবে?

**Options:**  
- Automated Backups + Point-in-Time Restore (PITR)
- Active Geo-Replication + Auto-Failover Group
- Traffic Manager + Secondary App Service

**Decision:**  
**Lab:** Automated Backups + PITR  
**Production Design:** Active Geo-Replication + Auto-Failover Group + Traffic Manager

**Reason:**  
- Lab-এ $100 Credit-এর মধ্যে থাকতে ব্যয়বহুল Geo-Replication/Failover Group বাদ।
- PITR-এ RTO/RPO কিছুটা বড়, কিন্তু Lab-এর জন্য যথেষ্ট।
- Production Design-এ RTO < 5 min, RPO < 1 min Achieve করা যায়।
- Documentation-এ Production Design স্পষ্ট দেখানো হয়েছে।

**Trade-off:**  
- Lab-এ RTO/RPO Target পুরোপুরি পূরণ হয় না, তবে Cost Optimization প্রাধান্য পেয়েছে।

---

### 3.11 Monitoring Strategy

**Context:**  
Application ও Infrastructure Monitoring কীভাবে হবে?

**Options:**  
- শুধু App Service Built-in Metrics
- Application Insights + Log Analytics
- Azure Monitor + Sentinel

**Decision:**  
Application Insights + Log Analytics Workspace

**Reason:**  
- Application Insights-এ Response Time, Failed Requests, Availability Test সব আছে।
- Real-Time Monitoring (Live Metrics) পাওয়া যায়।
- Log Analytics-এ Centralized Logs এবং Alert Query।
- Built-in Metrics-এ Application-level Insight নেই, Sentinel Lab-এ Overkill।

**Trade-off:**  
- Data Ingestion Cost থাকতে পারে, তাই Sampling ও Daily Cap সেট করা হয়।

---

### 3.12 Cost Management Approach

**Context:**  
$100 Credit-এর মধ্যে কীভাবে থাকা যাবে?

**Options:**  
- সব Resource সবসময় চালু রাখা
- Serverless/Auto-Pause ব্যবহার
- Demo-র সময় চালু, শেষে সম্পূর্ণ Delete (On/Off)

**Decision:**  
On/Off + Serverless + Budget Alert + Policies

**Reason:**  
- সবসময় চালু রাখলে $100 দ্রুত শেষ হয়ে যাবে।
- Serverless Database-এ Auto-Pause থাকায় Idle খরচ বাঁচে।
- Demo-র পরে সব মুছে দিলে খরচ প্রায় শূন্য।
- Budget Alert ও Policy দুর্ঘটনাজনিত খরচ রোধ করে।

**Trade-off:**  
- প্রতিবার Demo-র আগে Environment তৈরি করতে হয় (তবে IaC Automation করে সহজ)।
- Database Data সংরক্ষণের জন্য Seed Scripts প্রস্তুত রাখতে হয়।

---

## 4. Summary

এই ADR Document-এ আমরা দেখালাম, প্রতিটি বড় Decision-এর পেছনে Context, Options, Decision এবং Trade-off কী ছিল।  
এটি প্রমাণ করে যে, আমাদের Architecture শুধু Random Selection নয়, বরং Requirement ও Cost-এর Balance-এ চিন্তা করে নেওয়া হয়েছে।

---