# 04. Cost Management

## 1. Objective

এই Document-এ Student Portal Platform-এর **Cloud Cost Management Strategy** বর্ণনা করা হয়েছে।  
আমাদের মূল লক্ষ্য:

- Azure $100 Credit-এর মধ্যে থেকে সম্পূর্ণ Project পরিচালনা করা।
- Demo/Presentation-এর সময় মাত্র প্রয়োজনীয় Resources চালু রাখা।
- বাকি সময় সম্পূর্ণ Infrastructure বন্ধ রেখে খরচ কমানো।
- Cost Governance নিশ্চিত করা (Tags, Policies, Budget Alerts)।

---

## 2. Budget Limitations

| Item | Value |
|---|---|
| Total Azure Credit | $100 |
| Planned Maximum Spend | $80 (80% Safety Margin) |
| Project Environment | Lab / University Project |
| Primary Cost Drivers | App Service Plan, SQL Database |
| Cost-Saving Approach | Resource Group On/Off, Serverless, Policies |

---

## 3. Cost Management Strategy Overview

| Strategy | Description |
|---|---|
| Resource Group On/Off | Demo-র সময় চালু, শেষে সম্পূর্ণ Delete |
| Terraform Automation | `terraform apply` / `terraform destroy` দিয়ে নিয়ন্ত্রণ |
| Azure SQL Serverless | Auto-Pause থাকায় Idle Compute খরচ শূন্য |
| App Service Standard S1 | Lab-এর জন্য সাশ্রয়ী, Auto Scaling Supported |
| Monitoring Cost Control | Application Insights Sampling + Daily Cap |
| Governance | Tags, Azure Policy, Budget Alerts |

---

## 4. Resource Group Lifecycle

সব Resources একটি Resource Group-এ রাখা হয়:

```
Resource Group: studentportal-rg
Location: Southeast Asia
```

### 4.1 Demo শুরুর আগে

```bash
terraform init
terraform plan -var-file="environments/lab.tfvars"
terraform apply -var-file="environments/lab.tfvars"
```

এতে সম্পূর্ণ Environment তৈরি হয়:

- App Service Plan (Standard S1, Min 2)
- App Service (Production + Staging Slot)
- Azure SQL Database (Serverless)
- Key Vault
- Virtual Network + Subnet
- Application Insights + Log Analytics
- Budget Alerts & Policies (Optional)

### 4.2 Demo শেষে

```bash
terraform destroy -var-file="environments/lab.tfvars"
```

অথবা Resource Group Delete:

```bash
az group delete --name studentportal-rg --yes --no-wait
```

**ফলাফল:**

- সব ব্যয়বহুল Resources মুছে যায়।
- খরচ প্রায় শূন্য হয়ে যায়।
- Database চাইলে আলাদা রাখা যায় (নিচে Data Retention অংশ দেখুন)।

---

## 5. Data Retention Strategy

আমরা চাইলে Database-কে সম্পূর্ণ Delete না করে রেখে দিতে পারি, কারণ Azure SQL Database Serverless-এ Auto-Pause থাকায় Compute খরচ হয় না।  
শুধু Storage-এর জন্য সামান্য খরচ হয়।

| Option | Approach | সুবিধা | অসুবিধা |
|---|---|---|---|
| A | Database-ও Delete করে দেওয়া | খরচ সম্পূর্ণ শূন্য | Demo-র আগে Schema + Seed Script চালাতে হয় |
| B | Database রেখে দেওয়া | Data সংরক্ষিত থাকে, দ্রুত Demo শুরু | সামান্য Storage Cost (≈ $5/month) |

**Lab Decision:** Option A — Database Delete হবে, প্রয়োজন হলে SQL Script দিয়ে পুনরায় Seed করা হবে।

---

## 6. Tagging Policy

সব Resource-এ নিচের Tags বাধ্যতামূলক:

| Tag Key | Value |
|---|---|
| Project | StudentPortal |
| Environment | Lab |
| Owner | CloudEngineerTeam |
| CostCenter | UniversityLab |

**উদ্দেশ্য:**  
Azure Cost Management-এ Tag-ভিত্তিক খরচ বিশ্লেষণ করা সহজ হয় এবং কোন Resource কোন Project-এর তা স্পষ্ট থাকে।

---

## 7. Azure Policy Enforcement

দুটি Azure Policy ব্যবহার করা হয়:

### 7.1 Allowed App Service Plan SKU

শুধুমাত্র নিচের SKU গুলো Allow করা হয়:

- Standard S1
- Standard S2

অন্যান্য ব্যয়বহুল SKU (যেমন Premium v3, Isolated) Block করা হয়।

### 7.2 Require Resource Tags

প্রতিটি Resource-এ `Project`, `Environment`, `Owner`, `CostCenter` Tags থাকা বাধ্যতামূলক।  
Tag ছাড়া Deployment Fail হবে।

**উপকারিতা:**

- ভুল করে দামি Service তৈরি হলে Policy Block করবে।
- Resource-এর Ownership ও Cost Allocation পরিষ্কার থাকে।

---

## 8. Budget Alerts

Azure Cost Management-এ Budget Set করা হয়:

| Item | Value |
|---|---|
| Budget Scope | Resource Group: `studentportal-rg` |
| Budget Amount | $80 (100% ক্রেডিটের 80%) |
| Alert Threshold | 50%, 80%, 100% |
| Alert Action | Email Notification |

### Threshold Details:

| Threshold | Amount | Action |
|---|---|---|
| 50% | $40 | Warning Email |
| 80% | $64 | Critical Email |
| 100% | $80 | Stop Environment / Alert Admin |

**Terraform Example:**

```hcl
resource "azurerm_consumption_budget_resource_group" "lab_budget" {
  name              = "studentportal-budget"
  resource_group_id = azurerm_resource_group.rg.id

  amount     = 80
  time_grain = "Monthly"

  notification {
    enabled        = true
    threshold      = 50.0
    operator       = "GreaterThan"
    contact_emails = ["cloudengineer@studentportal.com"]
  }

  notification {
    enabled        = true
    threshold      = 80.0
    operator       = "GreaterThan"
    contact_emails = ["team@studentportal.com"]
  }

  notification {
    enabled        = true
    threshold      = 100.0
    operator       = "GreaterThan"
    contact_emails = ["admin@studentportal.com"]
  }
}
```

---

## 9. Cost-Saving Service Configurations

### 9.1 Azure App Service

- **Plan:** Standard S1 (১ vCPU, 1.75 GB RAM)
- **Min Instance:** 2 (HA)  
- **Max Instance:** 5 (Lab-এ Auto Scaling Cap)  
- **Demo শেষে Delete:** সম্পূর্ণ Plan ও App Service মুছে যায়

### 9.2 Azure SQL Database

- **Tier:** Serverless (General Purpose)
- **Min vCore:** 0.5
- **Max vCore:** 4 (Lab)
- **Auto-Pause:** 1 ঘণ্টা Idle থাকলে Compute বন্ধ
- **Storage:** LRS (Locally Redundant)
- **Backup:** Automatic + Point-in-Time Restore

### 9.3 Application Insights & Log Analytics

| Setting | Value |
|---|---|
| Adaptive Sampling | Enabled |
| Daily Data Cap | 0.5 GB/day |
| Data Retention | 30 days |

এতে Monitoring Data Ingestion খরচ Free Limit-এর মধ্যে থাকবে।

### 9.4 Key Vault, VNet, Service Endpoint

- Azure Key Vault: Standard Tier (সামান্য Operation Cost)
- Virtual Network + Service Endpoint: সম্পূর্ণ Free
- Managed Identity: Free
- Azure Policy: Free
- Azure Cost Management: Free

---

## 10. Expected Cost Estimate (Per Demo Session)

আনুমানিক Demo Session: **2–3 ঘণ্টা** (Resources চালু থাকবে)

| Resource | Unit | Duration | Estimated Cost |
|---|---|---|---|
| App Service Plan S1 (2 Instances) | ~$0.10/instance/hr | 3 hr | ~$0.60 |
| App Service Plan S1 (Scale-Out Extra 1-2 Instances) | ~$0.10/instance/hr | 1 hr | ~$0.20 |
| Azure SQL Database Serverless | vCore sec + storage | 3 hr | ~$0.50 |
| Key Vault | Operations | Limited | ~$0.01 |
| Application Insights | Data Ingestion | < 0.5 GB | Free/Included |
| Virtual Network | Free | 3 hr | $0.00 |
| **Total** | | | **~$1.31** |

**Note:**  
উপরের খরচ আনুমানিক। আসল খরচ Region ও Usage অনুযায়ী সামান্য পরিবর্তিত হতে পারে।  
তবে $100 Credit-এ ৫০+ Demo Session অনায়াসে চালানো যাবে।

---

## 11. Demo Cost Calendar

প্রতিটি Demo-র আগে/পরে খরচ Track করার জন্য একটি সাধারণ Calendar রাখা হয়:

| Session | Date | Duration | Resources | Estimated Cost | Actual Cost | Cumulative |
|---|---|---|---|---|---|---|
| Demo 1 | TBD | 3 hours | App Service + SQL + Monitoring | ~$1.31 | TBD | ~$1.31 |
| Demo 2 | TBD | 3 hours | Same | ~$1.31 | TBD | ~$2.62 |
| Demo 3 | TBD | 3 hours | Same | ~$1.31 | TBD | ~$3.93 |

**Actual Cost** Azure Cost Management থেকে সংগ্রহ করা হয়।

---

## 12. Cost Review & Governance

- প্রতি Demo-র পরে Azure Cost Management-এ খরচ Review করা হয়।
- Azure Advisor-এর Cost Recommendations চেক করা হয়।
- অপ্রয়োজনীয় Resource চলমান থাকলে সতর্কতা হিসেবে Budget Alert পাওয়া যায়।
- Terraform State ও Infrastructure Code GitHub-এ Version Control থাকে।

---

## 13. Decision Summary

| Decision | Status |
|---|---|
| Budget Limit | $80 (Safety Margin) |
| Budget Alert Thresholds | 50%, 80%, 100% |
| Resource Group | `studentportal-rg` |
| Tags | Project, Environment, Owner, CostCenter |
| Policy 1 | Allowed App Service SKU (S1, S2) |
| Policy 2 | Require Tags |
| Primary Cost Control | Terraform Apply/Destroy |
| Expected Cost per Demo | ~$1.31 |
| Monitoring Cost Control | Sampling + Daily Cap |
| Database Data Strategy | Delete + Seed Scripts (Option A) |

---

---

এটি হলো **চতুর্থ ডকুমেন্টেশন ফাইল: `04-cost-management.md`**।  
এখন আমরা চাইলে **`05-testing-and-verification.md`** তৈরি করতে পারি।  
আপনি প্রস্তুত থাকলে শুরু করি।
