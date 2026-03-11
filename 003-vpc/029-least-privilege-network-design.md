## VPC Least Privilege Networking Design — Segmentation and Microsegmentation

Modern cloud security follows the principle of **least privilege**.  
This principle applies not only to IAM permissions but also to **network access**.

In traditional networks, many systems could communicate freely once inside the network perimeter.  
In modern cloud architectures, we assume **no internal system is automatically trusted**.

Therefore, network communication should be **restricted to only what is required**.

---

### 1. What Is Least Privilege Networking?

Least privilege networking means:

```
Allow only the minimum network communication required.
```

Example:

Instead of allowing:

```
App Server → Any Database
```

Restrict communication to:

```
App Server → Specific Database → Specific Port
```

Example rule:

```
Allow TCP 5432 from App Security Group
```

All other traffic is denied.

---

### 2. Why Network Segmentation Matters

Without segmentation:

```
Single VPC
 |
-----------------------------------
|        |        |        |
App     DB      Cache    Admin
```

All resources can potentially talk to each other.

This creates security risks:

```
lateral movement
internal attacks
accidental access
```

Segmentation limits blast radius.

---

### 3. What Is Network Segmentation?

Segmentation divides infrastructure into **isolated network zones**.

Example segmented architecture:

```
VPC
 |
-------------------------------------
|          |           |            |
Web Tier  App Tier   Data Tier   Admin Tier
```

Each tier communicates only with specific tiers.

Example rules:

```
Web → App allowed
App → DB allowed
Web → DB blocked
```

---

### 4. VPC-Level Segmentation

Basic segmentation can be done at the **VPC level**.

Example architecture:

```
VPC-A → Production
VPC-B → Development
VPC-C → Security
```

Benefits:

```
strong isolation
separate routing
separate security policies
```

Communication occurs through:

```
VPC peering
Transit Gateway
PrivateLink
```

---

### 5. Subnet-Level Segmentation

Inside a VPC, segmentation begins with **subnets**.

Example:

```
VPC 10.0.0.0/16

Public Subnets
Private App Subnets
Private Database Subnets
```

Architecture:

```
Internet
   |
Public Subnet
   |
Application Subnet
   |
Database Subnet
```

Each layer has stricter rules.

---

### 6. Security Groups for Microsegmentation

Security groups provide **instance-level segmentation**.

Security groups are **stateful**.

Example:

```
Web-SG
App-SG
DB-SG
```

Rules:

```
Web-SG → Allow HTTP from internet
App-SG → Allow traffic from Web-SG
DB-SG → Allow traffic from App-SG
```

Architecture:

```
Internet
   |
Web Server (Web-SG)
   |
App Server (App-SG)
   |
Database (DB-SG)
```

This enforces least privilege communication.

---

### 7. Security Group Referencing

Instead of allowing IP ranges, security groups can reference other groups.

Example:

```
Allow TCP 8080 from App-SG
```

Benefits:

```
automatic scaling support
dynamic instance membership
stronger security boundaries
```

When instances scale, security rules automatically apply.

---

### 8. Network ACLs for Subnet-Level Controls

Network ACLs provide **stateless subnet firewalls**.

Example rule set:

```
Allow inbound HTTP
Allow outbound ephemeral ports
Deny all other traffic
```

NACLs are typically used for:

```
coarse subnet restrictions
blocking specific IP ranges
compliance requirements
```

Most segmentation is handled by **security groups**.

---

### 9. Microsegmentation Concept

Microsegmentation goes beyond subnet-level isolation.

Definition:

```
Fine-grained security policies controlling communication between individual workloads.
```

Example architecture:

```
Service A
Service B
Service C
```

Rules:

```
Service A → Service B allowed
Service A → Service C blocked
Service B → Service C allowed
```

Each service has its own policy.

---

### 10. Microsegmentation Example (Microservices)

Example microservice platform:

```
Frontend Service
Order Service
Payment Service
Auth Service
```

Security policies:

```
Frontend → Order allowed
Order → Payment allowed
Payment → Auth allowed
Frontend → Payment blocked
```

Architecture:

```
Frontend
   |
Order Service
   |
Payment Service
   |
Auth Service
```

Each hop is controlled.

---

### 11. Private Load Balancers for Segmentation

Internal load balancers help enforce service boundaries.

Example:

```
Frontend
   |
Internal ALB
   |
------------------------------
|              |             |
Order       Payment        Auth
```

Only allowed services communicate with each backend.

---

### 12. Segmentation Using Multiple VPCs

Large environments use **multi-VPC segmentation**.

Example:

```
Edge VPC
Application VPC
Data VPC
Security VPC
```

Architecture:

```
Internet
   |
Edge VPC
   |
Transit Gateway
   |
Application VPC
   |
Data VPC
```

Benefits:

```
strong isolation
controlled routing
centralized inspection
```

---

### 13. Inspection VPC Pattern

Traffic can be forced through security inspection.

Example:

```
Application VPC
     |
Transit Gateway
     |
Inspection VPC
     |
Firewall
     |
Internet
```

All traffic is inspected before leaving the network.

---

### 14. Zero Trust Networking

Microsegmentation aligns with **zero trust architecture**.

Zero trust principle:

```
Never trust
Always verify
```

This means:

```
every connection must be explicitly allowed
```

Even internal communication requires authorization.

---

### 15. Least Privilege Example Architecture

Example production architecture:

```
Internet
   |
Public ALB
   |
Web Tier
   |
Internal ALB
   |
Application Tier
   |
Database Tier
```

Security rules:

```
Internet → Web allowed
Web → App allowed
App → DB allowed
All other traffic blocked
```

---

### 16. Observability and Enforcement

Monitoring tools help enforce segmentation.

Tools include:

```
VPC Flow Logs
AWS Network Firewall
AWS GuardDuty
AWS Security Hub
```

These tools detect unauthorized communication.

---

### 17. Common Segmentation Mistakes

Common issues include:

```
allowing 0.0.0.0/0 internally
large flat networks
missing service isolation
broad security group rules
```

Example bad rule:

```
Allow ALL traffic from VPC CIDR
```

This defeats segmentation.

---

### 18. Mental Model

Think of segmentation like **secure rooms inside a building**.

```
Building → VPC
Floors → Subnets
Rooms → Instances
Doors → Security groups
```

Only specific doors allow movement between rooms.

---

### 19. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Least Privilege | Allow only required communication |
| Segmentation | Dividing network into isolated zones |
| Microsegmentation | Fine-grained workload-level control |
| Security Groups | Instance-level firewall |
| Network ACLs | Subnet-level firewall |

---

### 20. Checkpoint Questions

You should now be able to answer these.

#### What is least privilege networking?

Allowing only the minimum required network access.

---

#### What AWS feature is commonly used for microsegmentation?

```
Security Groups
```

---

#### What layer do NACLs operate at?

```
Subnet level
```

---

#### Why is segmentation important?

```
To prevent lateral movement and limit blast radius.
```

---

#### What architecture model aligns with microsegmentation?

```
Zero Trust Networking
```
