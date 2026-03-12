## EC2 with IAM Roles

In cloud environments, EC2 instances often need to interact with other AWS services such as:

- Amazon S3
- DynamoDB
- CloudWatch
- SQS
- Secrets Manager

For example, an application running on an EC2 instance might need to:

- upload files to an S3 bucket
- read configuration data from DynamoDB
- send logs to CloudWatch

To allow this access securely, AWS uses **IAM Roles for EC2**.

An **IAM role** allows an EC2 instance to securely obtain temporary credentials to access other AWS services without storing permanent credentials on the server.

Instead of placing access keys directly inside the application or configuration files, the instance assumes an IAM role that grants the required permissions.

This approach improves security and simplifies credential management.

---

### 1. Fine-Grained Permissions

**Fine-grained permissions** refer to the practice of granting **very specific and controlled access to AWS resources**.

Instead of giving broad permissions such as full access to a service, IAM policies can restrict access to:

- specific actions
- specific resources
- specific conditions

Example of overly broad permissions:

```
Allow: s3:*
Resource: *
```

This policy allows the instance to perform **any S3 action on any S3 bucket**, which is not secure.

A better example of fine-grained permissions:

```
Allow: s3:PutObject
Resource: arn:aws:s3:::my-app-bucket/uploads/*
```

This policy only allows the instance to **upload files to a specific directory in a specific bucket**.

Fine-grained permissions help ensure that EC2 instances only perform the operations they actually need.

This approach reduces the risk of accidental or malicious misuse of permissions.

---

### 2. Least Privilege

The **principle of least privilege** is one of the most important security concepts in AWS.

It means that every user, service, or instance should have **only the minimum permissions required to perform its tasks**.

No additional permissions should be granted beyond what is necessary.

For example:

If an EC2 instance only needs to read files from an S3 bucket, it should have permission for:

```
s3:GetObject
```

It should **not** have permissions such as:

```
s3:DeleteObject
s3:PutObject
s3:CreateBucket
```

Applying least privilege ensures that even if an EC2 instance is compromised, the attacker can only access a limited set of resources.

Benefits of the least privilege principle include:

- reduced security risks
- better access control
- minimized impact of compromised systems
- improved compliance with security standards

---

### 3. How EC2 Uses IAM Roles

When an IAM role is attached to an EC2 instance:

1. The instance receives temporary credentials from AWS.
2. The application on the instance uses those credentials to access AWS services.
3. The credentials are automatically rotated by AWS.

This means developers **do not need to store access keys in code or configuration files**.

Example architecture:

```
EC2 Instance
     ↓
IAM Role
     ↓
Permissions Policy
     ↓
Access to AWS Services (S3, DynamoDB, CloudWatch)
```

This mechanism provides secure and controlled access between EC2 and other AWS services.

---

### 4. Summary

Using IAM roles with EC2 allows instances to securely access other AWS services without storing long-term credentials.

Key concepts involved include:

| Concept | Description |
|------|------|
| IAM Role | Temporary credentials assigned to EC2 instances |
| Fine-Grained Permissions | Restrict access to specific actions and resources |
| Least Privilege | Grant only the minimum permissions required |

By combining IAM roles with fine-grained permission policies and the principle of least privilege, organizations can build **secure and well-controlled EC2 environments**.
