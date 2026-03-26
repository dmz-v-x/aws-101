## Writing a Rotation Lambda for AWS Secrets Manager — Basic Skeleton and Rotation Steps

### 1. Introduction

AWS Secrets Manager supports **automatic secret rotation** using **AWS Lambda functions**.

When rotation is enabled, Secrets Manager invokes a Lambda function that performs the rotation logic. This function interacts with:

- AWS Secrets Manager
- the target system (database, API service, etc.)
- credential validation mechanisms

AWS defines a **standard rotation workflow** consisting of four steps.

These steps are executed sequentially by the Lambda function.

1. `createSecret`
2. `setSecret`
3. `testSecret`
4. `finishSecret`

Every rotation Lambda must implement these steps.

---

### 2. Rotation Workflow Overview

The rotation process follows this sequence:

```
Secrets Manager
      |
      | Trigger rotation
      v
Lambda Rotation Function
      |
      | createSecret
      | setSecret
      | testSecret
      | finishSecret
      v
Secret successfully rotated
```

Each step performs a specific operation during the rotation lifecycle.

---

### 3. Lambda Event Structure

When Secrets Manager triggers the Lambda function, it sends an event containing information about the rotation request.

Example event payload:

```
{
  "Step": "createSecret",
  "SecretId": "my-database-secret",
  "ClientRequestToken": "unique-rotation-token"
}
```

Key fields include:

| Field | Purpose |
|------|---------|
| Step | Rotation step being executed |
| SecretId | The secret being rotated |
| ClientRequestToken | Version identifier for the new secret |

The Lambda function reads the **Step field** to determine which operation to execute.

---

### 4. Basic Lambda Rotation Skeleton

Below is a simplified skeleton of a rotation Lambda function.

Example (Python-style pseudocode):

```
def lambda_handler(event, context):

    step = event["Step"]
    secret_id = event["SecretId"]
    token = event["ClientRequestToken"]

    if step == "createSecret":
        create_secret(secret_id, token)

    elif step == "setSecret":
        set_secret(secret_id, token)

    elif step == "testSecret":
        test_secret(secret_id, token)

    elif step == "finishSecret":
        finish_secret(secret_id, token)

    else:
        raise ValueError("Invalid rotation step")
```

This structure ensures that the correct function executes based on the rotation stage.

---

### 5. Step 1 — createSecret

The **createSecret** step generates the new credential.

This credential becomes the **pending version** of the secret.

Typical operations:

- Generate a new password
- Generate a new API key
- Store the value in Secrets Manager with the `AWSPENDING` label

Example logic:

```
Generate new password
Store secret version with label AWSPENDING
```

Example pseudocode:

```
def create_secret(secret_id, token):

    new_password = generate_password()

    store_secret_value(
        secret_id,
        token,
        new_password,
        stage="AWSPENDING"
    )
```

At this stage, the external system is **not yet updated**.

---

### 6. Step 2 — setSecret

The **setSecret** step updates the target system with the new credential.

Examples include:

- updating a database password
- updating an API key
- updating authentication credentials

Example workflow:

```
Retrieve AWSPENDING secret
Update external system credential
```

Example pseudocode:

```
def set_secret(secret_id, token):

    pending_secret = get_secret_value(secret_id, stage="AWSPENDING")

    update_database_password(pending_secret)
```

This step ensures the external system recognizes the new credential.

---

### 7. Step 3 — testSecret

The **testSecret** step verifies that the new credential works.

This validation is critical before promoting the secret.

Typical tests include:

- database login attempt
- API authentication request
- service connectivity check

Example logic:

```
Attempt login using pending credential
If authentication fails → rotation stops
```

Example pseudocode:

```
def test_secret(secret_id, token):

    pending_secret = get_secret_value(secret_id, stage="AWSPENDING")

    if not authenticate(pending_secret):
        raise Exception("Credential validation failed")
```

If the test fails, the rotation process stops.

---

### 8. Step 4 — finishSecret

The **finishSecret** step promotes the new secret version.

This step moves staging labels.

Before promotion:

```
Old version → AWSCURRENT
New version → AWSPENDING
```

After promotion:

```
Old version → AWSPREVIOUS
New version → AWSCURRENT
```

Example pseudocode:

```
def finish_secret(secret_id, token):

    update_secret_version_stage(
        secret_id,
        version_token=token,
        stage="AWSCURRENT"
    )
```

Once this step completes, the new secret becomes the active credential.

---

### 9. Complete Rotation Flow

The full rotation lifecycle looks like this:

```
Step 1: createSecret
    |
    | Generate new credential
    v
Secret stored as AWSPENDING

Step 2: setSecret
    |
    | Apply credential to external system
    v
System updated

Step 3: testSecret
    |
    | Validate credential
    v
Authentication successful

Step 4: finishSecret
    |
    | Promote secret
    v
New credential becomes AWSCURRENT
```

---

### 10. Handling Failures

Rotation functions must handle errors carefully.

If any step fails:

- the rotation process stops
- the previous credential remains active

Example failure scenario:

```
createSecret → success
setSecret → success
testSecret → fails
```

Result:

```
Rotation aborted
AWSCURRENT remains unchanged
```

This prevents broken credentials from being deployed.

---

### 11. Required IAM Permissions

The rotation Lambda function needs permissions to:

- retrieve secrets
- update secrets
- decrypt secrets using KMS

Example IAM permissions:

```
secretsmanager:GetSecretValue
secretsmanager:PutSecretValue
secretsmanager:UpdateSecretVersionStage
kms:Decrypt
kms:GenerateDataKey
```

These permissions allow the Lambda function to interact with Secrets Manager and KMS securely.

---

### 12. Logging and Monitoring

Rotation functions should produce detailed logs for troubleshooting.

Logs typically include:

- rotation step execution
- credential generation
- validation results
- errors or failures

Logs are stored in:

```
CloudWatch Logs
```

Monitoring rotation logs helps detect issues early.

---

### 13. Best Practices

Follow these best practices when implementing rotation Lambda functions.

#### Use secure password generation

Generate strong credentials during `createSecret`.

---

#### Implement proper error handling

Failures should not break production systems.

---

#### Avoid logging secret values

Logs must never expose credentials.

---

#### Test rotation thoroughly

Always test rotation in staging environments.

---

### 14. Summary

AWS Secrets Manager rotation uses a **Lambda-based workflow** consisting of four standardized steps.

| Step | Purpose |
|-----|---------|
| createSecret | Generate and store a new secret version |
| setSecret | Update the external system with the new credential |
| testSecret | Verify the new credential works |
| finishSecret | Promote the new version as the active secret |

Rotation Lambda functions provide a flexible way to rotate credentials for:

- databases
- APIs
- SaaS services
- custom authentication systems

By implementing this structured rotation workflow, organizations can ensure secrets remain **secure, automatically updated, and safe to use across distributed systems**.
