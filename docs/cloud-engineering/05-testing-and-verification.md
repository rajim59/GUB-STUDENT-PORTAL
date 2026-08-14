# 05. Testing and Verification

## 1. Objective

এই Document-এ Student Portal Platform-এর **Testing এবং Verification Plan** বর্ণনা করা হয়েছে।  
মূল লক্ষ্য হলো প্রমাণ করা যে, Architecture-টি **Business Requirements ও Non-Functional Requirements (NFR)** পূরণ করছে।

Testing-এর মধ্যে থাকবে:

- Smoke Test
- Load / Performance Test
- Auto Scaling Verification
- High Availability / Failover Test
- Zero Downtime Deployment Test
- Disaster Recovery Test
- Monitoring & Alerting Test
- Security Configuration Verification

---

## 2. Testing Tools

| Tool | Purpose | Reason |
|---|---|---|
| **k6** (Local) | Load Testing, Performance Testing | Open Source, JavaScript Scripting, CI/CD Integration, Free |
| **Azure CLI / Portal** | Failover, Scale, Monitoring Verification | Native Azure Management |
| **GitHub Actions** | CI/CD Pipeline Testing | Automated Build/Test/Deploy Stages |
| **Application Insights / Log Analytics** | Availability Ping Test, Alert Verification | Real-time Monitoring |
| **Azure SQL Database** | Point-in-Time Restore Test | Disaster Recovery Verification |

---

## 3. Testing Environments

- **Staging Slot:** Smoke Test, Pre-Production Verification
- **Production Slot:** Load Test, Failover Test, Monitoring Verification
- **Secondary Region (Production Design):** DR Test (Documentation Only)

---

## 4. Test Scenarios & Execution

### 4.1 Smoke Test (Post-Deployment)

**Objective:** নিশ্চিত করা যে Application Basic ভাবে কাজ করছে।

**Steps:**
1. Staging Slot-এ Deploy হওয়ার পরে `/health` Endpoint-এ HTTP Request পাঠানো হয়।
2. Response Code `200 OK` এবং Response Body-তে `status: healthy` থাকা দরকার।
3. Database Connectivity পরীক্ষা করা হয়।
4. Login API-তে একটি Fake Request পাঠিয়ে Response আসছে কি না দেখা হয়।

**Success Criteria:**
- `/health` Endpoint HTTP 200 Returns.
- Database Connection সফল।
- Login API Response Time < 1 sec (Basic Check).

**Tools:** CI/CD Pipeline (GitHub Actions), `curl` বা Postman।

---

### 4.2 Load & Performance Test (Scaled-Down)

**Objective:** Application-এর Response Time, Throughput এবং Auto Scaling Trigger Verify করা।

**Tool:** k6 (Local)

**Test Scenarios:**

| Scenario | Description | Duration | Concurrent Users |
|---|---|---|---|
| Ramp-up Test | ধীরে ধীরে User সংখ্যা বাড়ানো | 5 min | 0 → 200 |
| Sustained Load | নির্দিষ্ট Load বজায় রাখা | 10 min | 200 |
| Spike Test | হঠাৎ User সংখ্যা বাড়িয়ে Auto Scaling Trigger করা | 2 min | 200 → 500 |

**Metrics Observed:**
- Average Response Time (< 200 ms Target, Lab-এ সামান্য বেশি হলে Document)
- Error Rate (< 1% Target)
- Requests per Second (Throughput)
- App Service Instance Count (Scale-Out প্রমাণ)
- Database CPU Percentage

**Success Criteria:**
- Error Rate < 1%
- Scale-Out Trigger হয় (CPU > 70%)
- Response Time Acceptable (Lab-এ < 500 ms Acceptable, Production Target < 200 ms)
- Database-এ কোনো Connection Failure নেই

**Evidence:** k6 Output, App Service Metrics Screenshot, Database Metrics Screenshot।

---

### 4.3 Auto Scaling Verification

**Objective:** প্রমাণ করা যে Load বাড়লে App Service-এ নতুন Instance তৈরি হয় এবং Load কমলে Instance কমে।

**Test Method:**
1. Spike Test চালানো হয় (k6)।
2. Azure Portal-এ App Service Plan-এর Instance Count পর্যবেক্ষণ করা হয়।
3. CPU > 70% হলে Scale-Out হওয়ার জন্য 5 মিনিট অপেক্ষা করা হয়।
4. Instance Count Min 2 → Max 5 পর্যন্ত বাড়ছে কি না Verify করা হয়।
5. Load কমানোর পরে 10 মিনিট অপেক্ষা করে Instance Count কমছে কি না দেখা হয়।

**Success Criteria:**
- Scale-Out Triggered (Instance Count বৃদ্ধি)।
- Scale-In Triggered (Load কমলে Instance Count হ্রাস)।
- Scale Operation-এ কোনো Downtime নেই।

**Evidence:** App Service Plan Scale Events Screenshot, Activity Log।

---

### 4.4 High Availability / Failover Test

**Objective:** প্রমাণ করা যে একটি Instance Down হলেও Service চালু থাকে।

**Test Method:**
1. App Service-এর একটি Instance Manual Stop করা হয় (Azure Portal)।
2. বাকি Instance-এ Traffic যাচ্ছে কি না Load Balancer Verify করে।
3. `/health` Endpoint-এ Continuous Request পাঠানো হয় (k6 বা curl Loop)।
4. Availability Ping Test-এ কোনো Failure আসে কি না দেখা হয়।

**Success Criteria:**
- Service Up থাকে, কোনো Downtime নেই।
- সব Requests সফলভাবে বাকি Instance-এ Serve হয়।
- Availability Ping Test 100% Pass করে।

**Evidence:** Azure Portal Instance Status, Application Insights Availability Result।

---

### 4.5 Zero Downtime Deployment Test

**Objective:** প্রমাণ করা যে নতুন Version Deploy করার সময় Service-এ কোনো Downtime হয় না।

**Test Method:**
1. GitHub-এ একটি ছোট Code Change Push করা হয় (যেমন Version Text Change)।
2. CI/CD Pipeline Trigger হয়, Staging Slot-এ Deploy হয়।
3. Swap Operation-এর সময় Continuous Load (k6) চালানো হয়।
4. Swap-এর আগে ও পরে Response Code এবং Error Rate Monitor করা হয়।

**Success Criteria:**
- Swap-এর সময় কোনো Request Fail হয় না (Failed Request = 0)।
- Response Time-এ সামান্য হেরফের হতে পারে, কিন্তু Downtime শূন্য।
- নতুন Version Production-এ সফলভাবে চলে।

**Evidence:** GitHub Actions Log, k6 Result during Swap, Application Insights Request Log।

---

### 4.6 Disaster Recovery (DR) Test — Point-in-Time Restore

**Objective:** প্রমাণ করা যে Database Data হারালে বা নষ্ট হলে তা পুনরুদ্ধার করা যায়।

**Test Method:**
1. Database-এ একটি Demo Table থেকে কিছু Data Delete করা হয় (Simulated Data Loss)।
2. Azure SQL Database-এর **Point-in-Time Restore** ব্যবহার করে নির্দিষ্ট সময়ের Database Restore করা হয়।
3. Restored Database-এ Data ফিরে এসেছে কি না Verify করা হয়।
4. RTO (Restore Time) Measure করা হয়।

**Success Criteria:**
- Data সফলভাবে Restore হয়।
- RTO Lab-এ 15–60 min (Production Design-এ < 5 min Document)।
- Application আবার Database-এ Connect করতে পারে (Connection String Update)।

**Evidence:** Restore Activity Log, Data Verification Screenshot।

---

### 4.7 Monitoring & Alerting Test

**Objective:** নিশ্চিত করা যে Alert Rules সঠিকভাবে কাজ করছে এবং Email Notification পাওয়া যাচ্ছে।

**Test Method:**
1. একটি App Service Instance বন্ধ করে Availability Alert Trigger করা হয়।
2. k6 দিয়ে CPU Load বাড়িয়ে CPU Alert Trigger করা হয়।
3. Email Notification এসেছে কি না Check করা হয়।
4. Azure Dashboard-এ Metrics Update হচ্ছে কি না Verify করা হয়।

**Success Criteria:**
- Alert Email পাওয়া যায়।
- Dashboard-এ Metrics দেখায়।
- Alert Rule Trigger হওয়ার Evidence পাওয়া যায়।

**Evidence:** Alert Email Screenshot, Azure Monitor Alert History।

---

### 4.8 Security Configuration Verification

**Objective:** Basic Security Settings সঠিকভাবে Configure করা হয়েছে কি না Verify করা।

**Test Method:**
1. HTTP Request পাঠিয়ে HTTPS-এ Redirect হচ্ছে কি না Check করা হয়।
2. Database-এ Public Endpoint থেকে সরাসরি Access চেষ্টা করা হয় (Expect Fail)।
3. Key Vault-এ Secrets শুধু Managed Identity-তে Access আছে কি না Verify করা হয়।
4. SCM (Deployment) Site-এ Developer IP Restriction কাজ করছে কি না Check করা হয়।

**Success Criteria:**
- HTTP → HTTPS Redirect সফল।
- Public Database Access Block।
- Key Vault Access শুধু App Service Managed Identity-তে।
- SCM Access Restriction কার্যকর।

**Evidence:** Browser Redirect Screenshot, Database Firewall Rules Screenshot, Key Vault Access Policies Screenshot।

---

## 5. Requirement Traceability Matrix (NFR Mapping)

| NFR | Test Method | Success Criteria | Status |
|---|---|---|---|
| High Availability (99.95%) | Failover Test, Availability Ping Test | Instance Down-এ Service Up, Ping Test 100% | Pending |
| Auto Scaling | Spike Test + Instance Count Observe | Scale-Out Triggered | Pending |
| Fast Response Time < 200 ms | k6 Load Test | Average Response Time < 200 ms (Lab-এ Acceptable) | Pending |
| Zero Downtime Deployment | Swap Test under Load | Failed Request = 0 | Pending |
| RTO/RPO | Database PITR Test | RTO Measured, RPO Documented | Pending |
| Real-Time Monitoring | Alert Trigger Test | Alert Email Received | Pending |
| Secure Authentication | Basic Config Verification | All Config Checked | Pending |
| Infrastructure as Code | Terraform Plan/Apply | Resources Created Successfully | Pending |
| Cost Optimization | Budget Alert, Cost Analysis | Spend < $80 | Pending |

---

## 6. Test Results Summary (Placeholder)

প্রতিটি Test-এর Result নিচের Table-এ সংরক্ষণ করা হবে:

| Test | Date | Environment | Result | Evidence |
|---|---|---|---|---|
| Smoke Test | TBD | Staging | TBD | TBD |
| Load Test (Ramp-up) | TBD | Production | TBD | TBD |
| Load Test (Sustained) | TBD | Production | TBD | TBD |
| Load Test (Spike) | TBD | Production | TBD | TBD |
| Auto Scaling Verification | TBD | Production | TBD | TBD |
| Failover Test | TBD | Production | TBD | TBD |
| Zero Downtime Deployment | TBD | Production | TBD | TBD |
| DR Test (PITR) | TBD | Production | TBD | TBD |
| Monitoring Alert Test | TBD | Production | TBD | TBD |
| Security Verification | TBD | Production | TBD | TBD |

---

## 7. Conclusion

এই Testing Plan-এর মাধ্যমে আমরা নিশ্চিত করব যে, Student Portal Platform-এর Architecture সমস্ত Business এবং Non-Functional Requirements পূরণ করছে।  
প্রতিটি Test-এর Evidence সংগ্রহ করে `testing-results/` ফোল্ডারে সংরক্ষণ করা হবে।

---