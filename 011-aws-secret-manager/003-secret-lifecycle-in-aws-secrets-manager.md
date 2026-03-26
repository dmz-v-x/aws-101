## Secret Lifecycle in AWS Secrets Manager

### 1. Understanding the Secret Lifecycle

A **secret lifecycle** describes the entire journey of a secret from the moment it is created until it is deleted or restored.

In AWS Secrets Manager, a secret goes through several stages during its lifetime. These stages allow secure management of sensitive credentials while ensuring that secrets can be updated, rotated, or recovered when needed.

The major stages in the lifecycle of a secret are:

1. Create a secret  
2. Retrieve a secret (`GetSecretValue`)  
3. Update a secret (`PutSecretValue` / `UpdateSecret`)  
4. Delete a secret  
5. Restore a secret  

Understanding this lifecycle is essential because every real-world system that uses Secrets Manager interacts with these operations regularly.

---

### 2. Stage 1 — Creating a Secret

The lifecycle begins when a **secret is created**.

Creating a secret means storing sensitive data securely inside AWS Secrets Manager.

When a secret is created:

- The secret value is encrypted using **AWS KMS**
- Metadata is stored for management
- The secret gets a **unique name**
- The first version of the secret is created

Typical data stored when creating a secret:

- Database credentials
- API keys
- Authentication tokens
- Application configuration secrets

Example of a JSON secret:

    {
      "username": "admin",
      "password": "SuperSecurePassword123",
      "host": "db.example.com",
      "port": 5432
    }

#### CLI Example

    aws secretsmanager create-secret \
        --name prod/database/credentials \
        --secret-string '{"username":"admin","password":"password123"}'

What happens internally:

1. AWS generates a **data encryption key** using AWS KMS.
2. The secret value is encrypted.
3. The encrypted secret is stored securely.
4. A version labeled **AWSCURRENT** is created.

---

### 3. Stage 2 — Retrieving a Secret (GetSecretValue)

Applications need to access secrets at runtime.

This is done using the **GetSecretValue API operation**.

Typical examples:

- An application retrieving a database password
- A microservice retrieving an API key
- A Lambda function retrieving authentication credentials

#### CLI Example

    aws secretsmanager get-secret-value \
        --secret-id prod/database/credentials

The response includes:

- Secret name
- Secret value
- Version ID
- Version stage (for example `AWSCURRENT`)

Example output:

    {
      "Name": "prod/database/credentials",
      "SecretString": "{\"username\":\"admin\",\"password\":\"password123\"}",
      "VersionStages": ["AWSCURRENT"]
    }

Applications typically parse this value and use it in their configuration.

#### Important Notes

- Access is controlled using **IAM permissions**
- Every retrieval is logged in **AWS CloudTrail**
- Applications should **cache secrets** instead of requesting them repeatedly

---

### 4. Stage 3 — Updating a Secret

Secrets often need to change over time.

For example:

- Database passwords must be rotated
- API tokens may expire
- Credentials may need to be replaced after a security event

AWS Secrets Manager supports updating secrets using two APIs:

- `PutSecretValue`
- `UpdateSecret`

---

#### 4.1 PutSecretValue

`PutSecretValue` is used when you want to **add a new version of the secret value**.

It does not change metadata. It only updates the value.

Example:

    aws secretsmanager put-secret-value \
        --secret-id prod/database/credentials \
        --secret-string '{"username":"admin","password":"NewPassword123"}'

What happens:

1. A **new version** of the secret is created.
2. The new version becomes **AWSCURRENT**.
3. The previous version becomes **AWSPREVIOUS**.

This versioning system allows safe updates and rollback if needed.

---

#### 4.2 UpdateSecret

`UpdateSecret` is used when modifying **secret metadata or configuration**.

Examples include:

- Changing the secret description
- Changing the KMS encryption key
- Updating the secret value along with metadata

Example:

    aws secretsmanager update-secret \
        --secret-id prod/database/credentials \
        --description "Production database credentials"

---

### 5. Stage 4 — Deleting a Secret

Sometimes secrets are no longer needed.

For example:

- A service is decommissioned
- Credentials are replaced permanently
- A system is migrated

Secrets Manager allows deleting secrets using the **DeleteSecret API**.

However, deletion is **not immediate by default**.

Instead, AWS places the secret into a **scheduled deletion state**.

Example CLI command:

    aws secretsmanager delete-secret \
        --secret-id prod/database/credentials \
        --recovery-window-in-days 7

Important details:

- Recovery window can be **7 to 30 days**
- During this period the secret cannot be retrieved
- The secret can still be restored if deletion was accidental

If immediate deletion is required:

    aws secretsmanager delete-secret \
        --secret-id prod/database/credentials \
        --force-delete-without-recovery

This permanently deletes the secret immediately.

Use this option carefully.

---

### 6. Stage 5 — Restoring a Deleted Secret

If a secret was deleted accidentally, AWS allows restoring it during the recovery window.

The **RestoreSecret API** cancels the scheduled deletion.

Example CLI command:

    aws secretsmanager restore-secret \
        --secret-id prod/database/credentials

After restoration:

- The secret becomes active again
- Applications can retrieve it normally
- No data is lost

This recovery feature helps prevent accidental credential loss.

---

### 7. Secret Version Lifecycle

Every secret has **multiple versions over time**.

AWS tracks versions using **staging labels**.

Common labels include:

| Label | Meaning |
|------|------|
| AWSCURRENT | Current active secret |
| AWSPREVIOUS | Previous version of the secret |
| AWSPENDING | New secret during rotation |

Example lifecycle during rotation:

    Version 1 → AWSCURRENT

After update:

    Version 1 → AWSPREVIOUS
    Version 2 → AWSCURRENT

This system allows safe credential rotation.

---

### 8. Example Real-World Secret Lifecycle

Consider a production database password.

Step 1 — Secret Created

    prod/database/password

Step 2 — Application retrieves password using GetSecretValue.

Step 3 — Password rotated every 30 days.

    PutSecretValue → new version created.

Step 4 — Old password becomes AWSPREVIOUS.

Step 5 — If service is retired:

    DeleteSecret scheduled.

Step 6 — If deletion was accidental:

    RestoreSecret restores access.

---

### 9. Best Practices for Managing the Secret Lifecycle

Follow these practices when managing secrets.

#### Enable automatic rotation

This reduces the risk of long-lived credentials.

#### Use least privilege IAM policies

Only allow applications to retrieve secrets they need.

#### Avoid hardcoding secrets

Always retrieve secrets dynamically.

#### Use versioning for safe updates

Never overwrite secrets without version tracking.

#### Monitor secret usage

Use CloudTrail to audit secret access.

---

### 10. Summary

The **secret lifecycle in AWS Secrets Manager** includes the complete process of managing secrets securely.

The lifecycle stages include:

1. **Create** — store sensitive data securely  
2. **Retrieve** — access secret using `GetSecretValue`  
3. **Update** — modify secret using `PutSecretValue` or `UpdateSecret`  
4. **Delete** — schedule deletion of secrets  
5. **Restore** — recover secrets during the recovery window  

Secrets also maintain **version history** using staging labels such as:

- `AWSCURRENT`
- `AWSPREVIOUS`
- `AWSPENDING`

Understanding this lifecycle is fundamental to securely managing credentials and building applications that rely on **AWS Secrets Manager**.
