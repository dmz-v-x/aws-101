## Cross-Account Access in AWS

In real-world AWS environments, organizations rarely operate with a **single AWS account**.

Instead, they use **multiple AWS accounts** for isolation, security, billing separation, and environment management.

Examples of account separation:

Development account  
Testing account  
Production account  
Security account  
Logging account  

However, sometimes identities in one account need to access resources in another account.

AWS solves this using **cross-account access**, which is primarily implemented through **IAM roles and STS role assumption**.

This mechanism allows identities in one account to temporarily access resources in another account **without creating duplicate users or sharing credentials**.

---

### 1. What Cross-Account Access Is

Cross-account access allows an identity in **Account A** to access resources in **Account B**.

This is done using **IAM roles** and the **AssumeRole mechanism**.

Basic idea:

    Account A Identity
            ↓
    Assumes Role in Account B
            ↓
    STS issues temporary credentials
            ↓
    Access resources in Account B

This approach ensures:

No credential sharing  
Temporary access  
Centralized access control

---

### 2. Why Cross-Account Access Is Needed

Cross-account access is widely used in organizations with **multi-account architectures**.

Common scenarios include:

Developers deploying infrastructure from a development account into a production account.

Security teams auditing logs across multiple accounts.

Centralized CI/CD pipelines deploying resources across environments.

A monitoring account collecting metrics from all accounts.

Without cross-account roles, administrators would have to create **duplicate IAM users in every account**, which becomes difficult to manage.

---

### 3. Example Multi-Account Architecture

A typical company might have this structure:

    AWS Organization
        │
        ├── Dev Account
        ├── Staging Account
        ├── Production Account
        ├── Security Account
        └── Logging Account

Developers work in the **Dev account** but need permission to deploy infrastructure into **Production**.

Instead of creating users in Production, developers assume a **DeploymentRole** in the Production account.

---

### 4. Key Components of Cross-Account Access

Cross-account access relies on three main components.

IAM Role  
Trust Policy  
STS AssumeRole

Each plays a specific role in enabling access.

---

### 5. IAM Role in the Target Account

The **target account** (the account being accessed) must create an IAM role.

This role defines:

Who can assume it  
What permissions it grants

Example:

Production account creates:

    DeploymentRole

This role will grant permissions for deploying infrastructure.

---

### 6. Trust Policy (Who Can Assume the Role)

The trust policy defines **which identity or account is allowed to assume the role**.

Example trust policy allowing Account A to assume the role.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "AWS": "arn:aws:iam::111111111111:root"
          },
          "Action": "sts:AssumeRole"
        }
      ]
    }

Explanation:

Principal → identifies the trusted account.

Action → allows the AssumeRole operation.

This means any allowed identity from Account A can assume the role.

---

### 7. Permission Policy (What the Role Can Do)

The role also has a **permission policy**.

This defines what the role can do inside the target account.

Example deployment permissions:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "ec2:RunInstances",
            "ec2:StartInstances",
            "ec2:StopInstances"
          ],
          "Resource": "*"
        }
      ]
    }

This means identities assuming the role can manage EC2 instances.

---

### 8. Source Account Permissions

The identity in the source account must also have permission to assume the role.

Example policy attached to a developer user.

    {
      "Effect": "Allow",
      "Action": "sts:AssumeRole",
      "Resource": "arn:aws:iam::222222222222:role/DeploymentRole"
    }

Without this permission, the user cannot assume the role.

---

### 9. Role Assumption Process

The cross-account role assumption process works like this.

Step 1  
Developer authenticates in Account A.

Step 2  
Developer calls the STS AssumeRole API.

Step 3  
AWS verifies the role trust policy.

Step 4  
STS generates temporary credentials.

Step 5  
Developer uses those credentials to access Account B.

---

### 10. Example AssumeRole CLI Command

Example CLI command to assume a role.

    aws sts assume-role \
        --role-arn arn:aws:iam::222222222222:role/DeploymentRole \
        --role-session-name DevDeploymentSession

STS returns temporary credentials.

Example response:

    AccessKeyId
    SecretAccessKey
    SessionToken
    Expiration

These credentials are used for subsequent API calls.

---

### 11. Role Session

When a role is assumed, AWS creates a **role session**.

A role session represents the temporary access period during which the role permissions are active.

Session characteristics:

Temporary credentials  
Limited session duration  
Automatically expires

Typical session duration:

15 minutes  
1 hour  
Several hours depending on configuration

After expiration, the role must be assumed again.

---

### 12. Cross-Account Access Using the AWS Console

Cross-account access can also occur through the AWS console.

Flow:

Step 1  
User logs into Account A.

Step 2  
User selects "Switch Role".

Step 3  
User enters target account ID and role name.

Step 4  
AWS assumes the role.

Step 5  
Console switches context to Account B.

This allows the user to manage resources in the target account.

---

### 13. Security Benefits of Cross-Account Roles

Cross-account roles provide several security advantages.

No credential sharing between accounts.

Temporary credentials instead of permanent keys.

Fine-grained permission control.

Centralized identity management.

Better environment isolation.

This makes them ideal for **large-scale cloud environments**.

---

### 14. Advanced Cross-Account Controls

Organizations often add additional security layers.

Examples include:

External IDs

Used to prevent confused deputy attacks.

MFA requirement

Require MFA before assuming sensitive roles.

Session policies

Limit permissions during the role session.

Service Control Policies (SCPs)

Restrict actions across entire accounts.

These mechanisms strengthen cross-account security.

---

### 15. Common Real-World Use Cases

Cross-account access is widely used for:

CI/CD pipelines deploying infrastructure.

Security teams auditing multiple accounts.

Centralized logging systems.

Monitoring accounts collecting metrics.

Shared services accessing multiple environments.

These patterns appear in nearly every enterprise AWS architecture.

---

### 16. Mental Model

Think of cross-account access like visiting another building in a company campus.

You normally work in Building A.

When you need to enter Building B:

You request temporary access.

Security verifies your identity.

You receive a temporary access badge.

You can enter Building B for a limited time.

Once the badge expires, you lose access.

This is exactly how cross-account role assumption works.

---

### Key Takeaways

Cross-account access allows identities in one AWS account to access resources in another account.

It is implemented using IAM roles and the STS AssumeRole API.

The target account creates a role with a trust policy allowing the source account.

The role permission policy defines what actions are allowed.

STS generates temporary credentials when the role is assumed.

Role sessions provide temporary access that automatically expires.

Cross-account roles are widely used in multi-account AWS architectures for secure resource access.
