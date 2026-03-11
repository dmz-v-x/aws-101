## Elastic Network Interfaces (ENI) and Secondary IPs — Multi-Homing Instances in AWS

Every server connected to a network needs a **network interface card (NIC)**.  
In physical machines this is a hardware device.

In AWS, the equivalent concept is the **Elastic Network Interface (ENI)**.

ENIs are one of the most fundamental building blocks of VPC networking because **all network communication to and from an EC2 instance flows through an ENI**.

Understanding ENIs is important because they enable:

- multiple IP addresses on a single instance
- attaching multiple network interfaces
- advanced networking architectures
- failover systems
- multi-network connectivity (multi-homing)

---

### 1. What Is an Elastic Network Interface (ENI)?

An **Elastic Network Interface (ENI)** is a virtual network interface attached to an EC2 instance.

Definition:

> An ENI is a virtual network card that enables communication between an EC2 instance and a VPC network.

Every EC2 instance has at least **one ENI**.

Example:

```
EC2 Instance
     |
     ENI
     |
     Subnet
     |
     VPC
```

The ENI connects the instance to the network.

---

### 2. Components of an ENI

An ENI contains several networking attributes.

Typical attributes include:

```
Primary private IP address
Secondary private IP addresses
Public IP address (optional)
Elastic IP (optional)
MAC address
Security groups
Subnet association
```

Example representation:

```
ENI
 ├ Primary IP
 ├ Secondary IPs
 ├ MAC address
 ├ Security groups
 └ Elastic IP
```

These properties control how the instance communicates.

---

### 3. Primary Private IP Address

When an EC2 instance launches, AWS assigns a **primary private IP address**.

Example:

```
10.0.1.15
```

Characteristics:

```
cannot be removed
persists for lifetime of ENI
used as main instance address
```

All instances must have a **primary private IP**.

---

### 4. Secondary Private IP Addresses

An ENI can have **multiple additional IP addresses**.

These are called **secondary private IPs**.

Example ENI configuration:

```
Primary IP
10.0.1.15

Secondary IPs
10.0.1.16
10.0.1.17
10.0.1.18
```

All of these addresses belong to the **same network interface**.

Applications can bind to different IPs.

---

### 5. Why Secondary IPs Exist

Secondary IPs allow a single instance to represent **multiple network identities**.

Example use cases:

```
multiple services on same instance
container networking
virtual IP failover
load balancer appliances
network appliances
```

Example scenario:

```
EC2 instance
 ├ Service A → 10.0.1.15
 ├ Service B → 10.0.1.16
 └ Service C → 10.0.1.17
```

Each service can have its own IP.

---

### 6. Public IP and Elastic IP

An ENI may also have a **public IP address**.

Example:

```
Private IP → 10.0.1.15
Public IP → 3.91.45.22
```

Alternatively, you can attach an **Elastic IP**.

Elastic IP:

```
static public IPv4 address
remains constant
```

Example:

```
Elastic IP → 54.201.32.10
```

This maps to the private IP of the ENI.

---

### 7. Multiple ENIs per Instance

Some instance types allow **multiple ENIs**.

Example:

```
EC2 Instance
 ├ ENI-1
 └ ENI-2
```

Each ENI connects to a subnet.

Example architecture:

```
ENI-1 → Public subnet
ENI-2 → Private subnet
```

This allows advanced network designs.

---

### 8. What Is Multi-Homing?

When an instance has **multiple network interfaces connected to different networks**, it is called **multi-homing**.

Definition:

> Multi-homing means connecting a system to multiple networks simultaneously.

Example architecture:

```
              Internet
                 |
           Public Subnet
                 |
              ENI-1
                 |
            EC2 Instance
                 |
              ENI-2
                 |
           Private Subnet
                 |
            Database
```

This instance acts as a **bridge between networks**.

---

### 9. Multi-Homing Use Cases

Multi-homed instances are used in many advanced networking systems.

Common examples:

```
firewalls
NAT appliances
security inspection systems
network routers
VPN gateways
```

Example firewall architecture:

```
Internet
   |
Public ENI
   |
Firewall EC2
   |
Private ENI
   |
Internal Network
```

Traffic flows through the firewall.

---

### 10. ENI Attachment Types

ENIs can be attached in two ways.

#### Primary ENI

Created automatically with instance launch.

```
cannot be detached
```

---

#### Secondary ENI

Additional network interface.

```
can be attached/detached
```

Used for flexible networking.

---

### 11. ENI Failover Example

ENIs allow **failover systems**.

Example architecture:

```
Primary Server
   |
ENI-A

Standby Server
```

If primary fails:

```
Detach ENI-A
Attach ENI-A → Standby Server
```

Traffic automatically moves to standby.

This pattern is used in:

```
high availability systems
network appliances
failover clusters
```

---

### 12. Security Groups and ENIs

Security groups are attached to **ENIs**, not instances.

Example:

```
ENI
 ├ Security Group A
 └ Security Group B
```

This controls traffic allowed through the network interface.

Example rules:

```
Allow SSH
Allow HTTPS
Allow internal communication
```

---

### 13. Example ENI Architecture

Example application server:

```
EC2 Instance

ENI
 ├ Primary IP → 10.0.2.15
 ├ Secondary IP → 10.0.2.16
 ├ Security Group
 └ MAC Address
```

Traffic flows through the ENI.

---

### 14. Multi-Interface Architecture Example

Example network appliance:

```
                Internet
                   |
              Public Subnet
                   |
                 ENI-1
                   |
             Security Appliance
                   |
                 ENI-2
                   |
              Private Subnet
                   |
               App Servers
```

The appliance inspects traffic between networks.

---

### 15. ENI Limits

Number of ENIs per instance depends on **instance type**.

Example:

| Instance Type | Max ENIs |
|------|------|
| t3.micro | 2 |
| m5.large | 3 |
| c5.4xlarge | 8 |

Each ENI also supports multiple private IPs.

---

### 16. Secondary IP Limits

Each ENI supports multiple IP addresses.

Example:

```
Primary IP
Secondary IPs
```

Exact limits depend on:

```
instance type
network performance tier
```

High-performance instances support many IPs.

---

### 17. Visual Example

```
EC2 Instance
     |
     ENI
     |
---------------------------------
| Primary IP     → 10.0.1.15     |
| Secondary IP   → 10.0.1.16     |
| Secondary IP   → 10.0.1.17     |
| Security Group → Web SG        |
| MAC Address    → aa:bb:cc:dd   |
---------------------------------
```

This ENI connects the instance to the network.

---

### 18. Mental Model

Think of ENIs like **network cables and ports on a server**.

```
Physical Server
 ├ Network Port 1
 └ Network Port 2
```

In AWS:

```
EC2 Instance
 ├ ENI-1
 └ ENI-2
```

Each interface connects to a network.

---

### 19. Key Concepts Summary

| Concept | Meaning |
|------|------|
| ENI | Virtual network interface |
| Primary IP | Main private IP |
| Secondary IP | Additional IPs on same interface |
| Multi-Homing | Multiple network interfaces |
| Elastic IP | Static public IP |
| Security Groups | Firewall rules attached to ENI |

---

### 20. Checkpoint Questions

You should now be able to answer these.

#### What is an ENI?

A virtual network interface that connects an EC2 instance to a VPC network.

---

#### What is a primary private IP?

The main private IP address assigned to an ENI.

---

#### What are secondary private IPs?

Additional IP addresses assigned to the same network interface.

---

#### What is multi-homing?

Connecting a system to multiple networks using multiple interfaces.

---

#### Where are security groups attached?

```
ENI
```
