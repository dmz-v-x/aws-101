## Security Groups in AWS — Stateful Firewall Basics, Default Rules, Inbound and Outbound Traffic

When running servers in the cloud, one of the most important requirements is **controlling who can access those servers**.

Without security controls, any system on the internet could potentially connect to your infrastructure.

AWS provides a firewall mechanism called **Security Groups** to protect resources inside a VPC.

Security groups act as **virtual firewalls** that control network traffic to and from AWS resources such as:

- EC2 instances
- Load balancers
- RDS databases
- ENIs
- ECS tasks
- EKS nodes

Understanding security groups is essential for designing **secure cloud architectures**.

---

### 1. What Is a Security Group?

A **Security Group** is a virtual firewall that controls network traffic to and from AWS resources.

Definition:

> A security group is a stateful firewall that allows or denies traffic based on defined rules.

Security groups operate at the **instance level**.

They are attached to:

```
Elastic Network Interfaces (ENIs)
```

Since every EC2 instance has an ENI, security groups effectively protect the instance.

---

### 2. Where Security Groups Are Applied

Security groups are attached to **network interfaces**, not directly to instances.

Example architecture:

```
EC2 Instance
     |
     ENI
     |
Security Group
```

Traffic reaching the ENI must pass through the security group rules.

---

### 3. What Traffic Security Groups Control

Security groups evaluate two types of traffic:

```
Inbound traffic
Outbound traffic
```

Inbound:

```
traffic coming INTO the instance
```

Outbound:

```
traffic leaving FROM the instance
```

Both directions can have rules.

---

### 4. Inbound Traffic

Inbound rules control **who can initiate connections to your resource**.

Example inbound rule:

| Type | Protocol | Port | Source |
|------|------|------|------|
| HTTP | TCP | 80 | 0.0.0.0/0 |

Meaning:

```
allow HTTP traffic from anywhere
```

Example use cases:

```
Allow SSH from your IP
Allow HTTPS from internet
Allow internal app communication
```

---

### 5. Outbound Traffic

Outbound rules control **where your instance can send traffic**.

Example outbound rule:

| Type | Protocol | Port | Destination |
|------|------|------|------|
| All traffic | All | All | 0.0.0.0/0 |

Meaning:

```
instance can connect to anywhere
```

Example use cases:

```
Allow database access
Allow API calls
Allow internet access
```

---

### 6. What Does Stateful Mean?

Security groups are **stateful firewalls**.

Definition:

> A stateful firewall automatically allows response traffic for permitted requests.

Example:

Inbound rule:

```
Allow HTTP (port 80)
```

User sends request:

```
User → Web Server
```

Security group allows the request.

Response traffic:

```
Web Server → User
```

Even if outbound rules don't explicitly allow it, the response is automatically allowed.

This behavior simplifies firewall management.

---

### 7. Stateful Example

Example rule:

```
Inbound
Allow TCP port 22 from your IP
```

Connection flow:

```
Laptop → EC2 (SSH)
```

Once allowed:

```
EC2 → Laptop (response)
```

Return traffic is automatically permitted.

You do not need to define reverse rules.

---

### 8. Stateless vs Stateful Comparison

| Feature | Stateful Firewall | Stateless Firewall |
|------|------|------|
| Tracks connections | Yes | No |
| Return traffic allowed automatically | Yes | No |
| Rule complexity | Lower | Higher |

Security groups are **stateful**.

Network ACLs (another AWS feature) are **stateless**.

---

### 9. Default Security Group

Every VPC automatically includes a **default security group**.

Characteristics:

Inbound rules:

```
Allow traffic from same security group
```

Outbound rules:

```
Allow all outbound traffic
```

Example inbound rule:

| Source | Action |
|------|------|
| Same SG | Allow |

This allows instances within the same security group to communicate.

---

### 10. Example Default Security Group Behavior

Example setup:

```
EC2 Instance A → Default SG
EC2 Instance B → Default SG
```

Traffic allowed:

```
A → B
B → A
```

But internet traffic is blocked unless rules are added.

---

### 11. Allow Rules Only

Security groups support **only allow rules**.

There are **no deny rules**.

Example:

```
Allow SSH
Allow HTTP
```

Everything else is implicitly blocked.

This is called **implicit deny**.

---

### 12. Example Security Group for Web Server

Typical web server rules:

Inbound:

| Type | Port | Source |
|------|------|------|
| SSH | 22 | Your IP |
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |

Outbound:

```
Allow all traffic
```

Architecture:

```
Internet
   |
Security Group
   |
Web Server
```

---

### 13. Application Tier Security Group

Application servers should not be publicly accessible.

Inbound rules:

| Type | Port | Source |
|------|------|------|
| HTTP | 8080 | Web SG |

Meaning:

```
Only web servers can connect
```

Architecture:

```
Web Servers → App Servers
```

Internet cannot reach app servers directly.

---

### 14. Database Security Group

Database servers should be highly restricted.

Inbound rules:

| Type | Port | Source |
|------|------|------|
| MySQL | 3306 | App SG |

Meaning:

```
Only application servers can access database
```

Architecture:

```
App Servers → Database
```

---

### 15. Multi-Tier Security Architecture

Example architecture:

```
Internet
   |
Web Security Group
   |
Web Servers
   |
App Security Group
   |
Application Servers
   |
DB Security Group
   |
Database
```

Each layer has restricted access.

---

### 16. Security Group Referencing

Security groups can reference **other security groups**.

Example rule:

```
Source → Web SG
```

Meaning:

```
Allow traffic from instances in Web SG
```

This is useful for microservice architectures.

---

### 17. Example Traffic Flow

User visits website.

```
User → Web Server (allowed)
```

Web server calls application server.

```
Web SG → App SG (allowed)
```

Application server queries database.

```
App SG → DB SG (allowed)
```

Traffic flows securely through layers.

---

### 18. Security Group Limits

Typical limits include:

```
60 inbound rules
60 outbound rules
```

Multiple security groups can be attached to an ENI.

Rules are combined.

---

### 19. Mental Model

Think of security groups like **security guards at building entrances**.

```
VPC → Building
Subnet → Floor
Instance → Room
Security Group → Guard controlling door access
```

The guard checks rules before allowing entry.

---

### 20. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Security Group | Virtual firewall |
| Inbound Rules | Control incoming traffic |
| Outbound Rules | Control outgoing traffic |
| Stateful Firewall | Automatically allows response traffic |
| Default SG | Allow internal communication |
| Implicit Deny | All unspecified traffic blocked |

---

### 21. Checkpoint Questions

You should now be able to answer these.

#### What is a security group?

A virtual firewall controlling traffic to AWS resources.

---

#### Where are security groups attached?

```
ENI (network interface)
```

---

#### What does stateful mean?

Return traffic for allowed connections is automatically permitted.

---

#### Do security groups support deny rules?

```
No
Only allow rules exist.
```

