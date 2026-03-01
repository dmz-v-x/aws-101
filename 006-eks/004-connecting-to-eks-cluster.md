## Connecting to EKS Cluster

### 1. Introduction

After creating an Amazon EKS cluster, the very first practical task is **connecting to the cluster so you can manage it**.

Until you connect to the cluster, you cannot:

- deploy applications
- inspect nodes
- create services
- manage namespaces
- debug workloads

In Kubernetes, cluster interaction happens through a command line tool called **kubectl**.

kubectl communicates with the Kubernetes API server using a configuration file called **kubeconfig**.

In Amazon EKS, there is an additional layer of authentication using **AWS IAM**, so understanding the complete workflow is critical.

This blog explains in detail how to connect your local machine or an EC2 instance to an EKS cluster and how kubectl interacts with it.

---

### 2. Understanding How kubectl Communicates with EKS

Before configuring anything, it is important to understand the communication architecture.

The workflow looks like this:

Local Machine / EC2 Instance  
↓  
kubectl command  
↓  
kubeconfig file  
↓  
AWS IAM authentication  
↓  
EKS API Server  
↓  
Cluster resources

The kubeconfig file tells kubectl:

- where the cluster API server is located
- how to authenticate
- which cluster context to use

Without this file, kubectl cannot talk to the cluster.

---

### 3. What is kubeconfig

kubeconfig is a configuration file used by kubectl to connect to Kubernetes clusters.

Default location:

    ~/.kube/config

This file contains:

Clusters  
Users  
Contexts  

Example structure:

    clusters:
    users:
    contexts:
    current-context:

Each section defines different aspects of cluster access.

This file allows a single kubectl client to interact with **multiple clusters**.

---

### 4. Components of kubeconfig

A kubeconfig file has three major sections.

Clusters

Defines cluster endpoints.

Example:

    cluster:
      server: https://ABCDE.gr7.ap-south-1.eks.amazonaws.com

Users

Defines authentication methods.

In EKS, authentication is done through AWS IAM tokens.

Contexts

Contexts connect users with clusters.

Example:

    context:
      cluster: production-cluster
      user: aws-user

Current Context

Defines which cluster kubectl should interact with by default.

---

### 5. Prerequisites Before Connecting to EKS

Several tools must be installed before connecting to the cluster.

AWS CLI

This allows interaction with AWS APIs.

Install AWS CLI:

    aws --version

kubectl

This is the Kubernetes CLI tool.

Check installation:

    kubectl version --client

IAM permissions

Your IAM user or role must have permission to access the EKS cluster.

Typical policy required:

    AmazonEKSClusterPolicy

Without proper permissions authentication will fail.

---

### 6. Getting kubeconfig for an EKS Cluster

AWS provides a command that automatically generates kubeconfig for EKS.

Command:

    aws eks update-kubeconfig \
    --region ap-south-1 \
    --name production-cluster

This command does several things:

1. Fetches cluster endpoint
2. Fetches cluster certificate authority
3. Updates kubeconfig file
4. Creates a new context

After running this command your machine can communicate with the cluster.

---

### 7. Understanding What update-kubeconfig Actually Does

This command queries the EKS API and retrieves:

Cluster endpoint URL  
Certificate authority data  

It then writes this information into:

    ~/.kube/config

Example entry:

    clusters:
    - cluster:
        server: https://ABCDE.eks.amazonaws.com
      name: production-cluster

It also configures IAM authentication using the AWS CLI.

This allows kubectl to generate authentication tokens dynamically.

---

### 8. Verifying Cluster Connectivity

After kubeconfig is configured, verify connectivity.

Check nodes:

    kubectl get nodes

Expected output:

    ip-192-168-10-45
    ip-192-168-20-67

If nodes appear, the connection is successful.

Check cluster information:

    kubectl cluster-info

Example output:

    Kubernetes control plane is running at https://ABCDE.eks.amazonaws.com

---

### 9. Understanding kubectl Contexts

kubectl supports multiple cluster connections.

Contexts help manage these connections.

A context defines:

Cluster  
User  
Namespace  

List contexts:

    kubectl config get-contexts

Example output:

    CURRENT   NAME
    *         production-cluster
              staging-cluster

The star indicates the active context.

---

### 10. Switching Between Contexts

If you work with multiple clusters, you may need to switch contexts.

Command:

    kubectl config use-context staging-cluster

Verify active context:

    kubectl config current-context

Example output:

    staging-cluster

This ensures kubectl commands target the correct cluster.

---

### 11. Inspecting kubeconfig

To view the entire kubeconfig file:

    kubectl config view

To view only cluster names:

    kubectl config get-clusters

To view contexts:

    kubectl config get-contexts

This is useful when debugging configuration issues.

---

### 12. Connecting from an EC2 Instance

Sometimes you want to manage the cluster from a **bastion host or DevOps EC2 machine** instead of your laptop.

Steps:

Install AWS CLI

    sudo apt install awscli

Install kubectl

    curl -LO https://dl.k8s.io/release/v1.29.0/bin/linux/amd64/kubectl

Make executable:

    chmod +x kubectl

Move binary:

    sudo mv kubectl /usr/local/bin

Configure AWS credentials:

    aws configure

Update kubeconfig:

    aws eks update-kubeconfig \
    --region ap-south-1 \
    --name production-cluster

Verify:

    kubectl get nodes

---

### 13. Understanding Kubernetes Namespaces

Namespaces divide cluster resources into logical groups.

They allow multiple teams or environments to share a single cluster.

Example namespaces:

default  
kube-system  
dev  
staging  
production  

Each namespace has isolated resources.

---

### 14. Listing Namespaces

Command:

    kubectl get namespaces

Example output:

    default
    kube-system
    kube-public

The kube-system namespace contains core Kubernetes services.

---

### 15. Creating Namespaces

Create a namespace:

    kubectl create namespace dev

Verify:

    kubectl get namespaces

This is commonly used to separate environments.

Example:

dev namespace for development workloads

production namespace for live applications

---

### 16. Running Commands in Specific Namespace

By default kubectl uses the **default namespace**.

To query another namespace:

    kubectl get pods -n dev

Example output:

    nginx-pod

To create resources in that namespace:

    kubectl apply -f deployment.yaml -n dev

---

### 17. Setting Default Namespace for Context

Instead of specifying namespace every time, you can set a default namespace.

Command:

    kubectl config set-context \
    --current \
    --namespace=dev

Now all kubectl commands operate inside the dev namespace.

Verify:

    kubectl config view --minify

---

### 18. Common Connectivity Issues

Authentication Failure

Error example:

    You must be logged in to the server

Cause

IAM user is not mapped in the cluster.

Solution

Add user to aws-auth ConfigMap.

---

### 19. Expired Authentication Token

EKS uses temporary tokens.

If authentication expires, rerun:

    aws eks update-kubeconfig

This refreshes credentials.

---

### 20. Networking Issues

If kubectl cannot reach the API server, possible causes include:

Private cluster configuration  
Security group restrictions  
Corporate firewall blocking access

In private clusters, access typically happens through:

Bastion host  
VPN  
Direct Connect

---

### 21. Useful kubectl Commands for Cluster Exploration

List nodes:

    kubectl get nodes

List pods:

    kubectl get pods -A

List services:

    kubectl get svc -A

Describe node:

    kubectl describe node <node-name>

These commands help verify that the cluster is functioning properly.

---

### 22. Production Best Practices

Do not store kubeconfig files in public repositories.

Use IAM roles instead of long-term AWS credentials.

Use separate namespaces for different environments.

Restrict cluster access using RBAC.

Use bastion hosts for private clusters.
