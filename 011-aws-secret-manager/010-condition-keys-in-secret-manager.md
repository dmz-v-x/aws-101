## IAM Condition Keys in Secrets Manager (aws:RequestTag, aws:TagKeys, kms:ViaService, kms:CallerAccount) and How to Scope Permissions

### 1. Introduction

In AWS IAM, **condition keys** allow you to enforce **fine-grained access control** beyond simple allow/deny rules.

Instead of granting broad permissions like:

    "Action": "secretsmanager:*",
    "Resource": "*"

you can restrict permissions using **conditions** that must be satisfied for access to be granted.

Condition keys help enforce policies such as:

- Allow creating secrets only with specific tags
- Restrict access to secrets belonging to a specific environment
- Ensure KMS keys are used only through approved AWS services
- Allow KMS decryption only for specific AWS accounts

Using condition keys is essential for implementing **least-privilege security and governance policies** in large AWS environments.

---

### 2. Structure of IAM Conditions

An IAM policy condition block typically looks like this:

    "Condition": {
      "ConditionOperator": {
        "ConditionKey": "ConditionValue"
      }
    }

Example:

    "Condition": {
      "StringEquals": {
        "aws:RequestTag/Environment": "Production"
      }
    }

This means the action is allowed **only if the request includes the specified tag**.

---

### 3. aws:RequestTag

#### What It Does

`aws:RequestTag` controls **tags that must be included when creating or modifying resources**.

This is commonly used to enforce **mandatory tagging policies**.

When a user creates a secret, they must include certain tags in the request.

---

#### Example Use Case

An organization requires all secrets to include the tag:

    Environment = Production

If a secret is created without this tag, the request should fail.

---

#### Example Policy

    {
      "Effect": "Allow",
      "Action": "secretsmanager:CreateSecret",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "aws:RequestTag/Environment": "Production"
        }
      }
    }

This policy allows creating secrets **only if the request contains the tag Environment=Production**.

---

#### Example Request That Works

    aws secretsmanager create-secret \
      --name prod/db/password \
      --secret-string "mypassword" \
      --tags Key=Environment,Value=Production

---

### 4. aws:TagKeys

#### What It Does

`aws:TagKeys` restricts **which tag keys can be used when tagging resources**.

This prevents users from adding arbitrary tags.

---

#### Example Use Case

An organization allows only these tag keys:

- Environment
- Application
- Owner

Users should not create new tag categories.

---

#### Example Policy

    {
      "Effect": "Allow",
      "Action": "secretsmanager:TagResource",
      "Resource": "*",
      "Condition": {
        "ForAllValues:StringEquals": {
          "aws:TagKeys": [
            "Environment",
            "Application",
            "Owner"
          ]
        }
      }
    }

This ensures that **only approved tag keys can be applied**.

---

### 5. kms:ViaService

#### What It Does

`kms:ViaService` ensures that a KMS key can be used **only through a specific AWS service**.

This prevents direct usage of the KMS key outside approved services.

This is extremely important when encrypting secrets using KMS.

---

#### Example Use Case

You want the KMS key to be used **only by AWS Secrets Manager**, not by other services or direct API calls.

---

#### Example Policy

    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "kms:Decrypt",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:ViaService": "secretsmanager.us-east-1.amazonaws.com"
        }
      }
    }

This ensures that **the KMS key can only be used through Secrets Manager**.

Direct calls like:

    kms:Decrypt

will fail if not coming from Secrets Manager.

---

### 6. kms:CallerAccount

#### What It Does

`kms:CallerAccount` restricts KMS operations to **requests originating from a specific AWS account**.

This is especially useful in **cross-account architectures**.

---

#### Example Use Case

Suppose a KMS key exists in **Account A**, and secrets encrypted with that key should only be accessed by services in **Account B**.

You can enforce this restriction using `kms:CallerAccount`.

---

#### Example Policy

    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "kms:Decrypt",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:CallerAccount": "123456789012"
        }
      }
    }

This ensures that only requests originating from **Account 123456789012** can use the key.

---

### 7. Combining Multiple Condition Keys

Policies can combine multiple conditions to enforce stricter rules.

Example:

    {
      "Effect": "Allow",
      "Action": "kms:Decrypt",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "kms:ViaService": "secretsmanager.us-east-1.amazonaws.com",
          "kms:CallerAccount": "123456789012"
        }
      }
    }

This means:

- The KMS key must be used **via Secrets Manager**
- The request must come from **Account 123456789012**

Both conditions must be satisfied.

---

### 8. How Condition Keys Help Scope Permissions

Condition keys allow you to scope permissions in several ways.

---

#### Tag-Based Governance

Require secrets to have mandatory tags.

Example:

    Environment = Production
    Application = BillingService

---

#### Enforce Organizational Standards

Restrict which tags can be used.

---

#### Limit Encryption Usage

Ensure KMS keys are only used by approved services.

---

#### Restrict Cross-Account Access

Ensure encryption keys are only used by trusted accounts.

---

### 9. Example Secure Secrets Manager Policy

Example policy enforcing tagging and restricted KMS usage:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "secretsmanager:CreateSecret",
          "Resource": "*",
          "Condition": {
            "StringEquals": {
              "aws:RequestTag/Environment": "Production"
            }
          }
        },
        {
          "Effect": "Allow",
          "Action": "kms:Decrypt",
          "Resource": "*",
          "Condition": {
            "StringEquals": {
              "kms:ViaService": "secretsmanager.us-east-1.amazonaws.com"
            }
          }
        }
      ]
    }

This policy ensures:

- Secrets must be tagged with Environment=Production
- KMS decryption is allowed only through Secrets Manager

---

### 10. Best Practices for Using Condition Keys

Follow these best practices when designing IAM conditions.

#### Enforce tagging policies

Require critical tags for all secrets.

---

#### Restrict KMS usage

Use `kms:ViaService` to limit key usage to approved services.

---

#### Control cross-account access

Use `kms:CallerAccount` to limit which accounts can use encryption keys.

---

#### Combine multiple conditions

Multiple conditions provide stronger security controls.

---

### 11. Summary

IAM condition keys provide powerful mechanisms for implementing **fine-grained security policies** in AWS.

Important condition keys include:

| Condition Key | Purpose |
|------|------|
| aws:RequestTag | Require specific tags when creating resources |
| aws:TagKeys | Restrict which tag keys can be used |
| kms:ViaService | Restrict KMS usage to specific AWS services |
| kms:CallerAccount | Restrict KMS usage to specific AWS accounts |

Using these condition keys helps enforce:

- Governance policies
- Security best practices
- Least-privilege access
- Controlled encryption usage

These mechanisms are essential for building **secure, scalable AWS environments where secrets and encryption keys are tightly controlled**.
