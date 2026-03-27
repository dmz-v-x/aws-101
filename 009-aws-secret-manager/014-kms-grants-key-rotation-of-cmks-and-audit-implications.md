## KMS Grants, Key Rotation of CMKs, and Audit Implications

### 1. Introduction

AWS Secrets Manager relies on **AWS Key Management Service (KMS)** to encrypt and decrypt secrets.  
When using **Customer Managed Keys (CMKs)**, organizations must understand three important operational aspects:

- **KMS Grants**
- **Key Rotation**
- **Audit and Monitoring**

These mechanisms ensure that encryption keys are:

- securely accessible to services
- rotated regularly for security
- fully auditable for compliance

Understanding how these features work together is critical for building secure systems using **AWS Secrets Manager and AWS KMS**.

---

### 2. What Are KMS Grants?

A **KMS Grant** is a permission mechanism that allows a principal to use a KMS key **without modifying the key policy**.

Grants provide **temporary delegated access** to a KMS key.

They are commonly used by AWS services such as:

- AWS Secrets Manager
- Amazon S3
- Amazon EBS
- AWS Lambda
- Amazon RDS

Instead of editing the key policy every time a service needs access, KMS automatically creates grants to allow controlled usage.

---

### 3. Why Grants Exist

Key policies are static and controlled by administrators.

However, AWS services often need **temporary operational access** to perform encryption operations.

For example:

- Secrets Manager needs to decrypt a secret
- A Lambda rotation function needs to encrypt a new secret value
- RDS needs to decrypt credentials during rotation

Instead of modifying the key policy repeatedly, AWS creates **temporary grants**.

Conceptually:

```
Key Policy → defines long-term permissions
Grant → provides temporary operational access
```

---

### 4. How Grants Work

When an AWS service needs access to a KMS key, the following process occurs:

```
Service (Secrets Manager)
        |
        | Request temporary permission
        v
AWS KMS creates a Grant
        |
        v
Grant allows service to perform operations
```

The grant allows specific operations such as:

- `Encrypt`
- `Decrypt`
- `GenerateDataKey`
- `DescribeKey`

Grants include:

- **Grantee Principal** – the entity receiving access
- **Allowed Operations**
- **Key ARN**
- **Optional constraints**

---

### 5. Example Grant Structure

A grant internally includes fields similar to:

```
Grant:
  GranteePrincipal: secretsmanager.amazonaws.com
  Operations:
    - Decrypt
    - GenerateDataKey
  KeyId: arn:aws:kms:region:account:key/key-id
```

This allows Secrets Manager to perform encryption operations on the key.

---

### 6. Grant Lifecycle

Grants are not permanent.

They can be:

- Created automatically by AWS services
- Revoked manually
- Retired automatically after use

Typical lifecycle:

```
Service needs key access
        |
        v
Grant created
        |
        v
Service performs encryption/decryption
        |
        v
Grant eventually retired or revoked
```

---

### 7. Viewing KMS Grants

You can view active grants using the AWS CLI.

Example:

```
aws kms list-grants \
  --key-id arn:aws:kms:region:account:key/key-id
```

The response shows:

- Grant ID
- Grantee principal
- Operations allowed

This helps administrators understand **which services are currently using a key**.

---

### 8. Revoking a Grant

Administrators can revoke grants if needed.

Example:

```
aws kms revoke-grant \
  --key-id key-id \
  --grant-id grant-id
```

Revoking a grant immediately removes access to the key.

This can be useful if:

- a service is compromised
- a resource is decommissioned
- temporary access is no longer required

---

### 9. CMK Key Rotation

Encryption keys should not remain static forever.

Long-lived encryption keys increase security risk.

AWS supports **automatic rotation of Customer Managed Keys (CMKs)**.

Key rotation creates a **new cryptographic key version** while keeping the same key identifier.

---

### 10. How Key Rotation Works

When rotation is enabled:

1. AWS generates a **new cryptographic backing key**
2. The CMK keeps the same **Key ID and ARN**
3. New encryption operations use the new key version
4. Old key versions remain available for decryption

Example:

```
Key ID: abc123

Version 1 → used for old secrets
Version 2 → used for new encryption
```

Old secrets remain decryptable because the previous key versions are retained.

---

### 11. Rotation Workflow

```
Original CMK
     |
     v
Key Version 1

Rotation Enabled
     |
     v
Key Version 2 Created
     |
     v
New secrets encrypted using Version 2
Old secrets still decrypt using Version 1
```

Rotation happens automatically every **365 days** when enabled.

---

### 12. Enabling Key Rotation

Key rotation can be enabled using the AWS Console or CLI.

Example CLI command:

```
aws kms enable-key-rotation \
  --key-id arn:aws:kms:region:account:key/key-id
```

Once enabled, AWS rotates the key automatically each year.

---

### 13. Manual Key Rotation

Sometimes organizations require manual rotation.

Manual rotation involves:

1. Creating a new CMK
2. Updating applications to use the new key
3. Re-encrypting data with the new key

This approach is sometimes required for **strict compliance environments**.

---

### 14. Impact of Key Rotation on Secrets Manager

Secrets Manager automatically works with rotated keys.

Because the **Key ID remains the same**, applications do not need to change anything.

Workflow:

```
Secret stored
       |
       v
Encrypted using data key
       |
       v
Data key protected by CMK
```

After rotation:

```
New data keys protected by new CMK version
Old secrets still decrypt using previous versions
```

No service interruption occurs.

---

### 15. Audit Implications

Encryption keys are critical security assets.

Organizations must monitor how keys are used.

AWS provides several auditing mechanisms.

---

### 16. CloudTrail Logging

AWS CloudTrail records all KMS API calls.

Examples of logged operations include:

- `Encrypt`
- `Decrypt`
- `GenerateDataKey`
- `CreateGrant`
- `RevokeGrant`

Example CloudTrail event:

```
EventName: Decrypt
UserIdentity: IAM Role
Resource: KMS Key ARN
```

This allows security teams to track:

- who used the key
- when the key was used
- which service accessed it

---

### 17. Monitoring Grant Activity

CloudTrail also logs grant-related events.

Examples:

- `CreateGrant`
- `RevokeGrant`
- `RetireGrant`

Monitoring these events helps detect:

- unexpected service access
- unauthorized key usage
- privilege escalation attempts

---

### 18. Compliance and Governance

Key rotation and audit logs are essential for compliance frameworks.

Common standards requiring encryption monitoring include:

- PCI DSS
- HIPAA
- SOC 2
- ISO 27001

Auditors often verify:

- keys are rotated regularly
- access logs are retained
- encryption operations are traceable

AWS KMS provides these capabilities through CloudTrail and key rotation.

---

### 19. Best Practices

Follow these best practices when managing KMS keys.

#### Enable automatic rotation

Ensure CMKs rotate regularly.

---

#### Use least privilege access

Only allow necessary services and roles to use the key.

---

#### Monitor CloudTrail logs

Track encryption and decryption operations.

---

#### Review active grants

Periodically audit grants using `list-grants`.

---

#### Restrict key usage

Use conditions like:

```
kms:ViaService
kms:CallerAccount
```

---

### 20. Summary

AWS KMS provides mechanisms for secure key usage and governance.

Key concepts include:

| Feature | Purpose |
|------|------|
| Grants | Temporary permissions allowing services to use a key |
| Key Rotation | Periodic generation of new cryptographic key versions |
| CloudTrail Auditing | Logging of all key usage and management actions |

Grants allow services like **Secrets Manager** to use encryption keys without modifying key policies.

Key rotation ensures encryption keys remain secure over time.

CloudTrail logging provides visibility into key usage for **security monitoring and compliance**.

Together, these mechanisms help organizations maintain a **secure, auditable encryption infrastructure when managing secrets with AWS KMS**.
