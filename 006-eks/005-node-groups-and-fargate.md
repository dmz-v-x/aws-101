## Managed Node Groups, Self Managed Node Groups and Fargate

### 1. Introduction

When you create an Amazon EKS cluster, the control plane alone is not enough to run workloads. Kubernetes needs compute resources where containers can actually run. These compute resources are provided through **worker nodes**.

In Amazon EKS, there are three major ways to run workloads:

Managed Node Groups  
Self-managed Node Groups  
AWS Fargate

All three provide compute for Kubernetes pods, but they differ in how much infrastructure you manage, how much control you have, and how operations are handled.

Understanding these models is critical when designing **production Kubernetes architectures**, because choosing the wrong model can lead to operational complexity, higher costs, or limited capabilities.

This guide explains everything from the fundamental concepts to advanced production scenarios.

---

### 2. Understanding Worker Nodes in Kubernetes

Before discussing the three approaches, we must understand **what a worker node is**.

A worker node is a machine that runs containers.

Each worker node contains the following core components:

kubelet  
kube-proxy  
container runtime (usually containerd)

These components communicate with the Kubernetes control plane.

The control plane schedules pods to nodes, and the node runs the containers required by the pods.

In Amazon EKS, worker nodes are typically **EC2 instances**, but they can also be **serverless compute environments through Fargate**.

---

### 3. Why Multiple Worker Node Models Exist

Different organizations have different operational needs.

Some teams want:

maximum control over infrastructure

Other teams want:

minimal operational overhead

AWS introduced multiple compute models so teams can choose between:

Control vs Simplicity

Operational responsibility vs Automation

Infrastructure flexibility vs Serverless convenience

This leads to three different worker node approaches.

Managed Node Groups  
Self-managed Node Groups  
Fargate

---

### 4. Managed Node Groups Overview

Managed Node Groups are AWS-managed collections of EC2 instances that act as Kubernetes worker nodes.

In this model:

AWS manages the lifecycle of worker nodes.

You still choose the instance type, scaling configuration, and AMI, but AWS handles most of the operational tasks.

Managed Node Groups internally use:

EC2 instances  
Auto Scaling Groups  
Launch templates

AWS automatically integrates these resources with the EKS control plane.

---

### 5. What AWS Manages in Managed Node Groups

AWS manages several important operational tasks.

These include:

Node provisioning  
Node registration with the cluster  
Health monitoring  
Automatic replacement of unhealthy nodes  
Rolling node upgrades  
Auto Scaling integration

Because of this automation, Managed Node Groups significantly reduce operational complexity.

---

### 6. Hands-on Example: Creating a Managed Node Group

Create a managed node group using eksctl.

```
eksctl create nodegroup \
--cluster production-cluster \
--name application-nodes \
--node-type t3.medium \
--nodes 3 \
--nodes-min 2 \
--nodes-max 5
```

This command creates:

Auto Scaling Group  
EC2 worker nodes  
IAM roles  
Security group configuration

Once created, verify nodes.

```
kubectl get nodes
```

The nodes should appear in the cluster.

---

### 7. Managed Node Group Architecture

Managed Node Groups operate as follows.

EKS Control Plane schedules pods  
↓

Auto Scaling Group provides EC2 instances  
↓

Instances join cluster through kubelet  
↓

Pods are scheduled to nodes

AWS monitors node health and replaces instances automatically if they fail.

---

### 8. Advantages of Managed Node Groups

Reduced operational complexity

AWS handles node lifecycle operations.

Automatic node replacement

Failed nodes are replaced automatically.

Integrated scaling

Works with Kubernetes cluster autoscaler.

Security updates

Node updates are easier to perform.

Managed Node Groups are widely used in **production clusters**.

---

### 9. Limitations of Managed Node Groups

Less flexibility compared to self-managed nodes.

Some custom AMI configurations may be harder to implement.

Less control over node bootstrap process.

Still requires EC2 infrastructure management.

---

### 10. Self-managed Node Groups Overview

Self-managed node groups give you **full control over worker nodes**.

You manually create and manage the EC2 instances.

This approach was the **original method used by EKS** before managed node groups were introduced.

In this model you must manage:

EC2 instances  
Auto Scaling Groups  
Launch templates  
Node upgrades  
Node health monitoring

---

### 11. Architecture of Self-managed Node Groups

In this architecture:

You create EC2 instances manually.

Each instance must run a bootstrap script that connects it to the cluster.

Example bootstrap command used by worker nodes.

```
/etc/eks/bootstrap.sh production-cluster
```

This script installs Kubernetes components and registers the node with the control plane.

---

### 12. Hands-on Example: Creating Self-managed Node Groups

Using eksctl:

```
eksctl create nodegroup \
--cluster production-cluster \
--name self-managed-nodes \
--node-type m5.large \
--nodes 2 \
--managed=false
```

This creates nodes but **does not use AWS managed node group services**.

You are responsible for node lifecycle management.

---

### 13. Responsibilities in Self-managed Node Groups

When using self-managed nodes, you must handle:

Node upgrades  
OS patching  
Scaling policies  
Failure replacement  
Cluster integration

If a node becomes unhealthy, AWS will not automatically replace it unless you configure automation.

---

### 14. Advantages of Self-managed Node Groups

Maximum customization

You can use custom AMIs or OS configurations.

Advanced networking setups

Custom storage drivers

Custom bootstrap logic

Full control over instance lifecycle

Some large enterprises prefer this model for **specialized workloads**.

---

### 15. Disadvantages of Self-managed Node Groups

Operational burden

You must maintain nodes yourself.

More complex infrastructure management

Upgrades require manual intervention.

Greater risk of misconfiguration

Because of these challenges, many teams prefer managed node groups.

---

### 16. AWS Fargate Overview

Fargate is a **serverless compute engine for containers**.

With Fargate you do not manage nodes at all.

Instead of launching EC2 instances, AWS runs pods directly on managed infrastructure.

In this model:

You deploy pods  
AWS provisions compute automatically

There are no visible worker nodes.

---

### 17. How Fargate Works Internally

Each pod runs in an isolated environment managed by AWS.

When a pod is scheduled:

EKS detects the pod matches a Fargate profile  
↓

AWS launches serverless compute  
↓

Pod runs on that infrastructure

Each pod receives:

Dedicated CPU  
Dedicated memory  
Dedicated network interface

---

### 18. Hands-on Example: Creating Fargate Profile

Create a Fargate profile.

```
eksctl create fargateprofile \
--cluster production-cluster \
--name default \
--namespace default
```

Now deploy an application.

```
kubectl create deployment nginx --image=nginx
```

Check pods.

```
kubectl get pods -o wide
```

The pods will run on Fargate instead of EC2 nodes.

---

### 19. Advantages of Fargate

No server management

No node patching

No capacity planning

Strong isolation between workloads

Fargate is ideal for teams that want a **fully serverless Kubernetes experience**.

---

### 20. Limitations of Fargate

Higher cost compared to EC2 nodes.

Limited support for certain Kubernetes features.

Some DaemonSets cannot run.

GPU workloads are not supported.

Less control over networking configuration.

---

### 21. Production Architecture Patterns

Most production EKS clusters use **a hybrid approach**.

Example architecture.

Managed Node Groups

Used for large microservices and backend APIs.

Fargate

Used for small services or event-driven workloads.

Specialized node groups

Used for machine learning workloads or GPU workloads.

This architecture balances cost and operational simplicity.

---

### 22. When to Use Managed Node Groups

Use Managed Node Groups when:

Running most production workloads

You want automated node lifecycle management

You want EC2 flexibility without heavy operational burden

This is the **default choice for most teams**.

---

### 23. When to Use Self-managed Node Groups

Use self-managed nodes when:

You require custom AMIs

You need advanced OS customization

You need specialized networking configurations

You require full infrastructure control

This is typically used by **large enterprises with complex infrastructure requirements**.

---

### 24. When to Use Fargate

Use Fargate when:

You want zero node management

You run small workloads

You want strong workload isolation

You run event-driven workloads

It is also useful for **running system workloads separately from application nodes**.
