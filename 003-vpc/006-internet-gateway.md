## Internet Gateway (IGW) and How a Subnet Becomes Public (Route Tables + SG + ENI)

To allow resources inside a VPC to communicate with the **internet**, AWS uses a component called an **Internet Gateway (IGW)**.

However, attaching an IGW alone does **not automatically make a subnet public**.

For a subnet to truly be public, three things must work together:

```
Internet Gateway (IGW)
Route Table
Instance Network Configuration (ENI + Security Group)
```



---

### 1. Why Internet Connectivity Is Not Automatic in AWS

When you create a **VPC**, it is completely **isolated**.

Example:

```
VPC (10.0.0.0/16)
```

Resources inside the VPC can talk to **each other**, but they cannot access the internet.

Example situation:

```
EC2 instance → cannot reach google.com
```

Why?

Because the VPC has **no gateway to the internet**.

To allow internet access, AWS introduces:

```
Internet Gateway
```

---

### 2. What Is an Internet Gateway?

An **Internet Gateway (IGW)** is a VPC component that enables communication between a VPC and the public internet.

Definition:

> An Internet Gateway is a highly available, horizontally scaled gateway that allows resources in a VPC to communicate with the internet.

Key characteristics:

```
managed by AWS
horizontally scalable
highly available
attached to a VPC
```

You typically attach **one IGW per VPC**.

---

### 3. High-Level Architecture

Without IGW:

```
Internet
   X
   |
VPC
 |
EC2
```

Traffic cannot leave the VPC.

With IGW:

```
Internet
   |
Internet Gateway
   |
VPC
 |
EC2
```

Now traffic can flow.

---

### 4. Attaching an Internet Gateway to a VPC

The process has two steps.

Step 1 — Create IGW

```
Create Internet Gateway
```

Step 2 — Attach IGW to VPC

```
Attach IGW → VPC
```

Example architecture:

```
VPC
 |
Internet Gateway
 |
Internet
```

However, **this still does not make instances public**.

---

### 5. The Most Important Rule

A subnet becomes **public only when**:

```
Route Table contains route to Internet Gateway
```

Specifically:

```
0.0.0.0/0 → IGW
```

---

### 6. Understanding Route Tables

A **route table** tells AWS where network traffic should go.

Example route table:

| Destination | Target |
|------|------|
| 10.0.0.0/16 | local |
| 0.0.0.0/0 | igw |

Explanation:

```
10.0.0.0/16 → internal traffic stays inside VPC
0.0.0.0/0 → all internet traffic goes to IGW
```

Once this route exists, the subnet becomes **public**.

---

### 7. What Actually Makes a Subnet Public

A subnet is considered **public** when:

```
1. Route table has route to IGW
2. Instance has public IP
3. Security group allows inbound traffic
```

All three conditions must be satisfied.

---

### 8. Role of Elastic Network Interface (ENI)

Every EC2 instance has a **network interface**.

In AWS this is called:

```
Elastic Network Interface (ENI)
```

The ENI contains:

```
Private IP
Public IP (optional)
MAC address
Security groups
```

Example:

```
EC2 Instance
   |
ENI
   |
Subnet
```

Internet traffic flows through the **ENI**.

---

### 9. Role of Public IP Address

To communicate with the internet, an instance must have a **public IP address**.

Example:

```
Private IP → 10.0.1.15
Public IP → 3.92.120.15
```

Traffic flow:

```
Internet → Public IP → ENI → EC2
```

Without a public IP, inbound internet traffic cannot reach the instance.

---

### 10. Role of Security Groups

Security groups act like **virtual firewalls**.

They control **allowed traffic**.

Example security group rules:

| Type | Port | Source |
|------|------|------|
| SSH | 22 | Your IP |
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |

If security groups block traffic, the instance remains inaccessible.

---

### 11. Complete Public Subnet Architecture

Example setup:

```
Internet
   |
Internet Gateway
   |
Route Table
   |
Public Subnet
   |
EC2 Instance
   |
ENI + Public IP
   |
Security Group
```

Each layer plays a role.

---

### 12. Internet Traffic Flow (Step-by-Step)

User opens website:

```
http://example.com
```

Step 1

DNS resolves domain.

```
example.com → 3.92.120.15
```

Step 2

Traffic reaches AWS.

```
Internet → IGW
```

Step 3

IGW forwards traffic into VPC.

```
IGW → Route Table
```

Step 4

Route table determines subnet.

```
Route → Public Subnet
```

Step 5

Packet reaches ENI.

```
Subnet → ENI
```

Step 6

Security group evaluates rules.

If allowed:

```
Packet → EC2 instance
```

Response travels back the same path.

---

### 13. Visual Architecture

```
                 Internet
                    |
             Internet Gateway
                    |
             Route Table
        (0.0.0.0/0 → IGW)
                    |
              Public Subnet
                    |
               EC2 Instance
                    |
                ENI + SG
```

---

### 14. Example AWS Network Design

```
VPC (10.0.0.0/16)

Public Subnet
 ├── Web Server
 ├── Bastion Host
 └── Load Balancer

Private Subnet
 ├── App Servers
 └── Workers

Isolated Subnet
 └── Databases
```

Only the **public subnet** connects to the internet directly.

---

### 15. Common Beginner Mistakes

#### Mistake 1

IGW attached but no route.

Result:

```
No internet access
```

---

#### Mistake 2

Route exists but instance has no public IP.

Result:

```
Cannot receive internet traffic
```

---

#### Mistake 3

Security group blocks ports.

Example:

```
Port 80 blocked
```

Result:

```
Website unreachable
```

---

### 16. Quick Checklist for Public Subnet

To confirm a subnet is public:

Check these items.

```
IGW attached to VPC
Route table has 0.0.0.0/0 → IGW
Subnet associated with route table
Instance has public IP
Security group allows traffic
```

If all conditions are satisfied, the subnet is public.

---

### 17. Example CIDR Layout

```
VPC → 10.0.0.0/16

Public Subnet
10.0.1.0/24

Private Subnet
10.0.2.0/24

Isolated Subnet
10.0.3.0/24
```

Public subnet route:

```
0.0.0.0/0 → IGW
```

Private subnet route:

```
0.0.0.0/0 → NAT Gateway
```

---

### 18. Mental Model

Think of IGW as the **internet door** for your VPC.

```
VPC → Building
Subnet → Rooms
IGW → Main entrance
Route table → Directions to door
ENI → Device network cable
Security group → Door security guard
```

All pieces must work together.

---

### 19. Checkpoint Questions

You should be able to answer these.

#### What is an Internet Gateway?

A gateway that allows communication between a VPC and the internet.

---

#### What makes a subnet public?

A route table containing:

```
0.0.0.0/0 → Internet Gateway
```

---

#### Why does an EC2 instance need a public IP?

Because the internet can only reach **public addresses**.

---

#### What role does the ENI play?

It acts as the **network interface** connecting the instance to the subnet.

---

#### What role do security groups play?

They control **allowed inbound and outbound traffic**.

