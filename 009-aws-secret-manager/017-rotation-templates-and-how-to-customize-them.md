## AWS Secrets Manager Rotation Templates and How to Customize Them

### 1. Introduction

AWS Secrets Manager supports **automatic secret rotation** using **Lambda rotation functions**.

To simplify the implementation of rotation logic, AWS provides **rotation templates**. These templates are pre-built Lambda function examples that implement the standard rotation workflow.

Rotation templates help developers:

- quickly enable secret rotation
- follow AWS recommended rotation patterns
- reduce the complexity of implementing rotation logic

Instead of writing rotation code from scratch, users can start with an AWS template and **customize it for their environment**.

---

### 2. What Are Rotation Templates?

Rotation templates are **pre-built Lambda function templates** provided by AWS that implement the standard **four-step secret rotation process**.

These templates include logic for:

- generating new credentials
- updating the target system
- validating the credentials
- promoting the new secret version

The templates are designed for common services such as databases.

---

### 3. Why AWS Provides Rotation Templates

Secret rotation requires careful coordination between:

- Secrets Manager
- the target system (database, API service, etc.)
- credential validation

Implementing this logic manually can be complex.

Rotation templates help by providing:

- a tested rotation framework
- secure credential handling patterns
- integration with Secrets Manager staging labels

Developers can then modify the template to match their infrastructure.

---

### 4. The Standard Rotation Workflow

All rotation templates follow the **same four-step rotation process** defined by AWS Secrets Manager.

These steps are triggered sequentially during rotation.

1. createSecret  
2. setSecret  
3. testSecret  
4. finishSecret  

Each step performs a specific part of the rotation process.

---

### 5. Step 1 — createSecret

This step generates the new credential.

Typical actions:

- generate a new password
- create a new API key
- store the credential in Secrets Manager

Example logic:

```
Generate new credential
Store secret version with staging label AWSPENDING
```

This step does not yet update the external system.

---

### 6. Step 2 — setSecret

In this step, the new credential is applied to the target system.

Examples include:

- updating the database password
- updating authentication credentials
- registering a new API key

Example logic:

```
Update database user password
Apply new credential to system
```

At this stage, both credentials may temporarily work.

---

### 7. Step 3 — testSecret

The rotation function must verify that the new credential works.

Typical tests include:

- connecting to the database
- performing an authentication request
- validating API access

Example logic:

```
Attempt login using new credential
If authentication fails → rotation stops
```

This ensures the new secret is valid before promoting it.

---

### 8. Step 4 — finishSecret

If the credential works, the rotation function promotes the new secret.

Example:

```
AWSPENDING → AWSCURRENT
Old secret → AWSPREVIOUS
```

After this step:

- applications automatically retrieve the new secret
- the previous credential becomes inactive

---

### 9. AWS Rotation Template Examples

AWS provides rotation templates for common databases.

Examples include:

- MySQL rotation template
- PostgreSQL rotation template
- Oracle rotation template
- Microsoft SQL Server rotation template
- Amazon Redshift rotation template

These templates already contain logic for:

- connecting to the database
- updating user passwords
- testing authentication

---

### 10. Template Programming Languages

Most AWS rotation templates are written in:

- **Python**
- **Node.js**

These Lambda functions use AWS SDK libraries to interact with:

- Secrets Manager
- the target service
- external systems

---

### 11. Example Rotation Template Structure

A typical Lambda rotation template contains functions like:

```
lambda_handler()

create_secret()

set_secret()

test_secret()

finish_secret()
```

The Lambda handler determines which rotation step is being executed.

Example flow:

```
Secrets Manager triggers Lambda
Lambda checks rotation step
Calls corresponding function
```

---

### 12. Simplified Template Logic

Example pseudo-code structure:

```
def lambda_handler(event, context):

    step = event["Step"]

    if step == "createSecret":
        create_secret()

    elif step == "setSecret":
        set_secret()

    elif step == "testSecret":
        test_secret()

    elif step == "finishSecret":
        finish_secret()
```

This logic ensures the correct rotation step executes at the correct time.

---

### 13. Customizing Rotation Templates

While AWS provides base templates, most real systems require customization.

Customization typically involves:

- modifying credential generation logic
- integrating with custom systems
- adding validation steps
- supporting external APIs

---

### 14. Example Customization — Password Rules

A database might require specific password policies.

Example requirements:

- minimum length
- special characters
- restricted symbols

You can modify the template to generate compliant passwords.

Example:

```
Generate password with:
- 16 characters
- numbers
- uppercase letters
- special symbols
```

---

### 15. Example Customization — External API Credentials

Suppose you want to rotate an API key for a SaaS provider.

The rotation template can be customized to:

1. call the provider API
2. generate a new key
3. deactivate the old key
4. update Secrets Manager

Example workflow:

```
Lambda function
      |
      v
External API request
      |
      v
New API key generated
      |
      v
Secret updated
```

---

### 16. Example Customization — Multi-User Rotation

Some database rotation strategies use **alternating users**.

Example users:

```
app_user_a
app_user_b
```

Rotation workflow:

```
Active user = app_user_a
Rotate → activate app_user_b
Next rotation → activate app_user_a
```

This ensures existing connections remain valid.

The rotation template can be modified to support this strategy.

---

### 17. Testing Custom Rotation Templates

Before deploying rotation templates in production, they must be tested carefully.

Testing steps include:

1. Deploy template in staging environment
2. Trigger manual rotation
3. Verify external system updates
4. Validate application authentication
5. Monitor logs for errors

Proper testing prevents production outages.

---

### 18. Monitoring Rotation Functions

Rotation functions run inside **AWS Lambda**, so monitoring is important.

Useful tools include:

- CloudWatch Logs
- CloudWatch Metrics
- CloudTrail

Logs help detect:

- rotation failures
- authentication errors
- API failures

---

### 19. Best Practices

Follow these best practices when working with rotation templates.

#### Start with AWS templates

Avoid writing rotation logic from scratch.

---

#### Customize carefully

Ensure templates match your system's authentication process.

---

#### Test rotation in staging

Never deploy untested rotation code to production.

---

#### Monitor rotation events

Track rotation success and failure using CloudWatch.

---

### 20. Summary

AWS Secrets Manager rotation templates provide a **starting framework for implementing automated credential rotation**.

They implement a standardized four-step rotation process:

1. createSecret  
2. setSecret  
3. testSecret  
4. finishSecret  

Templates are available for several database services and can be customized for:

- external APIs
- custom authentication systems
- specialized password policies

By customizing rotation templates carefully, organizations can build **secure and automated secret rotation workflows across both AWS services and external systems**.
