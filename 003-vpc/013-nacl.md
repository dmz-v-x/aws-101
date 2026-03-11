## Network ACLs (NACLs) in AWS — Stateless Firewall, Rule Order, Allow/Deny Model, and When to Use NACLs vs Security Groups

In AWS VPC networking, there are **two primary firewall mechanisms**:

```
Security Groups (SG)
Network Access Control Lists (NACL)
```

Although both control network traffic, they operate at **different layers of the network** and behave differently.

Security Groups protect **individual instances**, while NACLs protect **entire subnets**.

Understanding NACLs is important for designing **defense-in-depth network security architectures**.

---

### 1. What Is a Network ACL (NACL)?

A **Network Access Control List (NACL)** is a firewall that controls traffic **at the subnet level**.

Definition:

> A Network ACL is a stateless firewall that controls inbound and outbound traffic for an entire subnet.

Unlike security groups, which protect individual resources, NACLs protect **all resources inside a subnet**.

Example architecture:

```
VPC
 |
Subnet
 |
Network ACL
 |
Instances
```

Every packet entering or leaving the subnet must pass through the NACL.

---

### 2. Where NACLs Are Applied

NACLs are attached to **subnets**.

Example:

```
Public Subnet → NACL A
Private Subnet → NACL B
Database Subnet → NACL C
```

All instances inside the subnet are affected by the same NACL.

Example:

```
Subnet
 ├ EC2 instance
 ├ Load balancer
 └ NAT Gateway
```

All must follow the same NACL rules.

---

### 3. Stateless Firewall Behavior

NACLs are **stateless firewalls**.

Definition:

> Stateless means the firewall does not track connection state.

Every packet must be explicitly allowed.

Example:

```
Client → Server (request)
Server → Client (response)
```

For NACLs you must allow **both directions**.

Example rules required:

```
Inbound: allow port 80
Outbound: allow ephemeral ports
```

Without both rules, communication fails.

---

### 4. Security Groups vs NACLs (Statefulness)

| Feature | Security Group | NACL |
|------|------|------|
| Firewall type | Stateful | Stateless |
| Rule tracking | Tracks connections | Does not track |
| Response traffic | Automatically allowed | Must be explicitly allowed |

This is the **biggest conceptual difference**.

---

### 5. NACL Rule Structure

Each NACL rule contains:

```
Rule number
Protocol
Port range
Source/Destination
Allow or Deny
```

Example rule:

| Rule # | Protocol | Port | Source | Action |
|------|------|------|------|------|
| 100 | TCP | 80 | 0.0.0.0/0 | ALLOW |

---

### 6. Rule Numbering and Evaluation Order

NACL rules are evaluated **in ascending order**.

Example rules:

| Rule # | Action |
|------|------|
| 100 | Allow HTTP |
| 200 | Deny IP range |
| 300 | Allow SSH |

Evaluation flow:

```
Rule 100 → Rule 200 → Rule 300
```

As soon as a rule matches, processing stops.

Lower numbers = higher priority.

---

### 7. The Default NACL

Every VPC automatically includes a **default NACL**.

Default NACL behavior:

Inbound rules:

```
Allow all traffic
```

Outbound rules:

```
Allow all traffic
```

This means the default NACL **does not restrict traffic**.

Security groups provide the primary protection by default.

---

### 8. Custom NACL

When you create a **custom NACL**, it behaves differently.

Default behavior:

```
All traffic denied
```

You must explicitly create rules to allow traffic.

Example:

```
Allow HTTP
Allow HTTPS
Allow ephemeral ports
```

---

### 9. Example Inbound Rules

Example inbound NACL rules:

| Rule # | Protocol | Port | Source | Action |
|------|------|------|------|------|
| 100 | TCP | 80 | 0.0.0.0/0 | ALLOW |
| 110 | TCP | 443 | 0.0.0.0/0 | ALLOW |
| 120 | TCP | 22 | Your IP | ALLOW |
| * | ALL | ALL | ALL | DENY |

This allows web traffic but blocks everything else.

---

### 10. Example Outbound Rules

Outbound rules allow responses and outbound communication.

Example:

| Rule # | Protocol | Port | Destination | Action |
|------|------|------|------|------|
| 100 | TCP | 1024-65535 | 0.0.0.0/0 | ALLOW |
| * | ALL | ALL | ALL | DENY |

The range:

```
1024–65535
```

represents **ephemeral ports** used for response traffic.

---

### 11. Ephemeral Ports Explained

When a client connects to a server:

```
Client → Server:80
```

The response does not return to port 80.

Instead it uses a temporary port.

Example:

```
Server → Client:49152
```

These temporary ports are called **ephemeral ports**.

Typical range:

```
1024–65535
```

NACL outbound rules must allow these.

---

### 12. Example Web Subnet NACL

Example architecture:

```
Internet
   |
Internet Gateway
   |
Public Subnet
   |
Web Server
```

Inbound rules:

```
Allow 80
Allow 443
Allow SSH from admin IP
```

Outbound rules:

```
Allow ephemeral ports
Allow internet traffic
```

---

### 13. Example Database Subnet NACL

Database subnet should be heavily restricted.

Inbound rules:

```
Allow MySQL from App Subnet
```

Example:

| Rule # | Protocol | Port | Source | Action |
|------|------|------|------|------|
| 100 | TCP | 3306 | App subnet | ALLOW |

Outbound rules:

```
Allow ephemeral ports
```

---

### 14. Traffic Flow Through VPC Security Layers

Packet flow example:

```
Internet
   |
Internet Gateway
   |
NACL
   |
Subnet
   |
Security Group
   |
EC2 Instance
```

Both layers must allow traffic.

---

### 15. Layered Security Model

AWS recommends **defense in depth**.

Example layers:

```
NACL → Subnet protection
Security Group → Instance protection
Application firewall → App protection
```

Multiple layers improve security.

---

### 16. When to Use Security Groups

Security groups are used for **most access control needs**.

Best for:

```
instance-level security
microservice communication
application-tier isolation
```

Example:

```
Web SG → App SG → DB SG
```

---

### 17. When to Use NACLs

NACLs are useful for **network-level restrictions**.

Examples:

```
blocking malicious IP ranges
compliance requirements
extra security layer
restricting entire subnet access
```

Example:

```
Deny known attacker IPs
```

---

### 18. Security Groups vs NACLs Summary

| Feature | Security Groups | NACL |
|------|------|------|
| Scope | Instance | Subnet |
| Stateful | Yes | No |
| Allow rules | Yes | Yes |
| Deny rules | No | Yes |
| Rule order | Not evaluated | Evaluated by number |

---

### 19. Real Architecture Example

Example multi-tier system:

```
Internet
   |
IGW
   |
NACL
   |
Public Subnet
   |
Web SG
   |
Web Servers
   |
App SG
   |
Private Subnet
   |
Database SG
   |
DB Subnet
```

Each layer enforces security.

---

### 20. Mental Model

Think of network protection like **building security**.

```
City Gate → NACL
Building Door → Security Group
Room Lock → Application authentication
```

Each layer adds protection.

---

### 21. Key Concepts Summary

| Concept | Meaning |
|------|------|
| NACL | Subnet-level firewall |
| Stateless | No connection tracking |
| Rule Order | Evaluated by rule number |
| Allow/Deny | Both supported |
| Ephemeral Ports | Temporary response ports |

---

### 22. Checkpoint Questions

You should now be able to answer these.

#### What is a NACL?

A subnet-level firewall controlling inbound and outbound traffic.

---

#### Are NACLs stateful or stateless?

```
Stateless
```

---

#### Do NACLs support deny rules?

```
Yes
```

---

#### What determines rule priority?

```
Rule number
```

---

#### What is the main difference between SG and NACL?

```
SG → instance-level stateful firewall
NACL → subnet-level stateless firewall
```

