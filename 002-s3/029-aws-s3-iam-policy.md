## Amazon S3 IAM Policy Examples for Common Roles: Read-Only, Read/Write, and Full Admin

### 1. Introduction

AWS Identity and Access Management (IAM) policies are used to define what actions users, roles, or services can perform on AWS resources. In the context of Amazon S3, IAM policies control access to buckets and objects.

Organizations typically define **standard access roles** to simplify permission management. Some of the most common roles include:

- Read-only access
- Read and write access
- Full administrative access

These roles allow administrators to grant appropriate permissions based on the responsibilities of users or applications.

This guide explains typical IAM policies used for these roles and how they control access to S3 resources.

---

### 2. Structure of an IAM Policy

IAM policies follow a standard JSON structure.

Basic structure:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "...",
      "Action": "...",
      "Resource": "..."
    }
  ]
}
```

Key components include:

Effect

Determines whether the action is allowed or denied.

Action

Defines the AWS operations allowed.

Resource

Specifies which resources the policy applies to.

Policies can include multiple statements for more complex permission sets.

---

### 3. Resource Types in S3 Policies

S3 policies often need to reference both bucket-level and object-level resources.

Bucket-level resource:

```
arn:aws:s3:::example-bucket
```

Used for operations such as:

```
s3:ListBucket
```

Object-level resource:

```
arn:aws:s3:::example-bucket/*
```

Used for operations such as:

```
s3:GetObject
s3:PutObject
```

Most policies include both resource types.

---

### 4. Read-Only Access Role

A read-only role allows users or applications to **view and download objects** but prevents them from modifying or deleting data.

Typical permissions include:

- listing bucket contents
- retrieving objects

This role is commonly used for:

- analysts
- auditors
- monitoring tools
- data consumers

---

### 5. Read-Only IAM Policy Example

Example policy:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::example-bucket"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

Permissions granted:

```
List objects in the bucket
Download objects
```

Permissions not granted:

```
Upload objects
Delete objects
Modify bucket settings
```

---

### 6. Read-Only Access Behavior

With this policy, users can:

```
View bucket contents
Download files
```

But cannot:

```
Upload new objects
Delete existing objects
Change bucket configuration
```

This ensures that data cannot be modified.

---

### 7. Read/Write Access Role

A read/write role allows users or applications to:

- read objects
- upload objects
- update objects
- delete objects

This role is commonly used for:

- application services
- ETL pipelines
- data ingestion systems
- backend microservices

---

### 8. Read/Write IAM Policy Example

Example policy:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::example-bucket"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

Permissions granted:

```
List objects
Download objects
Upload objects
Delete objects
```

This policy allows applications to manage objects inside the bucket.

---

### 9. Read/Write Role Behavior

Users with this role can:

```
Upload files
Replace existing files
Delete files
Download files
```

However, they cannot:

```
Create or delete buckets
Modify bucket policies
Change encryption settings
```

This limits access to object operations only.

---

### 10. Full Administrative Role

A full administrative role allows complete control over S3 resources.

This role can:

- manage buckets
- modify policies
- configure encryption
- control lifecycle rules
- manage replication
- manage object access

This role is typically restricted to administrators.

---

### 11. Full Admin IAM Policy Example

Example policy:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }
  ]
}
```

This policy grants access to **all S3 actions on all resources**.

Actions allowed include:

```
Bucket creation
Bucket deletion
Object upload
Object deletion
Policy modification
Replication configuration
Lifecycle configuration
```

---

### 12. Safer Admin Policy (Scoped)

Instead of allowing access to all S3 resources, administrators often scope the policy to a specific bucket.

Example:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": [
        "arn:aws:s3:::example-bucket",
        "arn:aws:s3:::example-bucket/*"
      ]
    }
  ]
}
```

This limits administrative control to a specific bucket.

---

### 13. Role-Based Access Design

Organizations often create standardized IAM roles.

Example role structure:

```
DataAnalystRole → Read-only access
AppServiceRole → Read/write access
StorageAdminRole → Full admin access
```

Applications and users assume these roles to obtain appropriate permissions.

---

### 14. Principle of Least Privilege

AWS security best practices recommend the **principle of least privilege**.

This means:

Users should receive only the permissions they absolutely need.

Example:

```
Analytics user → Read-only
Data ingestion service → Read/write
Infrastructure admin → Full admin
```

Limiting permissions reduces security risks.

---

### 15. Combining IAM Policies with Bucket Policies

IAM policies are often used alongside bucket policies.

Example architecture:

```
IAM policy → controls what users can do
Bucket policy → controls who can access the bucket
```

Both policies are evaluated together when access requests occur.

---

### 16. Example Application Architecture

Consider a data processing pipeline.

Roles:

```
Upload service → Read/write
Analytics users → Read-only
Infrastructure team → Admin
```

Permissions are enforced through IAM policies attached to roles.

This ensures proper separation of responsibilities.

---

### 17. Testing IAM Policies

Policies should be tested before production deployment.

AWS provides tools such as:

IAM Policy Simulator

Access Analyzer

These tools help verify that policies behave as expected.

---

### 18. Monitoring Policy Usage

Access logs and monitoring tools help track policy usage.

Examples include:

AWS CloudTrail

Records API calls.

S3 access logs

Track object-level requests.

Monitoring helps detect unauthorized access attempts.

---

### 19. Common Mistakes

Common IAM policy mistakes include:

Granting overly broad permissions.

Using wildcard actions unnecessarily.

Allowing unrestricted access to sensitive buckets.

Failing to restrict resources properly.

Careful policy design reduces these risks.

---

### 20. Best Practices

Recommended practices include:

Use roles instead of individual user permissions.

Apply least privilege access.

Restrict policies to specific buckets when possible.

Audit permissions regularly.

Use monitoring tools to track access patterns.

These practices improve overall security posture.

---

### 21. Mental Model

A useful mental model is:

```
Read-only role → View data

Read/write role → Manage objects

Admin role → Manage storage infrastructure
```

Each role represents increasing levels of control.

---

### 22. Summary

IAM policies define what actions users, roles, or applications can perform on S3 resources.

Common policy patterns include:

Read-only access, which allows listing and downloading objects.

Read/write access, which allows uploading, modifying, and deleting objects.

Full administrative access, which provides complete control over S3 buckets and configuration.

By defining standard access roles and applying the principle of least privilege, organizations can securely manage access to S3 storage resources.
