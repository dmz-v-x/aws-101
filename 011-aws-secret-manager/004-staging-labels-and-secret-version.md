## Staging Labels and Secret Versions in AWS Secrets Manager

### 1. Why Secret Versioning Exists

Secrets rarely remain the same forever. In real systems, credentials must be **rotated regularly** to reduce security risks.

Examples:

- Database passwords must be changed periodically
- API keys may expire
- Compromised credentials must be replaced quickly

However, simply replacing a secret instantly can **break running applications**.

For example:

1. Database password changes
2. Application still uses the old password
3. Application fails to connect

To solve this problem, **AWS Secrets Manager uses secret versioning with staging labels**.

This allows **multiple versions of a secret to exist simultaneously**, while clearly identifying which version is active.

---

### 2. What is a Secret Version?

Each time a secret value changes, AWS Secrets Manager **creates a new version** of that secret.

Instead of overwriting the existing secret, AWS stores:

- Old versions
- Current version
- Pending versions during rotation

Example:

    Version 1 → Initial password
    Version 2 → Rotated password
    Version 3 → Updated credentials

Each version is identified by a **Version ID**.

But version IDs are not easy for applications to work with.  
Instead, AWS uses **staging labels** to indicate which version should be used.

---

### 3. What Are Staging Labels?

**Staging labels** are special tags attached to secret versions that indicate their status in the secret lifecycle.

They act as **aliases pointing to specific versions** of the secret.

Applications usually retrieve secrets using a staging label rather than a version ID.

Common staging labels include:

- `AWSCURRENT`
- `AWSPREVIOUS`
- `AWSPENDING`

These labels help AWS manage safe secret rotation.

---

### 4. AWSCURRENT — The Active Secret

`AWSCURRENT` represents the **currently active secret version**.

This is the version that:

- Applications should use
- Production systems retrieve by default
- Represents the latest valid credential

Example:

    Version ID: 1234
    Staging Label: AWSCURRENT

When an application calls:

    GetSecretValue

AWS automatically returns the version marked as `AWSCURRENT`.

This ensures applications always receive the **active credential**.

---

### 5. AWSPREVIOUS — The Last Secret Version

`AWSPREVIOUS` represents the **previous version of the secret**.

This version is kept temporarily to allow safe transitions during rotation.

Example scenario:

1. Old password is active
2. New password is created
3. Applications slowly update

State:

    Version 1 → AWSPREVIOUS
    Version 2 → AWSCURRENT

This allows systems still using the old credential to **continue working temporarily**.

After rotation completes successfully, the previous version can eventually be removed.

---

### 6. AWSPENDING — The Secret Being Rotated

`AWSPENDING` represents a **new secret version currently undergoing rotation**.

This version is not yet active.

Instead, it is:

- Created during the rotation process
- Tested before activation
- Verified before becoming `AWSCURRENT`

Example:

    Version 1 → AWSCURRENT
    Version 2 → AWSPENDING

The pending secret is validated before switching production systems to it.

---

### 7. How Secret Rotation Uses Staging Labels

AWS Secrets Manager rotation uses a **four-step rotation workflow**.

Each step uses staging labels to control the process safely.

Rotation is typically performed using a **Lambda rotation function**.

---

### Step 1 — Create New Secret

The rotation function generates a new credential.

Example:

- New database password
- New API key

This new version is stored in Secrets Manager with the label:

    AWSPENDING

State:

    Version 1 → AWSCURRENT
    Version 2 → AWSPENDING

The pending version is not yet used by applications.

---

### Step 2 — Set the Secret in the Target System

The new secret is applied to the system using it.

Examples:

- Update database password
- Update API service credentials
- Update authentication server

Now both credentials may exist temporarily.

State:

    Version 1 → AWSCURRENT
    Version 2 → AWSPENDING

---

### Step 3 — Test the New Secret

The rotation function verifies that the new credential works.

Example:

- Attempt database login
- Validate API authentication

If the test fails:

- Rotation stops
- `AWSCURRENT` remains unchanged

If the test succeeds:

Rotation proceeds to the final step.

---

### Step 4 — Promote the Secret

The new secret becomes active.

AWS moves the staging labels:

Before:

    Version 1 → AWSCURRENT
    Version 2 → AWSPENDING

After promotion:

    Version 1 → AWSPREVIOUS
    Version 2 → AWSCURRENT

Now applications automatically retrieve the new secret.

---

### 8. Visualizing the Rotation Process

Initial state:

    Version 1 → AWSCURRENT

During rotation:

    Version 1 → AWSCURRENT
    Version 2 → AWSPENDING

After rotation:

    Version 1 → AWSPREVIOUS
    Version 2 → AWSCURRENT

This ensures a **safe transition between credentials**.

---

### 9. Retrieving Secrets with Staging Labels

Applications can request specific secret versions using staging labels.

Example:

Retrieve current secret:

    aws secretsmanager get-secret-value \
        --secret-id my-secret \
        --version-stage AWSCURRENT

Retrieve previous secret:

    aws secretsmanager get-secret-value \
        --secret-id my-secret \
        --version-stage AWSPREVIOUS

Retrieve pending secret (for testing rotation):

    aws secretsmanager get-secret-value \
        --secret-id my-secret \
        --version-stage AWSPENDING

Normally, applications should only use **AWSCURRENT**.

---

### 10. Why Staging Labels Are Important

Staging labels solve several critical problems.

#### Safe Credential Rotation

Secrets can change without breaking running systems.

#### Zero Downtime

Old credentials remain available temporarily.

#### Rollback Capability

If a new secret fails, the previous version still exists.

#### Controlled Deployment

New credentials can be tested before activation.

---

### 11. Best Practices for Secret Version Management

Follow these best practices when working with secret versions.

#### Always retrieve secrets using AWSCURRENT

Applications should not depend on specific version IDs.

#### Enable automatic rotation

This ensures secrets remain fresh and secure.

#### Test rotation functions carefully

Rotation must verify credentials before promotion.

#### Monitor rotation with CloudWatch

Logs help detect failures in the rotation process.

---

### 12. Summary

AWS Secrets Manager uses **secret versions and staging labels** to safely manage credential changes.

Key staging labels include:

| Label | Purpose |
|------|------|
| AWSCURRENT | Active secret used by applications |
| AWSPREVIOUS | Previous secret version |
| AWSPENDING | New secret being rotated |

During rotation:

1. A new secret is created with `AWSPENDING`
2. The new credential is applied to the system
3. The credential is tested
4. Labels are shifted so the new secret becomes `AWSCURRENT`

This system allows **secure, safe, and automated secret rotation without disrupting applications**.
