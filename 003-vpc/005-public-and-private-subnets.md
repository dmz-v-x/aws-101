## Subnets in AWS — Public vs Private vs Isolated and How They Map to Availability Zones

In AWS networking, **subnets** are one of the most important building blocks of a VPC. Every compute resource (like EC2, load balancers, databases, Kubernetes nodes) runs inside a **subnet**.

Understanding subnets is critical because they control:

- **network segmentation**
- **security boundaries**
- **internet accessibility**
- **high availability architecture**

---

### 1. What Is a Subnet?

A **subnet** is a smaller network created inside a VPC.

Definition:

> A subnet is a range of IP addresses within a VPC.

When you create a VPC, you define a **CIDR block**.

Example:

```
VPC CIDR
10.0.0.0/16
```

This range contains **65,536 IP addresses**.

Instead of using the entire network for everything, we divide it into **smaller networks called subnets**.

Example:

```
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24
```

Each subnet can host different components.

Example architecture:

```
10.0.1.0/24 → Web servers
10.0.2.0/24 → Application servers
10.0.3.0/24 → Databases
```

---

### 2. Why Do Subnets Exist?

Subnets exist to create **network segmentation**.

Without subnets everything would live in one large network.

Example (bad design):

```
VPC 10.0.0.0/16

Web Servers
App Servers
Databases
Load Balancers
Cache
Workers
```

Problems:

- poor security
- difficult routing
- harder network control

Subnets allow us to **separate infrastructure layers**.

Example (better design):

```
Public Subnet → Load balancer
Private Subnet → Application servers
Isolated Subnet → Databases
```

This creates clear **network boundaries**.

---

### 3. Subnets Always Exist Inside a VPC

Subnets cannot exist alone.

They must belong to a **VPC**.

Example structure:

```
AWS Account
   |
   VPC (10.0.0.0/16)
   |
   ├── Subnet A (10.0.1.0/24)
   ├── Subnet B (10.0.2.0/24)
   └── Subnet C (10.0.3.0/24)
```

All instances launched must be placed into **one subnet**.

---

### 4. Subnets Are Bound to Availability Zones

One of the most important rules in AWS networking:

> **A subnet exists in only one Availability Zone.**

First we must understand **Availability Zones (AZs)**.

---

### 5. What Is an Availability Zone?

An **Availability Zone (AZ)** is an isolated data center inside an AWS region.

Example region:

```
ap-south-1 (Mumbai)
```

Possible Availability Zones:

```
ap-south-1a
ap-south-1b
ap-south-1c
```

Each AZ contains:

- data centers
- power systems
- networking
- cooling

They are physically separate for **fault tolerance**.

---

### 6. Why Subnets Are AZ-Specific

Because each AZ is a separate physical location.

Therefore a subnet cannot span multiple AZs.

Example:

```
Subnet A → ap-south-1a
Subnet B → ap-south-1b
Subnet C → ap-south-1c
```

This allows high availability.

Example architecture:

```
VPC

AZ-a
 └── Subnet A

AZ-b
 └── Subnet B
```

If one AZ fails, the other continues running.

---

### 7. Public vs Private vs Isolated Subnets

Subnets are typically categorized into three types.

These categories are based on **routing behavior**.

Types:

```
Public Subnet
Private Subnet
Isolated Subnet
```

---

### 8. Public Subnet

A **public subnet** is a subnet that has **direct internet access**.

Definition:

> A subnet is public when its route table has a route to an Internet Gateway.

Example route table:

```
Destination    Target
10.0.0.0/16    local
0.0.0.0/0      IGW
```

This means:

```
all internet traffic → Internet Gateway
```

Resources inside public subnets can:

- access the internet
- receive traffic from the internet

Examples:

```
Load balancers
Bastion hosts
Public web servers
```

Example diagram:

```
Internet
   |
Internet Gateway
   |
Public Subnet
   |
Web Server
```

---

### 9. Private Subnet

A **private subnet** does not allow direct internet access.

Route table example:

```
Destination    Target
10.0.0.0/16    local
0.0.0.0/0      NAT Gateway
```

Traffic flow:

```
Instance → NAT Gateway → Internet
```

Key rule:

```
Outbound internet allowed
Inbound internet blocked
```

Private subnets are commonly used for:

```
Application servers
Internal APIs
Worker nodes
Microservices
```

Example diagram:

```
Private Subnet
   |
Application Server
   |
NAT Gateway
   |
Internet Gateway
   |
Internet
```

---

### 10. Isolated Subnet

An **isolated subnet** has **no internet access at all**.

Route table example:

```
Destination    Target
10.0.0.0/16    local
```

There is no route to:

```
IGW
NAT Gateway
```

These subnets are completely internal.

Used for highly secure systems.

Examples:

```
Databases
Internal data stores
Security services
```

Example architecture:

```
Isolated Subnet
   |
Database Server
```

Only accessible from inside the VPC.

---

### 11. Comparing Subnet Types

| Subnet Type | Internet Access | Use Cases |
|------|------|------|
| Public | Direct | Load balancers, bastion hosts |
| Private | Outbound only | Application servers |
| Isolated | None | Databases |

---

### 12. Typical Production Subnet Layout

A common AWS architecture uses **multiple AZs and multiple subnet types**.

Example:

```
VPC (10.0.0.0/16)

AZ-a
 ├── Public Subnet
 ├── Private Subnet
 └── Isolated Subnet

AZ-b
 ├── Public Subnet
 ├── Private Subnet
 └── Isolated Subnet
```

This improves:

- availability
- fault tolerance
- scaling

---

### 13. Multi-AZ Architecture Example

```
                 Internet
                    |
              Internet Gateway
                    |
       -------------------------------
       |                             |
Public Subnet AZ-a           Public Subnet AZ-b
 Load Balancer                Load Balancer
       |                             |
Private Subnet AZ-a          Private Subnet AZ-b
 Application Servers         Application Servers
       |                             |
Isolated Subnet AZ-a         Isolated Subnet AZ-b
 Database                    Database
```

This is a **common production architecture**.

---

### 14. Example CIDR Design

Suppose we have:

```
VPC → 10.0.0.0/16
```

We can design subnets like this:

```
Public AZ-a     → 10.0.1.0/24
Public AZ-b     → 10.0.2.0/24

Private AZ-a    → 10.0.10.0/24
Private AZ-b    → 10.0.11.0/24

Isolated AZ-a   → 10.0.20.0/24
Isolated AZ-b   → 10.0.21.0/24
```

Each subnet contains:

```
256 IP addresses
```

---

### 15. Real Production Example

Typical infrastructure:

```
Public Subnets
 ├── Application Load Balancer
 └── NAT Gateways

Private Subnets
 ├── EC2 Instances
 ├── ECS Tasks
 └── Kubernetes Nodes

Isolated Subnets
 ├── RDS Databases
 ├── Redis
 └── Internal Services
```

---

### 16. Mental Model

Think of subnets like **rooms inside a building**.

```
VPC → Building
Subnets → Rooms
Instances → People inside rooms
```

Each room has:

- its own rules
- its own access control
- its own purpose

---

### 17. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Subnet | Smaller network inside VPC |
| AZ Mapping | Each subnet belongs to one AZ |
| Public Subnet | Internet accessible |
| Private Subnet | Outbound internet via NAT |
| Isolated Subnet | No internet connectivity |

---

### 18. Checkpoint Questions

You should be able to answer these.

#### What is a subnet?

A smaller network within a VPC used to organize resources.

---

#### Can a subnet span multiple Availability Zones?

No.  
A subnet exists in **one AZ only**.

---

#### What makes a subnet public?

A route to an **Internet Gateway**.

---

#### What makes a subnet private?

A route to a **NAT Gateway** instead of the Internet Gateway.

---

#### What is an isolated subnet?

A subnet with **no internet routes**.

