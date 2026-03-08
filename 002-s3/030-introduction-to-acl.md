## Amazon S3 ACLs Explained and Why They Are Rarely Recommended Today

### 1. Introduction

Amazon S3 originally provided **Access Control Lists (ACLs)** as one of the primary mechanisms for controlling access to buckets and objects.

ACLs allow permissions to be granted to specific AWS accounts or predefined groups.

However, as AWS evolved, more powerful and flexible access control mechanisms were introduced, including:

- IAM policies
- bucket policies
- block public access settings

Because of these newer systems, ACLs are now **rarely recommended** for managing S3 access. AWS best practices strongly encourage using IAM and bucket policies instead.

Understanding ACLs is still important because they are part of the S3 permission model and may exist in legacy systems.

---

### 2. What is an S3 ACL?

An **Access Control List (ACL)** is a set of permissions attached to a bucket or object that specifies which AWS accounts or predefined groups can access it.

ACLs define permissions in a simple list format.

Each entry in the ACL specifies:

- who the permission applies to
- what permissions they have

Example conceptual structure:

```
ACL
 ├── Grantee
 └── Permission
```

ACLs can be attached to:

- buckets
- individual objects

---

### 3. What is a Grantee?

A **grantee** is the entity that receives permission in an ACL.

Grantees can be:

AWS accounts

Specific AWS account IDs.

Canonical users

Unique identifiers representing AWS accounts.

Predefined groups

Special AWS-defined groups representing categories of users.

Each grantee receives a defined set of permissions.

---

### 4. Predefined ACL Groups

S3 includes several predefined groups.

Examples include:

AllUsers

Represents anyone on the internet.

AuthenticatedUsers

Represents any AWS account holder.

LogDelivery

Used for log delivery services.

Example conceptual ACL:

```
Grantee: AllUsers
Permission: READ
```

This configuration makes objects publicly readable.

---

### 5. ACL Permission Types

ACLs support a limited set of permissions.

For buckets:

```
READ
WRITE
READ_ACP
WRITE_ACP
FULL_CONTROL
```

For objects:

```
READ
READ_ACP
WRITE_ACP
FULL_CONTROL
```

Each permission controls specific actions.

---

### 6. Meaning of ACL Permissions

READ

Allows reading objects in the bucket.

WRITE

Allows writing objects to the bucket.

READ_ACP

Allows reading the ACL configuration.

WRITE_ACP

Allows modifying the ACL configuration.

FULL_CONTROL

Grants all ACL permissions.

These permissions are less granular compared to IAM policies.

---

### 7. Bucket ACL Example

Example conceptual ACL:

```
Bucket ACL

Owner:
  FULL_CONTROL

AWS Account 123456789012:
  READ

AllUsers:
  READ
```

This configuration would allow public read access.

---

### 8. Object ACL Example

Objects can also have their own ACLs.

Example:

```
Object: photo.jpg

Owner:
  FULL_CONTROL

AWS Account 111122223333:
  READ
```

This allows another account to read the object.

---

### 9. ACL-Based Public Access

One common historical use of ACLs was enabling public access to objects.

Example configuration:

```
Grantee: AllUsers
Permission: READ
```

This makes objects publicly accessible on the internet.

Today, public access is usually configured using **bucket policies instead**.

---

### 10. Bucket Owner vs Object Owner

ACLs introduce an important concept:

Object ownership.

When objects are uploaded by another AWS account, that account becomes the **object owner**.

Example:

```
Account A owns bucket
Account B uploads object
```

Result:

```
Account B owns the object
```

This can create complicated permission issues.

---

### 11. Cross-Account Upload Problem

Example scenario:

Account A owns bucket.

Account B uploads object.

Account B becomes object owner.

Account A may not automatically have full control over the object.

ACLs were often used to fix this problem by granting permissions to the bucket owner.

This complexity is one reason ACLs became problematic.

---

### 12. Modern Ownership Controls

AWS introduced **S3 Object Ownership** settings to simplify ownership management.

Example setting:

```
BucketOwnerEnforced
```

With this configuration:

- ACLs are disabled
- bucket owner automatically owns all objects

This eliminates many ACL-related permission issues.

---

### 13. Why ACLs Are Limited

ACLs have several limitations.

Limited permission types

ACLs support only a few permissions.

Poor scalability

ACLs are difficult to manage for large systems.

No conditional logic

ACLs cannot enforce conditions such as IP restrictions.

No integration with IAM roles

ACLs cannot express complex identity-based access control.

Because of these limitations, AWS introduced IAM and bucket policies.

---

### 14. IAM Policies vs ACLs

IAM policies provide much more flexibility.

Example capabilities:

- allow or deny specific actions
- restrict access by IP address
- enforce HTTPS
- require encryption
- control access by time

ACLs cannot support these advanced controls.

---

### 15. Bucket Policies vs ACLs

Bucket policies are resource-based policies attached to buckets.

They allow:

- complex permission rules
- cross-account access control
- service integrations
- conditional access logic

Bucket policies are therefore far more powerful than ACLs.

---

### 16. Policy-Based Access Model

Modern S3 access control typically uses:

```
IAM policies
Bucket policies
Block public access settings
```

ACLs are often disabled entirely.

This model simplifies security management.

---

### 17. S3 Block Public Access

AWS introduced **Block Public Access** to prevent accidental public exposure.

This feature blocks:

- public ACLs
- public bucket policies

When enabled, it prevents misconfigured ACLs from exposing data.

This further reduces the need for ACL-based access control.

---

### 18. Object Ownership Setting

A modern S3 bucket often uses:

```
Object Ownership = BucketOwnerEnforced
```

Effects:

```
ACLs disabled
Bucket owner controls all objects
```

This configuration simplifies permission management.

---

### 19. When ACLs Are Still Used

ACLs are rarely used today but may appear in certain cases.

Examples include:

Legacy applications

Older systems that rely on ACL-based permissions.

Log delivery services

Some AWS services historically used ACLs.

Cross-account object uploads in legacy systems.

Even in these cases, AWS now provides better alternatives.

---

### 20. Best Practices

Modern S3 security practices include:

Disable ACLs when possible.

Use IAM policies for identity-based access control.

Use bucket policies for resource-level permissions.

Enable Block Public Access settings.

Use Object Ownership with BucketOwnerEnforced.

These practices simplify security management and reduce misconfiguration risks.

---

### 21. Mental Model

A useful mental model is:

```
ACLs → Old permission system

IAM policies → Identity-based access control

Bucket policies → Resource-based access control
```

Modern architectures rely primarily on IAM and bucket policies.

---

### 22. Summary

Amazon S3 Access Control Lists (ACLs) were originally used to manage access to buckets and objects by assigning permissions to AWS accounts or predefined groups.

ACLs provide a simple permission model but support only limited permissions and lack advanced access control features.

Because of these limitations, AWS now recommends using IAM policies and bucket policies for managing S3 access.

Modern S3 configurations often disable ACLs entirely using Object Ownership settings and Block Public Access controls.

As a result, ACLs are now primarily relevant for understanding legacy systems rather than designing new architectures.
