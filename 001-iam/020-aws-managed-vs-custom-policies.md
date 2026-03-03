## AWS Managed Policies vs Custom Policies

When working with IAM policies in AWS, there are two common sources for policies:

AWS Managed Policies  
Customer Managed Policies (Custom Policies)

Both are types of **managed policies**, meaning they are standalone policies that can be attached to multiple users, groups, or roles.

The difference lies in **who creates and maintains the policy**.

---

### 1. What Is an AWS Managed Policy?

An **AWS Managed Policy** is a policy that is **created and maintained by AWS**.

AWS provides these policies to cover common use cases so that users do not need to manually create permissions from scratch.

These policies are pre-built and ready to attach to identities.

Example AWS managed policies include:

AdministratorAccess  
ReadOnlyAccess  
AmazonS3FullAccess  
AmazonEC2FullAccess  
AmazonDynamoDBFullAccess

These policies are maintained by AWS, which means AWS may update them whenever new features or permissions are added to services.

---

### 2. Example AWS Managed Policy

Example: **ReadOnlyAccess**

This AWS managed policy allows viewing resources across many AWS services but prevents modifying them.

Conceptually it allows actions like:

- Describe resources
- List resources
- View configurations

But denies actions like:

- Create resources
- Modify resources
- Delete resources

Example conceptual actions allowed:

    "Action": [
      "*:Describe*",
      "*:List*",
      "*:Get*"
    ]

This policy is very useful for roles like:

Auditors  
Support engineers  
Monitoring systems

---

### 3. Example Scenario Using AWS Managed Policies

Imagine a company hires a support engineer whose job is to **inspect infrastructure but not modify it**.

Instead of writing a custom policy, the administrator attaches the AWS managed policy:

ReadOnlyAccess

Architecture example:

    Support Engineer User
            ↓
    ReadOnlyAccess Policy

Now the engineer can:

- View EC2 instances
- View S3 buckets
- View CloudWatch metrics
- Inspect configurations

But cannot modify anything.

---

### 4. What Is a Customer Managed Policy?

A **Customer Managed Policy** (often called a **custom policy**) is a policy that **you create and manage yourself**.

Organizations create custom policies when AWS managed policies are:

Too broad  
Too generic  
Not specific enough

Custom policies allow organizations to define **precise permissions tailored to their infrastructure**.

---

### 5. Example Custom Policy

Imagine a company wants developers to:

- Start EC2 instances
- Stop EC2 instances
- View EC2 instances

But **not terminate instances**.

A custom policy can be created like this:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "ec2:StartInstances",
            "ec2:StopInstances",
            "ec2:DescribeInstances"
          ],
          "Resource": "*"
        }
      ]
    }

This policy is more controlled than the AWS managed policy:

AmazonEC2FullAccess

Because it prevents deleting instances.

---

### 6. Real AWS Example (Developer Permissions)

Suppose developers need permission to deploy applications.

They must:

- Upload files to a specific S3 bucket
- Restart EC2 servers
- View logs

But they should not:

- Modify IAM
- Delete infrastructure
- Access billing

An organization might create a custom policy like this:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "s3:PutObject",
            "s3:GetObject"
          ],
          "Resource": "arn:aws:s3:::app-deployments/*"
        },
        {
          "Effect": "Allow",
          "Action": [
            "ec2:StartInstances",
            "ec2:StopInstances"
          ],
          "Resource": "*"
        }
      ]
    }

This policy matches the **exact needs of the organization**.

---

### 7. Key Differences Between AWS Managed and Custom Policies

The major difference is **who controls the policy**.

AWS managed policies:

- Created by AWS
- Automatically updated by AWS
- Designed for common use cases
- Quick to attach and use

Customer managed policies:

- Created by the organization
- Fully customizable
- Maintained by the organization
- Designed for specific environments

---

### 8. Advantages of AWS Managed Policies

AWS managed policies are useful because they are:

Quick to use  
Pre-configured  
Maintained by AWS  
Suitable for common roles

For example:

ReadOnlyAccess is perfect for monitoring users.

AdministratorAccess is useful for temporary admin roles.

---

### 9. Advantages of Custom Policies

Custom policies provide much more **control and precision**.

Organizations can:

Limit permissions to specific resources  
Enforce security restrictions  
Follow least privilege principles  
Match company-specific workflows

This is important for production environments where security requirements are strict.

---

### 10. Best Practice in Real AWS Environments

In real-world systems, both types are used.

AWS managed policies are often used for:

Initial setup  
Testing environments  
Standard roles like read-only access

Customer managed policies are used for:

Production systems  
Security-sensitive environments  
Fine-grained permission control

Most mature AWS environments rely heavily on **custom policies** to implement least privilege access.

---

### Mental Model

You can think of the difference like this:

AWS Managed Policies → Ready-made permission templates created by AWS.

Customer Managed Policies → Custom permission rules designed by your organization.

Both serve the same purpose, but custom policies allow **more precise security control**.

---

### Key Takeaways

AWS managed policies are pre-built policies created and maintained by AWS.

Customer managed policies are custom policies created and maintained by the organization.

AWS managed policies are convenient for common use cases.

Custom policies allow fine-grained control and better implementation of least privilege security.

Most real-world production systems rely heavily on custom policies tailored to their infrastructure.
