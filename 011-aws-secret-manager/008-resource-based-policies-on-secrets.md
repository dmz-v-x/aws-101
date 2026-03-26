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
