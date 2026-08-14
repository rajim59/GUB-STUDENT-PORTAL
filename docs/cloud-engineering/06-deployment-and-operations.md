# 06. Deployment and Operations

## 1. Objective

এই Document-এ Student Portal Platform-এর **Deployment এবং Operations** প্রক্রিয়া বর্ণনা করা হয়েছে।  
এখানে Infrastructure Provisioning, Application Deployment, CI/CD Pipeline, Environment Start/Stop এবং Troubleshooting-এর ধাপগুলো স্পষ্টভাবে উল্লেখ আছে।

মূল লক্ষ্য:

- **Terraform** ব্যবহার করে Infrastructure Automation।
- **GitHub Actions** দিয়ে CI/CD Pipeline পরিচালনা।
- **Demo-র সময় দ্রুত Environment চালু করা।**
- **Demo শেষে সম্পূর্ণ Environment বন্ধ করে খরচ কমানো।**
- যেকোনো সমস্যা হলে দ্রুত সমাধানের জন্য Runbook।

---

## 2. Prerequisites

Deployment শুরু করার আগে নিচের জিনিসগুলো থাকতে হবে:

| Prerequisite | Version / Details |
|---|---|
| Azure Subscription | Active Subscription with $100 Credit |
| Azure CLI | Latest Version |
| Terraform CLI | v1.6+ |
| Git | Latest Version |
| GitHub Account | Repository Access |
| VS Code / Any Editor | Optional |

### 2.1 Azure CLI Login

```bash
az login
az account set --subscription <subscription-id>
```

### 2.2 Terraform Installation Check

```bash
terraform version
```

### 2.3 GitHub Repository Setup

- Source Code GitHub Repository-তে থাকবে।
- Repository-তে `terraform/` ফোল্ডার এবং Application Code থাকবে।
- GitHub Secrets-এ Azure Login Credentials (OIDC) Set করতে হবে।

---

## 3. Infrastructure as Code (Terraform)

### 3.1 Terraform Project Structure

```
project-root/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── providers.tf
│   ├── modules/
│   │   ├── appservice/
│   │   ├── sql/
│   │   ├── keyvault/
│   │   ├── network/
│   │   └── monitoring/
│   └── environments/
│       ├── lab.tfvars
│       └── production.tfvars
```

### 3.2 Terraform Variables (Lab Environment)

`environments/lab.tfvars` ফাইলে Lab-এর Configuration থাকবে:

```hcl
location            = "southeastasia"
resource_group_name = "studentportal-rg"
environment         = "lab"

app_service_plan_sku = "S1"
min_instance_count   = 2
max_instance_count   = 5

sql_server_name = "studentportal-sql"
database_name   = "StudentPortalDB"
database_sku    = "Serverless"
min_vcore       = 0.5
max_vcore       = 4

key_vault_name = "studentportal-kv"
app_insights_name = "studentportal-ai"
log_analytics_name = "studentportal-la"
```

### 3.3 Terraform Commands

#### 3.3.1 Initialize

```bash
cd terraform
terraform init
```

#### 3.3.2 Plan

```bash
terraform plan -var-file="environments/lab.tfvars"
```

#### 3.3.3 Apply (Create Environment)

```bash
terraform apply -var-file="environments/lab.tfvars"
```

#### 3.3.4 Destroy (Delete Environment)

```bash
terraform destroy -var-file="environments/lab.tfvars"
```

> **Note:**  
> `terraform destroy` চালালে Resource Group-সহ সব Resource মুছে যাবে।  
> Database Data হারাবে, তাই আগে থেকে SQL Seed Scripts প্রস্তুত রাখতে হবে।

---

## 4. CI/CD Pipeline (GitHub Actions)

### 4.1 Workflow Files

GitHub Repository-তে `.github/workflows/` ফোল্ডারে দুইটি Workflow থাকবে:

| Workflow | Purpose |
|---|---|
| `infra-deploy.yml` | Terraform Infrastructure Create/Destroy |
| `app-deploy.yml` | Application Build, Test, Deploy to App Service |

### 4.2 Application CI/CD Pipeline

`app-deploy.yml` ফাইলের মূল ধাপ:

| Stage | Description |
|---|---|
| Checkout | Source Code Pull |
| Setup | Runtime Setup (Node.js/.NET ইত্যাদি) |
| Build | Application Build |
| Test | Unit Test Run |
| Publish Artifact | Build Output Artifact |
| Deploy to Staging | Staging Slot-এ Deploy |
| Smoke Test | Health Check + DB Connectivity |
| Swap | Staging ↔ Production Slot Swap |

### 4.3 Pipeline Secrets

GitHub Secrets-এ নিচের তথ্যগুলো Set করতে হবে:

| Secret Name | Description |
|---|---|
| `AZURE_CLIENT_ID` | Service Principal Client ID (OIDC) |
| `AZURE_TENANT_ID` | Azure Tenant ID |
| `AZURE_SUBSCRIPTION_ID` | Subscription ID |
| `AZURE_APP_NAME` | App Service Name |
| `AZURE_RESOURCE_GROUP` | Resource Group Name |

**Note:**  
OIDC (OpenID Connect) ব্যবহার করা হয়, তাই কোনো Password/Long-lived Credential লাগে না।

---

## 5. Demo Session Runbook

### 5.1 Demo-র আগে (Environment Start)

```bash
# Step 1: Infrastructure Deploy
cd terraform
terraform init
terraform plan -var-file="environments/lab.tfvars"
terraform apply -var-file="environments/lab.tfvars"

# Step 2: Application Deploy (CI/CD)
# GitHub-এ main branch-এ Push করলে Pipeline স্বয়ংক্রিয়ভাবে চালু হবে।
# Or Manual Trigger:
gh workflow run app-deploy.yml

# Step 3: Verify Environment
az webapp show --name studentportal-app --resource-group studentportal-rg
az webapp list-instances --name studentportal-app --resource-group studentportal-rg
```

### 5.2 Demo-র সময় (Verification)

- Azure Portal-এ Resource Group-এ সব Resource Running আছে কি না Check করুন।
- Application Insights-এ Live Metrics দেখুন।
- Load Test (k6) চালান।
- Auto Scaling ও Failover Test করুন।
- Monitoring Dashboard দেখান।

### 5.3 Demo শেষে (Environment Stop)

```bash
# Option 1: Terraform Destroy
cd terraform
terraform destroy -var-file="environments/lab.tfvars"

# Option 2: Resource Group Delete
az group delete --name studentportal-rg --yes --no-wait
```

**Best Practice:**  
Demo শেষে অবশ্যই Destroy করে খরচ বন্ধ করতে হবে।  
Video Recording-এর জন্য আগে থেকেই সবকিছু Record করে রাখতে হবে।

---

## 6. Database Schema & Seed Data

### 6.1 Schema Creation

- Database Deploy-এর পরে Backend Team-এর দেওয়া `schema.sql` Script চালাতে হবে।
- এটি CI/CD Pipeline-এর একটি Step হিসেবেও Run করা যায়।

### 6.2 Seed Data

- Demo-র জন্য Sample Data `seed.sql` Script দিয়ে Load করা হয়।
- Student Users, Courses, Routine ইত্যাদি Sample Data থাকবে।

**Script Location:**
```
/database/
├── schema.sql
└── seed.sql
```

### 6.3 Manual Seed Command (যদি দরকার হয়)

```bash
sqlcmd -S studentportal-sql.database.windows.net -U sqladmin -P <password> -d StudentPortalDB -i database/schema.sql
sqlcmd -S studentportal-sql.database.windows.net -U sqladmin -P <password> -d StudentPortalDB -i database/seed.sql
```

---

## 7. Troubleshooting Guide

### 7.1 App Service Deployment Failure

**সমস্যা:** Deployment Pipeline Fail হচ্ছে।

**Check:**
- GitHub Actions Log দেখুন।
- Azure Service Principal Permission ঠিক আছে কি না।
- App Service Name/Resource Group সঠিক আছে কি না।
- Deployment Slot Configuration ঠিক আছে কি না।

**Solution:**
- Service Principal-কে `Contributor` Role দিন (Resource Group Scope)।
- App Service-এ Deployment Slot তৈরি আছে কি না নিশ্চিত করুন।
- Slot Swap Configuration check করুন।

---

### 7.2 Database Connection Failure

**সমস্যা:** App Service Database-এ Connect করতে পারছে না।

**Check:**
- VNet Integration এবং Service Endpoint ঠিক আছে কি না।
- Database Firewall-এ Subnet Allow আছে কি না।
- Key Vault-এ Connection String সঠিক আছে কি না।
- Managed Identity-র Key Vault Access Policy ঠিক আছে কি না।

**Solution:**
- VNet Subnet-এ Service Endpoint `Microsoft.Sql` enable করুন।
- Database Firewall-এ VNet Subnet Rule add করুন।
- Key Vault-এ Secret Value check করুন এবং App Service Managed Identity-কে Get permission দিন।

---

### 7.3 Auto Scaling Not Working

**সমস্যা:** Load দিলেও Instance বাড়ছে না।

**Check:**
- App Service Plan Standard S1 কি না।
- Autoscale Rules ঠিকমতো Configure করা আছে কি না।
- Scale-Out Threshold (CPU 70%) পৌঁছাচ্ছে কি না।

**Solution:**
- App Service Plan-এ `Standard S1` Tier নিশ্চিত করুন।
- Azure Portal-এ Scale Out (App Service Plan) settings check করুন।
- Load Test দিয়ে CPU 70% এর উপরে নিয়ে যান এবং 5 মিনিট অপেক্ষা করুন।

---

### 7.4 Budget Alert Triggered

**সমস্যা:** Budget Alert Email পেয়েছেন।

**Check:**
- Azure Cost Management-এ Current Spend।
- কোন Resource বেশি খরচ করছে।

**Solution:**
- Demo শেষে অবশ্যই `terraform destroy` করুন।
- অপ্রয়োজনীয় Resource বন্ধ করুন।
- Azure Advisor Recommendation দেখুন।

---

### 7.5 Terraform State Lock

**সমস্যা:** Terraform Apply/Destroy কাজ করছে না, State Lock হয়ে আছে।

**Check:**
- Local State File-এ Lock আছে কি না।

**Solution:**
- Lock File মুছে দিন (সাবধানে):
```bash
rm terraform.tfstate.lock.info
```
- যদি Remote State ব্যবহার করেন, তাহলে Blob Lease break করুন।

---

## 8. Operations Checklist

| Task | Command / Action | Frequency |
|---|---|---|
| Environment Start | `terraform apply` | Demo-র আগে |
| Environment Stop | `terraform destroy` | Demo শেষে |
| Budget Review | Azure Cost Management | প্রতি Demo পরে |
| Advisor Check | Azure Advisor | প্রতি Demo পরে |
| Log Review | Application Insights | Demo-র সময় |
| Alert Verification | Email Inbox | Alert Trigger হলে |
| Backup Verification | SQL Database Restore Test | একবার Demo-তে |
| Security Review | Key Vault, Firewall | Demo-র আগে |

---

## 9. Conclusion

এই Runbook-এর মাধ্যমে আমরা Student Portal Platform-এর Infrastructure এবং Application Management সহজে করতে পারব।  
Terraform এবং GitHub Actions ব্যবহার করে পুরো Environment Automation করা হয়েছে।  
Demo-র সময় চালু, বাকি সময় বন্ধ রেখে আমরা Cost Optimization-ও নিশ্চিত করছি।

---