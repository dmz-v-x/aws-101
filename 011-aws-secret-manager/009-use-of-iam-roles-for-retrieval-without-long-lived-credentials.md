## Using IAM Roles (EC2, Lambda, ECS Task Roles) to Retrieve Secrets Without Long-Lived Credentials

### 1. The Problem With Long-Lived Credentials

Before IAM roles existed, applications commonly accessed AWS services using **long-lived credentials**, such as:

- Access Key ID
- Secret Access Key

Example:

    AWS_ACCESS_KEY_ID=AKIAxxxxxxxx
    AWS_SECRET_ACCESS_KEY=abcd123456789

These credentials were often stored in:

- Application configuration files
- Environment variables
- Code repositories
- Deployment scripts

This approach creates several serious security risks:

- Credentials may leak through code repositories
- Developers may accidentally expose credentials
- Credentials rarely expire
- Compromised credentials provide long-term access

To solve this problem, AWS introduced **IAM roles with temporary credentials**.

---

### 2. What is an IAM Role?

An **IAM role** is a special type of identity in AWS that provides **temporary security credentials** to applications or services.

Unlike IAM users, roles do not have:

- Permanent access keys
- Static credentials

Instead, AWS automatically generates **temporary credentials** when a role is assumed.

Temporary credentials include:

- Access Key ID
- Secret Access Key
- Session Token

These credentials:

- Expire automatically
- Are rotated automatically
- Are delivered securely to AWS services

This allows applications to access services like **AWS Secrets Manager without storing credentials**.

---

### 3. Why IAM Roles Are Used With Secrets Manager

Applications often need to retrieve secrets such as:

- Database passwords
- API tokens
- Encryption keys
- Service credentials

Using IAM roles allows the application to:

1. Authenticate with AWS securely
2. Retrieve secrets dynamically
3. Avoid storing AWS credentials

This approach provides several benefits:

- Eliminates long-lived credentials
- Supports automatic credential rotation
- Reduces risk of credential leaks
- Enables least-privilege access control

---

### 4. How Role-Based Secret Retrieval Works

When using IAM roles, AWS services obtain credentials from the **Instance Metadata Service (IMDS)** or service identity systems.

The workflow looks like this:

    Application
         |
         | Request secret
         v
    AWS SDK
         |
         | Uses temporary credentials from role
         v
    AWS Secrets Manager
         |
         | IAM policy check
         v
    Secret returned

The application never manages credentials directly.

---

### 5. Using IAM Roles With EC2

EC2 instances can be assigned an **instance role**.

This role allows applications running on the EC2 instance to access AWS services securely.

#### Step 1 — Create IAM Role

Example role name:

    EC2SecretsAccessRole

Policy attached to the role:

    {
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:region:account-id:secret:prod/db/password-*"
    }

This allows the EC2 instance to retrieve a specific secret.

---

#### Step 2 — Attach Role to EC2 Instance

When launching an EC2 instance:

1. Select **IAM Role**
2. Choose `EC2SecretsAccessRole`

AWS automatically attaches the role.

---

#### Step 3 — Application Retrieves Secret

Inside the EC2 instance, the application can retrieve secrets using the AWS SDK.

Example (Node.js concept):

    secretsManager.getSecretValue({
      SecretId: "prod/db/password"
    })

The SDK automatically retrieves credentials from the **Instance Metadata Service**.

No credentials are stored in the application.

---

### 6. Using IAM Roles With AWS Lambda

Lambda functions can also assume IAM roles.

Each Lambda function is assigned an **execution role**.

This role defines what AWS services the Lambda function can access.

---

#### Step 1 — Create Lambda Role

Example role:

    LambdaSecretsAccessRole

Policy:

    {
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:region:account-id:secret:prod/api/key-*"
    }

---

#### Step 2 — Assign Role to Lambda Function

When creating or editing the Lambda function:

1. Select **Execution Role**
2. Attach `LambdaSecretsAccessRole`

---

#### Step 3 — Lambda Retrieves Secret

Example workflow:

    Lambda Function
         |
         | Assume execution role
         v
    Temporary credentials issued
         |
         | Call GetSecretValue
         v
    AWS Secrets Manager

The Lambda function retrieves the secret securely.

No credentials are stored in the function code.

---

### 7. Using IAM Roles With ECS (Task Roles)

Containers running in Amazon ECS also need secure access to secrets.

ECS provides **Task Roles**.

A task role is an IAM role that a container can assume during execution.

---

#### Step 1 — Create ECS Task Role

Example role:

    ECSSecretsAccessRole

Policy:

    {
      "Effect": "Allow",
      "Action": "secretsmanager:GetSecretValue",
      "Resource": "arn:aws:secretsmanager:region:account-id:secret:prod/payment/api-*"
    }

---

#### Step 2 — Attach Role to Task Definition

In ECS task definition:

1. Specify **Task Role**
2. Select `ECSSecretsAccessRole`

---

#### Step 3 — Container Retrieves Secret

Inside the container:

    Application
         |
         | Uses AWS SDK
         v
    Temporary credentials from task role
         |
         v
    AWS Secrets Manager

The container retrieves the secret securely.

---

### 8. Temporary Credential Flow

All AWS services follow a similar process when using IAM roles.

    Service (EC2 / Lambda / ECS)
            |
            | Assume IAM Role
            v
    AWS STS issues temporary credentials
            |
            v
    Application uses temporary credentials
            |
            v
    Call Secrets Manager API

Temporary credentials automatically expire and rotate.

---

### 9. Benefits of Role-Based Secret Access

Using IAM roles provides several advantages.

#### No Hardcoded Credentials

Applications never store AWS credentials.

---

#### Automatic Credential Rotation

Temporary credentials rotate automatically.

---

#### Improved Security

Compromised credentials expire quickly.

---

#### Fine-Grained Access Control

IAM policies restrict which secrets can be accessed.

---

### 10. Best Practices

Follow these best practices when retrieving secrets with IAM roles.

#### Use Least Privilege

Only grant access to specific secrets.

Example:

    arn:aws:secretsmanager:region:account-id:secret:prod/db/password-*

Avoid broad permissions.

---

#### Separate Roles Per Service

Different services should use different roles.

Example:

- EC2 role
- Lambda role
- ECS task role

---

#### Monitor Access

Use **CloudTrail** to monitor secret retrieval activity.

---

#### Avoid Hardcoded Secrets

Always retrieve secrets dynamically using AWS SDK.

---

### 11. Example Architecture

A common architecture looks like this:

    Application Service (EC2 / Lambda / ECS)
                |
                | IAM Role
                v
          AWS STS Temporary Credentials
                |
                v
          AWS Secrets Manager
                |
                v
          Secret Returned

This architecture removes the need for long-lived credentials.

---

### 12. Summary

IAM roles allow AWS services to retrieve secrets securely without storing long-lived credentials.

Supported services include:

- EC2 instance roles
- Lambda execution roles
- ECS task roles

The workflow includes:

1. Service assumes IAM role
2. Temporary credentials are issued
3. Application calls `GetSecretValue`
4. Secrets Manager returns the secret

Using IAM roles with Secrets Manager is considered the **recommended security best practice for accessing secrets in AWS applications**.
