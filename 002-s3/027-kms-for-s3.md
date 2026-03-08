## AWS KMS Basics for Amazon S3: Customer-Managed Keys vs AWS-Managed Keys, Key Policies, and IAM Interactions

### 1. Introduction

When using server-side encryption with AWS Key Management Service (SSE-KMS), Amazon S3 relies on **AWS KMS (Key Management Service)** to create, manage, and control encryption keys.

KMS provides centralized key management and fine-grained control over who can use encryption keys. It also enables auditing, access control, and automatic key rotation.

Understanding how KMS works with S3 requires understanding three core concepts:

- KMS keys (formerly called CMKs)
- key policies
- IAM permissions

These components work together to control how encryption keys are used to protect S3 objects.

---

### 2. What is AWS KMS?

AWS KMS is a managed service used to create and control **cryptographic keys** used to encrypt data.

KMS handles several tasks:

- secure key storage
- key generation
- key rotation
- key usage permissions
- encryption and decryption operations
- auditing of key usage

Many AWS services integrate with KMS, including:

- Amazon S3
- Amazon EBS
- Amazon RDS
- AWS Lambda
- AWS Secrets Manager

For S3, KMS is used when the **SSE-KMS encryption option** is selected.

---

### 3. How S3 Uses KMS

When SSE-KMS is enabled for an S3 object, S3 uses a KMS key to encrypt the object.

The simplified workflow is:

```
Client uploads object
        ↓
S3 requests encryption key from KMS
        ↓
KMS provides encryption key
        ↓
S3 encrypts object
        ↓
Encrypted object stored in S3
```

When the object is retrieved:

```
Client requests object
        ↓
S3 requests permission to use KMS key
        ↓
KMS decrypts encryption key
        ↓
S3 decrypts object
        ↓
Object returned to client
```

KMS therefore controls the encryption keys used for S3 objects.

---

### 4. What is a KMS Key (Formerly CMK)?

In AWS KMS, the main resource is a **KMS key**.

Previously, these were called **Customer Master Keys (CMKs)**. AWS now refers to them simply as **KMS keys**.

A KMS key is used to:

- encrypt data keys
- decrypt data keys
- control access to encrypted resources

Each KMS key has:

- a unique identifier
- a key policy
- optional key rotation
- usage permissions

S3 uses these keys during encryption and decryption operations.

---

### 5. Types of KMS Keys Used with S3

When using SSE-KMS with S3, there are two primary types of keys.

AWS-managed keys

Customer-managed keys

Both are stored in AWS KMS but differ in control and configuration.

---

### 6. AWS-Managed Keys

AWS-managed keys are created and maintained automatically by AWS services.

Example:

When SSE-KMS is enabled for S3 without specifying a key, AWS may use the default key:

```
aws/s3
``` 

Characteristics of AWS-managed keys:

- created automatically
- managed entirely by AWS
- rotated automatically
- minimal configuration required

These keys are suitable for many workloads where advanced key management is not required.

---

### 7. Advantages of AWS-Managed Keys

AWS-managed keys offer several benefits.

Simplicity

No need to create or configure keys manually.

Automatic rotation

AWS handles key rotation automatically.

Low operational overhead

Minimal management effort required.

These keys are often used in basic encryption setups.

---

### 8. Limitations of AWS-Managed Keys

Although convenient, AWS-managed keys provide limited control.

Limitations include:

Limited customization of key policies.

Restricted visibility into key usage.

No fine-grained access control over key permissions.

Organizations with strict security requirements usually prefer customer-managed keys.

---

### 9. Customer-Managed Keys

Customer-managed keys are KMS keys created and controlled by the user.

Administrators define:

- key policies
- IAM permissions
- key rotation settings
- usage permissions

These keys provide much more control over encryption operations.

Example key:

```
arn:aws:kms:region:account-id:key/key-id
```

S3 can use this key to encrypt objects.

---

### 10. Advantages of Customer-Managed Keys

Customer-managed keys provide several advanced capabilities.

Fine-grained access control

Administrators define who can use the key.

Audit logging

All key usage can be recorded in CloudTrail.

Key rotation control

Keys can be rotated automatically or manually.

Cross-account permissions

Keys can be shared across AWS accounts.

These features make customer-managed keys ideal for enterprise environments.

---

### 11. KMS Key Policies

Every KMS key has a **key policy**.

A key policy is a JSON document that defines who can use or manage the key.

Key policies determine:

- who can encrypt data
- who can decrypt data
- who can manage the key
- who can grant permissions

Example conceptual structure:

```
Key Policy
 ├── Allowed principals
 ├── Allowed actions
 └── Resource definitions
```

Without the correct key policy permissions, the key cannot be used.

---

### 12. IAM and KMS Interaction

KMS permissions are controlled through two layers:

Key policies

IAM policies

Both must allow access for a user or service to use a key.

This dual-permission model provides strong security control.

---

### 13. Permission Evaluation Model

When an S3 request attempts to use a KMS key, AWS checks:


IAM policy permissions
        +
KMS key policy permissions


If either denies access, the request fails.

Both must allow the action.

---

### 14. Example Access Scenario

Example workflow:

Application uploads object using SSE-KMS.

Steps:

Application role sends upload request
        ↓
S3 attempts to use KMS key
        ↓
AWS checks IAM permissions
        ↓
AWS checks KMS key policy
        ↓
If allowed → encryption proceeds


If the role lacks permission, the upload fails.

---

### 15. Required KMS Permissions for S3

Typical permissions required include:

```
kms:Encrypt
kms:Decrypt
kms:GenerateDataKey
kms:DescribeKey
```

These permissions allow S3 to use the KMS key during encryption and decryption.

---

### 16. Envelope Encryption with KMS

S3 uses **envelope encryption** when working with KMS.

Envelope encryption process:

```
KMS generates data key
        ↓
Data key encrypts object
        ↓
Data key encrypted using KMS key
        ↓
Encrypted data key stored with object
```

This design improves performance because large objects are encrypted using data keys rather than the master key.

---

### 17. KMS Request Costs

Using SSE-KMS introduces additional costs.

Each encryption or decryption request may generate a KMS API call.

Examples include:

Object upload

Object download

Key generation operations

For high-volume workloads, these requests can contribute to operational costs.

---

### 18. KMS Auditing and Monitoring

One advantage of KMS integration is auditing.

Every key usage event can be logged using:

AWS CloudTrail

These logs include:

- who used the key
- when the key was used
- which resource used the key

This auditing capability is valuable for compliance and security monitoring.

---

### 19. Real World Example

Consider a company storing confidential financial data in S3.

Requirements:

Strict control over encryption keys.

Audit logs of key usage.

Controlled access to encrypted data.

Solution:

Use SSE-KMS with a customer-managed key.

The key policy allows only authorized application roles to use the key.

All key usage events are logged through CloudTrail.

---

### 20. Best Practices

Recommended practices include:

Use SSE-KMS for sensitive workloads.

Use customer-managed keys for strong security control.

Enable key rotation.

Monitor key usage with CloudTrail.

Restrict key permissions carefully.

Proper key management is essential for secure encryption systems.

---

### 21. Mental Model

A useful mental model is:

```
S3 stores encrypted data
        ↓
KMS protects encryption keys
```

S3 protects the data, while KMS protects the keys used to encrypt that data.

Both systems work together to provide secure storage.

---

### 22. Summary

AWS KMS provides centralized encryption key management for services such as Amazon S3.

When using SSE-KMS, S3 relies on KMS keys to encrypt and decrypt stored objects.

Two types of keys can be used:

AWS-managed keys, which are automatically managed by AWS, and customer-managed keys, which provide full control over key policies and permissions.

Access to KMS keys is controlled through both key policies and IAM policies, creating a layered security model.

By combining S3 encryption with KMS key management, organizations can achieve strong data protection, auditing, and compliance capabilities.
