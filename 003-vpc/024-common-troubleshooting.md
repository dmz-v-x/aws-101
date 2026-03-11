## AWS VPC Networking Troubleshooting Checklist — SG, NACL, Route Tables, IGW/NAT, DNS, ENI, and OS Firewall

When a connection between resources in AWS fails (for example **EC2 cannot reach the internet**, **service cannot connect to database**, or **VPCs cannot communicate**), the root cause usually lies in one of a few common networking layers.

A systematic troubleshooting approach helps quickly isolate the issue.

A typical AWS network path looks like this:

```
Application
   |
OS Firewall
   |
ENI
   |
Security Group
   |
Subnet
   |
Network ACL
   |
Route Table
   |
Gateway (IGW / NAT / TGW / VPN)
   |
Destination
```

Troubleshooting should move **step-by-step across each layer**.

This checklist covers the most common networking issues in AWS environments.

---

### 1. Verify the Basic Connectivity Problem

First confirm the actual failure scenario.

Example questions:

```
Can the instance reach the internet?
Can the instance reach another instance?
Can the service resolve DNS?
Is only one port failing?
```

Useful tests:

```
ping
curl
telnet
nc (netcat)
traceroute
```

Example:

```
curl https://google.com
```

This confirms outbound internet connectivity.

---

### 2. Check Security Groups (SG)

Security groups are the **most common cause of connectivity failures**.

Security groups are **stateful**.

Checklist:

```
Inbound rule allows the required port
Outbound rule allows traffic
Source CIDR is correct
Security group references are correct
```

Example issue:

```
Web server cannot reach database
```

Possible problem:

```
DB SG does not allow inbound 3306 from App SG
```

Correct rule example:

```
Inbound
Type: MySQL
Port: 3306
Source: App-SG
```

---

### 3. Check Network ACLs (NACL)

NACLs are **stateless subnet firewalls**.

Checklist:

```
Inbound rule allows traffic
Outbound rule allows response traffic
Rule order is correct
No deny rule blocks traffic
```

Example issue:

```
Instance cannot receive response packets
```

Possible cause:

```
Outbound ephemeral ports blocked
```

Example required rule:

```
Allow ports 1024–65535
```

---

### 4. Check Route Tables

Route tables determine where traffic should go.

Checklist:

```
Correct destination CIDR
Correct target (IGW, NAT, TGW, Peering)
Subnet associated with correct route table
```

Example internet access route:

```
0.0.0.0/0 → Internet Gateway
```

Example private subnet route:

```
0.0.0.0/0 → NAT Gateway
```

Common problem:

```
Subnet using wrong route table
```

---

### 5. Check Internet Gateway (IGW)

For internet access, the VPC must have an **Internet Gateway attached**.

Checklist:

```
IGW attached to VPC
Public subnet route table points to IGW
Instance has public IP or Elastic IP
Security group allows traffic
```

Example architecture:

```
EC2
 |
Public Subnet
 |
Route Table → IGW
 |
Internet
```

Without IGW, the instance cannot reach the internet.

---

### 6. Check NAT Gateway

Private subnets require **NAT Gateway** for outbound internet access.

Checklist:

```
NAT exists in public subnet
Elastic IP attached
Private subnet route points to NAT
NAT subnet route points to IGW
```

Example routing:

```
Private Subnet
0.0.0.0/0 → NAT Gateway
```

Common issue:

```
NAT deployed in wrong subnet
```

---

### 7. Check DNS Resolution

DNS issues can prevent services from connecting.

Checklist:

```
VPC DNS resolution enabled
VPC DNS hostnames enabled
Correct DNS server configuration
Correct Route53 records
```

Example test:

```
nslookup example.com
dig example.com
```

Example failure:

```
DNS resolves to incorrect IP
```

---

### 8. Check Elastic Network Interface (ENI)

Each instance communicates through its **ENI**.

Checklist:

```
Correct subnet assignment
Correct security groups attached
Correct private IP
ENI not detached
```

Example architecture:

```
EC2
 |
ENI
 |
Subnet
```

Misconfigured ENIs can break connectivity.

---

### 9. Check OS Firewall Rules

Sometimes the issue is **inside the instance itself**.

Common OS firewalls include:

```
iptables
firewalld
ufw
Windows Firewall
```

Example Linux check:

```
sudo iptables -L
```

Example issue:

```
Port 8080 blocked by OS firewall
```

Even if AWS allows the traffic, the OS can still block it.

---

### 10. Check Application Listening Ports

Sometimes the network works correctly but the application is not listening.

Example check:

```
netstat -tulnp
```

Example output:

```
tcp 0 0 0.0.0.0:8080 LISTEN
```

If the application is not listening, the connection will fail.

---

### 11. Check VPC Peering or TGW Routes

For cross-VPC communication verify:

```
Peering connection active
Routes added on both sides
Security groups allow CIDR
CIDR ranges do not overlap
```

Example route:

```
10.1.0.0/16 → VPC Peering
```

Without routes on both sides, communication fails.

---

### 12. Check Load Balancer Health Checks

For applications behind load balancers:

Checklist:

```
Target group health checks passing
Correct port configured
Security groups allow load balancer traffic
```

Example failure:

```
Targets marked unhealthy
```

This prevents traffic routing.

---

### 13. Use VPC Flow Logs

VPC Flow Logs help analyze traffic behavior.

Flow logs show:

```
source IP
destination IP
port
accept/reject status
```

Example:

```
ACCEPT TCP 10.0.1.10 → 10.0.2.15:443
REJECT TCP 10.0.1.10 → 10.0.2.15:3306
```

Rejected entries indicate blocking rules.

---

### 14. Common Troubleshooting Workflow

A typical investigation order:

```
1. Application running?
2. OS firewall rules?
3. Security group rules?
4. NACL rules?
5. Route tables?
6. Gateway configuration?
7. DNS resolution?
8. VPC flow logs?
```

Moving step-by-step avoids missing root causes.

---

### 15. Example Troubleshooting Scenario

Problem:

```
EC2 cannot reach the internet
```

Checklist:

```
Instance has public IP?
Security group outbound allowed?
Route table → IGW?
IGW attached to VPC?
OS firewall blocking?
```

Following the checklist usually identifies the issue.

---

### 16. Mental Model

Think of network troubleshooting like **checking doors along a hallway**.

```
Application → OS firewall → SG → NACL → Route → Gateway → Destination
```

If any door is closed, traffic stops.

---

### 17. Key Concepts Summary

| Layer | What to Check |
|------|------|
| Application | Service running and listening |
| OS Firewall | iptables / ufw / Windows firewall |
| ENI | Correct interface configuration |
| Security Group | Correct inbound/outbound rules |
| NACL | Correct allow rules and order |
| Route Table | Correct routing target |
| Gateway | IGW or NAT configured correctly |
| DNS | Correct name resolution |

---

### 18. Quick Troubleshooting Cheat Sheet

Common failure → likely cause:

```
Cannot reach internet → missing IGW or NAT
Cannot reach database → SG rule missing
Connection times out → NACL blocking traffic
DNS failure → VPC DNS disabled
Service unreachable → application not listening
```

---

### 19. Key Takeaway

Almost all AWS networking problems come from **one of these layers**:

```
Security Groups
NACLs
Route Tables
Gateways
DNS
Instance Firewall
```

Systematically checking each layer quickly identifies the issue.

---

### 20. Final Troubleshooting Rule

Always troubleshoot **from the inside outward**:

```
Application
OS
Instance
VPC
Gateway
Internet
```

This structured approach is used by AWS engineers and cloud architects.
