## `eksctl` deep dive

### 1. Introduction

eksctl is the official command line tool used to create and manage Amazon EKS clusters. It is developed by Weaveworks and is widely used by developers and DevOps engineers because it simplifies many complex steps required to provision EKS infrastructure.

Without eksctl, creating an EKS cluster requires manually configuring multiple AWS resources such as VPCs, subnets, IAM roles, security groups, node groups, and autoscaling groups. eksctl automates these tasks and provides a Kubernetes-focused interface to manage EKS clusters efficiently.

This blog provides a deep dive into eksctl starting from installation, understanding how it works internally, exploring its configuration schema, and performing advanced cluster creation with custom networking, node groups, and SSH access.

---

### 2. What eksctl Actually Does Internally

Before using eksctl it is important to understand what it does behind the scenes.

When you run an eksctl command to create a cluster, the tool internally generates and deploys AWS CloudFormation stacks.

These stacks create resources such as:

VPC  
Subnets  
Route Tables  
Internet Gateway  
NAT Gateway  
Security Groups  
IAM Roles  
EKS Control Plane  
Node Groups  
Auto Scaling Groups  

eksctl essentially acts as a higher level abstraction on top of CloudFormation and AWS APIs.

Because of this architecture:

- eksctl operations are idempotent
- infrastructure state is tracked through CloudFormation stacks
- rollback mechanisms are available if deployment fails

---

### 3. Installing eksctl

eksctl can be installed on Linux, macOS, and Windows.

Linux installation:

    curl --silent --location \
    "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
    | tar xz -C /tmp

Move the binary:

    sudo mv /tmp/eksctl /usr/local/bin

Verify installation:

    eksctl version

Example output:

    0.174.0

This confirms that eksctl is installed correctly.

---

### 4. Required Prerequisites Before Using eksctl

eksctl relies on several dependencies.

You must have the following configured:

AWS CLI

    aws configure

kubectl

    kubectl version --client

IAM permissions that allow:

- EKS creation
- VPC creation
- EC2 provisioning
- IAM role management

Typical IAM policies required include:

AmazonEKSClusterPolicy  
AmazonEKSServicePolicy  
AmazonEC2FullAccess  

Without these permissions cluster creation will fail.

---

### 5. Understanding eksctl Command Structure

All eksctl commands follow a consistent structure.

Basic format:

    eksctl <resource> <action>

Examples:

    eksctl create cluster
    eksctl delete cluster
    eksctl get cluster
    eksctl create nodegroup
    eksctl delete nodegroup

The most common command categories include:

Cluster commands  
Node group commands  
IAM commands  
Fargate commands  
Addon commands  
Utility commands

---

### 6. eksctl Basic Help Commands

To see all commands:

    eksctl help

To see help for cluster commands:

    eksctl create cluster --help

To see help for node groups:

    eksctl create nodegroup --help

These commands list all available flags and options.

---

### 7. Creating a Basic EKS Cluster Using eksctl

The simplest command to create a cluster is:

    eksctl create cluster

This command automatically creates:

Default VPC  
Two node worker instances  
Managed node group  
Security groups  
IAM roles  

To specify cluster name and region:

    eksctl create cluster \
    --name production-cluster \
    --region ap-south-1

Once completed you can verify the cluster:

    kubectl get nodes

---

### 8. Important Flags for Cluster Creation

Several flags allow customization of cluster creation.

Cluster name

    --name

Example:

    --name production-cluster

Region

    --region

Example:

    --region ap-south-1

Kubernetes version

    --version

Example:

    --version 1.29

Node instance type

    --node-type

Example:

    --node-type t3.medium

Node count

    --nodes

Example:

    --nodes 3

Minimum nodes

    --nodes-min

Maximum nodes

    --nodes-max

Example:

    --nodes-min 2 --nodes-max 5

---

### 9. Creating Clusters with SSH Access

To enable SSH access to worker nodes you must provide an SSH key.

Example command:

    eksctl create cluster \
    --name production-cluster \
    --region ap-south-1 \
    --node-type t3.medium \
    --nodes 2 \
    --ssh-access \
    --ssh-public-key my-keypair

Where my-keypair refers to an existing EC2 key pair.

This allows SSH access to worker nodes:

    ssh ec2-user@node-ip

---

### 10. Creating Clusters with Custom VPC

By default eksctl creates a new VPC automatically.

However production environments typically use existing VPCs.

Example:

    eksctl create cluster \
    --name production-cluster \
    --region ap-south-1 \
    --vpc-private-subnets subnet-12345,subnet-67890 \
    --vpc-public-subnets subnet-11111,subnet-22222

This attaches the cluster to an existing VPC architecture.

---

### 11. eksctl Configuration Files

For reproducible infrastructure it is better to use configuration files instead of CLI flags.

These configuration files are written in YAML.

Example file structure:

    apiVersion: eksctl.io/v1alpha5
    kind: ClusterConfig

    metadata:
      name: production-cluster
      region: ap-south-1

    nodeGroups:
      - name: general-nodes
        instanceType: t3.medium
        desiredCapacity: 2

This file describes the entire cluster configuration.

---

### 12. Creating Cluster Using Configuration File

Once the configuration file is created:

    eksctl create cluster -f cluster.yaml

eksctl reads the configuration file and deploys the infrastructure accordingly.

This method is preferred because it allows:

Version control  
Team collaboration  
Reproducible environments

---

### 13. Full Configuration Schema Overview

An eksctl configuration file typically includes the following sections.

metadata

Cluster identity information.

vpc

Networking configuration.

nodeGroups

Worker node configuration.

managedNodeGroups

AWS-managed node groups.

fargateProfiles

Serverless pod execution configuration.

iam

IAM roles and service accounts.

addons

Cluster addons.

---

### 14. Metadata Section

Example:

    metadata:
      name: production-cluster
      region: ap-south-1
      version: "1.29"

Fields include:

name  
region  
version  
tags

These values define the cluster identity.

---

### 15. VPC Configuration

Example:

    vpc:
      id: vpc-12345
      subnets:
        private:
          ap-south-1a:
            id: subnet-111
          ap-south-1b:
            id: subnet-222

This allows attaching EKS to existing networking infrastructure.

---

### 16. Managed Node Groups Configuration

Example:

    managedNodeGroups:
      - name: application-nodes
        instanceType: t3.medium
        desiredCapacity: 3
        minSize: 2
        maxSize: 5

Managed node groups automatically handle node lifecycle operations.

---

### 17. SSH Configuration in Node Groups

Example:

    managedNodeGroups:
      - name: app-nodes
        instanceType: t3.medium
        desiredCapacity: 2
        ssh:
          allow: true
          publicKeyName: my-keypair

This enables SSH access to nodes.

---

### 18. Fargate Profiles

Example:

    fargateProfiles:
      - name: default
        selectors:
          - namespace: default

Pods in the default namespace will run on Fargate.

---

### 19. IAM Service Accounts

Example:

    iam:
      withOIDC: true

This enables IAM Roles for Service Accounts.

It allows Kubernetes services to access AWS resources securely.

---

### 20. Managing Clusters with eksctl

List clusters:

    eksctl get cluster

Delete cluster:

    eksctl delete cluster --name production-cluster

Describe cluster:

    eksctl utils describe-stacks --cluster production-cluster

---

### 21. Managing Node Groups

Create node group:

    eksctl create nodegroup \
    --cluster production-cluster \
    --name backend-nodes \
    --node-type t3.large \
    --nodes 3

Delete node group:

    eksctl delete nodegroup \
    --cluster production-cluster \
    --name backend-nodes

List node groups:

    eksctl get nodegroup --cluster production-cluster

---

### 22. Advanced Cluster Utilities

Update kubeconfig:

    aws eks update-kubeconfig \
    --region ap-south-1 \
    --name production-cluster

Scale node groups:

    eksctl scale nodegroup \
    --cluster production-cluster \
    --name backend-nodes \
    --nodes 5

---

### 23. Real Production Workflow

Typical production workflow using eksctl:

Step 1

Define cluster configuration YAML.

Step 2

Store configuration in version control.

Step 3

Create cluster using configuration file.

Step 4

Manage node groups separately.

Step 5

Deploy workloads using kubectl or CI/CD pipelines.
