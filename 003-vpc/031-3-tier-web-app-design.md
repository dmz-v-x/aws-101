## Architecture Design — Simple 3-Tier Web Application VPC (Public Web, Private App, Database Subnets)

A **3-tier architecture** is one of the most common network designs used in production systems.  
It separates application components into three layers:

```
Web Layer (presentation)
Application Layer (business logic)
Database Layer (data storage)
```

This separation improves:

```
security
scalability
fault isolation
network segmentation
```

In AWS, a typical 3-tier architecture uses **VPC networking components** such as:

```
VPC
subnets
internet gateway
NAT gateway
Application Load Balancer (ALB)
security groups
```

---

### 1. High-Level Architecture

Typical architecture:

```
                    Internet
                       |
                Internet Gateway
                       |
                Application Load Balancer
                   /           \
              Web Tier       Web Tier
               (AZ-a)        (AZ-b)
                   |             |
               App Tier       App Tier
               (Private)      (Private)
                   |             |
                 Database Tier (Multi-AZ)
```

Key principles:

```
web layer exposed to internet
application layer private
database layer isolated
```

---

### 2. VPC Configuration

Example VPC CIDR:

```
10.0.0.0/16
```

Subnet design:

```
Public Subnets
Private Application Subnets
Private Database Subnets
```

Example layout:

```
10.0.0.0/16 VPC

AZ-a
  10.0.1.0/24 → Public Subnet
  10.0.10.0/24 → App Subnet
  10.0.20.0/24 → DB Subnet

AZ-b
  10.0.2.0/24 → Public Subnet
  10.0.11.0/24 → App Subnet
  10.0.21.0/24 → DB Subnet
```

This provides **high availability across AZs**.

---

### 3. Public Subnets — Web Layer

Public subnets host components that must be reachable from the internet.

Example resources:

```
Application Load Balancer
NAT Gateways
Bastion Hosts (optional)
```

Routing:

```
0.0.0.0/0 → Internet Gateway
```

Architecture:

```
Internet
   |
Internet Gateway
   |
Public Subnet
   |
Application Load Balancer
```

The ALB distributes traffic to backend instances.

---

### 4. Web Tier

The **web tier** handles incoming HTTP requests.

Example services:

```
Nginx
Apache
Node.js frontend
React / Next.js frontend
```

Architecture:

```
Internet
   |
Application Load Balancer
   |
--------------------------
|                        |
Web Server A         Web Server B
```

Instances typically run in an **Auto Scaling Group**.

---

### 5. Private Application Subnets

Application servers run in **private subnets**.

They are not accessible directly from the internet.

Example services:

```
API servers
microservices
backend workers
business logic services
```

Architecture:

```
Web Tier
   |
Application Servers
   |
Private Subnet
```

Security rules:

```
Allow traffic only from ALB security group
```

---

### 6. NAT Gateway for Private Internet Access

Private instances sometimes need outbound internet access.

Examples:

```
install packages
download container images
call external APIs
```

This is done using **NAT Gateway**.

Architecture:

```
Private Subnet
   |
NAT Gateway
   |
Internet Gateway
   |
Internet
```

Private instances cannot receive inbound traffic.

---

### 7. Database Tier

The database tier stores application data.

Examples:

```
Amazon RDS
Aurora
PostgreSQL
MySQL
```

Database subnets are **highly restricted**.

Architecture:

```
Application Servers
   |
Database
```

Security rules:

```
Allow DB port only from App security group
```

Example:

```
Allow TCP 5432 from App-SG
```

---

### 8. Multi-AZ Database Deployment

High availability databases use **Multi-AZ replication**.

Architecture:

```
Primary DB (AZ-a)
       |
Synchronous Replication
       |
Standby DB (AZ-b)
```

If the primary fails:

```
standby automatically becomes primary
```

Applications reconnect automatically.

---

### 9. Security Group Design

Security groups enforce **least privilege networking**.

Example security groups:

```
ALB-SG
Web-SG
App-SG
DB-SG
```

Traffic rules:

```
Internet → ALB-SG (HTTP/HTTPS)

ALB-SG → Web-SG (HTTP)

Web-SG → App-SG (API port)

App-SG → DB-SG (DB port)
```

All other traffic is denied.

---

### 10. Example Traffic Flow

Client request path:

```
User Browser
     |
Internet
     |
Application Load Balancer
     |
Web Server
     |
Application Server
     |
Database
```

Response path returns through the same route.

---

### 11. High Availability Design

Multi-AZ architecture:

```
                Internet
                   |
            Application Load Balancer
             /                    \
        AZ-a Web Server       AZ-b Web Server
             |                    |
        AZ-a App Server       AZ-b App Server
             |                    |
        Multi-AZ Database Cluster
```

Benefits:

```
fault tolerance
automatic failover
scalability
```

---

### 12. Autoscaling

Web and application tiers use **Auto Scaling Groups**.

Example scaling architecture:

```
Application Load Balancer
        |
--------------------------------
|              |               |
App Server    App Server     App Server
```

Benefits:

```
scale with demand
automatic instance replacement
high availability
```

---

### 13. Monitoring and Observability

Monitoring tools include:

```
CloudWatch
VPC Flow Logs
ALB metrics
RDS monitoring
```

Metrics monitored:

```
CPU utilization
request latency
network throughput
error rates
```

This helps maintain application health.

---

### 14. Production Architecture Diagram

Complete architecture:

```
                     Internet
                        |
                 Internet Gateway
                        |
                Application Load Balancer
                  /                  \
              Public AZ-a         Public AZ-b
                |                    |
              Web Tier            Web Tier
                |                    |
           Private App AZ-a     Private App AZ-b
                |                    |
                   Application Layer
                         |
                  Database Subnets
                         |
                   Multi-AZ RDS
```

This architecture follows AWS best practices.

---

### 15. Key Security Properties

This architecture enforces:

```
internet access only through ALB
application servers private
database isolated
least privilege security groups
multi-AZ high availability
```

This is the **standard production network architecture**.

---

### 16. Mental Model

Think of a 3-tier architecture like a **secure building**.

```
Front Desk → Web Layer
Office Floor → Application Layer
Vault Room → Database Layer
```

Visitors only access the front desk.

Employees access internal areas.

The vault is highly restricted.

---

### 17. Key Concepts Summary

| Layer | Role |
|------|------|
| Web Tier | Handles incoming requests |
| App Tier | Business logic |
| DB Tier | Data storage |
| ALB | Traffic distribution |
| NAT | Outbound internet for private servers |

---

### 18. Checkpoint Questions

You should now be able to answer these.

#### Why place application servers in private subnets?

```
To prevent direct internet access.
```

---

#### What component distributes traffic to web servers?

```
Application Load Balancer
```

---

#### How do private instances access the internet?

```
NAT Gateway
```

---

#### Which layer stores application data?

```
Database layer
```

---

#### Why deploy resources across multiple AZs?

```
To provide high availability and fault tolerance.
```
