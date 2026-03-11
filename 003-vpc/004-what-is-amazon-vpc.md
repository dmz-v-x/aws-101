## What Is an Amazon VPC and Its Core Components (VPC, Subnets, Route Tables, IGW, NAT, ENI, Elastic IP)

Before deploying servers, databases, or applications in the cloud, we must first create a **network environment** where those resources will live.

In AWS, this network environment is called a **VPC**.

A VPC is the **foundation of AWS networking**. Every EC2 instance, database, load balancer, or container runs inside a VPC.

This guide explains from **absolute zero to deeper understanding**:

- What a **VPC** is
- Why VPCs exist
- How AWS networking works internally
- Core VPC components
- How traffic flows inside a VPC
- How VPC components interact together

We will cover these components in detail:

```
VPC
Subnets
Route Tables
Internet Gateway (IGW)
NAT Gateway
Elastic Network Interface (ENI)
Elastic IP
```

---

### 1. The Problem VPC Solves

Before cloud networking, companies built **physical networks**.

Example:

```
Office Datacenter
 ├── Router
 ├── Switch
 ├── Firewall
 ├── Servers
 └── Storage
```

Each company built its **own private network**.

But in cloud computing:

- thousands of companies share the same infrastructure
- resources run inside shared data centers

The question becomes:

```
How do we give every customer their own isolated network?
```

The solution is:

```
Virtual Private Cloud (VPC)
```

---

### 2. What Is a VPC?

A **Virtual Private Cloud (VPC)** is:

> A logically isolated virtual network inside AWS where you can launch resources.

Key characteristics:

- logically isolated
- fully customizable network
- defined IP address range
- controlled routing
- configurable security

Think of a VPC as:

```
Your own private network inside AWS
```

Example representation:

```
AWS Cloud
 └── Your VPC
      ├── Subnets
      ├── EC2 instances
      ├── Databases
      └── Load balancers
```

Each AWS account can create **multiple VPCs**.

---

### 3. VPC CIDR Block (Network Range)

When creating a VPC you must define an **IP address range**.

Example:

```
10.0.0.0/16
```

This is the **CIDR block** for the VPC.

This defines the **available IP space** inside the network.

Example capacity:

```
/16 → 65,536 IP addresses
```

Inside this range we create **subnets**.

---

### 4. High-Level VPC Architecture

Example architecture:

```
              Internet
                  |
           Internet Gateway
                  |
            Public Subnet
               (Web)
                  |
            Private Subnet
              (App)
                  |
            Private Subnet
               (DB)
```

Each layer serves a different role.

---

### 5. Subnets

A **subnet** is a smaller network inside a VPC.

Definition:

> A subnet is a range of IP addresses within a VPC.

Example VPC:

```
10.0.0.0/16
```

Possible subnets:

```
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24
```

Each subnet typically hosts specific components.

Example architecture:

```
10.0.1.0/24 → Public web servers
10.0.2.0/24 → Application servers
10.0.3.0/24 → Databases
```

---

### 6. Public vs Private Subnets

Subnets are categorized based on **internet accessibility**.

#### Public Subnet

A subnet is public when:

```
Route table contains route to Internet Gateway
```

Example:

```
0.0.0.0/0 → IGW
```

Resources in public subnets can reach the internet.

Examples:

```
Web servers
Load balancers
Bastion hosts
```

---

#### Private Subnet

Private subnets do **not have direct internet access**.

Used for:

```
Application servers
Databases
Internal services
```

Traffic typically flows through:

```
NAT Gateway
```

---

### 7. Route Tables

A **route table** controls how network traffic moves inside a VPC.

Definition:

> A route table is a set of rules that determine where network traffic is directed.

Example route table:

| Destination | Target |
|------|------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | igw |

Meaning:

```
internal traffic → local VPC
internet traffic → IGW
```

Each subnet must be associated with a **route table**.

---

### 8. Internet Gateway (IGW)

An **Internet Gateway** allows communication between a VPC and the internet.

Definition:

> An IGW is a horizontally scaled, highly available gateway that enables internet connectivity.

Traffic flow example:

```
EC2 → IGW → Internet
Internet → IGW → EC2
```

Without an IGW:

```
Instances cannot access internet
```

Important notes:

- one IGW per VPC
- must be attached to VPC
- used by public subnets

---

### 9. NAT Gateway

Private subnets cannot directly access the internet.

But sometimes servers need to:

- download updates
- install packages
- call external APIs

This is solved using **NAT Gateway**.

Definition:

> NAT Gateway allows instances in private subnets to access the internet while preventing inbound connections.

Traffic flow:

```
Private EC2
    |
NAT Gateway
    |
Internet Gateway
    |
Internet
```

Key rule:

```
Outbound allowed
Inbound blocked
```

This keeps private resources secure.

---

### 10. Elastic Network Interface (ENI)

An **ENI** is a virtual network interface attached to an EC2 instance.

It is similar to a **network card (NIC)** in physical machines.

Each ENI contains:

```
Private IP address
Public IP (optional)
MAC address
Security groups
```

Example:

```
EC2 Instance
  |
Elastic Network Interface
  |
Subnet
```

Some instances can have **multiple ENIs**.

This allows:

- multi-homing
- advanced networking setups

---

### 11. Elastic IP

An **Elastic IP** is a static public IPv4 address.

Definition:

> Elastic IP is a public IP address designed to remain constant even if instances stop or restart.

Example scenario:

Normal public IP:

```
changes when instance stops
```

Elastic IP:

```
remains fixed
```

Use cases:

```
web servers
DNS records
whitelisted IP systems
```

Example:

```
Elastic IP → 3.120.45.11
```

You can remap the IP to another instance.

---

### 12. Putting Everything Together

Let’s combine all VPC components.

Example architecture:

```
                    Internet
                       |
                 Internet Gateway
                       |
                Public Subnet
                  Web Server
                       |
                  Route Table
                       |
                Private Subnet
                 App Server
                       |
                  NAT Gateway
                       |
                Private Subnet
                  Database
```

Each component has a role.

---

### 13. Traffic Flow Example

User visits website.

Step 1

```
User → Internet
```

Step 2

```
Internet → IGW
```

Step 3

```
IGW → Public Subnet
```

Step 4

```
Web Server → App Server
```

Step 5

```
App Server → Database
```

Step 6

Response flows back to the user.

---

### 14. Full VPC Example Diagram

```
                    Internet
                       |
                 Internet Gateway
                       |
                -----------------
                |               |
         Public Subnet      Public Subnet
          Web Server        Load Balancer
                |
          NAT Gateway
                |
        -------------------
        |                 |
   Private Subnet     Private Subnet
    App Servers        Databases
```

---

### 15. Real-World VPC Design

Typical production architecture:

```
VPC (10.0.0.0/16)

Public Subnets
 ├── Load Balancer
 └── NAT Gateway

Private Subnets
 ├── Application Servers
 └── Worker Nodes

Isolated Subnets
 └── Databases
```

This structure improves:

- security
- scalability
- availability

---

### 16. Key Concepts Summary

| Component | Purpose |
|------|------|
| VPC | Virtual network |
| Subnet | Network segment |
| Route Table | Traffic routing |
| Internet Gateway | Internet connectivity |
| NAT Gateway | Private subnet internet access |
| ENI | Virtual network card |
| Elastic IP | Static public IP |

---

### 17. Quick Mental Model

Think of AWS networking like a **virtual datacenter**.

```
VPC → entire datacenter network
Subnets → network segments
Route tables → routers
IGW → internet router
NAT → private internet proxy
ENI → network card
Elastic IP → static public IP
```

---

### 18. Checkpoint Questions

You should be able to answer:

#### What is a VPC?

A logically isolated virtual network inside AWS.

---

#### What is a subnet?

A smaller IP network inside a VPC.

---

#### What does a route table do?

It determines how network traffic is routed.

---

#### What does an Internet Gateway do?

It allows communication between a VPC and the internet.

---

#### What does a NAT Gateway do?

It allows private instances to access the internet without exposing them publicly.

---

#### What is an ENI?

A virtual network interface attached to an instance.

---

#### What is an Elastic IP?

A static public IP address in AWS.

---
