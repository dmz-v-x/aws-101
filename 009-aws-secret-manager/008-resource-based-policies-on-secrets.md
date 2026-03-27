## Resource-Based Policies on AWS Secrets — Cross-Account Access Patterns

### 1. Introduction

AWS Secrets Manager stores sensitive credentials such as:

- Database passwords
- API keys
- Service authentication tokens
- Encryption keys

By default, secrets are accessible **only within the AWS account where they are created**. However, in many real-world architectures, multiple AWS accounts must share secrets securely.

Examples:

- A **central security account** storing secrets
- Application accounts needing to retrieve those secrets
- Shared database credentials across accounts
- Cross-account microservices accessing the same secret

To enable this securely, AWS Secrets Manager supports **resource-based policies**.

---

### 2. What is a Resource-Based Policy?

A **resource-based policy** is an IAM policy that is **attached directly to a resource**, instead of being attached to a user or role.

Examples of AWS services that support resource policies:

- Amazon S3
- AWS KMS
- AWS Lambda
- AWS Secrets Manager

Instead of granting permissions through IAM roles alone, the **resource itself defines who can access it**.

For Secrets Manager, a resource policy defines:

- Which **AWS accounts**
- Which **IAM roles**
- Which **IAM users**

are allowed to access the secret.

---

### 3. Identity-Based vs Resource-Based Policies

It is important to understand the difference between the two.

#### Identity-Based Policy

Attached to:

- IAM users
- IAM roles
- IAM groups

Example:

    Allow role to retrieve secrets

---

#### Resource-Based Policy

Attached to:

- AWS resources

Example:

    Allow specific external account to access this secret

---

#### Combined Authorization Model

For access to succeed:

1. The **identity policy** must allow the action.
2. The **resource policy** must allow the principal.

Both conditions must be satisfied.

---

### 4. When Resource Policies Are Used

Resource policies are mainly used for **cross-account access**.

Common scenarios include:

#### Scenario 1 — Centralized Secrets Account

A company stores all secrets in a **security account**.

Other accounts (application accounts) retrieve those secrets.

    Security Account
          |
          | Secret stored here
          v
    Secrets Manager
          |
          | Resource policy allows access
          v
    Application Account retrieves secret

---

#### Scenario 2 — Shared Infrastructure

Example:

- Database credentials stored in Account A
- Application running in Account B

Account B needs access to the secret.

---

#### Scenario 3 — Multi-Account Microservices

Large organizations often use:

- One account per team
- One account per environment
- One account per service

Secrets may need to be shared between services.

---

### 5. Structure of a Resource-Based Policy

A Secrets Manager resource policy follows the standard IAM policy structure.

Example:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "AWS": "arn:aws:iam::111122223333:role/ApplicationRole"
          },
          "Action": "secretsmanager:GetSecretValue",
          "Resource": "*"
        }
      ]
    }

Important components:

| Field | Meaning |
|------|------|
| Effect | Allow or Deny |
| Principal | Who can access the secret |
| Action | Allowed operations |
| Resource | Target secret |

---

### 6. Example Cross-Account Access Setup

Assume:

    Account A = 111111111111 (Secret owner)
    Account B = 222222222222 (Application account)

Account B runs an application that needs a secret stored in Account A.

---

### Step 1 — Create Secret in Account A

Example secret:

    prod/database/credentials

---

### Step 2 — Attach Resource Policy

Attach policy allowing Account B role access.

Example policy:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "AWS": "arn:aws:iam::222222222222:role/AppServiceRole"
          },
          "Action": "secretsmanager:GetSecretValue",
          "Resource": "*"
        }
      ]
    }

This allows the role **AppServiceRole in Account B** to retrieve the secret.

---

### Step 3 — Grant IAM Permission in Account B

In Account B, the role must also have permission to access the secret.

Example IAM policy:

    {
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:region:111111111111:secret:prod/database/credentials-*"
    }

Now both sides allow access.

---

### 7. Cross-Account Access Flow

The access process works like this:

    Application (Account B)
            |
            | Assume Role
            v
    IAM Role in Account B
            |
            | Request secret
            v
    Secrets Manager (Account A)
            |
            | Resource policy allows access
            v
    Secret returned

Both policies must allow access.

---

### 8. Using Conditions in Resource Policies

Conditions add **extra security controls**.

Example condition restricting access to a specific account:

    "Condition": {
      "StringEquals": {
        "aws:PrincipalAccount": "222222222222"
      }
    }

Other conditions include:

- aws:SourceVpc
- aws:SourceIp
- aws:PrincipalArn

These allow restricting access based on environment.

---

### 9. Important Security Considerations

Cross-account secret sharing introduces security risks if not configured correctly.

#### Principle of Least Privilege

Only allow required roles.

Avoid granting access to entire accounts unless necessary.

---

#### Avoid Wildcard Principals

Avoid:

    "Principal": "*"

This could expose the secret publicly.

---

#### Use Specific Secret ARNs

Avoid:

    "Resource": "*"

Instead specify the exact secret ARN.

---

#### Monitor Access with CloudTrail

CloudTrail records all Secrets Manager API calls.

This allows auditing who accessed the secret.

---

### 10. Viewing and Managing Resource Policies

You can view or manage resource policies using:

- AWS Console
- AWS CLI
- AWS SDK

Example CLI command:

    aws secretsmanager get-resource-policy \
      --secret-id prod/database/credentials

To attach a policy:

    aws secretsmanager put-resource-policy \
      --secret-id prod/database/credentials \
      --resource-policy file://policy.json

---

### 11. Common Cross-Account Architecture Pattern

Large organizations often implement a **central secrets account architecture**.

    Security Account
          |
          | Stores all secrets
          v
    AWS Secrets Manager
      /       |       \
     /        |        \
    App A   App B    App C
 (Account) (Account) (Account)

Benefits:

- Centralized security control
- Easier auditing
- Consistent secret management
- Reduced credential duplication

---

### 12. Best Practices

Follow these best practices when using resource-based policies.

#### Use role-based access instead of users

Roles provide better security and automation support.

---

#### Restrict principals tightly

Allow only specific roles or accounts.

---

#### Combine IAM and resource policies

Both must allow access.

---

#### Use conditions when possible

Restrict access to trusted environments.

---

#### Regularly audit policies

Ensure secrets are not unintentionally exposed.

---

### 13. Summary

AWS Secrets Manager supports **resource-based policies** that allow defining who can access a secret directly on the resource.

Key points:

- Resource policies enable **cross-account access**
- Both **identity policies and resource policies** must allow the action
- Policies define principals, actions, and conditions
- Cross-account sharing is common in **multi-account architectures**

This capability allows organizations to build **secure, centralized secret management systems while still enabling applications across accounts to retrieve credentials safely**.
