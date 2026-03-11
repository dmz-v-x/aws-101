## Multi-AZ Designs — Subnetting for High Availability and AZ-Aware Services

High availability (HA) is a core design principle in AWS architecture. One of the primary mechanisms AWS provides for HA is the use of **Availability Zones (AZs)**.

To build resilient systems, architectures should distribute infrastructure across **multiple Availability Zones** and design networking (especially **subnet layouts**) to support that distribution.

---

### 1. What Is an Availability Zone?

An **Availability Zone (AZ)** is an isolated data center (or group of data centers) inside an AWS region.

Example region:

```
ap-south-1 (Mumbai)
```

Example AZs:

```
ap-south-1a
ap-south-1b
ap-south-1c
```

Each AZ has:

```
independent power
independent networking
physical isolation
```

This reduces the risk of a **single failure affecting all infrastructure**.

---

### 2. Why Multi-AZ Design Is Important

If infrastructure is deployed in only **one AZ**, an outage in that AZ can bring down the entire application.

Example single-AZ architecture:

```
VPC
 |
Subnet (AZ-a)
 |
Application Server
 |
Database
```

If the AZ fails, the system becomes unavailable.

Multi-AZ architecture solves this.

Example:

```
AZ-a → Application Server
AZ-b → Application Server
```

If one AZ fails, traffic continues through the other.

---

### 3. Multi-AZ Architecture Example

Typical production architecture:

```
                Internet
                   |
           Application Load Balancer
             /                 \
          AZ-a               AZ-b
       Public Subnet      Public Subnet
            |                  |
       App Server         App Server
            |                  |
       Private Subnet     Private Subnet
            |                  |
        Database           Database
```

Traffic automatically shifts to healthy zones.

---

### 4. Subnet Design for Multi-AZ Environments

Subnets are **AZ-specific**.

Important rule:

```
A subnet exists in only one AZ.
```

Therefore, high availability requires creating **multiple subnets across AZs**.

Example VPC:

```
10.0.0.0/16
```

Subnet layout:

```
AZ-a
  10.0.1.0/24 → Public
  10.0.10.0/24 → Private
  10.0.20.0/24 → Database

AZ-b
  10.0.2.0/24 → Public
  10.0.11.0/24 → Private
  10.0.21.0/24 → Database
```

This layout supports multi-AZ workloads.

---

### 5. Typical Multi-AZ Subnet Layout

Production VPCs usually follow this pattern:

```
VPC 10.0.0.0/16

AZ-a
 ├ Public Subnet
 ├ Private App Subnet
 └ Private DB Subnet

AZ-b
 ├ Public Subnet
 ├ Private App Subnet
 └ Private DB Subnet
```

Benefits:

```
fault tolerance
traffic distribution
scaling capability
```

---

### 6. Load Balancers and Multi-AZ

Load balancers are **AZ-aware**.

Example:

```
Application Load Balancer
   |
-------------------------------
|                             |
AZ-a Target                AZ-b Target
EC2 Instance              EC2 Instance
```

The load balancer:

```
distributes traffic across AZs
detects unhealthy instances
routes traffic to healthy AZs
```

---

### 7. Auto Scaling Across AZs

Auto Scaling Groups (ASG) can launch instances across multiple AZs.

Example:

```
Auto Scaling Group
  |
-------------------------
|                       |
AZ-a Instance       AZ-b Instance
```

Benefits:

```
automatic instance recovery
balanced load across AZs
high availability
```

---

### 8. Multi-AZ Database Design

Databases must also support high availability.

Example using **Amazon RDS Multi-AZ**:

```
Primary DB (AZ-a)
        |
Synchronous Replication
        |
Standby DB (AZ-b)
```

If the primary database fails:

```
AWS automatically promotes the standby
```

Applications reconnect automatically.

---

### 9. Example Highly Available Architecture

Example production architecture:

```
                 Internet
                    |
            Application Load Balancer
              /                   \
          AZ-a                  AZ-b
       Public Subnet         Public Subnet
           |                      |
      Web Server            Web Server
           |                      |
      Private Subnet        Private Subnet
           |                      |
     App Service            App Service
           |                      |
        RDS Primary          RDS Standby
```

This architecture tolerates AZ failures.

---

### 10. NAT Gateway Placement

NAT Gateways are **AZ-specific resources**.

Best practice:

```
Deploy one NAT Gateway per AZ
```

Example:

```
AZ-a
  NAT Gateway

AZ-b
  NAT Gateway
```

Private subnets should route to the NAT in their AZ.

Benefits:

```
fault tolerance
avoids cross-AZ charges
improves performance
```

---

### 11. AZ-Aware AWS Services

Many AWS services are designed for multi-AZ deployments.

Examples:

| Service | AZ Awareness |
|------|------|
| EC2 | Instances launched per AZ |
| ALB/NLB | Distributes traffic across AZs |
| RDS Multi-AZ | Primary + standby DB |
| EFS | Multi-AZ storage access |
| Auto Scaling | Distributes instances across AZs |
| EKS | Worker nodes across AZs |

These services help build resilient architectures.

---

### 12. Storage and Multi-AZ

Some AWS storage services replicate data automatically across AZs.

Examples:

**Amazon S3**

```
Objects replicated across multiple AZs
```

**Amazon EFS**

```
File system accessible from multiple AZs
```

**Amazon RDS Multi-AZ**

```
Primary database with standby replica
```

This provides high durability and availability.

---

### 13. Cross-AZ Traffic Considerations

Traffic between AZs is allowed but may incur **data transfer charges**.

Example:

```
EC2 (AZ-a) → Database (AZ-b)
```

Best practice:

```
place dependent services in same AZ
but maintain redundancy across AZs
```

This balances performance and availability.

---

### 14. Failure Scenario Example

Example failure:

```
AZ-a becomes unavailable
```

Architecture before failure:

```
ALB
 |
AZ-a → Web Server
AZ-b → Web Server
```

During failure:

```
ALB detects AZ-a failure
Traffic routed to AZ-b
Application remains available
```

This demonstrates the value of multi-AZ architecture.

---

### 15. Mental Model

Think of Availability Zones like **separate buildings in the same campus**.

```
Building A → AZ-a
Building B → AZ-b
Building C → AZ-c
```

If one building loses power, operations continue in the others.

Similarly:

```
AZ failure → application still runs in other AZs
```

---

### 16. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Availability Zone | Isolated data center in a region |
| Multi-AZ Architecture | Deploying resources across AZs |
| AZ-Specific Subnets | Subnets exist in one AZ |
| Load Balancing | Traffic distributed across AZs |
| Multi-AZ Databases | Primary + standby DB |

---

### 17. Checkpoint Questions

You should now be able to answer these.

#### What is an Availability Zone?

A physically isolated data center within an AWS region.

---

#### Why use multi-AZ architectures?

```
To achieve high availability and fault tolerance.
```

---

#### Can a subnet span multiple AZs?

```
No
A subnet exists in only one AZ.
```

---

#### Why deploy NAT gateways per AZ?

```
To avoid cross-AZ traffic and improve availability.
```

---

#### What AWS service automatically distributes traffic across AZs?

```
Application Load Balancer
```
