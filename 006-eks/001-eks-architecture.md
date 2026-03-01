## Introduction to EKS

### 1. Introduction

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service provided by AWS that allows you to run Kubernetes clusters without managing the Kubernetes control plane yourself.

Before working with EKS in a production environment, it is critical to understand its internal architecture. Understanding how EKS is structured will help you troubleshoot problems, design production-ready clusters, and operate workloads effectively.

The core architectural components of EKS are:

- Managed Control Plane
- Worker Nodes
- Managed Node Groups
- AWS Fargate

In this blog we will start from the most fundamental concepts and gradually move toward advanced architectural understanding using hands-on scenarios and real-world production perspectives.

---

### 2. Quick Kubernetes Refresher (Why Architecture Matters)

Kubernetes clusters are composed of two major parts:

Control Plane  
Worker Nodes

The control plane makes decisions about the cluster, while worker nodes actually run application containers.

In a traditional Kubernetes setup, you must manage both of these components yourself. This means you must maintain:

- etcd
- kube-apiserver
- kube-controller-manager
- kube-scheduler
- networking
- node provisioning
- upgrades

Managing this entire stack is operationally heavy.

Amazon EKS simplifies this by managing the control plane for you while still allowing you to manage compute resources where your containers run.

---

### 3. High Level Architecture of Amazon EKS

An EKS cluster consists of two main layers.

AWS Managed Layer
Customer Managed Layer

AWS manages the Kubernetes control plane. You manage the compute infrastructure where containers run.

Architecture structure:

Control Plane (AWS Managed)

- kube-apiserver
- etcd
- scheduler
- controller manager

Worker Infrastructure (Customer Managed)

- EC2 Worker Nodes
- Managed Node Groups
- Fargate

This separation is extremely important when designing production infrastructure.

---

### 4. Understanding the Managed Control Plane

The control plane is the brain of a Kubernetes cluster.

It contains the components responsible for maintaining cluster state and scheduling workloads.

Key control plane components include:

kube-apiserver  
etcd  
kube-scheduler  
kube-controller-manager

In a self-managed Kubernetes cluster you would need to install and manage all of these manually.

In EKS, AWS manages the control plane for you.

This includes:

- High availability
- Scaling
- Security patches
- Version upgrades
- Backup of etcd
- Control plane monitoring

---

### 5. What AWS Actually Manages in the Control Plane

When you create an EKS cluster, AWS deploys the control plane inside an AWS-managed account.

This means you cannot see the control plane nodes directly.

However AWS ensures:

- Multi Availability Zone deployment
- Automatic failover
- Load-balanced API servers

The Kubernetes API endpoint is exposed to your AWS account.

You interact with the cluster through:

kubectl  
AWS CLI  
SDKs

Example of accessing the cluster:

    aws eks update-kubeconfig --region ap-south-1 --name my-cluster

This command updates your local kubeconfig file so kubectl can communicate with the EKS API server.

Once configured you can run:

    kubectl get nodes
    kubectl get pods -A

These commands interact with the control plane.

---

### 6. Hands-On: Creating an EKS Cluster (Control Plane)

The easiest way to create a cluster is using eksctl.

Install eksctl:

    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz

Move binary:

    sudo mv eksctl /usr/local/bin

Verify installation:

    eksctl version

Create a cluster:

    eksctl create cluster \
    --name production-cluster \
    --region ap-south-1

This command does several things automatically:

1. Creates a VPC
2. Creates subnets
3. Creates security groups
4. Creates an EKS control plane
5. Creates worker nodes

After cluster creation, run:

    kubectl get nodes

You should see nodes joining the cluster.

---

### 7. Worker Nodes: Where Containers Actually Run

Worker nodes are machines that run application containers.

They run several components that communicate with the control plane.

Important components on worker nodes:

kubelet  
kube-proxy  
container runtime (containerd)

kubelet communicates with the Kubernetes API server.

It ensures that containers defined in pods are running on the node.

kube-proxy manages networking rules so services can communicate with pods.

Worker nodes are usually EC2 instances in EKS.

---

### 8. Hands-On: Inspecting Worker Nodes

Once the cluster is created, inspect the nodes:

    kubectl get nodes

Example output:

    ip-192-168-10-45.ap-south-1.compute.internal
    ip-192-168-20-98.ap-south-1.compute.internal

Get detailed node information:

    kubectl describe node <node-name>

This command shows:

- CPU capacity
- memory capacity
- pods running on the node
- networking information

This is useful for debugging production workloads.

---

### 9. Managed Node Groups

Managed Node Groups are AWS-managed collections of worker nodes.

Without managed node groups, you would have to manually manage EC2 nodes.

Tasks you would normally manage manually include:

- node provisioning
- node scaling
- node upgrades
- node replacement
- auto scaling group configuration

Managed Node Groups automate these tasks.

AWS automatically:

- creates EC2 instances
- attaches them to the cluster
- manages lifecycle
- performs rolling updates

---

### 10. Hands-On: Creating Managed Node Groups

Create a node group with eksctl.

Example command:

    eksctl create nodegroup \
    --cluster production-cluster \
    --region ap-south-1 \
    --name application-nodes \
    --node-type t3.medium \
    --nodes 3 \
    --nodes-min 2 \
    --nodes-max 5

This creates an Auto Scaling Group with EC2 instances.

Verify nodes:

    kubectl get nodes

You should now see multiple nodes belonging to this node group.

---

### 11. Managed Node Group Architecture

Managed Node Groups internally use:

EC2 Instances  
Auto Scaling Groups  
Launch Templates  
IAM Roles

AWS ensures nodes remain healthy.

If a node fails:

Auto Scaling replaces it automatically.

In production environments, node groups are usually separated by purpose.

Example node groups:

frontend-nodes  
backend-nodes  
batch-processing-nodes

Each node group may have different instance types and scaling policies.

---

### 12. Advanced Node Group Strategies in Production

Production clusters often use multiple node groups.

Examples include:

Compute optimized nodes for CPU heavy workloads

Memory optimized nodes for large in-memory applications

GPU nodes for machine learning workloads

Example:

frontend-nodegroup → t3.medium  
backend-nodegroup → m5.large  
ml-nodegroup → g4dn.xlarge

Workloads are scheduled using node selectors and taints.

Example node selector:

    spec:
      nodeSelector:
        workload: backend

This ensures pods run only on specific nodes.

---

### 13. AWS Fargate in EKS

Fargate allows you to run Kubernetes pods without managing servers.

Instead of provisioning EC2 nodes, AWS directly runs pods on serverless infrastructure.

This means:

No node management  
No instance patching  
No capacity planning

You simply deploy pods.

AWS automatically allocates compute resources.

---

### 14. How Fargate Works Internally

With Fargate:

Pods run in isolated compute environments.

Each pod receives:

Dedicated CPU  
Dedicated memory  
Dedicated networking

Unlike EC2 nodes where many pods share a machine, Fargate pods run in fully isolated environments.

This improves security and simplifies operations.

---

### 15. Hands-On: Creating a Fargate Profile

First create a Fargate profile.

Example:

    eksctl create fargateprofile \
    --cluster production-cluster \
    --region ap-south-1 \
    --name fp-default \
    --namespace default

This tells EKS that pods in the default namespace should run on Fargate.

Deploy a sample application.

    kubectl create deployment nginx --image=nginx

Check pods:

    kubectl get pods -o wide

You will notice pods do not run on EC2 nodes.

Instead they run on Fargate infrastructure.

---

### 16. Comparing Worker Node Approaches

EC2 Worker Nodes

Advantages:

- Full control
- GPU support
- cheaper for large workloads
- flexible networking

Disadvantages:

- you manage node scaling
- you manage OS patching
- operational overhead

Managed Node Groups

Advantages:

- AWS handles lifecycle
- automatic upgrades
- integrated scaling

Disadvantages:

- still need to manage EC2 infrastructure

Fargate

Advantages:

- serverless
- zero node management
- strong isolation

Disadvantages:

- higher cost
- limited configuration
- some Kubernetes features unsupported

---

### 17. Real Production Architecture Example

A typical production EKS architecture may look like this:

Control Plane (AWS Managed)

Worker Layer

Managed Node Groups

- general workloads
- backend APIs

Fargate

- small microservices
- event driven workloads

Special Node Groups

- GPU workloads
- high memory workloads

This hybrid approach balances cost, performance, and operational complexity.
