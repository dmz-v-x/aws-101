## IAM Roles

IAM Roles are one of the **most important and most frequently used concepts in AWS**.

If you understand IAM roles properly, many advanced AWS architectures become much easier to understand.

Roles are heavily used for:

- Applications accessing AWS services
- Services interacting with other services
- Cross-account access
- Temporary access for users
- Secure automation

In modern AWS architectures, **roles are used far more often than IAM users**.

---

### 1. Why IAM Users Are Not Enough

IAM users work well for **human identities**, such as developers or administrators.

However, IAM users have several limitations.

First, IAM users typically rely on **long-term credentials**, such as passwords or access keys.

Long-term credentials introduce risks:

- Credentials may be leaked
- Credentials may be stored in code
- Credentials may be reused improperly
- Credentials may be forgotten and never rotated

Second, many systems that interact with AWS are **not humans**.

Examples include:

- Applications running on servers
- AWS services interacting with other AWS services
- Automation systems
- CI/CD pipelines

These systems still need permissions, but they should **not rely on permanent credentials**.

AWS solves this problem using **IAM Roles**.

---

### 2. What Is an IAM Role?

An **IAM role** is an identity in AWS that provides **temporary permissions to perform specific actions**.

Unlike IAM users, roles do **not have permanent credentials**.

Instead, roles are **assumed temporarily** when access is needed.

When an identity assumes a role:

- AWS provides temporary credentials
- Those credentials allow specific actions
- The credentials expire automatically

This makes roles significantly **safer than long-term credentials**.

---

### 3. Key Difference Between Users and Roles

IAM User:

- Permanent identity
- Long-term credentials
- Usually represents a human

IAM Role:

- Temporary identity
- Temporary credentials
- Assumed when needed

In most modern AWS architectures, roles are preferred because they avoid long-term credential risks.

---

### 4. Temporary Credentials

One of the most important characteristics of IAM roles is the use of **temporary credentials**.

When a role is assumed, AWS generates temporary security credentials that include:

- Temporary access key
- Temporary secret key
- Security session token

These credentials are valid for a **limited period of time**.

Typical durations might be:

- 15 minutes
- 1 hour
- A few hours

After the credentials expire, they **automatically stop working**.

This reduces the risk of credential misuse.

---

### 5. The Role Assumption Concept

A role is not used directly.

Instead, it must be **assumed**.

Assuming a role means:

An identity temporarily receives the permissions defined by that role.

The process looks like this:

Identity → Assumes Role → Receives Temporary Permissions

Example conceptual flow:

1. An identity requests to assume a role.
2. AWS verifies that the identity is allowed to assume the role.
3. AWS issues temporary credentials.
4. The identity uses those credentials to perform actions.

When the session expires, the permissions disappear automatically.

---

### 6. Why Role Assumption Is Powerful

Role assumption creates a **dynamic and flexible permission system**.

Instead of giving permanent permissions to identities, permissions can be **granted temporarily when needed**.

This provides several benefits:

- Reduced credential exposure
- Better security
- Fine-grained permission control
- Easier automation

This concept is heavily used throughout AWS.

---

### 7. Roles for Human Users

Sometimes human users need **temporary elevated permissions**.

Example scenario:

A developer normally has limited permissions.

Occasionally they may need administrator permissions for troubleshooting.

Instead of permanently granting admin permissions, they can **assume an admin role temporarily**.

Workflow example:

Developer logs in → assumes admin role → performs task → role expires.

This approach improves security.

---

### 8. Roles for AWS Services

Many AWS services need to interact with other AWS services.

Example scenarios include:

- A server storing files in storage
- A compute service reading from a database
- A monitoring system collecting logs

Instead of embedding credentials in applications, AWS allows services to **assume roles automatically**.

Example:

A compute instance may assume a role that allows it to upload files to storage.

The application running on that server receives **temporary credentials automatically**.

This avoids storing secrets in code.

---

### 9. Cross-Account Roles

Large organizations often use **multiple AWS accounts**.

For example:

- One account for development
- One account for testing
- One account for production

Sometimes users or systems in one account need access to resources in another account.

IAM roles enable this through **cross-account access**.

Example scenario:

A DevOps engineer in Account A needs to manage infrastructure in Account B.

Instead of creating duplicate users, the engineer can **assume a role in Account B**.

This approach simplifies access control across accounts.

---

### 10. Three Major Types of Role Usage

IAM roles are commonly used in three major scenarios.

Human roles:

Used when users temporarily assume roles to gain additional permissions.

Service roles:

Used when AWS services need permissions to interact with other AWS services.

Cross-account roles:

Used when identities from one AWS account need access to another account.

These patterns appear constantly in real-world AWS architectures.

---

### 11. Mental Model for IAM Roles

Think of a role like a **temporary permission badge**.

An employee normally has their regular badge.

But sometimes they may receive a **temporary badge** that allows them into restricted areas.

Once their task is complete, the badge expires.

IAM roles work exactly like that.

Permissions are **granted temporarily**, then automatically removed.

---

### Key Takeaways

IAM roles are identities that provide temporary permissions in AWS.

Roles do not have permanent credentials.

Instead, identities assume roles to receive temporary credentials.

Temporary credentials automatically expire, improving security.

Roles are used by humans, applications, AWS services, and cross-account systems.

IAM roles are one of the most important security mechanisms in AWS and are widely used in modern cloud architectures.
