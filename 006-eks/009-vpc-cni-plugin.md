## VPC CNI Plugin

### 1. Introduction

One of the most important differences between Amazon EKS and many local Kubernetes environments is **how networking works**.

If you have previously used local clusters such as:

- kind
- minikube
- Docker Desktop Kubernetes

you likely used networking plugins like:

Flannel  
Calico  
Weave

These plugins create **overlay networks** that assign internal IP addresses to pods.

Amazon EKS uses a very different networking model called the **Amazon VPC CNI plugin**.

Instead of giving pods internal overlay IPs, EKS assigns **real IP addresses from the AWS VPC** to pods.

This design deeply integrates Kubernetes networking with AWS infrastructure and introduces several unique concepts such as:

Elastic Network Interfaces (ENIs)  
IP allocation strategies  
Subnet IP exhaustion  
Pod density limits

Understanding how the **VPC CNI plugin works internally** is critical for operating EKS clusters in production.

---

### 2. What is CNI (Container Network Interface)

Before discussing the AWS-specific implementation, we must first understand **what CNI is**.

CNI stands for:

Container Network Interface

CNI is a **standard specification** that defines how container runtimes configure networking for containers.

When Kubernetes creates a pod, it must assign:

- an IP address
- network routing rules
- connectivity with other pods

The CNI plugin performs these tasks.

The sequence looks like this:

Pod is created  
↓  
Kubelet requests networking  
↓  
CNI plugin executes  
↓  
Network interface is created  
↓  
IP address is assigned

Every Kubernetes cluster must use a CNI plugin.

---

### 3. Overlay Networking vs Native Networking

Most Kubernetes clusters use **overlay networking**.

Overlay networks create a virtual network on top of the existing infrastructure.

Example:

Pod network CIDR

```
10.244.0.0/16
```

Pods communicate through encapsulated traffic.

Overlay solutions include:

Flannel  
Calico  
Weave Net

These networks hide infrastructure complexity but introduce:

Encapsulation overhead  
Additional routing layers

AWS took a different approach.

Instead of overlay networking, EKS uses **native VPC networking**.

---

### 4. What is the Amazon VPC CNI Plugin

The **Amazon VPC CNI plugin** allows Kubernetes pods to receive **IP addresses directly from the AWS VPC**.

This means pods become first-class citizens inside the VPC network.

Example architecture:

VPC Subnet

```
10.0.1.0/24
```

EC2 Node

```
10.0.1.15
```

Pod running on that node

```
10.0.1.21
```

The pod receives a **real VPC IP address** instead of an overlay IP.

This allows pods to communicate directly with:

Other pods  
EC2 instances  
Load balancers  
Databases  
AWS services

---

### 5. Components of the VPC CNI Plugin

The VPC CNI plugin runs inside the Kubernetes cluster.

It consists of several components.

aws-node DaemonSet

Runs on every worker node and manages networking.

ipamd

IP address management daemon responsible for allocating and releasing IPs.

CNI binary

Executes when Kubernetes creates a pod.

These components work together to assign IP addresses to pods.

---

### 6. Inspecting VPC CNI in an EKS Cluster

The VPC CNI plugin runs as a daemonset in the kube-system namespace.

Check running pods.

```
kubectl get pods -n kube-system -l k8s-app=aws-node
```

You should see a pod on every worker node.

Check daemonset configuration.

```
kubectl get daemonset aws-node -n kube-system
```

This daemon manages IP allocation for pods.

---

### 7. Understanding Elastic Network Interfaces (ENI)

To understand how VPC CNI works, you must understand **Elastic Network Interfaces**.

An ENI is a virtual network card attached to an EC2 instance.

Each ENI has:

Private IP address  
MAC address  
Security group association

Example EC2 instance:

Primary ENI

```
10.0.1.15
```

Additional ENI attached:

```
10.0.1.30
10.0.1.31
10.0.1.32
```

Each ENI can hold multiple IP addresses.

These IP addresses are used by Kubernetes pods.

---

### 8. How Pods Get IP Addresses

When a pod is created, the following sequence occurs.

Pod scheduled to node  
↓  
Kubelet requests networking  
↓  
VPC CNI plugin checks available IP pool  
↓  
IP assigned from ENI  
↓  
Pod network interface created

The pod receives one of the IP addresses assigned to the node’s ENI.

---

### 9. ENI Allocation Strategy

Each EC2 instance has limits on:

Number of ENIs  
Number of IP addresses per ENI

Example for t3.medium:

Maximum ENIs: 3  
IPs per ENI: 6

Total possible pod IPs roughly:

```
3 ENIs × 6 IPs = 18 IP addresses
```

However some IPs are reserved for the node.

Actual pod capacity becomes smaller.

---

### 10. Calculating Maximum Pods per Node

AWS publishes a table showing pod limits per instance type.

Example:

t3.medium

Maximum pods ≈ 17

m5.large

Maximum pods ≈ 29

These limits exist because pods require real VPC IP addresses.

If IP capacity is exhausted, pods cannot start.

---

### 11. IP Address Pooling Behavior

The VPC CNI plugin tries to **pre-allocate IP addresses** for faster pod startup.

Example behavior:

Node launches  
↓  
VPC CNI allocates several IP addresses  
↓  
Pods consume those IPs

When IP pool becomes low, the plugin requests additional ENIs.

---

### 12. Secondary ENI Attachment

If IP capacity on the primary ENI is exhausted, the plugin attaches additional ENIs.

Process:

Node running pods  
↓  
IP pool becomes low  
↓  
ipamd requests new ENI  
↓  
AWS attaches ENI  
↓  
New IP addresses become available

This dynamic attachment enables scaling.

---

### 13. What is IP Exhaustion

IP exhaustion occurs when no more IP addresses are available in the subnet.

Example:

Subnet CIDR

```
10.0.1.0/24
```

Total addresses = 256

If too many pods or nodes are created, the subnet runs out of IP addresses.

Symptoms include:

Pods stuck in Pending state  
Network errors  
Scheduling failures

---

### 14. Detecting IP Exhaustion

Check pod status.

```
kubectl get pods
```

Pods may show:

```
Pending
```

Check aws-node logs.

```
kubectl logs <aws-node-pod> -n kube-system
```

Common error:

```
failed to assign IP address
```

This indicates subnet exhaustion.

---

### 15. Common Causes of IP Exhaustion

Small subnet CIDR blocks.

Example:

```
/27
```

This provides only 32 IP addresses.

Large clusters require much larger CIDR blocks.

High pod density on nodes.

Running too many pods on limited IP capacity nodes.

Improper subnet planning.

---

### 16. Best Practices for Subnet Design

Use sufficiently large CIDR ranges.

Example production recommendation:

```
/19 or /20
```

This provides thousands of IP addresses.

Separate node subnets and service subnets.

Use multiple availability zones.

These practices prevent IP exhaustion.

---

### 17. Advanced Feature: Prefix Delegation

Newer versions of VPC CNI support **prefix delegation**.

Instead of assigning individual IP addresses, AWS allocates **IP blocks** to nodes.

Example:

```
/28 prefix
```

This provides 16 IP addresses per prefix.

Benefits include:

Higher pod density  
Reduced ENI usage  
Better scalability

---

### 18. Monitoring IP Usage

Monitoring IP allocation is critical.

Useful commands:

Check nodes.

```
kubectl get nodes
```

Check aws-node logs.

```
kubectl logs -n kube-system <aws-node-pod>
```

AWS CloudWatch metrics also provide IP usage visibility.

---

### 19. Production Networking Gotchas

IP exhaustion due to small subnet ranges.

Pods failing to schedule because ENI limits are reached.

High startup latency due to ENI attachment delays.

Security group misconfiguration blocking pod traffic.

Understanding these issues helps troubleshoot EKS networking problems.
