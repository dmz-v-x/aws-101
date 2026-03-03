## Amazon S3 Access Control: Bucket Policies vs ACLs vs IAM Policies (Comparison and Use Cases)

### 1. Introduction

Amazon S3 stores data, but controlling **who can access that data** is equally important. Access control ensures that only authorized users, applications, or services can read, write, or manage objects in a bucket.

S3 provides multiple mechanisms for controlling access:

- IAM Policies
- Bucket Policies
- Access Control Lists (ACLs)

These mechanisms work together as part of AWS’s broader **Identity and Access Management (IAM)** system.

Understanding the differences between these access control methods is important because using the wrong one can lead to:

- security vulnerabilities
- accidental public data exposure
- complex permission management
- difficult troubleshooting

---

### 2. The Goal of Access Control in S3

Access control determines **who can perform which actions on which resources**.

Typical S3 actions include:

- uploading objects
- downloading objects
- deleting objects
- listing bucket contents
- modifying bucket configuration

Permissions must define:

Who can perform the action

Which resource the action applies to

What actions are allowed or denied

AWS implements these rules using policy-based access control.

---

### 3. Overview of the Three Access Control Mechanisms

Amazon S3 supports three primary access control mechanisms.

IAM Policies

Permissions attached to AWS identities such as users, groups, and roles.

Bucket Policies

Policies attached directly to S3 buckets that control access to bucket resources.

Access Control Lists (ACLs)

Legacy permission systems attached to buckets or objects.

Each mechanism operates at a different level in the AWS permission model.

---

### 4. IAM Policies

IAM policies are the **most commonly used access control mechanism in AWS**.

An IAM policy is a JSON document that defines permissions for an IAM identity.

IAM identities include:

- IAM users
- IAM groups
- IAM roles

IAM policies define what actions these identities can perform on AWS resources.

---

### 5. Example IAM Policy for S3

Example IAM policy allowing read access to a bucket:

```
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject"
  ],
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

This policy allows an IAM user or role to download objects from the bucket.

IAM policies are often used to control access for:

- applications
- developers
- backend services

---

### 6. Bucket Policies

Bucket policies are resource-based policies attached directly to an S3 bucket.

Unlike IAM policies, bucket policies define **who can access the bucket resource itself**.

They are also written in JSON format.

Bucket policies are useful when you want to control access to a bucket from multiple accounts or services.

---

### 7. Example Bucket Policy

Example bucket policy allowing public read access:

```
{
  "Effect": "Allow",
  "Principal": "*",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

This policy allows anyone on the internet to read objects from the bucket.

Bucket policies are commonly used for:

- public static websites
- cross-account access
- service integrations

---

### 8. Access Control Lists (ACLs)

ACLs are the **oldest access control mechanism in S3**.

ACLs allow permissions to be granted to specific AWS accounts.

ACL permissions include actions such as:

- read
- write
- read ACL
- write ACL

ACLs can be applied to:

- buckets
- individual objects

However, ACLs are now considered **legacy** and are rarely recommended for modern architectures.

---

### 9. Example ACL Permissions

ACL permissions might grant access like:

```
Account A → Read access
Account B → Write access
```

ACLs do not support the rich conditional logic that policies support.

Because of this limitation, they are less flexible than IAM or bucket policies.

---

### 10. Why ACLs Are Less Common Today

AWS recommends using **IAM policies and bucket policies instead of ACLs**.

Reasons include:

Limited flexibility

ACLs support fewer permission options.

Complex management

ACLs can be difficult to track across large numbers of objects.

Security risks

ACLs have historically caused accidental public exposure of data.

To simplify access control, AWS introduced **S3 Object Ownership settings that disable ACLs entirely**.

---

### 11. Comparison Overview

Each access control mechanism has a different purpose.

IAM policies control permissions for identities.

Bucket policies control permissions for bucket resources.

ACLs provide basic legacy permissions.

Understanding when to use each one is essential.

---

### 12. Comparison Chart

| Feature | IAM Policies | Bucket Policies | ACLs |
|-------|-------|-------|-------|
| Attached to | Users, groups, roles | Buckets | Buckets and objects |
| Policy format | JSON | JSON | Simple permission list |
| Supports conditions | Yes | Yes | No |
| Supports cross-account access | Yes | Yes | Limited |
| Fine-grained permissions | Yes | Yes | Limited |
| Recommended for modern systems | Yes | Yes | No |
| Can grant public access | Indirectly | Yes | Yes |

This table summarizes the major differences.

---

### 13. IAM Policy Use Cases

IAM policies are typically used when granting permissions to AWS identities.

Common use cases include:

Allowing application servers to upload objects.

Allowing developers to read data from buckets.

Allowing backend services to process files stored in S3.

IAM policies are ideal for **identity-based access control**.

---

### 14. Bucket Policy Use Cases

Bucket policies are commonly used when access must be granted to **external entities or services**.

Examples include:

Allowing a CloudFront distribution to access S3 objects.

Allowing another AWS account to read objects.

Making objects publicly accessible for static websites.

Bucket policies are useful when the bucket itself needs to control access.

---

### 15. ACL Use Cases

ACLs are rarely needed today but may appear in older systems.

Possible use cases include:

Legacy systems that rely on object-level permissions.

Interoperability with older AWS tools.

However, modern architectures usually disable ACLs entirely.

---

### 16. Combining IAM Policies and Bucket Policies

In many systems, IAM policies and bucket policies work together.

Example architecture:

An application role has an IAM policy allowing object uploads.

The bucket policy allows that role to write objects.

Both permissions must allow the action for access to succeed.

AWS evaluates all relevant policies before granting access.

---

### 17. AWS Permission Evaluation Logic

When a request is made to S3, AWS evaluates permissions in multiple places.

This includes:

IAM policies

Bucket policies

ACLs (if enabled)

Block Public Access settings

Explicit deny rules always override allow rules.

Understanding this evaluation process is important when troubleshooting access problems.

---

### 18. Security Best Practices

Modern S3 architectures follow several best practices.

Prefer IAM policies for identity-based permissions.

Use bucket policies for cross-account or public access.

Disable ACLs when possible.

Enable Block Public Access to prevent accidental exposure.

Regularly audit bucket permissions.

Following these practices improves security.

---

### 19. Example Real World Architecture

Consider a web application storing images in S3.

Application servers upload images.

IAM role attached to servers allows:

```
s3:PutObject
```

Bucket policy allows:

Application role access to the bucket.

Public users can read images if required.

ACLs are disabled to simplify access control.

This architecture uses IAM and bucket policies together.

---

### 20. Summary

Amazon S3 supports multiple access control mechanisms.

IAM policies

Used to define permissions for users, roles, and applications.

Bucket policies

Used to control access to bucket resources.

ACLs

Legacy permissions that are rarely used in modern systems.

In most modern architectures:

IAM policies handle identity permissions.

Bucket policies handle resource-level access.

ACLs are disabled or avoided.

Understanding how these mechanisms differ allows engineers to design secure and maintainable S3 storage systems.
