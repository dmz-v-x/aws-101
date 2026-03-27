## KMS Key Policy vs IAM Policy Interactions — Troubleshooting Denied Access

### 1. Introduction

When AWS Secrets Manager encrypts or decrypts secrets, it interacts with **AWS Key Management Service (KMS)**.

Because KMS keys protect sensitive data, AWS enforces **multiple layers of access control** before allowing an encryption or decryption operation.

Two types of policies control access to a KMS key:

- **KMS Key Policy**
- **IAM Policy**

For an operation like `kms:Decrypt` or `kms:GenerateDataKey` to succeed, **both policies must allow the action**.

If either policy blocks the request, the result will be:

    AccessDeniedException

Understanding how these policies interact is essential when troubleshooting permission errors in Secrets Manager.

---

### 2. The Two Permission Layers in KMS

KMS uses a **dual authorization model**.

Two separate checks must pass:

1. **Key Policy Check**
2. **IAM Policy Check**

The request must be allowed by **both**.

Conceptually:

    Request to use KMS Key
           |
           v
    Check IAM Policy
           |
           v
    Check Key Policy
           |
           v
    If BOTH allow → operation succeeds
    If ANY deny → operation fails

---

### 3. What is a KMS Key Policy?

A **KMS Key Policy** is a resource-based policy attached directly to the KMS key.

It defines **who can use or manage the key**.

Without an appropriate key policy, IAM permissions alone cannot grant access.

Key policies control actions such as:

- Encrypt
- Decrypt
- GenerateDataKey
- DescribeKey
- ScheduleKeyDeletion

Example key policy:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "AWS": "arn:aws:iam::123456789012:role/AppRole"
          },
          "Action": [
            "kms:Decrypt",
            "kms:GenerateDataKey"
          ],
          "Resource": "*"
        }
      ]
    }

This allows the role **AppRole** to use the key.

---

### 4. What is an IAM Policy?

An **IAM policy** is attached to:

- IAM users
- IAM roles
- IAM groups

It defines **what actions the identity can perform**.

Example IAM policy:

    {
      "Effect": "Allow",
      "Action": [
        "kms:Decrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "arn:aws:kms:us-east-1:123456789012:key/abcd1234"
    }

This allows the role to perform encryption operations on the specified key.

---

### 5. How Secrets Manager Uses KMS Permissions

When Secrets Manager encrypts or decrypts secrets, it requires several KMS permissions.

Common required actions include:

- `kms:GenerateDataKey`
- `kms:Decrypt`
- `kms:Encrypt`
- `kms:DescribeKey`

Typical workflow:

    Application
        |
        v
    Secrets Manager
        |
        | Request data key
        v
    AWS KMS (GenerateDataKey)
        |
        v
    Secret encrypted

When retrieving a secret:

    Application
        |
        v
    Secrets Manager
        |
        | Decrypt encrypted data key
        v
    AWS KMS (Decrypt)

If KMS denies access, Secrets Manager cannot decrypt the secret.

---

### 6. Common Access Denied Scenarios

Several common misconfigurations can cause denied access errors.

---

### Scenario 1 — IAM Policy Allows But Key Policy Denies

Example:

IAM policy allows:

    kms:Decrypt

But the KMS key policy **does not include the role**.

Result:

    AccessDeniedException

Reason:

The key policy does not trust the IAM role.

---

### Scenario 2 — Key Policy Allows But IAM Policy Denies

Example:

Key policy includes:

    AppRole

But IAM policy does not include:

    kms:Decrypt

Result:

    AccessDeniedException

Reason:

The identity itself is not authorized.

---

### Scenario 3 — Wrong KMS Key ARN

Sometimes IAM policies reference the wrong key ARN.

Example:

    arn:aws:kms:us-east-1:123456789012:key/incorrect-key

Result:

    AccessDeniedException

Even if permissions exist, they apply to a different key.

---

### Scenario 4 — Missing Secrets Manager Service Permission

When Secrets Manager uses KMS, the KMS key must also allow the **Secrets Manager service**.

Example key policy requirement:

    "kms:ViaService": "secretsmanager.region.amazonaws.com"

Without this permission, Secrets Manager cannot decrypt the secret.

---

### Scenario 5 — Cross-Account Key Access Issues

In cross-account environments:

- Secret stored in Account A
- KMS key in Account A
- Application running in Account B

Both accounts must allow access.

Requirements:

1. IAM policy in Account B must allow KMS usage
2. Key policy in Account A must trust Account B

If either is missing → access denied.

---

### 7. Example Correct Policy Setup

Example IAM policy for application role:

    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "kms:Decrypt"
      ],
      "Resource": "*"
    }

Example key policy allowing the role:

    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/AppRole"
      },
      "Action": [
        "kms:Decrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "*"
    }

Both policies must allow the action.

---

### 8. Troubleshooting Workflow

When debugging KMS access issues, follow this checklist.

---

#### Step 1 — Identify the KMS Key

Determine which KMS key the secret uses.

Command:

    aws secretsmanager describe-secret \
      --secret-id my-secret

Check the field:

    KmsKeyId

---

#### Step 2 — Check IAM Policy

Verify that the role or user includes permissions such as:

- `kms:Decrypt`
- `kms:GenerateDataKey`
- `kms:DescribeKey`

---

#### Step 3 — Check Key Policy

Open the KMS key in the AWS console and verify:

- The IAM role is listed in the key policy
- Required actions are allowed

---

#### Step 4 — Check Service Conditions

Verify whether the key policy includes service restrictions such as:

    kms:ViaService

Example:

    secretsmanager.us-east-1.amazonaws.com

---

#### Step 5 — Review CloudTrail Logs

CloudTrail logs show the exact reason for KMS failures.

Look for events like:

- `Decrypt`
- `GenerateDataKey`

Error messages often identify which policy denied the request.

---

### 9. Recommended Key Policy Pattern

A common key policy structure includes:

1. Root account access
2. IAM role access
3. Service access

Example:

    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/AppRole"
      },
      "Action": [
        "kms:Decrypt",
        "kms:GenerateDataKey"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:ViaService": "secretsmanager.us-east-1.amazonaws.com"
        }
      }
    }

This ensures the key is used only through Secrets Manager.

---

### 10. Best Practices

Follow these best practices when configuring KMS permissions.

#### Always allow both IAM and key policy

Both must permit the operation.

---

#### Restrict key usage with conditions

Use conditions like:

- `kms:ViaService`
- `kms:CallerAccount`

---

#### Avoid overly permissive policies

Avoid:

    "Principal": "*"

---

#### Audit KMS usage

Enable CloudTrail to monitor encryption operations.

---

### 11. Summary

AWS KMS uses a **dual authorization model** involving both IAM policies and key policies.

Key points:

- IAM policies define what identities can do
- Key policies define who can use the key
- Both must allow the request

If either policy denies access, KMS returns:

    AccessDeniedException

When troubleshooting denied access:

1. Check IAM policy permissions
2. Check KMS key policy permissions
3. Verify key ARN correctness
4. Inspect service conditions
5. Review CloudTrail logs

Understanding these interactions is essential for diagnosing **Secrets Manager encryption failures and KMS access errors**.
