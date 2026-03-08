## Amazon S3 Cross-Account Access Patterns: Role Assumption and Bucket Policy Access

### 1. Introduction

In many organizations, AWS environments are structured using **multiple AWS accounts**. This approach improves security isolation, cost management, and operational separation.

Example multi-account architecture:

```
Account A → Application workloads
Account B → Shared storage
Account C → Analytics
```

In such environments, it is often necessary for one AWS account to access resources owned by another account. This is called **cross-account access**.

For Amazon S3, cross-account access is typically implemented using two primary patterns:

- IAM role assumption
- bucket policies that grant access to another account

Both methods allow secure access while maintaining clear separation between accounts.

---

### 2. Why Cross-Account Access Is Needed

Cross-account access is common in modern cloud architectures.

Typical scenarios include:

Centralized storage accounts

One account stores shared datasets used by multiple teams.

Data lake architectures

Analytics accounts access data stored in ingestion accounts.

Shared services

Backup or logging accounts store data from many application accounts.

Vendor access

External partners may require limited access to specific buckets.

These scenarios require controlled cross-account permissions.

---

### 3. Cross-Account Access Concepts

Cross-account access involves three main components.

```
Resource owner account
Account that owns the S3 bucket

Requesting account
Account requesting access

Permission mechanism
Policies that allow the access
```

Permissions must be configured correctly in both accounts for access to work.

---

### 4. Two Common Access Patterns

The two most common cross-account access patterns are:

```
1. IAM role assumption
2. Bucket policy granting another account access
```

Both approaches are widely used depending on the architecture.

---

### 5. Pattern 1: IAM Role Assumption

The **role assumption pattern** uses AWS Security Token Service (STS).

In this approach:

- the bucket owner creates an IAM role
- the requesting account is allowed to assume the role
- the role has permissions to access the S3 bucket

The requesting account temporarily assumes the role to gain access.

---

### 6. Role Assumption Architecture

Example architecture:

```
Account A (requesting account)
        ↓
Assume role via STS
        ↓
Account B role
        ↓
Access S3 bucket
```

The role exists in the bucket owner account.

This method is considered the **most secure and flexible approach**.

---

### 7. Step 1: Create Role in Bucket Owner Account

In the bucket owner account:

Create an IAM role.

Attach permissions allowing access to the S3 bucket.

Example permissions:

```
s3:GetObject
s3:ListBucket
```

The role defines what actions can be performed.

---

### 8. Step 2: Configure Trust Policy

The role must include a **trust policy** that allows the requesting account to assume it.

Example conceptual trust relationship:

```
Account A allowed to assume role in Account B
```

This trust relationship allows secure delegation of permissions.

---

### 9. Step 3: Assume the Role

The requesting account uses AWS STS to assume the role.

Workflow:

```
Application requests role assumption
        ↓
STS issues temporary credentials
        ↓
Application uses temporary credentials
        ↓
S3 access granted
```

These credentials expire automatically after a short time.

---

### 10. Advantages of Role Assumption

Role assumption provides several advantages.

Temporary credentials

Credentials expire automatically.

Fine-grained permissions

Roles can define precise access rules.

Strong security

No permanent cross-account credentials are shared.

Auditability

Role usage is logged in AWS CloudTrail.

For these reasons, role assumption is often the preferred method.

---

### 11. Pattern 2: Bucket Policy Access

Another method is granting cross-account access directly using a **bucket policy**.

In this model, the bucket owner grants access to another AWS account using the `Principal` field.

This allows identities in the other account to access the bucket.

---

### 12. Bucket Policy Architecture

Example architecture:

```
Account A user
        ↓
Requests access to S3
        ↓
Bucket policy in Account B allows it
        ↓
Access granted
```

The policy defines which external accounts are allowed to access the bucket.

---

### 13. Example Bucket Policy for Cross-Account Access

Example conceptual policy:

```
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::111122223333:root"
  },
  "Action": [
    "s3:GetObject"
  ],
  "Resource": "arn:aws:s3:::example-bucket/*"
}
```

This policy allows another AWS account to read objects.

The principal represents the external account.

---

### 14. Required Permissions in Requesting Account

The requesting account must also allow the access through IAM policies.

Example:

```
Allow s3:GetObject on example-bucket
```

Both conditions must be satisfied:

```
IAM policy allows access
Bucket policy allows access
```

If either denies access, the request fails.

---

### 15. Policy Evaluation Model

When a cross-account request occurs, AWS evaluates:

```
IAM policies in requesting account
+
Bucket policy in resource account
```

Access is granted only if both allow the action.

Explicit deny rules override any allows.

---

### 16. Role Assumption vs Bucket Policy

Both approaches provide cross-account access but differ in architecture.

Role assumption:

```
Account assumes role in resource account
Temporary credentials used
```

Bucket policy:

```
External account directly allowed in policy
```

Role assumption is usually more secure and flexible.

---

### 17. Comparison Table

| Feature | Role Assumption | Bucket Policy |
|------|------|------|
| Credentials | Temporary | Permanent identity |
| Security | Strong | Moderate |
| Control | Fine-grained | Less flexible |
| Auditing | Clear role usage logs | Standard access logs |
| Complexity | Higher setup | Simpler setup |

---

### 18. Hybrid Approach

Some architectures combine both methods.

Example:

```
Bucket policy allows role access
Role assumption used by external account
```

In this design:

- bucket policy trusts a role
- external account assumes the role

This provides stronger security control.

---

### 19. Example Real-World Scenario

Consider a company with two AWS accounts.

```
Account A → Application services
Account B → Centralized storage
```

Applications in Account A must read data stored in Account B.

Implementation:

Account B creates a role with S3 permissions.

Account A assumes the role using STS.

Applications access the bucket using temporary credentials.

This architecture isolates storage from application workloads.

---

### 20. Best Practices

Recommended cross-account access practices include:

Prefer role assumption over direct bucket policies.

Use temporary credentials instead of long-term credentials.

Limit permissions using least privilege principles.

Audit access using CloudTrail.

Avoid granting access to entire accounts when possible.

These practices improve security and manageability.

---

### 21. Mental Model

A useful mental model is:

```
Role assumption → Borrow permissions temporarily

Bucket policy → Grant permissions directly
```

Role assumption acts like temporarily entering another account with approved permissions.

---

### 22. Summary

Cross-account access allows AWS accounts to securely access resources owned by other accounts.

For Amazon S3, the two primary access patterns are role assumption and bucket policy-based access.

Role assumption uses AWS STS to grant temporary credentials, providing strong security and flexibility.

Bucket policies allow external accounts to access buckets directly but offer less control.

Most modern architectures prefer role assumption combined with bucket policies for secure and scalable cross-account access.
