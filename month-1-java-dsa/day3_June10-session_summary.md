# Day 3 — Session Summary
**Date:** June 10, 2026 | **Location:** Page Turn Study Hall, Ameenpur
**Planned:** 8 blocks | **Completed:** 6 blocks | **Rolled over:** 2 blocks

---

## ✅ Completed Topics

### 1. DSA — Longest Subarray with Sum K
- **Approach:** Prefix sum + HashMap (handles negative values; sliding window breaks with negatives)
- **Key insight:** Store `prefixSum → index` in map; check if `prefixSum - K` exists
- **Pattern:** Carry-forward / prefix sum family

---

### 2. DSA — 3Sum + Container With Most Water

**3Sum**
- **Approach:** Sort array → fix one element → two pointers for the rest
- **Key trap:** Skip duplicates at all three pointer positions to avoid duplicate triplets
- **Time complexity:** O(n²)

**Container With Most Water**
- **Approach:** Two pointers from both ends; move the pointer with the shorter height
- **Key insight:** Moving the taller pointer can only decrease area; always move the shorter one

---

### 3. AWS — EC2 Deep Dive
- Instance types, pricing models (On-Demand, Reserved, Spot)
- Security Groups — stateful, instance-level, allow rules only
- Key pairs, AMIs, user data scripts
- Interview angle: EC2 + IAM Role = no hardcoded credentials

---

### 4. AWS — S3 + Presigned URL Lab

**S3 Core Concepts**
- **Buckets & objects** — globally unique bucket names, object key = full path
- **Bucket Policy** — resource-based access control; Block Public Access overrides everything
- **S3 Static Website Hosting** — serves `index.html`; no native HTTPS (needs CloudFront for HTTPS)
- **Versioning** — once enabled, can only suspend (not fully disable); delete adds a delete marker, data still exists; `s3:DeleteObjectVersion` needed to truly purge

**Access Control Hierarchy**
```
Block Public Access (account/bucket level)
    ↓
Bucket Policy
    ↓
IAM Policy
    ↓
ACLs (legacy — mostly ignore)
```

**Presigned URLs**
```java
S3Presigner presigner = S3Presigner.create();

GetObjectPresignRequest presignRequest = GetObjectPresignRequest.builder()
    .signatureDuration(Duration.ofMinutes(15))
    .getObjectRequest(r -> r.bucket("my-bucket").key("file.pdf"))
    .build();

PresignedGetObjectRequest presigned = presigner.presignGetObject(presignRequest);
URL url = presigned.url();
```
- **Key gotcha:** URL is signed with the *generator's* credentials — if those credentials are revoked, URL stops working immediately even before expiry
- **Interview answer:** Credentials are checked at *request time*, not signing time

---

### 5. Java — Virtual Threads Deep Dive (Java 21)

**Core concepts**
- Virtual threads are lightweight threads managed by the JVM, not the OS
- Carrier thread = the platform (OS) thread that *mounts* a virtual thread
- When a virtual thread hits blocking I/O, JVM *unmounts* it from the carrier — carrier is free to run another virtual thread
- Result: high throughput without hundreds of OS threads

**Key APIs**
```java
// Option 1 — direct
Thread.ofVirtual().start(() -> System.out.println("Hello"));

// Option 2 — executor (preferred for many tasks)
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    IntStream.range(0, 10_000).forEach(i ->
        executor.submit(() -> Thread.sleep(Duration.ofMillis(100)))
    );
}
```

**Pinning — the main gotcha**
```java
// BAD — synchronized + blocking = pinning (carrier thread stuck)
synchronized (lock) {
    Thread.sleep(Duration.ofMillis(100));
}

// GOOD — ReentrantLock allows unmounting
lock.lock();
try {
    Thread.sleep(Duration.ofMillis(100));
} finally {
    lock.unlock();
}
```

**Why you don't pool virtual threads**
- They are cheap to create (not OS threads)
- Pooling defeats the purpose — just create one per task
- `newVirtualThreadPerTaskExecutor()` is the right pattern

**Interview answer for "Why virtual threads over thread pool for REST API with DB calls?"**
> DB calls are blocking I/O — with a traditional thread pool, those threads sit blocked and wasted. With virtual threads, the JVM unmounts the virtual thread from the carrier thread the moment it hits the blocking call, freeing the carrier to run another virtual thread. So you get high throughput without needing hundreds of OS threads.

**Resources used**
- JEP 444: https://openjdk.org/jeps/444
- Baeldung Virtual Threads: https://www.baeldung.com/java-virtual-threads

---

### 6. AWS — VPC Fundamentals

**The 5-Component Mental Model**

```
                    INTERNET
                       |
                 [ IGW ]  ← attach to VPC
                       |
        ┌──────── VPC (10.0.0.0/16) ────────┐
        │                                    │
        │  ┌─ Public Subnet (10.0.1.0/24) ─┐│
        │  │   EC2 (web server)             ││
        │  │   NAT Gateway                  ││
        │  └────────────────────────────────┘│
        │                                    │
        │  ┌─ Private Subnet (10.0.2.0/24) ─┐│
        │  │   EC2 (app server)             ││
        │  │   RDS (database)               ││
        │  └────────────────────────────────┘│
        └────────────────────────────────────┘
```

**The 5 Components**

| Component | What it does |
|---|---|
| **VPC** | Isolated network; CIDR block defines IP range |
| **Subnets** | Subdivisions across AZs; Public = route to IGW, Private = no direct internet |
| **Internet Gateway (IGW)** | Door between VPC and internet; one per VPC |
| **NAT Gateway** | Sits in public subnet; lets private resources call OUT; blocks inbound |
| **Route Tables** | Traffic rules; public subnet: `0.0.0.0/0 → IGW`; private: `0.0.0.0/0 → NAT` |

**Security layers**

| Layer | Scope | Type | Rules |
|---|---|---|---|
| Security Group | Instance level | Stateful | Allow only |
| NACL | Subnet level | Stateless | Allow + Deny |

**Stateful vs Stateless — key distinction**
- Security Groups remember the connection — return traffic automatically allowed
- NACLs don't — you must explicitly allow both inbound AND outbound

**Interview answer — "RDS in private subnet needs to download a patch"**
> The private subnet's route table gets an entry: `0.0.0.0/0 → NAT Gateway`. RDS sends outbound traffic to NAT, NAT forwards it to the IGW, IGW reaches the internet. Return traffic comes back the same path. RDS never gets a public IP — NAT handles address translation.

**Traffic path:**
```
RDS (private subnet)
  → Route table: 0.0.0.0/0 → NAT Gateway
    → NAT Gateway (public subnet)
      → IGW
        → Internet
```

---

## 🎯 Interview Q&A — Day 3 (All Questions + Final Answers)

---

**Q1. Your EC2 app needs to read from S3 without hardcoding credentials. How?**

> Attach an IAM Role to the EC2 instance with a policy that grants `s3:GetObject` on the target bucket. The AWS SDK automatically picks up credentials from the instance metadata service (IMDS) — no hardcoding needed.

**Why it works:** IMDS (`169.254.169.254`) serves temporary credentials to the EC2 instance. The SDK reads from IMDS automatically. It's an IAM Role on the instance, not a user, not access keys.

---

**Q2. What happens if the IAM user who generated a presigned URL loses their permissions before the URL expires?**

> The URL stops working immediately. Credentials are checked at request time, not signing time. Revoking the IAM permissions invalidates all presigned URLs generated by that user — even ones with a future expiry.

---

**Q3. Why would you use virtual threads over a traditional thread pool for a REST API with heavy DB calls?**

> DB calls are blocking I/O — with a traditional thread pool, those threads sit blocked and wasted. With virtual threads, the JVM unmounts the virtual thread from the carrier thread the moment it hits the blocking call, freeing the carrier to run another virtual thread. So you get high throughput without needing hundreds of OS threads.

---

**Q4. Your RDS in a private subnet needs to download a patch from the internet. How?**

> The private subnet's route table gets an entry: `0.0.0.0/0 → NAT Gateway`. RDS sends outbound traffic to NAT, NAT forwards it to the IGW, IGW reaches the internet. Return traffic comes back the same path. RDS never gets a public IP — NAT handles the address translation.

**Traffic path:**
```
RDS (private subnet)
  → Route table: 0.0.0.0/0 → NAT Gateway
    → NAT Gateway (public subnet)
      → IGW
        → Internet
```

---

**Q5. What is the difference between a Security Group and a NACL in AWS?**

> Security Groups are at the instance level and are stateful — return traffic is automatically allowed. NACLs are at the subnet level and are stateless — you must explicitly allow both inbound and outbound traffic. Security Groups support allow rules only; NACLs support both allow and deny.

---

**Q6. A user deletes an object in a versioning-enabled S3 bucket. Is the data gone?**

> No. S3 adds a delete marker on top of the existing versions — the data is still there. To truly purge an object you need `s3:DeleteObjectVersion` permission and must delete each version explicitly.

---

## 🔄 Rolled Over to Day 4

| Topic | Priority | Suggested Slot |
|---|---|---|
| DSA — HashMap patterns (3 problems) | HIGH | First DSA slot (morning, fresh brain) |
| Effective Java Ch 1–4 (catch-up) | MEDIUM | Home reading before leaving |

---

## 📊 Day 3 Stats

- **Focused hours:** ~6.5 hours
- **DSA problems solved:** 3 (Longest Subarray, 3Sum, Container With Most Water)
- **AWS services covered:** EC2, S3, VPC
- **Java topics:** Virtual Threads (Java 21)
- **Interview questions practiced:** 4
  - EC2 without hardcoded credentials
  - Presigned URL credential revocation gotcha
  - Virtual threads vs thread pool for REST API
  - RDS patch download via NAT

---

## 🔑 Key Takeaways

1. **Prefix sum + HashMap** beats sliding window when array has negative values
2. **Block Public Access overrides everything** in S3 — know the hierarchy cold
3. **Presigned URL credentials checked at request time** — not signing time
4. **Virtual threads unmount on blocking I/O** — carrier thread stays free
5. **`synchronized` + blocking = pinning** — use `ReentrantLock` with virtual threads
6. **NAT Gateway = one-way door** — private resources can call out, internet can't call in
7. **NACLs are stateless** — need explicit inbound + outbound rules unlike Security Groups
