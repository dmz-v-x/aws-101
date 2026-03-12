## EC2 + Multi-AZ Architecture

When designing applications in the cloud, one of the most important goals is ensuring that the system **remains available even when failures occur**. Hardware failures, power outages, or network issues can affect servers inside a single data center.

To reduce the impact of such failures, AWS provides multiple **Availability Zones (AZs)** within each region.

A **Multi-AZ architecture** is a design pattern where EC2 instances are distributed across **multiple Availability Zones** instead of being placed in a single zone.

By spreading infrastructure across multiple AZs, applications can continue running even if one Availability Zone experiences problems.

A typical architecture looks like this:

```
Users
   ↓
Load Balancer
   ↓
AZ-1: EC2 Instance(s)
AZ-2: EC2 Instance(s)
AZ-3: EC2 Instance(s)
```

In this setup, traffic is distributed across instances located in different Availability Zones.

---

### 1. Fault Tolerance

**Fault tolerance** refers to the ability of a system to **continue operating even when part of the system fails**.

In a single-AZ architecture, if the Availability Zone experiences an outage, all EC2 instances in that zone may become unavailable.

Example of single-AZ risk:

```
Users → Load Balancer → AZ-1 → EC2 Instances
```

If AZ-1 fails, the entire application becomes unavailable.

In a **Multi-AZ architecture**, instances are distributed across multiple zones.

Example:

```
Users → Load Balancer
        ↓
   AZ-1 → EC2 Instances
   AZ-2 → EC2 Instances
```

If AZ-1 fails:

- traffic is automatically routed to instances in AZ-2
- the application continues running

This architecture ensures that the system can **tolerate infrastructure failures** without completely shutting down.

Auto Scaling Groups are often used together with Multi-AZ deployments to automatically launch new instances in healthy zones.

---

### 2. High Availability

**High availability** refers to designing systems so that they remain **accessible and operational for users with minimal downtime**.

Multi-AZ architecture improves availability because traffic can be served from multiple independent data centers.

Availability Zones in AWS are designed to be:

- physically separated
- connected with high-speed networking
- isolated from failures in other zones

By placing EC2 instances in multiple zones, the system avoids a **single point of failure**.

Typical high availability architecture includes:

```
Internet Users
      ↓
Application Load Balancer
      ↓
AZ-1 → EC2 Instance
AZ-2 → EC2 Instance
```

The load balancer distributes traffic between instances in both zones.

Benefits of this architecture include:

- improved uptime
- better resilience against infrastructure failures
- ability to perform maintenance without downtime
- better load distribution

High availability is especially important for:

- production web applications
- financial systems
- large-scale APIs
- enterprise applications

---

### 3. Summary

A **Multi-AZ architecture** distributes EC2 instances across multiple Availability Zones to improve reliability and uptime.

Key advantages include:

| Concept | Description |
|------|------|
| Multi-AZ Architecture | Deploy EC2 instances across multiple Availability Zones |
| Fault Tolerance | System continues working even if one zone fails |
| High Availability | Application remains accessible with minimal downtime |

By combining **load balancers, Auto Scaling Groups, and Multi-AZ deployments**, AWS architectures can achieve highly resilient and scalable systems that remain operational even during infrastructure failures.
