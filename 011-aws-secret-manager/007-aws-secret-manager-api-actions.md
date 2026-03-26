## AWS Secrets Manager API Actions and Required IAM Permissions

### 1. Why IAM Permissions Are Important for Secrets Manager

AWS Secrets Manager stores **sensitive credentials**, so access to secrets must be tightly controlled.

AWS uses **IAM (Identity and Access Management)** to control who can:

- Create secrets
- Retrieve secrets
- Update secrets
- Rotate secrets
- Delete secrets
- Manage metadata

Each operation in Secrets Manager corresponds to an **API action**, and each API action requires a specific **IAM permission**.

If a user, role, or service does not have the required permission, AWS returns:

    AccessDeniedException

Understanding these permissions is critical when designing **secure production systems**.

---

### 2. Structure of IAM Permissions

IAM permissions follow this general format:

    service:Action

For AWS Secrets Manager:

    secretsmanager:ActionName

Example:

    secretsmanager:GetSecretValue

An IAM policy grants permissions like this:

    {
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "*"
    }

This allows retrieving secrets.

In real environments, permissions should be **restricted to specific secrets**, not all resources.

---

### 3. Core Secrets Manager API Actions

These are the most important API actions used when working with secrets.

---

### 4. CreateSecret

#### Purpose

Creates a new secret in AWS Secrets Manager.

#### Required IAM Permission

    secretsmanager:CreateSecret

#### Example Use Cases

- Storing database credentials
- Storing API keys
- Storing application tokens

Example CLI command:

    aws secretsmanager create-secret \
        --name prod/database/password \
        --secret-string "MySecurePassword123"

Without the `CreateSecret` permission, users cannot create new secrets.

---

### 5. GetSecretValue

#### Purpose

Retrieves the secret value.

This is the **most commonly used API action**, because applications use it to access credentials at runtime.

#### Required IAM Permission

    secretsmanager:GetSecretValue

Example CLI command:

    aws secretsmanager get-secret-value \
        --secret-id prod/database/password

This operation returns:

- Secret value
- Version ID
- Version stage

Important security note:

Applications should only have permission to **retrieve specific secrets they need**, not all secrets.

---

### 6. PutSecretValue

#### Purpose

Adds a **new version of a secret value**.

It does not change metadata, only the secret content.

#### Required IAM Permission

    secretsmanager:PutSecretValue

Example CLI command:

    aws secretsmanager put-secret-value \
        --secret-id prod/database/password \
        --secret-string "NewPassword456"

What happens internally:

- A new version is created
- The staging label `AWSCURRENT` moves to the new version
- The previous version becomes `AWSPREVIOUS`

This API is commonly used during **secret rotation**.

---

### 7. UpdateSecret

#### Purpose

Updates **secret metadata or configuration**.

Examples:

- Change description
- Change KMS encryption key
- Update secret value
- Modify other secret settings

#### Required IAM Permission

    secretsmanager:UpdateSecret

Example CLI command:

    aws secretsmanager update-secret \
        --secret-id prod/database/password \
        --description "Updated production DB credentials"

---

### 8. DeleteSecret

#### Purpose

Schedules a secret for deletion.

By default, deletion happens after a **recovery window (7–30 days)**.

#### Required IAM Permission

    secretsmanager:DeleteSecret

Example CLI command:

    aws secretsmanager delete-secret \
        --secret-id prod/database/password \
        --recovery-window-in-days 7

This prevents accidental data loss.

Immediate deletion requires:

    --force-delete-without-recovery

---

### 9. RestoreSecret

#### Purpose

Cancels scheduled deletion and restores the secret.

#### Required IAM Permission

    secretsmanager:RestoreSecret

Example CLI command:

    aws secretsmanager restore-secret \
        --secret-id prod/database/password

This is useful when a secret was **deleted accidentally**.

---

### 10. DescribeSecret

#### Purpose

Retrieves **metadata about the secret**, not the secret value.

Information returned includes:

- Secret name
- Description
- KMS key
- Tags
- Rotation configuration

#### Required IAM Permission

    secretsmanager:DescribeSecret

Example CLI command:

    aws secretsmanager describe-secret \
        --secret-id prod/database/password

---

### 11. ListSecrets

#### Purpose

Lists all secrets in an AWS account.

#### Required IAM Permission

    secretsmanager:ListSecrets

Example CLI command:

    aws secretsmanager list-secrets

Administrators often need this permission for **inventory management**.

---

### 12. RotateSecret

#### Purpose

Triggers secret rotation manually.

#### Required IAM Permission

    secretsmanager:RotateSecret

Example CLI command:

    aws secretsmanager rotate-secret \
        --secret-id prod/database/password

Rotation may trigger a **Lambda rotation function**.

---

### 13. TagResource and UntagResource

#### Purpose

Manage tags attached to secrets.

Required permissions:

    secretsmanager:TagResource
    secretsmanager:UntagResource

Example CLI command:

    aws secretsmanager tag-resource \
        --secret-id prod/database/password \
        --tags Key=Environment,Value=Production

Tags help organize secrets and enforce governance.

---

### 14. GetRandomPassword

#### Purpose

Generates a random password using AWS Secrets Manager.

This helps create secure credentials during automation.

Required permission:

    secretsmanager:GetRandomPassword

Example CLI command:

    aws secretsmanager get-random-password

This can generate passwords with specific constraints such as length and character types.

---

### 15. Common Secrets Manager Permissions Summary

| API Action | IAM Permission | Purpose |
|------|------|------|
| CreateSecret | secretsmanager:CreateSecret | Create a new secret |
| GetSecretValue | secretsmanager:GetSecretValue | Retrieve secret value |
| PutSecretValue | secretsmanager:PutSecretValue | Add new secret version |
| UpdateSecret | secretsmanager:UpdateSecret | Update metadata or value |
| DeleteSecret | secretsmanager:DeleteSecret | Schedule secret deletion |
| RestoreSecret | secretsmanager:RestoreSecret | Restore deleted secret |
| DescribeSecret | secretsmanager:DescribeSecret | View secret metadata |
| ListSecrets | secretsmanager:ListSecrets | List all secrets |
| RotateSecret | secretsmanager:RotateSecret | Trigger secret rotation |
| TagResource | secretsmanager:TagResource | Add tags to secret |
| UntagResource | secretsmanager:UntagResource | Remove tags |
| GetRandomPassword | secretsmanager:GetRandomPassword | Generate secure password |

---

### 16. Example IAM Policy for Application Access

Example policy allowing an application to **read only one secret**.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "secretsmanager:GetSecretValue",
          "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/database/password-*"
        }
      ]
    }

This follows the **least privilege principle**.

---

### 17. Best Practices for Secrets Manager Permissions

Follow these best practices when designing IAM permissions.

#### Use least privilege

Applications should only access the secrets they need.

#### Avoid wildcard access

Avoid:

    "Resource": "*"

Use specific secret ARNs.

#### Separate admin and application permissions

Admins may need full access, but applications should only retrieve secrets.

#### Monitor access using CloudTrail

All API calls to Secrets Manager are logged.

---

### 18. Summary

AWS Secrets Manager operations are performed through **API actions**, and each action requires a corresponding **IAM permission**.

Important permissions include:

- `secretsmanager:GetSecretValue` — retrieve secrets  
- `secretsmanager:CreateSecret` — create secrets  
- `secretsmanager:PutSecretValue` — add new secret versions  
- `secretsmanager:UpdateSecret` — modify secret configuration  
- `secretsmanager:DeleteSecret` — delete secrets  

Proper IAM policy design ensures that secrets remain **secure, controlled, and accessible only to authorized services and users**.
