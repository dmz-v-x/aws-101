## How AWS Secrets Manager Integrates with AWS KMS (Default AWS Managed CMK vs Customer Managed CMK)

### 1. Introduction

AWS Secrets Manager stores highly sensitive information such as:

- Database passwords
- API keys
- OAuth tokens
- Service credentials
- Encryption secrets

Because this data is critical, AWS Secrets Manager always **encrypts secrets at rest**.

To perform encryption securely, Secrets Manager integrates with **AWS Key Management Service (AWS KMS)**.

KMS provides the **master encryption keys** used to protect secrets.

When you store a secret, AWS Secrets Manager:

1. Requests encryption keys from AWS KMS
2. Encrypts the secret using envelope encryption
3. Stores the encrypted secret securely

Understanding this integration is important because it determines:

- Who can decrypt secrets
- Which services can use encryption keys
- How encryption is audited
- How keys are rotated

---

### 2. What is AWS KMS?

AWS KMS is a **managed key management service** that allows you to create and control cryptographic keys used for encryption.

KMS provides:

- Secure key storage
- Hardware Security Module (HSM) protection
- Encryption and decryption APIs
- Key usage auditing
- Access control via IAM policies

Instead of managing encryption keys manually, AWS services use KMS to perform encryption operations securely.

---

### 3. How Secrets Manager Uses KMS

Whenever a secret is stored or retrieved, AWS Secrets Manager interacts with KMS.

The workflow looks like this:

    Application
        |
        | Store Secret
        v
    AWS Secrets Manager
        |
        | Request data key
        v
    AWS KMS
        |
        | Return encrypted + plaintext data key
        v
    Secret encrypted
        |
        v
    Encrypted secret stored

During retrieval:

    Application
        |
        v
    AWS Secrets Manager
        |
        | Send encrypted data key
        v
    AWS KMS decrypts data key
        |
        v
    Secret decrypted and returned

Secrets Manager never exposes encryption keys directly.

---

### 4. Types of KMS Keys Used by Secrets Manager

Secrets Manager supports two types of KMS keys:

1. **AWS Managed Keys**
2. **Customer Managed Keys (CMK)**

Both encrypt secrets, but they differ in **control, customization, and governance**.

---

### 5. AWS Managed KMS Key (Default Option)

When you create a secret without specifying a KMS key, AWS automatically uses an **AWS-managed key**.

The default key used is:

    aws/secretsmanager

This key is created automatically by AWS.

---

### Characteristics of AWS Managed Keys

AWS-managed keys have the following characteristics:

- Created automatically by AWS
- Managed entirely by AWS
- Automatically rotated
- Used only by the specific AWS service
- Minimal configuration required

Users do not manage these keys directly.

---

### Example Encryption Flow with AWS Managed Key

    Secret
       |
       | Request encryption
       v
    Secrets Manager
       |
       | Uses aws/secretsmanager key
       v
    AWS KMS encrypts data key
       |
       v
    Secret stored securely

This is the **simplest setup**.

---

### Advantages of AWS Managed Keys

AWS managed keys are easy to use.

Benefits include:

- No key management required
- Automatic rotation handled by AWS
- Secure by default
- Quick setup

They are suitable for:

- Development environments
- Small applications
- Basic security requirements

---

### Limitations of AWS Managed Keys

However, AWS managed keys have some limitations.

You cannot:

- Modify key policies
- Control key usage in detail
- Restrict access to specific roles
- Disable the key
- Schedule key deletion

Because AWS manages the key, customization is limited.

---

### 6. Customer Managed KMS Keys (CMK)

Customer-managed keys are **KMS keys created and controlled by the user**.

These keys provide **full control over encryption policies**.

Example CMK ARN:

    arn:aws:kms:us-east-1:123456789012:key/abcd1234-5678-90ef-1234-abcdef567890

When creating a secret, you can choose this key instead of the default.

---

### Example Secret Creation Using CMK

    aws secretsmanager create-secret \
      --name prod/payment/api-key \
      --kms-key-id arn:aws:kms:us-east-1:123456789012:key/abcd1234 \
      --secret-string '{"apiKey":"abc123"}'

This instructs Secrets Manager to use the specified CMK.

---

### Advantages of Customer Managed Keys

Customer-managed keys provide advanced capabilities.

---

#### Fine-Grained Access Control

You can control which roles or users can use the key.

Example:

    Allow only payment-service role to decrypt secrets

---

#### Cross-Account Access

CMKs support secure cross-account encryption.

This is useful in **multi-account architectures**.

---

#### Key Policy Customization

You can define detailed key policies controlling:

- who can encrypt
- who can decrypt
- which services can use the key

---

#### Key Rotation Control

You can enable automatic rotation or rotate keys manually.

---

#### Key Deletion Scheduling

You can schedule key deletion if the key is no longer needed.

---

### 7. Encryption Flow with Customer Managed Key

When using a CMK, the encryption flow looks like this:

    Secret
        |
        v
    Secrets Manager
        |
        | Request data key
        v
    AWS KMS (Customer Managed Key)
        |
        | Return encrypted data key
        v
    Secret encrypted
        |
        v
    Encrypted secret stored

The CMK protects the **data encryption key** used for the secret.

---

### 8. Security Comparison

| Feature | AWS Managed Key | Customer Managed Key |
|------|------|------|
| Created by | AWS | Customer |
| Key policy control | No | Yes |
| Access restrictions | Limited | Full control |
| Cross-account access | Limited | Supported |
| Key deletion | Not allowed | Allowed |
| Key rotation | Automatic (AWS controlled) | Optional (customer controlled) |

Customer managed keys provide **greater flexibility and control**.

---

### 9. When to Use Each Type

Use **AWS managed keys** when:

- Security requirements are simple
- You want minimal configuration
- You are building prototypes or small systems

Use **customer managed keys** when:

- Working in production environments
- Compliance requirements exist
- You need strict access control
- Cross-account encryption is required
- Auditing and governance are important

Most enterprise environments use **customer-managed keys**.

---

### 10. Monitoring Encryption Operations

Every encryption or decryption operation performed by KMS is recorded in **AWS CloudTrail**.

Example logged events:

- Encrypt
- Decrypt
- GenerateDataKey

This allows security teams to audit:

- Who accessed a secret
- When decryption occurred
- Which role performed the action

---

### 11. Best Practices

Follow these best practices when integrating Secrets Manager with KMS.

#### Use customer managed keys for production secrets

This gives better control and auditing.

---

#### Restrict key access

Limit which IAM roles can use the key.

---

#### Use kms:ViaService condition

Ensure the key is only used by Secrets Manager.

---

#### Monitor key usage

Enable CloudTrail logging for all KMS operations.

---

### 12. Summary

AWS Secrets Manager integrates tightly with **AWS KMS** to protect secrets using envelope encryption.

Two types of KMS keys can be used:

| Key Type | Description |
|------|------|
| AWS Managed Key | Automatically created and managed by AWS |
| Customer Managed Key (CMK) | Created and fully controlled by the user |

AWS-managed keys are simple and require minimal setup.

Customer-managed keys provide:

- detailed access control
- compliance support
- cross-account security
- stronger governance

Understanding how Secrets Manager integrates with KMS is essential for designing **secure secret management systems in AWS**.
