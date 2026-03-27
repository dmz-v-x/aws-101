### 1. Introduction

Applications running inside Kubernetes often need to access AWS services such as:

S3 (for storing files)  
DynamoDB (for databases)  
SQS (for messaging)  
Secrets Manager (for secrets)  

The question is:

**How do pods securely access AWS services?**

There are three approaches:

Hardcoding AWS credentials (bad practice)  
Using node IAM roles (older approach)  
Using IAM Roles for Service Accounts (IRSA) (recommended)

IRSA is the **modern, secure, production-grade approach** used in Amazon EKS.

This guide explains IRSA from absolute basics to advanced usage, including why it is safer than node IAM roles and how to set it up step by step.

---

### 2. The Core Problem

Consider this scenario.

You have a pod running an application:

    app → needs access to S3 bucket

How does the pod authenticate with AWS?

Possible naive solution:

Store AWS access keys inside the container.

Problems:

Credentials can leak  
Hard to rotate  
Security risk  

We need a **secure, dynamic authentication mechanism**.

---

### 3. Old Approach: Node IAM Roles

Before IRSA, the common approach was:

Assign an IAM role to EC2 worker nodes.

Example:

EC2 instance role:

    AmazonS3FullAccess

Pods running on that node automatically inherit permissions.

---

### 4. Why Node IAM Roles Are Dangerous

Node IAM roles have a major security problem.

All pods on that node share the same permissions.

Example:

Pod A needs S3 access  
Pod B should NOT have access  

But both run on same node → both get S3 access

This violates the principle of **least privilege**.

This is a serious issue in multi-tenant clusters.

---

### 5. What is IRSA (IAM Roles for Service Accounts)

IRSA allows **individual Kubernetes pods** to assume specific IAM roles.

Instead of giving permissions to nodes, permissions are given to **service accounts**.

Flow:

Pod  
↓

Kubernetes ServiceAccount  
↓

IAM Role  
↓

AWS Service

Each pod can have its own IAM role.

---

### 6. Key Components of IRSA

IRSA involves three main components.

Kubernetes ServiceAccount

Represents identity inside Kubernetes.

IAM Role

Defines AWS permissions.

OIDC Identity Provider

Allows AWS to trust Kubernetes-issued tokens.

---

### 7. What is OIDC (OpenID Connect)

OIDC is an identity layer that allows Kubernetes to issue identity tokens.

EKS clusters have an OIDC provider.

This allows AWS IAM to trust tokens issued by Kubernetes.

Without OIDC, IRSA would not work.

---

### 8. IRSA Authentication Flow

When a pod uses IRSA:

Pod requests AWS access  
↓

Kubernetes injects service account token  
↓

AWS STS validates token using OIDC  
↓

IAM role is assumed  
↓

Temporary credentials returned  
↓

Pod accesses AWS service

This process happens automatically.

---

### 9. Why IRSA Is Safer Than Node IAM

Key advantages:

Fine-grained access

Each pod gets only required permissions.

Isolation

Pods do not share credentials.

Temporary credentials

No long-lived secrets.

Better auditing

Each role can be tracked separately.

This makes IRSA the **recommended approach for production EKS clusters**.

---

### 10. Step 1: Enable OIDC Provider

First, associate OIDC provider with EKS cluster.

```
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster \
  --approve
```

This enables identity federation.

---

### 11. Step 2: Create IAM Policy

Create a policy defining permissions.

Example:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "*"
    }
  ]
}
```

Create policy:

```
aws iam create-policy \
  --policy-name s3-read-policy \
  --policy-document file://policy.json
```

---

### 12. Step 3: Create IAM Role for Service Account

Create role with trust policy.

Example trust relationship:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:oidc-provider/<OIDC_PROVIDER>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity"
    }
  ]
}
```

Attach policy to role.

---

### 13. Step 4: Create Kubernetes ServiceAccount

Example:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: s3-access
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<ACCOUNT_ID>:role/s3-role
```

This annotation links Kubernetes ServiceAccount to IAM role.

---

### 14. Step 5: Use ServiceAccount in Pod

Example pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: app
spec:
  serviceAccountName: s3-access
  containers:
  - name: app
    image: amazonlinux
    command: ["sleep", "3600"]
```

This pod now has access to S3 using IAM role.

---

### 15. Step 6: Test Access

Exec into pod:

```
kubectl exec -it app -- bash
```

Run:

```
aws s3 ls
```

If configured correctly, it will list S3 buckets.

---

### 16. How Credentials Are Injected

Kubernetes automatically mounts a token inside the pod.

Location:

```
/var/run/secrets/eks.amazonaws.com/serviceaccount/token
```

AWS SDK reads this token and exchanges it for temporary credentials.

---

### 17. IRSA with Applications

Most AWS SDKs automatically detect IRSA credentials.

Example in Node.js:

```
const s3 = new AWS.S3();
```

No credentials needed in code.

SDK uses IRSA automatically.

---

### 18. Common Gotchas

OIDC not enabled

IRSA will not work.

Incorrect trust policy

Role cannot be assumed.

Wrong annotation

ServiceAccount not linked to role.

Using default ServiceAccount

May unintentionally grant permissions.

---

### 19. Production Best Practices

Use one IAM role per application.

Follow least privilege principle.

Avoid using node IAM roles.

Use descriptive role names.

Audit IAM roles regularly.

---

### 20. IRSA vs Node IAM Summary

Node IAM

Permissions at node level  
All pods share access  
Less secure

IRSA

Permissions at pod level  
Fine-grained access  
More secure  
Recommended approach

---

### 21. Real Production Example

Example architecture:

EKS cluster  
↓

Pod A → IAM role for S3  
Pod B → IAM role for DynamoDB  
Pod C → IAM role for SQS  

Each pod has independent permissions.

---

### 22. Final Summary

IAM Roles for Service Accounts (IRSA) is a secure mechanism that allows Kubernetes pods in Amazon EKS to access AWS services using IAM roles.

It replaces the older node IAM model by providing fine-grained, pod-level permissions, improving security and enabling least privilege access.

IRSA uses OIDC identity federation to allow Kubernetes service accounts to assume IAM roles and obtain temporary credentials dynamically.

This approach is essential for building secure, production-grade Kubernetes applications on AWS.
