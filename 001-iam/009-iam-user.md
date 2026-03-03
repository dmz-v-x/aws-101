## IAM Users

### 1. What Is an IAM User?

An **IAM user** is an identity created inside an AWS account that represents **a specific person or system**.

Most commonly, IAM users represent **human users**, such as:

- Developers
- DevOps engineers
- Security engineers
- Data engineers
- Administrators

Each IAM user has their own **credentials** and **permissions**.

This means every person gets their **own identity**, rather than sharing a single login.

---

### 2. Why IAM Users Exist

In a real organization, many people need access to AWS.

Examples include:

- Developers deploying applications
- DevOps engineers managing infrastructure
- Security teams reviewing configurations
- Data engineers accessing analytics systems

Sharing the root account among all these people would be extremely dangerous.

Instead, AWS allows the creation of **separate IAM users** for each person.

This provides several benefits:

- Individual accountability
- Controlled permissions
- Better security
- Easier auditing

Each user can be given **only the permissions they actually need**.

---

### 3. Difference Between Root User and IAM User

The root user and IAM users are very different in terms of power and purpose.

Root user characteristics:

- Created automatically when the AWS account is created
- Has unlimited permissions
- Cannot be restricted by IAM policies
- Should rarely be used

IAM user characteristics:

- Created manually inside the AWS account
- Permissions are controlled using IAM policies
- Designed for daily usage
- Can be restricted to specific actions

In practice, the root user should almost never be used.

Instead, IAM users should be used for **normal operations and daily work**.

---

### 4. When IAM Users Are Used

IAM users are typically used whenever **a human needs to access AWS**.

Common scenarios include:

Developers logging into the AWS Console to deploy applications.

DevOps engineers configuring infrastructure.

Security engineers reviewing IAM policies and security settings.

Operations teams monitoring systems and logs.

Because each user has a unique identity, AWS can track **who performed which action**.

This improves accountability and security.

---

### 5. IAM User Credentials

IAM users authenticate themselves using **credentials**.

Credentials are the pieces of information used to prove identity when accessing AWS.

The two most common IAM user credentials are:

Password  
Access keys

Each type of credential is used for a different purpose.

---

### 6. Password (Console Access)

Passwords are used when an IAM user logs into the **AWS Management Console**.

The AWS Management Console is the **web interface** used to manage AWS services through a browser.

Example workflow:

A developer opens the AWS console login page.

They enter:

- IAM username
- Password

If the credentials are correct, they are authenticated and allowed to access AWS.

Depending on their permissions, they may be able to:

- Deploy infrastructure
- View logs
- Manage resources
- Monitor services

The password provides **interactive login access** to AWS.

---

### 7. Access Keys (Programmatic Access)

Access keys are used when AWS is accessed **programmatically**.

Programmatic access means interacting with AWS through:

- Command Line Interface (CLI)
- SDKs
- Applications
- Automation scripts

An access key consists of two parts:

Access Key ID  
Secret Access Key

These credentials allow programs and tools to authenticate with AWS.

---

### 8. Example Use Case for Access Keys

Imagine a developer using the AWS CLI.

The CLI must authenticate with AWS before executing commands.

The developer configures their CLI with:

- Access Key ID
- Secret Access Key

Once configured, the CLI can perform actions such as:

- Creating servers
- Uploading files
- Deploying infrastructure
- Querying logs

These actions are still restricted by **IAM policies**.

Even with access keys, the user can only perform actions that their permissions allow.

---

### 9. Security Considerations for IAM User Credentials

Because IAM user credentials provide access to AWS, they must be protected carefully.

Important security practices include:

- Use strong passwords
- Enable Multi-Factor Authentication (MFA)
- Never share credentials
- Rotate access keys regularly
- Avoid embedding access keys in source code

If credentials are exposed, attackers could gain access to AWS resources.

Proper credential management is therefore essential.

---

### 10. Mental Model for IAM Users

Think of IAM users like **employee accounts in a company system**.

Each employee has:

- Their own username
- Their own password
- Their own permissions

Employees can only access the systems and data required for their job.

IAM users work the same way in AWS.

They represent **individual identities with controlled permissions**.

---

### Key Takeaways

IAM users are identities created inside an AWS account that typically represent human users.

IAM users allow multiple people to access AWS safely without sharing the root account.

Unlike the root user, IAM users have permissions controlled through IAM policies.

IAM users are used for everyday operations such as deploying applications or managing infrastructure.

IAM user authentication typically uses two types of credentials: passwords for console access and access keys for programmatic access.
