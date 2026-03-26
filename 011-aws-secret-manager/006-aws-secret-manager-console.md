## AWS Secrets Manager Console Walk-Through (Create Secret Wizard)

### 1. Introduction

AWS Secrets Manager provides a **web-based console interface** that allows you to create, manage, and monitor secrets without using the CLI or SDK.

The **Create Secret Wizard** in the AWS Console guides you step-by-step through the process of creating a new secret. It helps configure:

- The **secret value**
- The **type of secret**
- The **encryption settings**
- The **secret name and metadata**
- **Automatic rotation**
- **Access control**

Understanding each option in this wizard is important because these settings determine **how the secret is stored, secured, and managed**.

---

### 2. Accessing the Create Secret Wizard

To start creating a secret in the AWS Console:

1. Open the **AWS Management Console**
2. Navigate to **Secrets Manager**
3. Click **Store a new secret**

You will be guided through several configuration pages.

The wizard usually includes these major steps:

1. Choose secret type  
2. Configure secret value  
3. Configure encryption settings  
4. Name and describe the secret  
5. Configure rotation  
6. Review and create

---

### 3. Step 1 — Choose Secret Type

The first step is selecting the **type of secret** you want to store.

The console provides predefined templates for common credential types.

Available options include:

- **Credentials for Amazon RDS database**
- **Credentials for other database**
- **Credentials for Redshift cluster**
- **Credentials for Amazon DocumentDB**
- **Other type of secret**

Each option helps AWS understand the **structure of the secret** and enables certain features like automatic rotation.

---

#### 3.1 Database Credentials (RDS)

This option is used when storing credentials for an **Amazon RDS database**.

Information typically required:

- Database username
- Database password
- Database instance identifier

Benefits of using this template:

- AWS can automatically configure **rotation**
- Rotation Lambda templates are preconfigured
- Integration with the database is simplified

---

#### 3.2 Other Database Credentials

This option is used when storing credentials for **databases not managed directly by AWS RDS**.

Examples:

- Self-managed PostgreSQL
- MySQL running on EC2
- External databases

Required fields may include:

- Username
- Password
- Database engine
- Host
- Port

Rotation can still be configured but may require **custom rotation logic**.

---

#### 3.3 Other Type of Secret

This option is used when storing **general secrets**.

Examples include:

- API keys
- OAuth tokens
- Application configuration secrets
- Service authentication credentials

Here you can choose to store:

- **Plain string secret**
- **JSON structured secret**
- **Binary data**

Example JSON secret:

    {
      "api_key": "abc123",
      "api_secret": "xyz456"
    }

---

### 4. Step 2 — Configure Secret Value

In this step, you enter the **actual secret data**.

The console provides two main formats:

#### Plaintext Key/Value Format

The console allows entering secrets as key-value pairs.

Example:

| Key | Value |
|-----|------|
| username | admin |
| password | MyStrongPassword |

This format automatically generates a **JSON secret** internally.

Example result:

    {
      "username": "admin",
      "password": "MyStrongPassword"
    }

---

#### Plaintext Secret

You can also paste a raw string value.

Example:

    sk_live_abc123456

This is commonly used for:

- API keys
- tokens
- simple passwords

---

#### Binary Secret

You can upload **binary data** such as:

- certificate files
- private keys
- keystores

Secrets Manager will store the binary data securely.

---

### 5. Step 3 — Encryption Settings (KMS Key)

All secrets in AWS Secrets Manager are **encrypted at rest**.

In this step, you choose the **AWS KMS key** used to encrypt the secret.

Options include:

- **AWS managed key** (default)
- **Customer managed KMS key**

---

#### AWS Managed KMS Key

Default option:

    aws/secretsmanager

Characteristics:

- Managed automatically by AWS
- No configuration required
- Suitable for most basic use cases

---

#### Customer Managed KMS Key

You may choose your own KMS key if you need:

- Custom access control
- Compliance requirements
- Advanced encryption policies

Example key:

    arn:aws:kms:region:account-id:key/key-id

Using your own key provides **greater control over encryption access**.

---

### 6. Step 4 — Secret Name and Description

Every secret must have a **unique name within the region**.

Example naming patterns:

    prod/database/password
    dev/payment-service/api-key
    staging/auth-service/jwt-secret

Best practice is to use a **hierarchical naming structure**.

---

#### Secret Description

You can also provide a description explaining the secret’s purpose.

Example:

    "Production PostgreSQL credentials for Order Service"

Descriptions help teams understand the purpose of the secret.

---

#### Tags

You can also add **tags** to organize secrets.

Example:

| Key | Value |
|-----|------|
| Environment | Production |
| Application | OrderService |
| Owner | DevOpsTeam |

Tags help with:

- resource organization
- access control
- cost allocation
- automation

---

### 7. Step 5 — Configure Automatic Rotation

Secrets Manager allows **automatic secret rotation**.

Rotation improves security by **periodically updating credentials**.

In this step, you can choose:

- **Enable automatic rotation**
- **Disable automatic rotation**

---

#### If Rotation is Enabled

You must configure:

- Rotation Lambda function
- Rotation interval (e.g., 30 days)
- Rotation window

AWS provides **rotation templates** for supported services like RDS.

Rotation process typically includes:

1. Create new credential
2. Apply credential to the system
3. Test credential
4. Promote credential to active version

---

### 8. Step 6 — Review and Create

The final page displays a **summary of all configuration settings**.

It includes:

- Secret type
- Secret value format
- Encryption key
- Secret name
- Tags
- Rotation settings

Once everything is confirmed, click:

    Store

AWS then:

1. Encrypts the secret using KMS
2. Creates the secret resource
3. Assigns the staging label **AWSCURRENT**
4. Stores the secret securely

The secret is now available for retrieval via:

- AWS Console
- AWS CLI
- AWS SDK
- AWS API

---

### 9. Viewing the Created Secret

After creation, the secret appears in the **Secrets Manager dashboard**.

The secret details page shows:

- Secret name
- Description
- Tags
- KMS key used
- Secret versions
- Rotation configuration

You can also perform operations such as:

- Retrieve secret value
- Rotate secret
- Edit secret
- Delete secret

---

### 10. Best Practices When Using the Console

Follow these best practices when creating secrets via the console.

#### Use clear naming conventions

Example:

    environment/service/credential-type

---

#### Add meaningful descriptions

This helps teams understand secret usage.

---

#### Enable rotation for sensitive credentials

Especially for:

- database passwords
- service accounts
- API credentials

---

#### Use customer-managed KMS keys for critical workloads

This allows stronger access control and auditing.

---

### 11. Summary

The **Create Secret Wizard** in AWS Secrets Manager provides a guided interface for securely storing secrets.

The wizard typically includes the following steps:

1. Choose secret type  
2. Enter secret value  
3. Select encryption key  
4. Name and describe the secret  
5. Configure rotation  
6. Review and store  

This process ensures that secrets are:

- securely encrypted
- properly organized
- easily retrievable
- safely rotated

Understanding the console workflow is essential for managing secrets effectively in AWS environments.
