## Resource-Based vs Identity-Based Policies

In AWS IAM, permissions can be defined in **two different places**.

Permissions can be attached to:

An identity (user, group, or role)  
or  
A resource (like an S3 bucket or a queue)

Because of this, AWS supports two types of policies:

Identity-based policies  
Resource-based policies

Both define **who can do what**, but they are attached to **different parts of the system**.

---

### 1. Identity-Based Policies

An **identity-based policy** is a policy that is attached to an **IAM identity**.

The identity could be:

IAM User  
IAM Group  
IAM Role  

These policies define **what the identity is allowed to do**.

Example structure:

    IAM User
        ↓
    Identity Policy
        ↓
    Permissions

Example question identity policies answer:

What actions can this user perform?

---

### 2. Example Identity-Based Policy

Example policy attached to an IAM role that allows reading objects from S3.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::company-data/*"
        }
      ]
    }

This policy means:

The identity can read objects inside the **company-data S3 bucket**.

The permission is defined **on the identity side**.

---

### 3. Real AWS Example (Identity Policy)

Imagine a developer role that needs to upload files to an S3 bucket.

Policy attached to the role:

    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::project-assets/*"
    }

Now when the developer uploads a file, AWS checks:

Does this identity have permission?

If yes → request allowed.

---

### 4. Resource-Based Policies

A **resource-based policy** is a policy that is attached directly to an AWS resource.

Instead of asking:

What can this user do?

The question becomes:

Who can access this resource?

Resource-based policies are supported by certain AWS services such as:

S3 buckets  
SNS topics  
SQS queues  
Lambda functions  
KMS keys  

These policies are written directly on the **resource itself**.

---

### 5. Example Resource-Based Policy (S3 Bucket)

Example bucket policy allowing a specific IAM user to read objects.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "AWS": "arn:aws:iam::123456789012:user/Developer"
          },
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::company-data/*"
        }
      ]
    }

Notice something important.

This policy includes a **Principal field**.

Principal defines **who is allowed to access the resource**.

Identity-based policies do not include this field because the identity is already known.

---

### 6. Identity Policy vs Resource Policy Structure

Identity-based policy focuses on:

What actions the identity can perform.

Resource-based policy focuses on:

Who can access the resource.

Example comparison.

Identity policy:

    Identity → Allowed actions → Resource

Resource policy:

    Resource → Allowed identities → Actions

Both achieve permission control but from **opposite directions**.

---

### 7. Real AWS Example (S3 Cross-Account Access)

Imagine two AWS accounts.

Account A owns an S3 bucket.

Account B has an application that needs to read data from that bucket.

Account A can add a **resource-based policy** to the bucket.

Example:

    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::999999999999:role/AppRole"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::shared-data/*"
    }

This allows a role in another AWS account to read files.

This type of access is **not possible using only identity policies**.

Resource policies enable **cross-account access**.

---

### 8. How AWS Evaluates Both Together

When a request is made, AWS may evaluate **both identity policies and resource policies**.

Example scenario:

A user tries to read an S3 object.

AWS checks:

Identity policy attached to the user.

Resource policy attached to the bucket.

Both must allow the action.

If either denies the request, access fails.

---

### 9. Services That Commonly Use Resource Policies

Some AWS services frequently use resource-based policies.

Examples include:

S3 bucket policies  
SNS topic policies  
SQS queue policies  
Lambda resource policies  
KMS key policies  

These services allow administrators to control access **directly at the resource level**.

---

### 10. Mental Model

You can think of access control from two perspectives.

Identity perspective:

What can this identity access?

Resource perspective:

Who is allowed to access this resource?

Identity-based policies answer the first question.

Resource-based policies answer the second.

Together they provide a **complete access control system**.

---

### Key Takeaways

AWS supports identity-based policies and resource-based policies.

Identity-based policies are attached to IAM users, groups, or roles and define what actions they can perform.

Resource-based policies are attached directly to AWS resources and define who can access the resource.

Resource policies include a Principal field that identifies the allowed identity.

Both policy types may be evaluated together when determining access.

Resource-based policies are often used for services like S3, SNS, SQS, Lambda, and KMS.

Understanding both policy types is essential for designing secure AWS architectures.
