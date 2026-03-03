## Least Privilege Principle

One of the most important security principles in cloud computing — and in cybersecurity in general — is the **Principle of Least Privilege (PoLP)**.

AWS IAM is designed specifically to help enforce this principle.

The idea is simple:

Give identities **only the permissions they absolutely need**, and nothing more.

This approach significantly reduces security risks and prevents accidental damage to infrastructure.

---

### 1. What Least Privilege Means

The **Principle of Least Privilege** means that every identity (user, role, or service) should receive **only the minimum permissions required to perform its job**.

Nothing extra.

Example:

A developer who only needs to **view logs** should not be able to:

- Delete databases
- Modify IAM policies
- Shut down production servers
- Access billing information

Instead, they should only receive permissions such as:

- Viewing logs
- Reading application data
- Accessing monitoring dashboards

By limiting permissions, the impact of mistakes or compromised credentials is reduced.

---

### 2. Example Without Least Privilege (Bad Practice)

Imagine a company that gives developers the following policy:

    {
      "Effect": "Allow",
      "Action": "*",
      "Resource": "*"
    }

This effectively gives **full administrator access**.

Now the developer can:

- Delete databases
- Modify IAM permissions
- Terminate production servers
- Access sensitive data
- Change security settings

Even if the developer has no malicious intent, this is extremely risky.

A simple mistake could cause a **major outage or data loss**.

---

### 3. Example With Least Privilege (Good Practice)

Instead, the developer receives only the permissions required for development work.

Example policy:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "logs:DescribeLogStreams",
            "logs:GetLogEvents"
          ],
          "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": [
            "ec2:StartInstances",
            "ec2:StopInstances"
          ],
          "Resource": "*"
        }
      ]
    }

Now the developer can:

- View application logs
- Restart servers

But cannot:

- Delete infrastructure
- Modify IAM
- Access sensitive data

This is **least privilege in practice**.

---

### 4. Why Least Privilege Is Critical

Least privilege reduces the impact of several major risks.

These include:

Credential theft  
Insider threats  
Accidental mistakes  
Privilege escalation

Example scenario:

If an attacker steals credentials from an application server that has **full admin permissions**, the attacker could destroy the entire infrastructure.

But if the server only has permission to **upload logs**, the damage is minimal.

---

### 5. Over-Permission Risks

Over-permission occurs when an identity receives **more permissions than it actually needs**.

This is extremely common in poorly designed IAM systems.

Example over-permission risks include:

Developers having full administrative access.

Applications having permission to delete infrastructure.

Automation tools having unrestricted resource access.

These excessive permissions increase the potential damage from:

Human error  
Security breaches  
Misconfigurations

Over-permission is one of the most common security weaknesses in cloud environments.

---

### 6. Example Real-World Risk

Imagine an application server that stores logs in S3.

The application should only need:

    s3:PutObject

But instead it receives:

    s3:*

This means the application could now:

- Delete objects
- Modify bucket settings
- Read sensitive data

If the server is compromised, the attacker gains **far more access than necessary**.

This is why permissions should always be minimized.

---

### 7. IAM Policy Scoping Strategy

To implement least privilege effectively, policies should be **carefully scoped**.

This means restricting permissions using:

Actions  
Resources  
Conditions

---

### 8. Restricting Actions

The **Action** field should only include the operations required.

Bad practice:

    "Action": "s3:*"

Better practice:

    "Action": [
      "s3:GetObject",
      "s3:PutObject"
    ]

This limits what operations can be performed.

---

### 9. Restricting Resources

The **Resource** field should limit access to specific resources whenever possible.

Bad practice:

    "Resource": "*"

Better practice:

    "Resource": "arn:aws:s3:::app-logs/*"

Now the identity can only interact with a **specific bucket**.

---

### 10. Using Conditions

Conditions add another layer of control.

Example: require MFA.

    "Condition": {
      "Bool": {
        "aws:MultiFactorAuthPresent": "true"
      }
    }

Other examples include:

Restrict access by IP address.

Restrict access by time of day.

Restrict access to specific AWS services.

Conditions make policies **much more secure and precise**.

---

### 11. Example Fully Scoped Policy

Example least-privilege policy for an application uploading logs.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:PutObject",
          "Resource": "arn:aws:s3:::app-logs/*"
        }
      ]
    }

This policy allows exactly one action:

Upload logs to a specific bucket.

Nothing else.

---

### 12. Mental Model

Think of least privilege like giving employees access badges.

An employee should only have access to:

The rooms they need for their job.

Not the entire building.

Similarly, an IAM identity should only have access to:

The AWS resources required for its role.

Not the entire cloud environment.

---

### Key Takeaways

The Principle of Least Privilege means giving identities only the permissions they absolutely need.

Over-permission increases security risks and potential damage from mistakes or attacks.

IAM policies should be carefully scoped using actions, resources, and conditions.

Restricting permissions to specific resources and operations is critical for secure AWS environments.

Least privilege is one of the most important security practices in AWS and is fundamental to designing safe cloud architectures.
