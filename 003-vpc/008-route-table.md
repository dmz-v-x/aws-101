## Route Tables and the Implicit VPC Router — Main vs Custom Route Tables and Subnet Associations

In AWS networking, **route tables** control how network traffic moves inside a VPC and how it leaves the VPC.

Whenever a packet travels from one resource to another, AWS must determine **where that packet should go next**. This decision is made using **route tables**.

However, route tables do not operate alone. Every VPC also contains an **implicit router** that automatically routes traffic between subnets.

Understanding this relationship is critical for designing correct network architectures.

---

### 1. What Is Routing?

Routing means:

> Deciding where network traffic should go next.

When a packet leaves a machine, the network must determine its destination.

Example:

```
EC2 → Database
EC2 → Another subnet
EC2 → Internet
```

Each case requires a **different route decision**.

These decisions are stored inside **route tables**.

---

### 2. What Is a Route Table?

A **route table** is a list of rules that determine how traffic is directed within a network.

Each rule is called a **route**.

Example route table:

| Destination | Target |
|------|------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | igw |

Explanation:

```
10.0.0.0/16 → traffic stays inside the VPC
0.0.0.0/0 → traffic goes to Internet Gateway
```

Route tables exist **inside a VPC**.

---

### 3. The Hidden Component — VPC Implicit Router

Inside every VPC, AWS automatically provides a **virtual router**.

This router is **not visible or configurable directly**.

It is called the **implicit router**.

Its job is to:

```
forward packets according to route tables
```

Example packet flow:

```
EC2 → Subnet
Subnet → VPC Router
Router → Route Table
Route Table → Destination
```

So traffic always goes through the **VPC router**.

---

### 4. The Default Local Route

Every route table automatically contains a **local route**.

Example:

```
10.0.0.0/16 → local
```

This means:

```
all traffic inside the VPC CIDR stays inside the VPC
```

Example communication:

```
EC2 (10.0.1.10) → Database (10.0.2.15)
```

Even if they are in different subnets, AWS routes traffic internally.

No additional configuration is required.

---

### 5. Communication Between Subnets

Example architecture:

```
VPC (10.0.0.0/16)

Subnet A (10.0.1.0/24)
Subnet B (10.0.2.0/24)
```

Example traffic:

```
10.0.1.10 → 10.0.2.20
```

Flow:

```
EC2 → VPC Router → Destination Subnet
```

The route used:

```
10.0.0.0/16 → local
```

This is why **all subnets inside a VPC can communicate by default**.

---

### 6. Main Route Table

Every VPC has a **main route table**.

Definition:

> The main route table is the default route table used by subnets that do not have an explicitly associated route table.

When you create a VPC:

```
AWS automatically creates a main route table
```

Example:

```
Main Route Table
10.0.0.0/16 → local
```

Initially it contains **only the local route**.

---

### 7. What Happens When You Create a Subnet

When a subnet is created:

```
it automatically associates with the main route table
```

Example:

```
Subnet A → Main Route Table
Subnet B → Main Route Table
```

So both subnets use the same routing rules.

---

### 8. Why Custom Route Tables Exist

In real architectures, different subnets require **different routing rules**.

Example:

```
Public Subnet → Internet access
Private Subnet → NAT Gateway
Isolated Subnet → No internet
```

This cannot be achieved with one route table.

So we create **custom route tables**.

---

### 9. Custom Route Tables

A **custom route table** is any route table created by the user.

Example:

```
Public Route Table
Private Route Table
Isolated Route Table
```

Each route table can contain different routing rules.

---

### 10. Example Architecture

```
VPC (10.0.0.0/16)

Public Subnet
Private Subnet
Database Subnet
```

We create route tables like this:

```
Public Route Table
Private Route Table
Isolated Route Table
```

Each subnet uses its own route table.

---

### 11. Public Route Table Example

Used for internet-facing resources.

Example routes:

| Destination | Target |
|------|------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | Internet Gateway |

Subnet association:

```
Public Subnet → Public Route Table
```

This allows internet access.

---

### 12. Private Route Table Example

Used for application servers.

Example routes:

| Destination | Target |
|------|------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | NAT Gateway |

Subnet association:

```
Private Subnet → Private Route Table
```

This allows outbound internet only.

---

### 13. Isolated Route Table Example

Used for highly secure resources.

Example routes:

| Destination | Target |
|------|------|
| 10.0.0.0/16 | local |

No internet routes exist.

Subnet association:

```
Database Subnet → Isolated Route Table
```

---

### 14. Subnet Associations

A route table works only if it is **associated with a subnet**.

Association means:

```
This subnet should use this route table.
```

Example:

```
Public Subnet → Public Route Table
Private Subnet → Private Route Table
Database Subnet → Isolated Route Table
```

Each subnet can use **only one route table at a time**.

---

### 15. Full Routing Example

Example architecture:

```
VPC (10.0.0.0/16)

Internet Gateway
NAT Gateway

Subnets:
10.0.1.0/24 → Public
10.0.2.0/24 → Private
10.0.3.0/24 → Database
```

Route tables:

```
Public RT
0.0.0.0/0 → IGW

Private RT
0.0.0.0/0 → NAT

Isolated RT
only local
```

Associations:

```
Public Subnet → Public RT
Private Subnet → Private RT
DB Subnet → Isolated RT
```

---

### 16. Packet Flow Example

User connects to web server.

```
Internet → IGW → Public Subnet → Web Server
```

Web server talks to application server.

```
Web → VPC Router → Private Subnet → App Server
```

App server downloads updates.

```
App → NAT → Internet
```

Each step uses **route table rules**.

---

### 17. Mental Model

Think of routing like **road signs in a city**.

```
VPC → City
Subnets → Neighborhoods
Route Tables → Road signs
VPC Router → Traffic controller
```

Traffic moves according to the rules defined in route tables.

---

### 18. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Route Table | List of routing rules |
| Route | Destination + target |
| Main Route Table | Default route table for VPC |
| Custom Route Table | User-created route table |
| Subnet Association | Assign route table to subnet |
| VPC Router | AWS-managed router forwarding traffic |

---

### 19. Checkpoint Questions

You should be able to answer these.

#### What is a route table?

A set of rules that determine where network traffic should go.

---

#### What is the main route table?

The default route table automatically created with a VPC.

---

#### What is a custom route table?

A route table created by the user with custom routing rules.

---

#### What is a subnet association?

Linking a subnet to a specific route table.

---

#### Can a subnet use multiple route tables?

No.

```
A subnet can use only one route table at a time.
```
