## Managed Policies vs Inline Policies

In AWS IAM, policies can be attached to identities in **two different ways**.

These are called:

Managed Policies  
Inline Policies

Both types define permissions, but they differ in **how they are created, reused, and managed**.

---

### 1. What Is a Managed Policy?

A **managed policy** is a standalone IAM policy that exists **independently of users, groups, or roles**.

This means the policy is created once and can be **attached to multiple identities**.

Think of a managed policy as a **reusable permission template**.

Example idea:

You create a policy called:

DeveloperEC2Access

That policy allows:

- Starting EC2 instances
- Stopping EC2 instances
- Viewing EC2 instances

Now this single policy can be attached to:

- Multiple IAM users
- Multiple IAM roles
- Multiple IAM groups

This avoids repeating the same permissions many times.

---

### 2. Example Managed Policy (Real AWS Example)

Example policy that allows viewing EC2 instances.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "ec2:DescribeInstances",
            "ec2:DescribeVolumes"
          ],
          "Resource": "*"
        }
      ]
    }

Once this policy is created, it can be attached to many identities.

Example structure:

    Developer Group
            ↓
    EC2ReadOnlyPolicy

Now all developers inherit these permissions.

---

### 3. Types of Managed Policies

AWS supports **two types of managed policies**.

AWS Managed Policies  
Customer Managed Policies

---

### 4. AWS Managed Policies

These policies are created and maintained by AWS.

They are designed for common use cases.

Examples include policies like:

AdministratorAccess  
ReadOnlyAccess  
AmazonS3FullAccess  
AmazonEC2FullAccess

Example usage:

You attach **ReadOnlyAccess** to a user.

That user can view resources across AWS but cannot modify them.

Example:

    IAM User
        ↓
    ReadOnlyAccess Policy

AWS automatically maintains these policies.

If AWS updates service permissions, the policy may also be updated.

---

### 5. Customer Managed Policies

Customer managed policies are created by **your organization**.

You define the permissions yourself.

Example:

A company creates a policy called:

DeveloperEC2Access

This policy might allow:

- Starting EC2 instances
- Stopping EC2 instances
- Viewing instance information

But not deleting instances.

Customer managed policies allow organizations to create **custom permission models** tailored to their infrastructure.

---

### 6. What Is an Inline Policy?

An **inline policy** is a policy that is directly embedded inside a specific identity.

Unlike managed policies, inline policies **cannot be reused**.

They belong to **only one identity**.

Example structure:

    IAM User
        ↓
    Inline Policy

If the user is deleted, the policy is deleted automatically.

Inline policies are tightly coupled to the identity they belong to.

---

### 7. Example Inline Policy

Example inline policy attached directly to a role.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::company-reports/*"
        }
      ]
    }

This policy allows reading files from the **company-reports S3 bucket**.

But it applies **only to the identity it is attached to**.

No other user or role can reuse it.

---

### 8. Managed vs Inline Policies (Comparison)

Managed policies and inline policies differ in several important ways.

Managed policies:

- Created separately from identities
- Can be reused across multiple users, groups, or roles
- Easier to manage at scale
- Can be centrally updated

Inline policies:

- Embedded inside a single identity
- Cannot be reused
- Deleted automatically with the identity
- Useful for very specific permissions

---

### 9. Real AWS Scenario

Imagine a company with many developers.

All developers need permission to:

- Read logs
- Deploy applications
- Access specific S3 buckets

Instead of attaching the same inline policy to every user, the company creates a **managed policy**.

Example architecture:

    Developers Group
            ↓
    DeveloperAccessPolicy

Whenever a new developer joins the company:

Add user → add to group → permissions automatically applied.

This makes permission management much easier.

---

### 10. When Inline Policies Are Useful

Inline policies are useful in situations where a permission should belong **only to one identity**.

Example scenario:

A specific IAM role needs permission to access a single S3 bucket used by one application.

Instead of creating a reusable policy, the permission can be embedded directly as an inline policy.

Example:

    Application Role
            ↓
    Inline Policy

This keeps the permission tightly linked to that role.

---

### 11. Best Practice in AWS

In most real-world AWS environments:

Managed policies are preferred.

This is because they are:

Reusable  
Easier to maintain  
Centralized  

Inline policies are used only for **very specific identity-level permissions**.

---

### Mental Model

Think of managed policies as **reusable permission templates**.

Inline policies are **custom permissions written directly into a specific identity**.

Managed policies are better for **scaling permission management** across large teams and systems.

Inline policies are useful for **unique one-off permissions**.

---

### Key Takeaways

IAM policies can be attached as managed policies or inline policies.

Managed policies exist independently and can be attached to multiple identities.

There are two types of managed policies: AWS managed policies and customer managed policies.

Inline policies are embedded directly inside a single user, group, or role.

Managed policies are preferred for large environments because they are reusable and easier to manage.
