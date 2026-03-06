## Common STS APIs

AWS Security Token Service (STS) provides several APIs that generate **temporary security credentials**.

These APIs allow identities to obtain **short-lived credentials** instead of using permanent access keys.

The most commonly used STS APIs are:

AssumeRole  
AssumeRoleWithWebIdentity  
GetSessionToken

Each API is designed for a **different authentication scenario**.

Understanding when and why each API is used is important for designing secure AWS access systems.

---

### 1. AssumeRole

AssumeRole is the **most commonly used STS API**.

It allows an identity to **temporarily assume an IAM role and receive temporary credentials**.

These credentials can then be used to access AWS resources with the permissions defined in that role.

AssumeRole is heavily used for:

Cross-account access  
Applications assuming roles  
Temporary administrative access  
DevOps automation

---

### Example Scenario — Developer Assuming Admin Role

A developer normally has limited permissions.

When performing administrative tasks, they assume an admin role.

Step-by-step flow:

Step 1  
Developer logs in with IAM credentials.

Step 2  
Developer calls the AssumeRole API.

Step 3  
STS verifies the role trust policy.

Step 4  
STS returns temporary credentials.

Step 5  
Developer performs admin tasks using the temporary credentials.

Example CLI command:

    aws sts assume-role \
        --role-arn arn:aws:iam::123456789012:role/AdminRole \
        --role-session-name DevSession

STS response includes:

    AccessKeyId
    SecretAccessKey
    SessionToken
    Expiration

These credentials remain valid until the session expires.

---

### 2. AssumeRoleWithWebIdentity

AssumeRoleWithWebIdentity is used when users authenticate through an **external identity provider**.

This is commonly used in **federated authentication systems**.

Instead of logging in directly with AWS credentials, users authenticate through services such as:

Google  
Facebook  
Apple  
Corporate SSO systems  
OIDC providers

After authentication, the identity provider returns a **web identity token**.

That token is exchanged with AWS STS to receive temporary credentials.

---

### Example Scenario — Mobile Application Login

A mobile application allows users to log in using Google.

Flow:

Step 1  
User signs in with Google.

Step 2  
Google returns an identity token.

Step 3  
The application sends the token to AWS STS.

Step 4  
STS verifies the token.

Step 5  
STS returns temporary AWS credentials.

Example API call conceptually:

    sts:AssumeRoleWithWebIdentity

These credentials allow the mobile app to access AWS services like:

S3  
DynamoDB  
API Gateway

This allows applications to authenticate users **without storing AWS credentials**.

---

### 3. GetSessionToken

GetSessionToken is used to obtain **temporary credentials for an existing IAM user**.

It is commonly used when **MFA authentication is required**.

This API generates temporary credentials that are tied to the IAM user's permissions.

The permissions remain the same as the IAM user, but the credentials expire automatically.

---

### Example Scenario — MFA-Protected Access

A company requires MFA before performing sensitive operations.

Step-by-step flow:

Step 1  
User logs in with IAM credentials.

Step 2  
User provides MFA code.

Step 3  
User calls GetSessionToken.

Step 4  
STS verifies MFA.

Step 5  
STS returns temporary credentials.

Example CLI command:

    aws sts get-session-token \
        --serial-number arn:aws:iam::123456789012:mfa/user \
        --token-code 123456

STS returns:

    AccessKeyId
    SecretAccessKey
    SessionToken
    Expiration

These credentials can then be used for secure API requests.

---

### 4. Comparison of Common STS APIs

Each STS API serves a different purpose.

AssumeRole

Used when an identity needs to temporarily assume an IAM role.

Common use cases:

Cross-account access  
Temporary elevated privileges  
Application roles

AssumeRoleWithWebIdentity

Used when authentication comes from an external identity provider.

Common use cases:

Mobile apps  
Web applications  
Federated login systems

GetSessionToken

Used to obtain temporary credentials for an IAM user, often with MFA.

Common use cases:

MFA-protected CLI access  
Temporary API access for IAM users

---

### 5. How STS APIs Improve Security

All STS APIs generate **temporary credentials that automatically expire**.

Benefits include:

Reduced credential exposure  
No long-term secrets stored in applications  
Improved security through short-lived sessions  
Support for external authentication systems

This is why modern AWS architectures rely heavily on STS.

---

### Mental Model

Think of STS APIs as **temporary credential generators for different authentication scenarios**.

AssumeRole → borrow permissions from a role temporarily.

AssumeRoleWithWebIdentity → exchange external login tokens for AWS credentials.

GetSessionToken → convert long-term IAM credentials into temporary credentials.

Each API solves a different authentication problem while maintaining strong security.

---

### Key Takeaways

AWS STS provides APIs that generate temporary AWS credentials.

AssumeRole is used to temporarily assume IAM roles.

AssumeRoleWithWebIdentity allows external identity providers to authenticate users.

GetSessionToken generates temporary credentials for IAM users, often with MFA.

These APIs reduce reliance on long-term credentials and improve overall security.
