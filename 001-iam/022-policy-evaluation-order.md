## Policy Evaluation Order

In AWS IAM, a permission decision is often influenced by **multiple layers of policies**.

When a request is made, AWS does not simply check one policy and return a result.  
Instead, AWS evaluates **several policy types together** before deciding whether the request should be allowed or denied.

This process is called **policy evaluation order**.

Understanding this order is extremely important because in real systems an identity may be affected by:

Identity policies  
Resource policies  
Permission boundaries  
Service Control Policies (SCPs)

Each of these layers contributes to the **final permission decision**.

---

### 1. Why Policy Evaluation Order Exists

AWS environments can become very complex.

An identity might have:

- A policy attached directly to the user
- A policy attached to a group
- A role policy
- A resource policy on the target resource
- A permission boundary
- An organization-level policy

All of these policies may influence the same request.

AWS therefore follows a **deterministic evaluation order** to ensure the decision is consistent and secure.

---

### 2. High-Level Evaluation Flow

When a request is made, AWS evaluates policies in this general order.

1. Check authentication
2. Evaluate Service Control Policies (SCPs)
3. Evaluate Permission Boundaries
4. Evaluate Identity-Based Policies
5. Evaluate Resource-Based Policies
6. Apply explicit deny rules
7. Determine final allow or deny

The key rule always remains:

Explicit Deny overrides Allow.

---

### 3. Identity-Based Policies

Identity-based policies are policies attached to IAM identities.

These identities include:

IAM users  
IAM groups  
IAM roles  

Identity policies define **what actions the identity is allowed to perform**.

Example identity policy:

    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::company-data/*"
    }

If a user has this policy, they are allowed to read objects from the S3 bucket.

However, identity policies alone do not always determine the final permission.

Other policy types may still affect the decision.

---

### 4. Resource-Based Policies

Resource-based policies are attached directly to AWS resources.

Examples include:

S3 bucket policies  
SNS topic policies  
SQS queue policies  
Lambda function policies  
KMS key policies  

These policies specify **who can access the resource**.

Example S3 bucket policy:

    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:user/Developer"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::company-data/*"
    }

In this case, the bucket itself allows the developer to read objects.

For the request to succeed, both the identity policy and the resource policy must permit the action.

---

### 5. Permission Boundaries

Permission boundaries are an advanced IAM feature used to **limit the maximum permissions an identity can have**.

A permission boundary acts like a **guardrail**.

Even if an identity policy allows an action, the permission boundary can restrict it.

Example scenario:

Identity policy allows:

    "Action": "ec2:*"

But the permission boundary allows only:

    "Action": [
      "ec2:StartInstances",
      "ec2:StopInstances"
    ]

Final result:

The user can only start and stop instances.

The broader permissions in the identity policy are restricted by the boundary.

Permission boundaries are often used in environments where administrators want to **delegate IAM creation safely**.

---

### 6. Service Control Policies (SCPs)

Service Control Policies are part of **AWS Organizations**.

They operate at the **organization or account level**.

SCPs define the **maximum permissions allowed for accounts within the organization**.

Even if an identity policy allows an action, the action will fail if the SCP denies it.

Example SCP:

    {
      "Effect": "Deny",
      "Action": "ec2:TerminateInstances",
      "Resource": "*"
    }

If this SCP is applied, no user in that account can terminate EC2 instances.

SCPs are often used by organizations to enforce **global security restrictions**.

For example:

Prevent deleting audit logs  
Prevent disabling security services  
Prevent deleting production infrastructure

---

### 7. Example Full Evaluation Scenario

Imagine a developer tries to delete a file from an S3 bucket.

AWS evaluates the following layers.

SCP  
Allows S3 actions.

Permission Boundary  
Allows only read access.

Identity Policy  
Allows deleting objects.

Bucket Policy  
Allows deleting objects.

Evaluation result:

Identity policy allows delete.

Bucket policy allows delete.

But permission boundary allows only read actions.

Final decision:

Access denied.

The boundary restricts the operation.

---

### 8. Simplified Mental Model

You can think of policy evaluation as a **series of security filters**.

Request enters AWS.

First filter:

SCPs limit what the entire account can do.

Second filter:

Permission boundaries limit the identity.

Third filter:

Identity policies grant permissions.

Fourth filter:

Resource policies grant access to the resource.

At any point, if a **deny rule is triggered**, the request fails.

---

### 9. Why This Matters in Real AWS Systems

Understanding policy evaluation order helps with:

Debugging permission errors

Designing secure multi-account architectures

Enforcing organizational security rules

Preventing privilege escalation

Many real-world AWS permission issues occur because engineers overlook **how these policy layers interact**.

---

### Key Takeaways

AWS evaluates multiple policy types when deciding whether a request is allowed.

Identity-based policies define what actions an identity can perform.

Resource-based policies define who can access a resource.

Permission boundaries restrict the maximum permissions an identity can have.

Service Control Policies (SCPs) restrict permissions at the organization level.

Explicit deny always overrides allow during policy evaluation.
