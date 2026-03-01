## Cluster Updates vs Version Upgrades

### 1. Introduction

Kubernetes clusters are not static systems. Over time, new Kubernetes versions are released that include security fixes, performance improvements, new APIs, and deprecations. Running outdated clusters can lead to security vulnerabilities, unsupported APIs, and operational instability.

In Amazon EKS, maintaining cluster health requires performing **regular cluster upgrades**. However, upgrading an EKS cluster is not a single-step operation. It involves multiple components that must be upgraded carefully.

The two main upgrade areas are:

Control Plane Upgrades  
Worker Node Upgrades

Understanding the relationship between these two components is critical for performing safe and reliable upgrades in production environments.

---

### 2. Understanding Kubernetes Versioning

Kubernetes follows semantic versioning:

Major.Minor.Patch

Example:

1.29.3

Major version changes are rare. Most updates occur in the minor version field.

Example minor upgrades:

1.27 → 1.28  
1.28 → 1.29

Patch versions include bug fixes and security patches.

Example:

1.29.1 → 1.29.3

Amazon EKS supports only specific Kubernetes versions, and each version eventually reaches end-of-life.

Running unsupported versions can cause cluster failures or unsupported API behavior.

---

### 3. Why Cluster Upgrades Are Required

Cluster upgrades are required for several reasons.

Security patches

Older Kubernetes versions may contain vulnerabilities.

Feature improvements

New versions introduce improvements to scheduling, networking, and API behavior.

API deprecations

Some APIs are removed in newer versions. If the cluster is not upgraded gradually, workloads may break.

Compatibility with AWS components

EKS add-ons such as the VPC CNI and CoreDNS require compatible Kubernetes versions.

Because of these factors, cluster upgrades must be part of regular operational maintenance.

---

### 4. Components Involved in EKS Upgrades

An EKS cluster contains multiple components that may require upgrades.

Control plane components

API server  
Scheduler  
Controller manager  
etcd

Worker nodes

EC2 instances running kubelet and container runtime

Cluster add-ons

CoreDNS  
kube-proxy  
VPC CNI

Applications running inside the cluster

Each component may have different upgrade procedures.

---

### 5. Understanding the Control Plane

The control plane is the central management system of Kubernetes.

It includes:

kube-apiserver  
kube-controller-manager  
kube-scheduler  
etcd

In self-managed Kubernetes clusters, administrators must upgrade these components manually.

In Amazon EKS, the control plane is fully managed by AWS.

This means AWS handles:

Control plane infrastructure  
High availability  
Patch management  
Component updates

However, administrators must still initiate the upgrade process.

---

### 6. Control Plane Upgrade Process in EKS

Upgrading the control plane changes the Kubernetes version used by the cluster API server.

Example command:

    aws eks update-cluster-version \
    --name production-cluster \
    --kubernetes-version 1.29

During this process AWS performs several steps.

Create new control plane instances  
Deploy updated Kubernetes components  
Verify cluster health  
Switch traffic to new control plane instances

Because the control plane runs across multiple Availability Zones, this process happens with minimal downtime.

---

### 7. Control Plane Upgrade Characteristics

Control plane upgrades in EKS are designed to be safe.

Important characteristics include:

High availability during upgrade  
Automatic rollback if failures occur  
No interruption to running pods

However, API changes can affect workloads if deprecated APIs are used.

Therefore compatibility checks should always be performed before upgrades.

---

### 8. Understanding Worker Nodes

Worker nodes run the containers that make up application workloads.

Each worker node includes:

kubelet  
container runtime  
kube-proxy

The kubelet version should remain compatible with the control plane version.

Kubernetes supports a **version skew policy** which allows kubelet to be slightly behind the control plane version.

However, large version differences are not supported.

---

### 9. Why Worker Nodes Must Be Upgraded

When the control plane version changes, worker nodes must eventually be upgraded as well.

Reasons include:

Security patches in node OS

New kubelet features

Compatibility with updated APIs

Support for new Kubernetes functionality

Running outdated nodes may cause scheduling problems or security risks.

---

### 10. Node Upgrade Approaches

There are several approaches for upgrading worker nodes.

Rolling node replacement

Blue-green node groups

Managed node group upgrades

Each approach ensures application workloads continue running during the upgrade process.

---

### 11. Managed Node Group Upgrades

Managed node groups simplify the upgrade process.

AWS performs rolling upgrades of worker nodes.

Upgrade command:

    eksctl upgrade nodegroup \
    --cluster production-cluster \
    --name application-nodes

During this process:

A new EC2 instance is launched with updated AMI  
Pods are drained from old node  
Workloads move to new node  
Old node is terminated

This process repeats until all nodes are upgraded.

---

### 12. Rolling Upgrade Strategy

Rolling upgrades ensure that applications remain available during node upgrades.

The process works as follows.

New node launches  
↓

Pods from old node are drained  
↓

Workloads move to other nodes  
↓

Old node is terminated

This ensures zero or minimal downtime.

---

### 13. Blue-Green Node Upgrade Strategy

Another strategy is creating a new node group with a newer version.

Example process.

Create new node group running updated AMI

Example:

    eksctl create nodegroup \
    --cluster production-cluster \
    --name new-nodegroup \
    --node-type m5.large

Migrate workloads to new nodes.

Delete old node group.

This approach provides safer upgrades and allows easy rollback.

---

### 14. Understanding Node Draining

When upgrading nodes, pods must be moved safely.

This is done using the drain command.

Example:

    kubectl drain node-1 \
    --ignore-daemonsets \
    --delete-emptydir-data

This command safely evicts pods from the node so they can run on other nodes.

DaemonSets are excluded because they automatically run on every node.

---

### 15. Pod Disruption Budgets

Pod Disruption Budgets ensure application availability during upgrades.

Example configuration:

    apiVersion: policy/v1
    kind: PodDisruptionBudget
    metadata:
      name: api-pdb
    spec:
      minAvailable: 2
      selector:
        matchLabels:
          app: api

This ensures at least two pods remain available during upgrades.

Without this configuration, upgrades could cause temporary outages.

---

### 16. Upgrading Cluster Add-ons

Cluster add-ons must also be upgraded after control plane updates.

Example:

    aws eks update-addon \
    --cluster-name production-cluster \
    --addon-name coredns

Add-ons must match the Kubernetes version to ensure compatibility.

Important add-ons include:

CoreDNS  
kube-proxy  
VPC CNI

---

### 17. Recommended Upgrade Order

Production upgrade strategy usually follows this order.

Step 1

Upgrade control plane.

Step 2

Upgrade cluster add-ons.

Step 3

Upgrade worker nodes.

Step 4

Verify workloads and application health.

Following this order ensures compatibility across cluster components.

---

### 18. Testing Upgrades Before Production

Never upgrade production clusters without testing.

Recommended approach:

Maintain staging cluster

Upgrade staging first

Run workload validation tests

Verify application functionality

After verification perform production upgrade.

---

### 19. Common Upgrade Gotchas

Deprecated APIs

Some APIs are removed in new Kubernetes versions.

Example:

extensions/v1beta1

Applications using deprecated APIs will fail.

Node version mismatch

Nodes too far behind control plane version can cause scheduling issues.

Insufficient cluster capacity

Rolling upgrades require spare capacity.

If cluster capacity is too small, workloads cannot move during upgrades.

---

### 20. Observability During Upgrades

Monitoring cluster health during upgrades is essential.

Key signals to monitor:

Node readiness

    kubectl get nodes

Pod health

    kubectl get pods -A

Cluster events

    kubectl get events

Application logs should also be monitored for failures.

---

### 21. Production Best Practices

Upgrade clusters regularly instead of skipping versions.

Always test upgrades in staging environments.

Use rolling upgrades for minimal downtime.

Maintain spare cluster capacity.

Monitor cluster health during upgrade operations.

Upgrade add-ons and node groups after control plane updates.
