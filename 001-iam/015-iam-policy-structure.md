## IAM Policy Structure

IAM policies are the **core mechanism that controls permissions in AWS**.

Every IAM policy follows a specific structure so that AWS can understand it and enforce permissions correctly.

The most important components of a policy are:

Version  
Statement  
Effect  
Action  
Resource  
Condition

---

### 1. The Basic Structure of an IAM Policy

Every IAM policy follows a general JSON structure like this:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::my-bucket/*"
        }
      ]
    }

Even though policies can become more complex, **this structure always remains the same**.

Let’s break down each component.

---

### 2. Version

The **Version** field specifies the version of the IAM policy language.

Example:

    "Version": "2012-10-17"

This does **not represent the date you created the policy**.

Instead, it represents the **policy language version AWS uses to interpret the policy**.

Currently, almost all IAM policies use:

    2012-10-17

This version supports modern policy features.

So in practice:

You will almost always write:

    "Version": "2012-10-17"

---

### 3. Statement

The **Statement** section contains the **actual permission rules**.

A policy can contain **one or multiple statements**.

Each statement defines a specific permission rule.

Example:

    "Statement": [
      {
        "Effect": "Allow",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::my-bucket/*"
      }
    ]

If multiple rules are needed, multiple statements can be added.

Example:

    "Statement": [
      { ...rule1... },
      { ...rule2... },
      { ...rule3... }
    ]

Each rule is evaluated by AWS when deciding permissions.

---

### 4. Effect

The **Effect** field determines whether the rule:

Allows an action  
or  
Denies an action

Possible values are:

Allow  
Deny

Example:

    "Effect": "Allow"

This means the action defined in the policy is **permitted**.

Example:

    "Effect": "Deny"

This means the action is **explicitly blocked**.

Deny rules are extremely important because **explicit deny overrides allow**.

---

### 5. Action

The **Action** field specifies **which AWS operations are allowed or denied**.

Every AWS service exposes many actions.

Examples for S3:

- s3:GetObject  
- s3:PutObject  
- s3:DeleteObject  

Examples for EC2:

- ec2:StartInstances  
- ec2:StopInstances  
- ec2:TerminateInstances  

Example policy action:

    "Action": "s3:GetObject"

This allows reading objects from S3.

Multiple actions can also be specified:

    "Action": [
      "ec2:StartInstances",
      "ec2:StopInstances"
    ]

This allows starting and stopping EC2 instances.

---

### 6. Resource

The **Resource** field specifies **which AWS resources the policy applies to**.

AWS resources are identified using something called an **ARN (Amazon Resource Name)**.

Example:

    "Resource": "arn:aws:s3:::my-bucket/*"

This ARN represents:

All objects inside the S3 bucket named **my-bucket**.

Another example:

    "Resource": "arn:aws:ec2:us-east-1:123456789012:instance/*"

This represents all EC2 instances in that account and region.

If you want to apply the policy to **all resources**, you can use:

    "Resource": "*"

However, this should be avoided when possible because it grants **very broad permissions**.

---

### 7. Condition (Optional but Powerful)

The **Condition** field adds **extra rules that must be satisfied** before a permission is granted.

Conditions allow fine-grained access control.

Example:

A user may be allowed to access S3 **only from a specific IP address**.

Example policy condition:

    "Condition": {
      "IpAddress": {
        "aws:SourceIp": "203.0.113.0/24"
      }
    }

This means the action is allowed **only if the request comes from that IP range**.

Conditions can enforce rules such as:

- Access only during certain times
- Access only from specific IP addresses
- Access only when MFA is enabled
- Access only from certain AWS services

Conditions make IAM policies **extremely flexible and powerful**.

---

### 8. Full Example Policy with All Components

Here is a more complete IAM policy example.

This policy allows a user to **read objects from a specific S3 bucket**, but only if MFA is enabled.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::company-data/*",
          "Condition": {
            "Bool": {
              "aws:MultiFactorAuthPresent": "true"
            }
          }
        }
      ]
    }

Explanation:

Version

Policy language version.

Statement

Defines the permission rule.

Effect

Allows the action.

Action

Allows reading S3 objects.

Resource

Applies to the bucket **company-data**.

Condition

Requires MFA authentication.

---

### 9. Real AWS Example (EC2 Control)

Imagine a DevOps engineer needs permission to:

- Start EC2 instances
- Stop EC2 instances

But they should **not terminate instances**.

Policy example:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "ec2:StartInstances",
            "ec2:StopInstances",
            "ec2:DescribeInstances"
          ],
          "Resource": "*"
        }
      ]
    }

This allows:

- Starting instances
- Stopping instances
- Viewing instance details

But **does not allow deleting instances**.

---

### 10. Mental Model for Policy Structure

Think of IAM policies as **security rules written in a structured format**.

Each statement answers five questions:

What version of the policy language is used?  
What rule is being defined?  
Is the action allowed or denied?  
Which actions are affected?  
Which resources are affected?  
Under what conditions is the action allowed?

AWS evaluates these rules whenever someone attempts to perform an action.

---

### Key Takeaways

IAM policies define permissions in AWS.

Policies are written using JSON and follow a consistent structure.

The most important policy components are Version, Statement, Effect, Action, Resource, and Condition.

Effect determines whether an action is allowed or denied.

Action specifies which AWS operations are affected.

Resource identifies which AWS resources the policy applies to.

Conditions allow additional rules such as IP restrictions or MFA requirements.

Understanding IAM policy structure is critical for designing secure AWS environments.
