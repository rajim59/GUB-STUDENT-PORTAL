# Green University of Bangladesh High Availability Student Portal Platform

### Microsoft Azure

---

## 1. Document Information

| Item               | Details                                              |
| ------------------ | ---------------------------------------------------- |
| **Project Name**   | Enterprise High Availability Student Portal Platform |
| **Organization**   | Green University of Bangladesh                       |
| **Cloud Platform** | Microsoft Azure                                      |
| **Document Type**  | Project Requirements                                 |
| **Version**        | 1.0                                                  |

---

## 2. Project Overview

Green University of Bangladesh-এর Student Portal সাধারণ সময়ে তুলনামূলকভাবে কম Traffic পরিচালনা করলেও Pre-Registration, Course Registration, Tuition Fee Payment এবং Result Publication-এর সময় অল্প সময়ের মধ্যে বিপুল সংখ্যক শিক্ষার্থী একসাথে Portal ব্যবহার করে।

এই Project-এর লক্ষ্য হলো Microsoft Azure-এর উপর একটি **Highly Available, Scalable, Secure এবং Cost-Optimized Student Portal Platform** তৈরি করা।

---

## 3. Business Problem

বর্তমান Infrastructure-এ প্রধান সমস্যাগুলো হলো:

* Single Web Server-এর কারণে Peak Time-এ CPU Overload হয়।
* Database Connection Limit দ্রুত শেষ হয়ে যায়।
* Session Database-এ রাখার কারণে Performance কমে যায়।
* Auto Scaling নেই।
* Real-Time Monitoring নেই।
* Manual Deployment-এর কারণে Service Interruption হতে পারে।
* Server Failure হলে Failover ব্যবস্থা না থাকায় পুরো Portal বন্ধ হয়ে যেতে পারে।

এর ফলে Course Registration, Tuition Fee Payment এবং অন্যান্য গুরুত্বপূর্ণ Academic Service ব্যাহত হয়।

---

## 4. Business Objectives

নতুন Platform-এর লক্ষ্য:

* Peak Time-এ Student Portal সচল রাখা।
* Traffic অনুযায়ী Auto Scaling করা।
* Server/Instance Failure হলেও Service চালু রাখা।
* দ্রুত ও নিরাপদ Service প্রদান করা।
* Zero Downtime Deployment নিশ্চিত করা।
* System Monitoring এবং Disaster Recovery নিশ্চিত করা।
* Cloud Cost নিয়ন্ত্রণে রাখা।

---

## 5. Business Requirements

| ID         | Requirement              |
| ---------- | ------------------------ |
| **BR-001** | High Availability        |
| **BR-002** | Auto Scaling             |
| **BR-003** | Zero Downtime Deployment |
| **BR-004** | Disaster Recovery        |
| **BR-005** | Secure Authentication    |
| **BR-006** | Fast Response Time       |
| **BR-007** | Real-Time Monitoring     |
| **BR-008** | Infrastructure as Code   |
| **BR-009** | Cost Optimization        |

---

## 6. Functional Requirements

### Student

| ID         | Function             |
| ---------- | -------------------- |
| **FR-001** | Student Login        |
| **FR-002** | Pre-Registration     |
| **FR-003** | Course Registration  |
| **FR-004** | Add/Drop Course      |
| **FR-005** | Tuition Fee Payment  |
| **FR-006** | Result Viewing       |
| **FR-007** | Class & Exam Routine |
| **FR-008** | Notice Board         |
| **FR-009** | Student Profile      |
| **FR-010** | Transcript Request   |

### Teacher & Admin

| ID         | Function                    |
| ---------- | --------------------------- |
| **FR-011** | Teacher Dashboard           |
| **FR-012** | Result Publishing           |
| **FR-013** | Admin Dashboard             |
| **FR-014** | Student & Course Management |

---

## 7. Non-Functional Requirements

| ID          | Requirement           |          Target |
| ----------- | --------------------- | --------------: |
| **NFR-001** | Availability          |    **≥ 99.95%** |
| **NFR-002** | Concurrent Users      |     **20,000+** |
| **NFR-003** | Average Response Time |    **< 200 ms** |
| **NFR-004** | RTO                   | **< 5 Minutes** |
| **NFR-005** | RPO                   |  **< 1 Minute** |
| **NFR-006** | Deployment Downtime   |   **0 Seconds** |

---

## 8. Usage Characteristics

### Normal Traffic

সাধারণ সময়ে Portal-এর ব্যবহার তুলনামূলকভাবে কম।

**Estimated Daily Users: 200–500**

### Peak Traffic

Registration এবং Result Publication-এর সময় Traffic হঠাৎ বৃদ্ধি পেতে পারে।

**Expected Peak:**
**8,000–15,000 users within 5–10 minutes**

Architecture-কে এই Traffic Spike Handle করতে সক্ষম হতে হবে।

---

## 9. Constraints & Considerations

* Solution Microsoft Azure-এর উপর তৈরি হবে।
* Cost Optimization গুরুত্বপূর্ণ।
* Normal Time-এর Low Traffic এবং Peak Time-এর High Traffic উভয় বিবেচনা করতে হবে।
* অপ্রয়োজনীয় Azure Services ব্যবহার করা যাবে না।
* প্রতিটি গুরুত্বপূর্ণ Architecture Decision-এর যথাযথ কারণ থাকতে হবে।
* Solution-টি Production-like Security ও Reliability বিবেচনা করবে।

---

## 10. Assumptions

* Primary Users হলো Green University-এর Students, Teachers এবং Administrators।
* Peak Traffic মূলত Academic Registration এবং Result Publication-এর সময় তৈরি হবে।
* Application Development Team মূল Application তৈরি করবে।
* Cloud Engineering Team Infrastructure, Security, Deployment, Monitoring এবং Reliability নিয়ে কাজ করবে।
* Final Resource Configuration Testing-এর ফলাফলের ভিত্তিতে নির্ধারণ করা হবে।

---

## 11. Success Criteria

Project সফল হবে যদি:

* High Availability নিশ্চিত করা যায়।
* Peak Load-এর সময় Auto Scaling কাজ করে।
* Instance Failure-এর পরও Service সচল থাকে।
* Response Time Target অর্জন করা যায়।
* Zero Downtime Deployment প্রমাণ করা যায়।
* Monitoring ও Alerting কার্যকর থাকে।
* Disaster Recovery Target অর্জন করা যায়।
* Infrastructure Code-এর মাধ্যমে পুনরায় তৈরি করা যায়।
* Cost Optimization Demonstrate করা যায়।

---

## 12. Requirement Verification

কোনো Requirement শুধু Configuration দেখে সম্পন্ন ধরা হবে না।

প্রতিটি গুরুত্বপূর্ণ Requirement Test করে Evidence-এর মাধ্যমে Verify করা হবে।

**Requirement → Implementation → Test → Evidence → PASS/FAIL**

উদাহরণ:

| Requirement       | Verification      |
| ----------------- | ----------------- |
| High Availability | Failure Test      |
| Auto Scaling      | Load Test         |
| Zero Downtime     | Deployment Test   |
| Disaster Recovery | Recovery Test     |
| Security          | Security Test     |
| Performance       | Performance Test  |
| Monitoring        | Alert Test        |
| IaC               | Re-Provision Test |
| Cost              | Cost Analysis     |

---

## 13. Requirement Traceability

প্রতিটি Requirement-এর ID ব্যবহার করে Architecture, Implementation এবং Testing-এর মধ্যে সম্পর্ক বজায় রাখা হবে।

```text
Requirement
     ↓
Architecture Decision
     ↓
Implementation
     ↓
Testing
     ↓
Evidence
     ↓
PASS / FAIL
```

### Requirement Status

* **NOT STARTED**
* **IN PROGRESS**
* **IMPLEMENTED**
* **VALIDATING**
* **PASS**
* **FAIL**

এই Requirement Document পরবর্তী Architecture Design এবং Implementation-এর Baseline হিসেবে ব্যবহৃত হবে।
