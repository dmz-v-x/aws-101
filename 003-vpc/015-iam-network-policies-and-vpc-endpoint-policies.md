## IAM Network-Related Policies and VPC Endpoint Policies — Applying the Principle of Least Privilege to Network Resources

In AWS, security is enforced at multiple layers:

```
Network Layer → VPC, Security Groups, NACLs
Identity Layer → IAM policies
Application Layer → Authentication and authorization
```

While networking controls **who can connect to a resource**, IAM controls **who can manage or interact with AWS services and infrastructure**.

For secure cloud architecture, both networking controls and IAM policies must follow the **principle of least privilege**.


---

### 1. What Is IAM?

IAM stands for **Identity and Access Management**.

IAM is the AWS service used to control:

```
who can access AWS resources
what actions they can perform
which resources they can access
```

IAM manages:

```
Users
Groups
Roles
Policies
```

Example IAM user:

```
DevOps Engineer
```

Example permissions:

```
Create EC2
Modify VPC
Describe subnets
Delete security groups
```

---

### 2. What Is an IAM Policy?

An **IAM policy** is a JSON document that defines permissions.

Definition:

> An IAM policy defines what actions are allowed or denied on specific AWS resources.

Example IAM policy structure:

```
Effect
Action
Resource
Condition (optional)
```

Example policy:

```
Allow EC2 read-only access
```

Example JSON:

```
{
 "Effect": "Allow",
 "Action": "ec2:DescribeInstances",
 "Resource": "*"
}
```

---

### 3. Network-Related IAM Permissions

Many AWS networking components are managed using IAM permissions.

Examples include:

```
Create VPC
Create subnets
Modify route tables
Attach internet gateway
Create security groups
Delete NAT gateways
```

Example actions:

```
ec2:CreateVpc
ec2:CreateSubnet
ec2:CreateRouteTable
ec2:CreateSecurityGroup
ec2:CreateInternetGateway
```

These permissions determine **who can manage network infrastructure**.

---

### 4. Example Networking IAM Policy

Example policy allowing VPC read access:

```
{
 "Effect": "Allow",
 "Action": [
   "ec2:DescribeVpcs",
   "ec2:DescribeSubnets",
   "ec2:DescribeRouteTables"
 ],
 "Resource": "*"
}
```

This allows users to **view networking resources but not modify them**.

---

### 5. Principle of Least Privilege

The **principle of least privilege (PoLP)** means:

> Grant only the permissions necessary to perform a task.

Do not give excessive permissions.

Bad example:

```
AdministratorAccess
```

This allows:

```
Delete VPC
Modify security groups
Terminate instances
```

Good example:

```
Allow only EC2 describe permissions
```

This reduces the risk of accidental or malicious changes.

---

### 6. Example Least Privilege Policy

Example DevOps role policy:

```
Allow:
Create EC2
Describe VPC
Describe subnets

Deny:
Delete VPC
Delete route tables
Modify NACL
```

This prevents destructive network actions.

---

### 7. VPC Endpoints Overview

Normally, when instances access AWS services, traffic goes through the **public internet**.

Example:

```
EC2 → Internet → S3
```

To improve security, AWS provides **VPC endpoints**.

Definition:

> A VPC endpoint allows private connections between a VPC and AWS services without using the public internet.

Example:

```
EC2 → VPC Endpoint → S3
```

Traffic stays inside the AWS network.

---

### 8. Types of VPC Endpoints

AWS provides two primary endpoint types.

```
Gateway Endpoints
Interface Endpoints
```

Gateway endpoints are used for:

```
S3
DynamoDB
```

Interface endpoints are used for many AWS services.

Example:

```
SSM
Secrets Manager
CloudWatch
```

---

### 9. What Is an Endpoint Policy?

A **VPC endpoint policy** controls which actions can be performed through the endpoint.

Definition:

> An endpoint policy is an IAM-style policy attached to a VPC endpoint that controls access to the target service.

Example:

```
Restrict S3 access through endpoint
```

---

### 10. Example Endpoint Policy

Example policy allowing access only to a specific S3 bucket.

```
{
 "Effect": "Allow",
 "Principal": "*",
 "Action": [
   "s3:GetObject",
   "s3:PutObject"
 ],
 "Resource": "arn:aws:s3:::company-data/*"
}
```

This means:

```
Only this bucket is accessible through the endpoint
```

All other S3 buckets are blocked.

---

### 11. Endpoint Policy Use Case

Suppose your VPC contains application servers.

Architecture:

```
Private Subnet
   |
EC2 Instance
   |
VPC Endpoint
   |
S3 Bucket
```

You want to restrict access so that:

```
EC2 instances can only access company backup bucket
```

Endpoint policy enforces this rule.

---

### 12. Combining IAM and Endpoint Policies

Security is strongest when multiple layers work together.

Example:

```
IAM Role → controls who can access S3
Endpoint Policy → controls what resources can be accessed
S3 Bucket Policy → controls bucket-level permissions
```

All three must allow the request.

---

### 13. Example Request Flow

Application server tries to access S3 object.

```
EC2 → VPC Endpoint → S3
```

Checks performed:

```
IAM policy
Endpoint policy
Bucket policy
```

If any deny the request:

```
Access denied
```

---

### 14. Enterprise Security Architecture

Example secure architecture:

```
Private Subnet
   |
Application Servers
   |
VPC Endpoint
   |
S3 Service
```

Security controls:

```
IAM Role
Endpoint Policy
Bucket Policy
```

This ensures **least privilege access**.

---

### 15. IAM Roles for EC2

Best practice is to use **IAM roles** instead of access keys.

Example:

```
EC2 Instance
   |
IAM Role
   |
Permissions
```

This allows secure service access without storing credentials.

---

### 16. Example IAM Role for S3 Access

Example policy:

```
Allow:
s3:GetObject
s3:PutObject

Resource:
arn:aws:s3:::company-data/*
```

Instances can access only this bucket.

---

### 17. Security Best Practices

Best practices include:

```
Use least privilege policies
Avoid wildcard permissions
Use IAM roles instead of keys
Use endpoint policies to restrict access
Monitor permissions with CloudTrail
```

These reduce security risks.

---

### 18. Example Secure Architecture

```
Internet
   |
WAF
   |
Load Balancer
   |
Private Subnet
   |
Application Servers
   |
VPC Endpoint
   |
S3 Storage
```

Access controls:

```
Security Groups
IAM Roles
Endpoint Policies
```

---

### 19. Mental Model

Think of access control like **multiple security checkpoints**.

```
Identity Check → IAM policy
Network Gate → VPC endpoint
Resource Lock → Service policy
```

All checks must pass.

---

### 20. Key Concepts Summary

| Concept | Meaning |
|------|------|
| IAM | Identity and access management |
| IAM Policy | Defines permissions |
| Least Privilege | Grant minimal permissions |
| VPC Endpoint | Private service access |
| Endpoint Policy | Restricts endpoint access |

---

### 21. Checkpoint Questions

You should now be able to answer these.

#### What is IAM?

A service that controls who can access AWS resources and what actions they can perform.

---

#### What is the principle of least privilege?

Granting only the permissions necessary to perform a task.

---

#### What is a VPC endpoint?

A private connection between a VPC and AWS services.

---

#### What does an endpoint policy do?

It restricts which actions and resources are accessible through a VPC endpoint.

---

#### Why combine IAM and endpoint policies?

To enforce stronger security using multiple access control layers.

