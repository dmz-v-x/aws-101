## Different ways to create EKS Cluster

### 1. Introduction

Amazon Elastic Kubernetes Service (EKS) is AWS’s managed Kubernetes offering. One of the first practical decisions you must make when working with EKS is **how to create and manage the cluster infrastructure**.

AWS provides multiple ways to create EKS clusters, each designed for different use cases and levels of automation.

The main methods used in real environments are:

- AWS Management Console
- eksctl CLI
- AWS CloudFormation
- Terraform
- AWS CDK

Each of these methods represents a different **infrastructure management philosophy**.

Some are manual and useful for learning, while others are used for **production Infrastructure as Code (IaC)** workflows.

In this blog we will start from the most basic method and gradually move toward advanced infrastructure automation used in production systems.

---

### 2. Understanding the Resources Required for an EKS Cluster

Before discussing the creation methods, it is important to understand **what resources are actually created when you build an EKS cluster**.

Creating an EKS cluster is not a single resource operation. Multiple AWS services are involved.

Typical resources include:

- VPC
- Subnets
- Route tables
- Internet gateway
- NAT gateway
- Security groups
- IAM roles
- EKS control plane
- Node groups
- Auto Scaling Groups
- EC2 instances

Because of this complexity, different tools exist to simplify the process.

Some tools create these resources automatically, while others require you to define them manually.

---

### 3. Method 1 — Creating EKS Clusters Using AWS Management Console

The **AWS Management Console** is the graphical interface provided by AWS.

It is often the first method beginners use when learning EKS.

This method involves manually configuring settings through the browser.

---

### 4. Step-by-Step: Creating an EKS Cluster Using the Console

Step 1: Open the AWS Console

Navigate to:

Services → EKS → Create Cluster

Step 2: Provide Cluster Details

Cluster name:

```
production-cluster
```

Kubernetes version:

```
1.29
```

Step 3: Configure IAM Role

EKS requires a role that allows it to manage AWS resources.

Create a role such as:

```
AmazonEKSClusterRole
```

Step 4: Configure Networking

Select:

- VPC
- Subnets
- Security groups

These determine how the cluster communicates with other services.

Step 5: Create the Cluster

Click **Create**.

AWS now provisions the control plane.

This usually takes **10–15 minutes**.

---

### 5. Creating Worker Nodes from Console

Once the control plane is ready, worker nodes must be created.

Go to:

Compute → Add Node Group

Configure:

Node group name

```
general-node-group
```

Instance type

```
t3.medium
```

Scaling configuration

```
desired: 2
min: 1
max: 4
```

AWS creates an Auto Scaling Group with EC2 instances.

Once complete:

```
kubectl get nodes
```

You will see the worker nodes registered.

---

### 6. Pros and Cons of Console-Based Cluster Creation

Advantages

Easy for beginners

Good for experimentation

Visual configuration

No tooling required

Disadvantages

Manual and not reproducible

Hard to track infrastructure changes

Difficult to automate

Not suitable for production environments

Console-based creation is generally **not used in production infrastructure workflows**.

---

### 7. Method 2 — Creating EKS Clusters Using eksctl

eksctl is a CLI tool specifically designed for creating and managing EKS clusters.

It was created by Weaveworks and is widely used by developers.

eksctl simplifies cluster creation by automatically provisioning the required infrastructure.

---

### 8. Installing eksctl

Linux installation:

```
curl --silent --location \
"https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
| tar xz
```

Move binary:

```
sudo mv eksctl /usr/local/bin
```

Verify installation:

```
eksctl version
```

---

### 9. Creating an EKS Cluster Using eksctl

A single command can create the entire cluster infrastructure.

```
eksctl create cluster \
--name production-cluster \
--region ap-south-1 \
--nodegroup-name general-nodes \
--node-type t3.medium \
--nodes 2
```

This command automatically creates:

- VPC
- Subnets
- EKS control plane
- IAM roles
- Managed node groups
- Security groups

Once the cluster is created:

```
kubectl get nodes
```

The nodes should appear.

---

### 10. Using eksctl Configuration Files

For better reproducibility, eksctl supports configuration files.

Example:

```
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: production-cluster
  region: ap-south-1

nodeGroups:
  - name: general-nodes
    instanceType: t3.medium
    desiredCapacity: 2
```

Create cluster:

```
eksctl create cluster -f cluster.yaml
```

This approach allows cluster infrastructure to be stored in version control.

---

### 11. Pros and Cons of eksctl

Advantages

Very simple cluster creation

Automatically manages infrastructure

Good for development environments

Great for quick testing

Disadvantages

Limited customization compared to full IaC tools

Not ideal for very complex infrastructure setups

Harder to integrate with enterprise infrastructure pipelines

eksctl is widely used for **development clusters and experimentation**.

---

### 12. Method 3 — Creating EKS Clusters Using AWS CloudFormation

CloudFormation is AWS’s native **Infrastructure as Code service**.

Infrastructure is defined using JSON or YAML templates.

CloudFormation then provisions the resources.

Example resources created through CloudFormation:

- VPC
- Subnets
- EKS cluster
- Node groups
- IAM roles

---

### 13. Example CloudFormation Template

Basic cluster definition:

```
Resources:

  EKSCluster:
    Type: AWS::EKS::Cluster
    Properties:
      Name: production-cluster
      Version: "1.29"
      RoleArn: arn:aws:iam::123456789012:role/EKSClusterRole
```

Deploy the stack:

```
aws cloudformation deploy \
--template-file cluster.yaml \
--stack-name eks-cluster
```

CloudFormation creates the defined resources.

---

### 14. Pros and Cons of CloudFormation

Advantages

Native AWS solution

Deep AWS integration

Supports large infrastructure stacks

Change sets allow preview of infrastructure changes

Disadvantages

Templates can become complex

Difficult to maintain large configurations

Less flexible compared to Terraform

CloudFormation is widely used in **AWS-native enterprise environments**.

---

### 15. Method 4 — Creating EKS Clusters Using Terraform

Terraform is a popular multi-cloud Infrastructure as Code tool.

It allows infrastructure to be described declaratively using HCL (HashiCorp Configuration Language).

Terraform is widely used in production environments.

---

### 16. Example Terraform Configuration

Provider configuration:

```
provider "aws" {
  region = "ap-south-1"
}
```

EKS cluster resource:

```
resource "aws_eks_cluster" "cluster" {
  name     = "production-cluster"
  role_arn = aws_iam_role.eks_role.arn

  vpc_config {
    subnet_ids = var.subnet_ids
  }
}
```

Initialize Terraform:

```
terraform init
```

Plan infrastructure:

```
terraform plan
```

Apply infrastructure:

```
terraform apply
```

Terraform then provisions the cluster.

---

### 17. Pros and Cons of Terraform

Advantages

Cloud agnostic

Strong community modules

Highly modular

Excellent for large infrastructure environments

Easy state management

Disadvantages

Requires Terraform state management

Extra tooling required

Not native to AWS

Terraform is extremely popular for **production infrastructure automation**.

---

### 18. Method 5 — Creating EKS Clusters Using AWS CDK

AWS CDK (Cloud Development Kit) allows infrastructure to be defined using programming languages.

Supported languages include:

- TypeScript
- Python
- Java
- C#
- Go

Instead of writing YAML or HCL, you write infrastructure using code.

---

### 19. Example CDK EKS Cluster

Example using TypeScript:

```
const cluster = new eks.Cluster(this, 'MyCluster', {
  version: eks.KubernetesVersion.V1_29,
  defaultCapacity: 2
});
```

Deploy infrastructure:

```
cdk deploy
```

CDK internally generates CloudFormation templates and deploys them.

---

### 20. Pros and Cons of AWS CDK

Advantages

Infrastructure defined using real programming languages

Reusable constructs

Better abstraction compared to raw CloudFormation

Powerful automation capabilities

Disadvantages

Steeper learning curve

Still dependent on CloudFormation

Not as widely adopted as Terraform in multi-cloud environments

CDK is commonly used in **modern AWS-native infrastructure systems**.

---

### 21. Comparing All EKS Creation Methods

Console

Best for:

Learning and manual experimentation

Automation level:

None

eksctl

Best for:

Developers and quick cluster creation

Automation level:

Moderate

CloudFormation

Best for:

AWS-native infrastructure teams

Automation level:

High

Terraform

Best for:

Enterprise production infrastructure

Automation level:

Very high

CDK

Best for:

Developers who prefer programming languages for infrastructure

Automation level:

Very high

---

### 22. Real Production Recommendation

In real-world environments:

Console is rarely used.

Common production approaches include:

Terraform for infrastructure management

CloudFormation for AWS-native environments

CDK for application-driven infrastructure

eksctl is mostly used for development clusters or experimentation.
