## Temporary Credentials (STS Intro)

In modern AWS security architecture, **temporary credentials are strongly preferred over long-term credentials**.

Instead of using permanent access keys that never expire, AWS provides a system for generating **short-lived, automatically expiring credentials**.

This system is powered by a service called **AWS STS (Security Token Service)**.

Temporary credentials are one of the most important concepts in secure AWS design.

They are used heavily in:

IAM Roles  
Federated access  
Cross-account access  
EC2 instance roles  
Lambda execution roles  

Understanding temporary credentials is essential for building secure cloud systems.

---

### 1. What Temporary Credentials Are

Temporary credentials are short-lived AWS credentials that automatically expire after a defined period of time.

They consist of:

Access Key ID  
Secret Access Key  
Session Token  

Example structure:

    Access Key ID
    Secret Access Key
    Session Token

The session token is what makes the credential temporary.

These credentials are issued by **AWS STS** when a role is assumed.

---

### 2. Why Temporary Credentials Are Safer

Temporary credentials are safer than long-term access keys for several reasons.

First, they automatically expire.

Even if stolen, they only work for a limited time.

Second, they reduce the need to store permanent secrets.

Applications do not need to embed long-term access keys.

Third, they support fine-grained access control for short periods.

Fourth, they reduce blast radius in case of compromise.

Example scenario:

If a long-term access key is leaked, the attacker can use it indefinitely.

If temporary credentials are leaked and expire in 1 hour, the attack window is very small.

This significantly improves security posture.

---

### 3. Example Without Temporary Credentials (Risky)

Application configuration:

    ACCESS_KEY=AKIA123456EXAMPLE
    SECRET_KEY=abc123secret

These credentials never expire.

If leaked:

Attacker can access AWS until the key is manually revoked.

This creates long-term exposure.

---

### 4. Example With Temporary Credentials (Safer)

Instead of storing permanent keys, an EC2 instance assumes a role.

AWS automatically provides temporary credentials via the instance metadata service.

The credentials look like:

    AccessKeyId
    SecretAccessKey
    Token
    Expiration

These credentials expire automatically, often within one hour.

If compromised:

They stop working after expiration.

No manual rotation required.

---

### 5. Where Temporary Credentials Come From

Temporary credentials are issued by **AWS Security Token Service (STS)**.

STS generates temporary credentials in several scenarios.

Common sources include:

Assuming an IAM role

Federated login (e.g., corporate SSO)

EC2 instance roles

Lambda execution roles

Cross-account access

Whenever a role is assumed, STS generates temporary credentials for that session.

---

### 6. Example — Assume Role Flow

Step 1  
Identity requests to assume a role.

Step 2  
AWS verifies that the identity is allowed to assume the role.

Step 3  
STS issues temporary credentials.

Step 4  
Identity uses temporary credentials to make API calls.

Step 5  
Credentials expire after session duration.

This mechanism avoids long-lived secrets.

---

### 7. Session Duration

Temporary credentials have a limited lifetime.

This is called the **session duration**.

Common durations include:

15 minutes  
1 hour  
Several hours (depending on configuration)

After expiration:

The credentials automatically stop working.

If continued access is required, a new role session must be created.

Shorter session durations increase security but may require more frequent re-authentication.

Longer durations provide convenience but slightly increase risk.

Organizations balance these factors based on security requirements.

---

### 8. Example — Temporary Credentials in AWS CLI

Assume a role using AWS CLI.

Conceptual example:

    aws sts assume-role \
        --role-arn arn:aws:iam::123456789012:role/AdminRole \
        --role-session-name MySession

STS returns:

    AccessKeyId
    SecretAccessKey
    SessionToken
    Expiration

These credentials can now be used for temporary administrative access.

After expiration, they become invalid.

---

### 9. Temporary Credentials in AWS Services

Many AWS services automatically use temporary credentials.

Examples:

EC2 instances using instance roles.

Lambda functions using execution roles.

ECS tasks using task roles.

In these cases:

No access keys are stored in code.

The service retrieves temporary credentials automatically.

This is the recommended secure approach.

---

### 10. Real-World Security Benefit

Imagine an attacker compromises an EC2 server.

If the server uses:

Long-term access keys → attacker gains indefinite access.

Temporary credentials → attacker access expires automatically.

Even if the attacker copies the credentials, they cannot use them after expiration.

This significantly reduces damage potential.

---

### 11. Mental Model

Think of temporary credentials as a **temporary access badge**.

Permanent access keys are like a master key that never expires.

Temporary credentials are like a visitor badge that works for one hour and then automatically stops working.

Modern AWS security relies heavily on temporary credentials rather than permanent keys.

---

### Key Takeaways

Temporary credentials are short-lived AWS credentials that expire automatically.

They are issued by AWS Security Token Service (STS).

Temporary credentials include an access key ID, secret key, and session token.

They are safer than long-term access keys because they limit the exposure window.

Session duration controls how long temporary credentials remain valid.

Temporary credentials are widely used with IAM roles, EC2, Lambda, and cross-account access.

Modern AWS best practice is to prefer temporary credentials over long-lived access keys.
