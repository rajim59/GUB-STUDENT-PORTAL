# 02. Requirements

## 1. Business Requirements

নতুন প্ল্যাটফর্মে অবশ্যই থাকতে হবে:

- High Availability
- Auto Scaling
- Zero Downtime Deployment
- Disaster Recovery
- Secure Authentication
- Fast Response Time
- Real-Time Monitoring
- Infrastructure as Code (IaC)
- Cost Optimization

---

## 2. Functional Requirements

Student Portal-এ নিম্নলিখিত ফিচারগুলো থাকতে হবে:

| Module | Feature |
|---|---|
| Student Login | Secure Login, Logout |
| Registration | Pre-Registration, Course Registration, Add/Drop Course |
| Payment | Tuition Fee Payment |
| Result | Result Publishing |
| Routine | Class Routine, Exam Routine |
| Notice | Notice Board |
| Profile | Student Profile, Transcript Request |
| Dashboard | Teacher Dashboard, Admin Dashboard |

> **Note:**  
> Functional Requirement-এর বিস্তারিত Implementation Frontend এবং Backend Developer-দের দায়িত্ব।  
> Cloud Engineering-এর জন্য এই তালিকাটি মূলত Infrastructure Capacity Planning ও Feature Availability নিশ্চিত করতে ব্যবহৃত হবে।

---

## 3. Non-Functional Requirements (NFR)

### 3.1 NFR Table

| Requirement | Target |
|---|---|
| Availability | 99.95% |
| Concurrent Users | 20,000+ |
| Average Response Time | Less than 200 ms |
| RTO (Recovery Time Objective) | Less than 5 Minutes |
| RPO (Recovery Point Objective) | Less than 1 Minute |
| Deployment Downtime | Zero Seconds |

### 3.2 NFR Explanation

| NFR | কেন দরকার |
|---|---|
| Availability 99.95% | Registration/Payment-এর সময় Portal Down থাকলে শিক্ষার্থী ভোগান্তিতে পড়ে |
| Concurrent Users 20,000+ | Peak Time-এ ৮,০০০–১৫,০০০+ শিক্ষার্থী একসাথে Portal ব্যবহার করবে |
| Response Time < 200 ms | দ্রুত User Experience নিশ্চিত করার জন্য |
| RTO < 5 Minutes | কোনো Disaster হলে দ্রুত Service ফিরিয়ে আনা |
| RPO < 1 Minute | Data Loss সর্বনিম্ন রাখা |
| Deployment Downtime Zero Seconds | নতুন Feature/Update দিতে গিয়ে Service বন্ধ রাখা যাবে না |

---

## 4. Requirement Verification Checklist

| Requirement | Verification Method | Status |
|---|---|---|
| High Availability | Failover Test, Availability Ping Test | Pending |
| Auto Scaling | Load Test (Spike Test) | Pending |
| Zero Downtime Deployment | Swap Test under Continuous Load | Pending |
| Disaster Recovery | Database Point-in-Time Restore Test | Pending |
| Secure Authentication | Basic Security Configuration Review | Pending |
| Fast Response Time | Load Test (k6) | Pending |
| Real-Time Monitoring | Alert Trigger Test | Pending |
| Infrastructure as Code | Terraform Deployment Test | Pending |
| Cost Optimization | Budget Alert & Cost Analysis | Pending |

> **Note:**  
> Verification-এর বিস্তারিত Plan এবং Result পরে `05-testing-and-verification.md`-এ থাকবে।

---

## 5. Lab Implementation vs Production Design

Lab Project-এ $100 Azure Credit-এর সীমাবদ্ধতার কারণে কিছু NFR Target (বিশেষত RTO/RPO) Production-এর মতো পুরোপুরি বাস্তবায়ন না করে **Simplified Lab Strategy** ব্যবহার করা হয়েছে।  
তবে Architecture Document-এ Production Design-এ পূর্ণ Target উল্লেখ থাকবে।

| NFR | Lab Implementation | Production Design |
|---|---|---|
| RTO < 5 min | 15–60 min (Database Restore) | < 5 min (Auto-Failover Group) |
| RPO < 1 min | 5–10 min (Backup Log) | < 1 min (Active Geo-Replication) |
| Concurrent Users 20k+ | Scaled-down Load Test (200–500) | Full-scale Load Test (20k) |