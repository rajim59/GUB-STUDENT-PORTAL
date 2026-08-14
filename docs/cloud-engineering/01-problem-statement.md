# 01. Problem Statement

## Project Title

**Enterprise High Availability Student Portal Platform**  
**Platform:** Microsoft Azure  
**Institution:** Green University of Bangladesh

---

## Problem Statement

Green University of Bangladesh-এর Student Portal বর্তমানে প্রতি সেমিস্টারের **Pre-Registration, Course Registration, Tuition Fee Payment এবং Result Publication**-এর সময় গুরুতর **Performance ও Availability সমস্যার** সম্মুখীন হয়।

মাত্র **৫–১০ মিনিটের মধ্যে প্রায় ৮,০০০–১৫,০০০ শিক্ষার্থী** একসাথে Portal-এ লগইন করার চেষ্টা করে।  
এই অতিরিক্ত লোডের কারণে বর্তমান Infrastructure সঠিকভাবে কাজ করতে পারে না।

---

## Current System Issues

বর্তমান সিস্টেমে যে সমস্যাগুলো রয়েছে:

| Issue | Description |
|---|---|
| Single Web Server | CPU Utilization 100% হয়ে যায় |
| Database Connection Limit | দ্রুত শেষ হয়ে যায় |
| Session Management | Database-এ সংরক্ষণ হওয়ায় Performance কমে যায় |
| Auto Scaling | কোনো ব্যবস্থা নেই |
| Real-Time Monitoring | কোনো ব্যবস্থা নেই |
| Deployment Process | Manual Deployment হওয়ায় Deployment-এর সময় Service বন্ধ থাকে |
| Failover | কোনো ব্যবস্থা না থাকায় Server Crash করলে পুরো Portal অচল হয়ে যায় |

---

## Impact of the Problem

এই সমস্যাগুলোর কারণে শিক্ষার্থী ও বিশ্ববিদ্যালয়ের উপর নেতিবাচক প্রভাব পড়ে:

- শিক্ষার্থীরা সময়মতো **Course Registration** করতে পারে না।
- **Tuition Fee Payment** ব্যর্থ হয়।
- অনেক শিক্ষার্থী **Registration Deadline** মিস করে।
- **IT Support Team**-এর উপর অতিরিক্ত চাপ সৃষ্টি হয়।
- **Green University of Bangladesh**-এর সেবার মান ও সুনাম ক্ষতিগ্রস্ত হয়।

---

## Business Goal

University Management একটি **আধুনিক, নিরাপদ এবং High Availability ভিত্তিক Student Portal Platform** চায়, যা—

- **Peak Registration Time-এও নিরবচ্ছিন্নভাবে চলবে।**
- প্রয়োজন অনুযায়ী **স্বয়ংক্রিয়ভাবে Auto Scale** করবে।
- কোনো **Server Failure হলেও Service চালু থাকবে।**
- **দ্রুত, নিরাপদ এবং নির্ভরযোগ্যভাবে** সকল শিক্ষার্থীকে সেবা প্রদান করবে।

---

## Business Requirements

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