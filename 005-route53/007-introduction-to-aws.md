## What is AWS?

### 1. Absolute Zero — What is AWS?

AWS stands for:

**Amazon Web Services**

At the simplest level:

AWS is a platform that lets you rent computing resources over the Internet.

Instead of buying physical servers, you use Amazon’s infrastructure.

Think of AWS as:

A giant collection of remote computers you can use on demand.

---

### 2. Why AWS Exists

Before cloud platforms like AWS, companies had to:

• Buy physical servers  
• Build data centers  
• Maintain hardware  
• Handle scaling manually  
• Pay large upfront costs

This created problems:

• Expensive  
• Slow to scale  
• Hard to maintain  
• Resource waste

AWS solves these problems.

---

### 3. What Cloud Computing Means

Cloud computing means:

Using computing resources delivered over the Internet instead of owning physical machines.

Instead of:

“My server in my building”

You have:

“My server in someone else’s data center”

But with full control.

---

### 4. Key Idea Behind Cloud Computing

Cloud = On-demand remote infrastructure

You can instantly access:

• Servers  
• Storage  
• Databases  
• Networking  
• Security systems  
• AI services

Without managing hardware.

---

### 5. Traditional Infrastructure vs Cloud

Traditional:

• Buy hardware  
• Install manually  
• Fixed capacity  
• High upfront cost

Cloud:

• Rent resources  
• Scale instantly  
• Pay-as-you-go  
• Flexible capacity

This shift changed the entire tech industry.

---

### 6. AWS as a Cloud Provider

AWS provides:

Infrastructure + Services

Examples:

• Virtual servers (EC2)  
• Storage (S3)  
• Databases (RDS, DynamoDB)  
• Networking (VPC)  
• DNS (Route 53)

AWS is not a single product.

It is an ecosystem.

---

### 7. Where Does AWS Actually Run?

AWS runs inside:

**Massive global data centers**

These are physical buildings filled with:

• Servers  
• Networking equipment  
• Cooling systems  
• Redundant power  
• Security systems

Cloud is still physical.

Just abstracted.

---

### 8. AWS Global Infrastructure — The Big Picture

AWS infrastructure is organized into:

• Regions  
• Availability Zones  
• Edge Locations

Each layer serves a specific purpose.

---

### 9. What is an AWS Region?

A **Region** is:

A geographic area containing AWS data centers.

Examples:

• Mumbai  
• Singapore  
• Frankfurt  
• Virginia

Each region is isolated from others.

---

### 10. Why Regions Exist

Regions provide:

• Geographic proximity  
• Data residency control  
• Fault isolation  
• Latency optimization  
• Legal compliance

Choosing a region is a critical design decision.

---

### 11. Region Isolation — Critical Insight

Regions are largely independent.

Failure in one region:

Does NOT automatically affect others.

This enables:

Disaster recovery strategies.

---

### 12. What is an Availability Zone (AZ)?

An **Availability Zone** is:

One or more data centers inside a region.

Each region contains:

Multiple AZs.

Example:

Mumbai region → Multiple AZs

---

### 13. Why Availability Zones Exist

AZs provide:

• High availability  
• Fault tolerance  
• Redundancy  
• Failure isolation

Instead of relying on one data center:

Workloads span multiple AZs.

---

### 14. AZ Isolation — Extremely Important Concept

Availability Zones are designed to be:

• Physically separate  
• Independent power  
• Independent networking  
• Low-latency connected

Failure in one AZ:

Does NOT collapse entire region.

---

### 15. High Availability Design Pattern

Typical architecture:

Application servers across:

• AZ-1  
• AZ-2  
• AZ-3

Result:

One AZ failure → System still runs.

---

### 16. Gotcha — Region ≠ Availability Zone

Common beginner confusion.

Region:

Geographic grouping.

Availability Zone:

Failure-isolated data center cluster.

---

### 17. Latency and Regions

Closer region → Lower latency.

User in India:

Mumbai region faster than US region.

Distance still matters.

Physics still matters.

---

### 18. Data Residency and Compliance

Some regulations require:

Data stored within specific geography.

Regions enable compliance.

Example:

Indian financial systems → Mumbai region.

---

### 19. AWS Edge Locations (Advanced Layer)

Used by services like:

• CloudFront  
• Route 53

Edge locations sit closer to users.

Purpose:

Reduce latency globally.

---

### 20. Now the Interesting Question:

Why is Route 53 GLOBAL, not regional?

This is where deeper infrastructure thinking begins.

---

### 21. What is Route 53?

Route 53 is AWS’s:

**DNS (Domain Name System) service**

It translates:

Domain names → IP addresses

Just like DNS anywhere else.

---

### 22. Why DNS Cannot Be Regional

DNS is part of:

**Global Internet infrastructure**

When someone types:

    example.com

The lookup must work worldwide.

Not tied to one region.

---

### 23. DNS is Fundamentally Global

DNS must:

• Resolve from anywhere  
• Handle global queries  
• Provide low-latency answers  
• Remain highly available  
• Survive regional failures

Hence:

DNS services are global by design.

---

### 24. Why Route 53 is a Global Service

Route 53 operates across:

Multiple regions + Edge Locations

Benefits:

• Ultra-low latency resolution  
• High resilience  
• Global availability  
• Fault tolerance

DNS cannot afford regional dependency.

---

### 25. What Happens If DNS Were Regional?

Imagine DNS tied to Mumbai region.

Mumbai outage → Entire domain resolution fails globally.

Unacceptable design.

---

### 26. Route 53 Uses Distributed Infrastructure

Route 53 leverages:

**Anycast routing**

Multiple servers share same IP.

User automatically connects to nearest healthy node.

---

### 27. Anycast — Key Performance Mechanism

Advantages:

• Faster responses  
• Automatic failover  
• Load distribution  
• Global resilience

Critical for DNS systems.

---

### 28. Gotcha — Global Service ≠ No Regions

Global services still use regional infrastructure.

But abstract complexity away.

Users don’t choose region.

System handles routing.

---

### 29. AWS Service Scope Types (Advanced Insight)

AWS services fall into categories:

• Regional (EC2, RDS, VPC)  
• Global (Route 53, IAM, CloudFront)

Depends on service purpose.

---

### 30. Why Compute Services Are Regional

Because:

• Latency sensitive  
• Data locality important  
• Regulatory constraints  
• Failure isolation

Running servers globally by default is inefficient.

---

### 31. Why DNS Must Be Global

Because:

DNS is discovery infrastructure.

Must work:

Anywhere → Anytime → Under failure conditions.

---

### 32. Final Mental Model

AWS is:

A cloud platform built on global physical infrastructure.

Organized into:

• Regions → Geographic isolation  
• Availability Zones → Failure isolation  
• Edge Locations → Latency optimization

Route 53 is global because:

DNS is inherently global infrastructure.

Not tied to one geography.

---

### 33. Elegant Big Picture

AWS abstracts:

Physical data centers → Regions → AZs → Services

Users focus on:

Applications & systems.

AWS handles:

Infrastructure complexity.
