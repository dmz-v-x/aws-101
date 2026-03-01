## Cluster Deletion and Cleanup in Amazon EKS (Cost Control) — From Absolute Zero to Advanced

### 1. Introduction

Running Kubernetes clusters in the cloud is powerful but also expensive if resources are left running unnecessarily. One of the most common mistakes beginners make when working with Amazon EKS is forgetting to delete clusters and their associated infrastructure after experiments or temporary workloads. Because EKS clusters create multiple AWS resources automatically, leaving them running can lead to unexpected charges.

Cluster deletion and cleanup is therefore an essential operational skill. It is not just about deleting the cluster itself. A proper cleanup process ensures that all supporting resources such as load balancers, node groups, storage volumes, networking components, and IAM roles are removed so that no hidden costs remain.

### 2. Why Cluster Deletion and Cleanup Matters

Amazon EKS clusters rely on many AWS resources. Even if the cluster is no longer being used, these resources may still continue running in the background and incur charges.

Examples of costs that can continue after a cluster is forgotten include:

- EKS control plane hourly cost
- EC2 instances used as worker nodes
- Elastic Load Balancers created by Kubernetes services
- Elastic Block Store volumes attached to pods
- NAT Gateways used for private networking
- Elastic IP addresses
- Data transfer costs

If these resources are not deleted properly, the AWS bill will keep increasing even though the cluster is not actively being used.

Proper cleanup ensures:

- no orphaned resources remain
- infrastructure costs stop immediately
- the AWS account remains organized
- future deployments avoid conflicts with old infrastructure

### 3. What Exactly Gets Created When You Create an EKS Cluster

Before learning how to delete a cluster, it is important to understand what components exist inside an EKS environment.

An EKS cluster usually consists of several layers of infrastructure.

Cluster control plane

The EKS control plane is managed by AWS and includes components like the Kubernetes API server and etcd. AWS runs these components in their own infrastructure and charges a fixed hourly fee for maintaining the cluster.

Worker nodes

Worker nodes are EC2 instances that run containers. These nodes host the actual application workloads and are usually grouped into node groups.

Networking components

EKS clusters run inside a Virtual Private Cloud (VPC). The networking layer may include:

- public and private subnets
- route tables
- NAT gateways
- internet gateways
- security groups

Load balancers

When Kubernetes services expose applications externally, AWS automatically creates load balancers such as:

- Application Load Balancer
- Network Load Balancer

Storage resources

Applications that require persistent storage may create resources such as:

- EBS volumes
- EFS file systems

IAM roles and policies

EKS uses IAM roles for:

- cluster control plane permissions
- worker node permissions
- service account access (IRSA)

Because all these resources are interconnected, deleting the cluster must be done carefully so that no dependent resources remain.

### 4. Types of Resources That Must Be Cleaned Up

A full cleanup typically involves removing the following resources.

Compute resources

- worker node EC2 instances
- Auto Scaling groups
- launch templates

Networking resources

- VPC components
- security groups
- elastic network interfaces

Storage resources

- persistent volumes
- EBS volumes
- snapshots

Load balancers

- ALB or NLB created by Kubernetes services
- target groups

Identity resources

- IAM roles
- IAM policies

Kubernetes resources

- namespaces
- deployments
- services
- ingress objects

Understanding this ecosystem helps prevent incomplete deletion.

### 5. When Should You Delete an EKS Cluster

Cluster deletion is usually required in the following situations.

Temporary testing environments

Engineers often create clusters to test applications or infrastructure configurations. Once testing is complete, the cluster should be removed.

CI/CD environments

Some organizations create clusters dynamically during automated testing pipelines and delete them afterward.

Training environments

When learning Kubernetes or EKS, clusters may only be needed for a few hours or days.

Migration or infrastructure redesign

Sometimes clusters are replaced by new clusters during architecture changes.

Proper cleanup after these activities prevents unnecessary costs.

### 6. Pre-Deletion Checklist

Before deleting a cluster, it is important to verify a few conditions to avoid accidental data loss.

Check running workloads

Ensure that no important applications are currently running.

Backup persistent data

If the cluster uses persistent storage such as EBS volumes, take backups before deletion.

Export Kubernetes manifests

Save deployment YAML files or Helm charts so the application can be redeployed later.

Delete external resources

If services created load balancers or external DNS records, remove them before deleting the cluster.

Confirm node group status

Ensure that node groups are healthy and manageable before initiating deletion.

### 7. Deleting an EKS Cluster Using eksctl

The easiest way to delete an EKS cluster created using eksctl is to use the same tool for deletion.

The command looks like this:

    eksctl delete cluster --name my-cluster --region us-east-1

When this command is executed, eksctl performs several cleanup tasks automatically.

These tasks include:

- deleting node groups
- removing CloudFormation stacks
- deleting associated networking components
- removing the EKS cluster

Eksctl internally tracks the infrastructure resources created during cluster creation and deletes them in the correct order.

### 8. Deleting an EKS Cluster Using AWS CLI

Clusters can also be deleted using the AWS CLI.

Example command:

    aws eks delete-cluster --name my-cluster

However, this command only deletes the control plane. Worker nodes and other resources must be deleted separately.

Therefore the AWS CLI approach often requires manual cleanup of additional resources.

### 9. Deleting Node Groups

Before removing the cluster control plane, worker nodes must be removed.

Node groups can be deleted using:

    aws eks delete-nodegroup --cluster-name my-cluster --nodegroup-name my-nodegroup

Deleting node groups removes the EC2 instances that host Kubernetes workloads.

If node groups are not deleted first, the cluster deletion process may fail.

### 10. Cleaning Up Load Balancers

Kubernetes services with type LoadBalancer automatically create AWS load balancers.

These load balancers may remain active even after applications are removed.

To identify them:

- check the EC2 load balancer dashboard
- inspect Kubernetes service resources

If orphaned load balancers exist, delete them manually.

### 11. Cleaning Up Persistent Volumes

Persistent volumes backed by EBS may remain even after pods are deleted.

You should inspect the EBS console and verify whether any unused volumes remain.

Unused volumes should be deleted to avoid storage charges.

### 12. Cleaning Up Elastic IPs

Elastic IP addresses are often used by NAT gateways or load balancers.

If they remain allocated after cluster deletion, they may incur charges.

Always verify and release unused Elastic IPs.

### 13. Cleaning Up VPC Resources

If the cluster was created with a dedicated VPC, the VPC itself may still exist after the cluster is deleted.

Check for leftover components such as:

- subnets
- route tables
- internet gateways
- NAT gateways

Deleting unused networking infrastructure prevents unnecessary costs.

### 14. Monitoring Costs After Deletion

After cluster deletion, it is good practice to monitor AWS billing dashboards.

Useful tools include:

- AWS Cost Explorer
- AWS Billing Dashboard
- AWS Budgets

These tools help verify that no unexpected charges remain.

### 15. Best Practices for Cost Control

To prevent unnecessary costs, engineers should follow several best practices.

Use infrastructure automation

Tools like Terraform or eksctl help track resources and ensure clean deletion.

Implement environment lifecycles

Temporary environments should automatically expire after a defined period.

Monitor unused resources

Regular audits help detect orphaned infrastructure.

Use tagging

Tagging AWS resources helps identify which resources belong to which clusters.

Schedule cluster shutdowns

Non-production clusters can be automatically stopped or deleted during idle periods.

### 16. Common Mistakes During Cluster Cleanup

Engineers often encounter the following mistakes.

Forgetting NAT gateways

NAT gateways are expensive and often remain after clusters are removed.

Leaving persistent volumes

Unused EBS volumes may continue accumulating storage costs.

Ignoring load balancers

Load balancers created by Kubernetes services may remain active.

Deleting clusters without backing up data

This may cause permanent data loss.

Proper procedures prevent these issues.

### 17. Advanced Automation for Cluster Cleanup

In large organizations, cleanup is often automated.

Examples include:

- scheduled cluster deletion pipelines
- infrastructure lifecycle automation
- GitOps-based cluster management
- automated cost monitoring

These systems ensure that clusters are created and destroyed safely and consistently.
