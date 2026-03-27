## Rotation Strategies in AWS Secrets Manager — Managed Rotation vs Lambda Rotation Functions

### 1. Introduction

Secret rotation is a core feature of **AWS Secrets Manager** that automatically updates credentials to reduce security risks.

When a secret is rotated, AWS performs operations such as:

- Generating a new credential
- Updating the external system (database, API service, etc.)
- Testing the credential
- Promoting the credential as the active secret

AWS Secrets Manager supports **two main rotation strategies**:

1. **Managed Rotation (AWS Managed Rotation)**
2. **Custom Rotation using Lambda Functions**

Both strategies automate credential updates but differ in **control, complexity, and supported services**.

Understanding these strategies helps determine the best approach for different workloads.

---

### 2. What is Secret Rotation?

Secret rotation replaces an existing credential with a new one automatically.

Example:

Before rotation:

    Database password = MyPassword123

After rotation:

    Database password = MyPassword456

Secrets Manager updates the secret value and ensures the application can continue retrieving the correct credential.

---

### 3. Managed Rotation (AWS Managed Rotation)

Managed rotation is a feature where **AWS automatically handles the entire rotation process**.

AWS manages the rotation workflow without requiring users to write any custom code.

This method is available for certain AWS services where AWS understands how credentials are structured.

---

### 4. Supported Services for Managed Rotation

Managed rotation currently supports several AWS services.

Examples include:

- Amazon RDS
- Amazon Aurora
- Amazon DocumentDB
- Amazon Redshift

For these services, AWS already knows how to:

- generate new credentials
- update the database user password
- validate the credential

Because AWS knows the service internals, rotation can be performed automatically.

---

### 5. Managed Rotation Workflow

The managed rotation process works like this:

```
Secrets Manager
       |
       | Create new credential
       v
Target service updated
       |
       | Test credential
       v
Credential promoted to active secret
```

AWS internally uses a rotation mechanism similar to the Lambda rotation workflow but **without requiring custom code**.

---

### 6. Example Managed Rotation Scenario

Example:

```
Database: Amazon RDS
Secret: db-credentials
Rotation schedule: every 30 days
```

Rotation process:

1. Secrets Manager generates a new password
2. Updates the RDS database user password
3. Tests the connection
4. Updates the secret value

The entire process is handled by AWS automatically.

---

### 7. Advantages of Managed Rotation

Managed rotation offers several benefits.

#### No Code Required

Users do not need to write Lambda functions.

---

#### Simplified Setup

Rotation can be enabled directly in the console.

---

#### AWS Maintains Rotation Logic

AWS handles credential updates internally.

---

#### Reduced Operational Overhead

There is no need to maintain rotation scripts.

---

### 8. Limitations of Managed Rotation

Managed rotation only works for **specific AWS services**.

Examples of unsupported scenarios include:

- Third-party APIs
- Custom authentication systems
- Self-managed databases
- SaaS service credentials

If the target system is not supported, **custom rotation must be implemented**.

---

### 9. Lambda Rotation Functions

For services that AWS does not support natively, Secrets Manager allows **custom rotation using AWS Lambda**.

In this model, the rotation process is implemented inside a **Lambda function** that performs credential updates.

This provides full flexibility for rotating secrets in **any external system**.

---

### 10. Lambda Rotation Architecture

The rotation workflow looks like this:

```
Secrets Manager
       |
       | Trigger rotation
       v
Lambda Rotation Function
       |
       | Update external system
       v
Secret updated
```

Secrets Manager invokes the Lambda function during rotation events.

---

### 11. Lambda Rotation Steps

A Lambda rotation function follows a standard **four-step rotation process**.

Secrets Manager invokes the Lambda function multiple times with different rotation steps.

---

### Step 1 — createSecret

The function generates a new credential.

Example:

```
Generate new password
Store version with AWSPENDING
```

---

### Step 2 — setSecret

The function updates the external system.

Example:

```
Update database password
Update API key
Update authentication service
```

---

### Step 3 — testSecret

The function verifies that the new credential works.

Example:

```
Attempt login with new credential
Verify API authentication
```

If validation fails, rotation stops.

---

### Step 4 — finishSecret

If testing succeeds, the new credential becomes active.

Example:

```
AWSPENDING → AWSCURRENT
Old secret → AWSPREVIOUS
```

Applications retrieving the secret will now receive the new value.

---

### 12. Example Lambda Rotation Scenario

Example:

```
Service: Stripe API
Secret: stripe-api-key
Rotation schedule: every 60 days
```

Rotation process:

1. Lambda generates new API key
2. Updates Stripe account settings
3. Tests API authentication
4. Updates Secrets Manager

---

### 13. Advantages of Lambda Rotation

Lambda rotation offers several advantages.

#### Supports Any System

Custom rotation works for:

- SaaS APIs
- third-party services
- self-managed databases
- internal authentication systems

---

#### Full Custom Logic

Users can implement any rotation process required.

---

#### Integration with Custom Infrastructure

Lambda functions can interact with:

- external APIs
- internal systems
- container platforms

---

### 14. Challenges of Lambda Rotation

Custom rotation also introduces additional complexity.

---

#### Requires Development Effort

Rotation logic must be implemented manually.

---

#### Operational Maintenance

Lambda functions must be maintained and updated.

---

#### Error Handling

Failures in rotation logic must be handled carefully.

---

#### Security Considerations

Lambda functions require secure IAM permissions.

---

### 15. Example Rotation Strategy Comparison

| Feature | Managed Rotation | Lambda Rotation |
|--------|------------------|-----------------|
| Code required | No | Yes |
| Supported systems | Limited AWS services | Any system |
| Operational complexity | Low | Higher |
| Custom logic support | Limited | Full |
| Maintenance | Minimal | Requires maintenance |

---

### 16. When to Use Managed Rotation

Managed rotation is best for:

- RDS database credentials
- Aurora database users
- DocumentDB credentials
- Redshift authentication

It is ideal when AWS provides built-in rotation support.

---

### 17. When to Use Lambda Rotation

Lambda rotation should be used when:

- rotating API keys
- rotating SaaS credentials
- rotating secrets for custom applications
- rotating credentials for non-AWS systems

This provides maximum flexibility.

---

### 18. Best Practices

Follow these best practices when implementing rotation.

#### Prefer Managed Rotation When Available

Managed rotation reduces operational overhead.

---

#### Use Lambda Rotation for Custom Systems

Custom systems require programmable rotation logic.

---

#### Implement Robust Error Handling

Rotation failures should not break production systems.

---

#### Monitor Rotation Events

Use CloudWatch and CloudTrail to track rotation activity.

---

### 19. Summary

AWS Secrets Manager provides two main rotation strategies.

| Strategy | Description |
|--------|-------------|
| Managed Rotation | AWS automatically rotates supported service credentials |
| Lambda Rotation | Custom rotation implemented using AWS Lambda |

Managed rotation is simple and works best for supported AWS services.

Lambda rotation provides flexibility for rotating credentials in **custom or external systems**.

Choosing the right strategy ensures secrets remain **secure, automatically updated, and safely managed across applications**.
