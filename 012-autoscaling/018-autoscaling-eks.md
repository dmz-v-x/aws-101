## Autoscaling in Amazon EKS (Kubernetes) — Cluster Autoscaler, Karpenter, Horizontal Pod Autoscaler (HPA), and Node Autoscaling

### 1. Introduction

Modern applications built on Kubernetes must handle dynamic and unpredictable workloads. As user demand increases or decreases, the system must adjust resources automatically to maintain performance and efficiency.

In Kubernetes environments running on **Amazon Elastic Kubernetes Service (EKS)**, autoscaling occurs at multiple levels.

Autoscaling can affect:

```
Application containers (pods)
Worker nodes (EC2 instances)
Cluster infrastructure capacity
```

These different scaling layers ensure that both the application and the underlying infrastructure can adjust dynamically.

Kubernetes-based autoscaling mechanisms include:

```
Horizontal Pod Autoscaler (HPA)
Cluster Autoscaler
Karpenter
```

Each of these components serves a different role in managing cluster capacity.

---

### 2. The Two Levels of Scaling in Kubernetes

Autoscaling in Kubernetes occurs at two primary levels.

```
Pod-level scaling
Node-level scaling
```

Understanding these levels is essential for designing scalable Kubernetes systems.

---

#### Pod-Level Scaling

Pod-level scaling adjusts the number of **application pods** running inside the cluster.

Example:

```
Web application running 3 pods
Traffic increases
System scales to 10 pods
```

This type of scaling ensures the application can handle increased workload.

---

#### Node-Level Scaling

Node-level scaling adjusts the number of **worker nodes** in the cluster.

Example:

```
Cluster has 3 nodes
Pods require more resources
Cluster adds 2 additional nodes
```

This ensures there is enough infrastructure capacity to run the pods.

---

### 3. Understanding Kubernetes Pods and Nodes

To understand autoscaling, it is important to understand Kubernetes resources.

---

#### Pods

A **pod** is the smallest deployable unit in Kubernetes.

It usually contains:

```
One or more containers
Shared network configuration
Shared storage
```

Example:

```
Pod → container running a web server
```

Pods represent the actual application workloads.

---

#### Nodes

A **node** is a machine that runs pods.

In Amazon EKS, nodes are typically:

```
EC2 instances
```

Each node provides:

```
CPU resources
Memory resources
Networking capacity
Storage
```

Pods are scheduled onto nodes by the Kubernetes scheduler.

---

### 4. The Scaling Problem in Kubernetes

When demand increases, two challenges arise.

Example scenario:

```
Traffic spike occurs
↓
Application requires more pods
↓
Pods scheduled by Kubernetes
```

However, if the cluster lacks resources:

```
Pods remain pending
No nodes available with sufficient CPU or memory
```

This creates a resource bottleneck.

To solve this problem, Kubernetes uses multiple autoscaling systems that work together.

---

### 5. Horizontal Pod Autoscaler (HPA)

The **Horizontal Pod Autoscaler (HPA)** automatically adjusts the number of pods in a deployment.

HPA scales pods horizontally by increasing or decreasing pod replicas.

Example scaling rule:

```
Target CPU utilization = 60%
```

If CPU usage rises above the target:

```
HPA increases the number of pods
```

If CPU usage drops:

```
HPA reduces the number of pods
```

This ensures the application has enough instances to handle workload demand.

---

### 6. How HPA Works

HPA operates using metrics collected from the Kubernetes metrics system.

Typical workflow:

```
Metrics server collects pod metrics
↓
HPA controller evaluates metrics
↓
Desired replica count calculated
↓
Deployment replica count updated
```

Example scaling flow:

```
CPU utilization increases to 80%
Target CPU = 60%
↓
HPA increases pod replicas
```

More pods share the workload.

---

### 7. Metrics Used by HPA

HPA can scale pods based on different types of metrics.

Common metrics include:

```
CPU utilization
Memory usage
Custom application metrics
External metrics
```

Example custom metric:

```
Requests per second
Queue length
Active user sessions
```

These metrics allow scaling based on application demand.

---

### 8. Limitations of HPA

HPA only manages **pod scaling**, not infrastructure scaling.

Example scenario:

```
HPA requests 10 pods
But cluster has resources for only 5 pods
```

Remaining pods remain:

```
Pending
```

To solve this issue, node-level autoscaling is required.

---

### 9. Node Autoscaling in Kubernetes

Node autoscaling adjusts the number of worker nodes in the cluster.

Example scenario:

```
Pods require additional CPU
No nodes available
Cluster autoscaler launches new EC2 instances
```

New nodes allow Kubernetes to schedule pending pods.

Node autoscaling ensures the cluster infrastructure grows as application demand increases.

---

### 10. Cluster Autoscaler

The **Cluster Autoscaler** is a Kubernetes component that automatically adjusts the number of nodes in the cluster.

Cluster Autoscaler works with:

```
EC2 Auto Scaling Groups
```

It monitors pods that cannot be scheduled due to insufficient resources.

Workflow:

```
Pods remain pending
↓
Cluster Autoscaler detects resource shortage
↓
New nodes launched in Auto Scaling Group
↓
Pods scheduled onto new nodes
```

---

### 11. How Cluster Autoscaler Works

Cluster Autoscaler continuously evaluates the cluster state.

Scaling process:

```
Unschedulable pods detected
↓
Cluster Autoscaler determines required node size
↓
Auto Scaling Group launches new EC2 instances
↓
Nodes join the cluster
↓
Pods scheduled onto nodes
```

Cluster Autoscaler also supports scale-in.

Example:

```
Nodes underutilized
Pods rescheduled elsewhere
Node terminated
```

This optimizes cluster resource usage.

---

### 12. Limitations of Cluster Autoscaler

Although Cluster Autoscaler works well, it has some limitations.

---

#### Slow Provisioning

Node provisioning may take several minutes.

Example:

```
EC2 instance launch
Node bootstrap
Cluster registration
```

This delay may impact scaling speed.

---

#### Limited Flexibility

Cluster Autoscaler depends on predefined Auto Scaling Groups.

Example configuration:

```
Node group with specific instance type
```

If that instance type is unavailable, scaling may fail.

---

### 13. Karpenter

Karpenter is a newer autoscaling solution designed for Kubernetes clusters.

It was developed by AWS to improve cluster autoscaling efficiency.

Karpenter dynamically provisions nodes based on pod requirements.

Key characteristics include:

```
Direct EC2 provisioning
Flexible instance selection
Faster node startup
Better resource efficiency
```

Karpenter replaces traditional node groups with more dynamic infrastructure provisioning.

---

### 14. How Karpenter Works

Karpenter observes unscheduled pods and provisions nodes that match their resource requirements.

Workflow:

```
Pods pending
↓
Karpenter analyzes resource requirements
↓
Appropriate EC2 instances launched
↓
Node joins cluster
↓
Pods scheduled
```

Karpenter selects the most suitable instance type automatically.

---

### 15. Instance Flexibility with Karpenter

Unlike Cluster Autoscaler, Karpenter can choose from many instance types.

Example:

```
m5.large
m5a.large
c5.large
c6i.large
```

Benefits include:

```
Better EC2 capacity availability
Lower cost using Spot instances
More efficient resource allocation
```

Karpenter optimizes instance selection dynamically.

---

### 16. Node Consolidation with Karpenter

Karpenter can also consolidate nodes when resources are underutilized.

Example scenario:

```
Three nodes lightly utilized
Karpenter consolidates workloads
Two nodes removed
```

This improves cost efficiency.

---

### 17. HPA vs Node Autoscaling

It is important to understand the difference between pod scaling and node scaling.

| Scaling Type | Component | What It Scales |
|--------------|-----------|---------------|
| Pod scaling | Horizontal Pod Autoscaler | Number of pods |
| Node scaling | Cluster Autoscaler | Number of nodes |
| Node provisioning | Karpenter | Dynamically creates nodes |

These mechanisms work together to maintain cluster performance.

---

### 18. Example End-to-End Autoscaling Flow

Example scenario in an EKS cluster:

Initial configuration:

```
3 nodes
6 pods
```

Traffic spike occurs.

Process:

```
CPU utilization increases
↓
HPA increases pod count
↓
Pods exceed available node capacity
↓
Cluster Autoscaler or Karpenter launches new nodes
↓
Pods scheduled on new nodes
```

When traffic decreases:

```
HPA reduces pod replicas
↓
Nodes become underutilized
↓
Cluster Autoscaler or Karpenter removes nodes
```

This maintains efficient cluster utilization.

---

### 19. Best Practices for EKS Autoscaling

When designing autoscaling for Kubernetes clusters, several best practices should be followed.

---

#### Use HPA for Application Scaling

Scale application pods based on real workload metrics.

---

#### Enable Node Autoscaling

Ensure the cluster can grow when pods require additional resources.

---

#### Consider Using Karpenter

Karpenter provides faster and more flexible node provisioning compared to traditional node groups.

---

#### Monitor Cluster Metrics

Use monitoring tools such as:

```
Prometheus
CloudWatch
Grafana
```

to observe scaling behavior.

---

### 20. Summary

Autoscaling in Amazon EKS operates at multiple levels to ensure both application workloads and cluster infrastructure scale efficiently.

Key autoscaling components include:

```
Horizontal Pod Autoscaler (HPA) → scales application pods
Cluster Autoscaler → scales node groups
Karpenter → dynamically provisions nodes
```

HPA ensures that applications have enough replicas to handle workload demand.

Cluster Autoscaler and Karpenter ensure the cluster has enough infrastructure capacity to run those pods.

Together, these autoscaling mechanisms allow Kubernetes clusters on AWS to become:

```
Highly scalable
Cost efficient
Resilient
Automatically responsive to workload changes
```
