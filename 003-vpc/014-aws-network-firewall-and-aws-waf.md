## AWS Network Firewall and AWS WAF — Perimeter Protection for VPC Networks

When building secure cloud architectures, organizations must protect their infrastructure from **external attacks, malicious traffic, and unauthorized access**.

AWS provides multiple security layers to protect applications and networks:

```
Security Groups
Network ACLs
AWS Network Firewall
AWS WAF (Web Application Firewall)
```

Security Groups and NACLs control **basic network access**, but large production systems require **advanced inspection and filtering of traffic**.

This is where **AWS Network Firewall** and **AWS WAF** are used.

These services help protect your VPC at the **network perimeter and application layer**.

---

### 1. What Is Perimeter Security?

Perimeter security means protecting the **boundary of your network** from external threats.

Think of it as a **security checkpoint at the edge of your infrastructure**.

Example architecture:

```
Internet
   |
Perimeter Firewall
   |
Internal Network
```

All incoming traffic must pass through security inspection before entering the internal network.

In AWS, this perimeter protection can be implemented using:

```
AWS Network Firewall
AWS WAF
```

---

### 2. Layers of Cloud Security

AWS security typically operates in multiple layers.

Example layered security:

```
Internet
   |
WAF
   |
Network Firewall
   |
VPC
   |
Security Groups
   |
Application
```

Each layer protects against different types of threats.

---

### 3. What Is AWS Network Firewall?

AWS Network Firewall is a **managed network firewall service** for VPCs.

Definition:

> AWS Network Firewall is a stateful, managed firewall that inspects and filters network traffic entering or leaving a VPC.

It allows organizations to implement **advanced network security policies**.

Example capabilities:

```
deep packet inspection
stateful traffic filtering
domain blocking
IP blocking
protocol filtering
```

---

### 4. Where AWS Network Firewall Is Deployed

AWS Network Firewall is typically placed in a **dedicated firewall subnet**.

Example architecture:

```
Internet
   |
Internet Gateway
   |
Firewall Subnet
   |
AWS Network Firewall
   |
Application Subnets
```

All traffic flows through the firewall before reaching internal systems.

---

### 5. Network Firewall Traffic Flow

Example flow:

```
Internet
   |
IGW
   |
Firewall Endpoint
   |
Inspection Rules
   |
Application Subnet
   |
EC2 / Services
```

The firewall evaluates traffic against **security policies** before forwarding packets.

---

### 6. Network Firewall Rule Types

Network Firewall supports two main rule categories.

#### Stateless Rules

Stateless rules inspect **individual packets**.

Example:

```
Allow HTTP
Deny traffic from IP range
```

---

#### Stateful Rules

Stateful rules inspect **connection flows**.

Example:

```
Allow outbound HTTPS sessions
Block malicious traffic patterns
```

Stateful inspection allows deeper analysis of traffic behavior.

---

### 7. Example Network Firewall Rules

Example policies:

```
Block known malicious IP ranges
Allow outbound HTTPS only
Block cryptocurrency mining domains
Block TOR network access
```

Example rule:

```
deny tcp any any -> 1.2.3.4/32
```

This blocks traffic to a specific IP.

---

### 8. What Is AWS WAF?

AWS WAF stands for **Web Application Firewall**.

Definition:

> AWS WAF protects web applications by filtering and monitoring HTTP and HTTPS requests.

Unlike Network Firewall, WAF focuses specifically on **application-layer traffic (Layer 7)**.

It protects against attacks targeting web applications.

Example threats:

```
SQL injection
Cross-site scripting (XSS)
HTTP floods
malicious bots
```

---

### 9. Where AWS WAF Is Used

AWS WAF is typically deployed in front of services such as:

```
Application Load Balancer (ALB)
Amazon CloudFront
Amazon API Gateway
AWS AppSync
```

Example architecture:

```
Internet
   |
CloudFront / ALB
   |
AWS WAF
   |
Application Servers
```

WAF inspects HTTP requests before they reach the application.

---

### 10. WAF Rule Types

AWS WAF supports multiple rule types.

Examples:

```
IP blocking rules
rate limiting rules
SQL injection detection
XSS detection
geolocation blocking
```

Example rule:

```
Block traffic from specific country
```

Example:

```
Allow requests under 1000 per minute
```

---

### 11. Example WAF Protection

Suppose attackers attempt SQL injection:

```
GET /login?id=1 OR 1=1
```

WAF detects malicious pattern and blocks the request.

Traffic flow:

```
Attacker → WAF → Blocked
```

The application never sees the request.

---

### 12. AWS Network Firewall vs AWS WAF

| Feature | AWS Network Firewall | AWS WAF |
|------|------|------|
| Layer | Network Layer (L3/L4) | Application Layer (L7) |
| Traffic inspected | All network traffic | HTTP/HTTPS |
| Deployment | VPC perimeter | ALB / CloudFront |
| Threats handled | IP, protocol filtering | Web attacks |

Both services complement each other.

---

### 13. Example Combined Architecture

Production architecture may combine both services.

```
Internet
   |
CloudFront
   |
AWS WAF
   |
Internet Gateway
   |
AWS Network Firewall
   |
VPC
   |
Application Servers
```

Security layers protect against multiple attack vectors.

---

### 14. Enterprise Security Architecture

Large enterprises often implement **centralized inspection VPCs**.

Example:

```
Internet
   |
Transit Gateway
   |
Inspection VPC
   |
AWS Network Firewall
   |
Application VPC
```

This architecture centralizes security enforcement.

---

### 15. Example Attack Protection

Suppose attackers attempt:

```
DDoS
SQL injection
Malicious bot traffic
```

Protection layers:

```
AWS Shield → DDoS protection
AWS WAF → blocks application attacks
AWS Network Firewall → blocks malicious network traffic
Security Groups → restrict instance access
```

Multiple layers improve resilience.

---

### 16. When to Use AWS Network Firewall

Use Network Firewall when you need:

```
centralized VPC traffic inspection
advanced network filtering
compliance security policies
deep packet inspection
large-scale enterprise security
```

---

### 17. When to Use AWS WAF

Use WAF when protecting:

```
web applications
public APIs
CloudFront distributions
load-balanced applications
```

It protects applications from **Layer 7 attacks**.

---

### 18. Example Production Deployment

Example architecture:

```
Internet
   |
CloudFront
   |
AWS WAF
   |
ALB
   |
Private Subnets
   |
Application Servers
   |
Database
```

Network Firewall may also inspect traffic between VPCs.

---

### 19. Mental Model

Think of these components like a **secured airport system**.

```
Airport Entrance → Network Firewall
Security Checkpoint → WAF
Boarding Gate → Security Groups
```

Each layer filters threats before they reach critical systems.

---

### 20. Key Concepts Summary

| Concept | Meaning |
|------|------|
| Perimeter Security | Protecting network boundary |
| AWS Network Firewall | Network traffic inspection |
| AWS WAF | Web application protection |
| Stateless Rules | Packet-based filtering |
| Stateful Rules | Connection-based filtering |

---

### 21. Checkpoint Questions

You should now be able to answer these.

#### What does AWS Network Firewall do?

It inspects and filters network traffic entering or leaving a VPC.

---

#### What does AWS WAF protect?

Web applications from HTTP-based attacks.

---

#### Which layer does Network Firewall operate on?

```
Network Layer (L3/L4)
```

---

#### Which layer does WAF operate on?

```
Application Layer (L7)
```

---

#### Can both be used together?

```
Yes
Large architectures commonly use both.
```
