## Identity Comparison 

### 1. Identity Comparison Table

| Identity | Used By | Credentials | Lifetime |
|---------|---------|------------|----------|
| Root | Account owner | Permanent | Permanent |
| User | Humans | Permanent | Permanent |
| Role | Users / services | Temporary | Temporary |

This table captures the **core mental model** of AWS identities.

---

### 2. Root Identity (Account Owner)

The **root identity** represents the **AWS account itself**.

It is created automatically when the AWS account is created.

Characteristics:

- Has full control of the account
- Cannot be restricted using IAM policies
- Credentials are permanent
- Should rarely be used

Root access is typically reserved for:

- Initial account setup
- Account recovery
- Billing configuration
- Critical administrative actions

Because of its unlimited permissions, the root identity must be **protected extremely carefully**.

---

### 3. IAM User (Human Identity)

IAM users represent **individual human users** inside an AWS account.

Examples include:

- Developers
- DevOps engineers
- Security engineers
- Administrators

Characteristics:

- Permanent identity
- Permanent credentials
- Permissions controlled using IAM policies
- Designed for daily access

IAM users typically authenticate using:

- Console password
- Access keys

However, modern AWS best practices recommend **minimizing the use of IAM users** in favor of roles where possible.

---

### 4. IAM Role (Temporary Identity)

IAM roles are identities designed for **temporary access**.

Roles are not tied to a specific person.

Instead, they are **assumed when needed**.

Characteristics:

- Temporary credentials
- Automatically expire
- Used by humans, services, and systems
- No long-term credentials

Roles are commonly used for:

- Applications accessing AWS services
- AWS services interacting with other services
- Cross-account access
- Temporary elevated permissions

Because credentials are temporary, roles provide **stronger security** than long-term credentials.

---

### 5. Permanent vs Temporary Credentials

A key difference between these identities is the **lifetime of their credentials**.

Root user credentials:

- Permanent
- Long-lived
- High risk if compromised

IAM user credentials:

- Permanent
- Long-lived
- Must be managed carefully

IAM role credentials:

- Temporary
- Automatically expire
- Lower risk

This is one reason AWS strongly encourages **role-based access** in modern architectures.

---

### 6. Mental Model

You can think of AWS identities like this:

Root → Owner of the entire building  

User → Employee with a permanent badge  

Role → Temporary badge issued for a specific task  

This model helps explain how AWS controls access safely across large cloud environments.

---

### Key Takeaways

AWS provides three primary identity types: root user, IAM user, and IAM role.

The root identity represents the account owner and has unlimited permissions.

IAM users represent human identities with long-term credentials.

IAM roles provide temporary permissions through temporary credentials.

Modern AWS architectures rely heavily on roles because temporary credentials improve security and reduce credential management risks.
