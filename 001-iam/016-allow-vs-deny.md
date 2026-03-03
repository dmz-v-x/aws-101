## Allow vs Deny

One of the most important concepts in AWS IAM is understanding how **Allow** and **Deny** permissions work.

Every permission decision in AWS ultimately comes down to these two outcomes:

Allow  
Deny

Whenever someone tries to perform an action in AWS — such as starting a server, reading a file, or accessing a database — AWS must decide whether the request should be **allowed or denied**.

IAM policies control this decision using the **Effect** field inside policy statements.

---

### 1. The Effect Field Controls Allow or Deny

Inside every IAM policy statement there is a field called **Effect**.

This field tells AWS whether the action should be:

Allowed  
or  
Denied

Example:

    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }

This policy allows the identity to **read objects from the S3 bucket**.

If the effect is changed to Deny:

    {
      "Effect": "Deny",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }

Then the identity is **explicitly blocked from reading objects** from that bucket.

---

### 2. Implicit Deny (Default Behavior)

AWS follows an important security rule:

Everything is **denied by default**.

This means that unless a policy explicitly allows an action, AWS will automatically deny it.

This is called **implicit deny**.

Example scenario:

A new IAM user is created but no policies are attached.

If the user tries to start an EC2 instance:

AWS response:

Access Denied

Why?

Because no policy explicitly allowed that action.

So the request is denied by default.

---

### 3. Explicit Allow

An action becomes allowed only when a policy **explicitly allows it**.

Example policy:

    {
      "Effect": "Allow",
      "Action": "ec2:StartInstances",
      "Resource": "*"
    }

Now the user can start EC2 instances.

But the user **still cannot perform other actions** like:

- Terminating instances
- Creating instances
- Modifying networking

Those actions remain denied because they were **never allowed**.

---

### 4. Explicit Deny

An **explicit deny** is a policy statement that intentionally blocks an action.

Example:

    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::company-data/*"
    }

This policy explicitly prevents deleting objects from the bucket.

Even if another policy allows deleting objects, the deny will override it.

This rule is extremely important.

---

### 5. Explicit Deny Always Wins

In AWS IAM policy evaluation:

Explicit Deny always overrides Allow.

This means if a request matches both an allow rule and a deny rule, AWS will **deny the request**.

Example scenario:

Policy 1:

    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }

This allows all S3 actions.

Policy 2:

    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "*"
    }

This denies deleting objects.

If a user tries to delete an object:

AWS sees:

Allow → s3:*  
Deny → s3:DeleteObject  

Because deny takes priority, the request is **blocked**.

---

### 6. Why Explicit Deny Exists

Explicit deny is used to **create strong security boundaries**.

It ensures that certain actions are **never allowed**, even if other policies grant broad permissions.

Example use cases:

Prevent deleting critical backups  
Prevent modifying IAM policies  
Prevent deleting production databases  

Example deny policy:

    {
      "Effect": "Deny",
      "Action": "rds:DeleteDBInstance",
      "Resource": "*"
    }

This prevents database deletion even if someone has admin permissions elsewhere.

---

### 7. Real AWS Example (Production Protection)

Imagine a company wants to ensure that no developer can delete production S3 data.

Even if developers have broad permissions, a deny policy can enforce this rule.

Example:

    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::production-data/*"
    }

Now developers may be able to:

- Read files
- Upload files
- Process data

But they **cannot delete files** in the production bucket.

---

### 8. The IAM Permission Decision Flow

When a request is made to AWS, IAM evaluates permissions in this order.

Step 1

Check if there is an **explicit deny**.

If yes → request is denied.

Step 2

Check if there is an **explicit allow**.

If yes → request is allowed.

Step 3

If no allow exists → implicit deny applies.

This logic ensures AWS remains secure by default.

---

### 9. Visual Mental Model

You can think of IAM permission evaluation like a security gate.

Step 1  
Check for deny rules.

If a deny rule matches → access blocked immediately.

Step 2  
Check for allow rules.

If an allow rule matches → access granted.

Step 3  
If no rule matches → access denied by default.

This process happens for **every AWS API request**.

---

### 10. Common Beginner Mistake

A common mistake is thinking that a policy must explicitly deny actions.

That is not necessary.

If an action is not allowed, it is automatically denied.

Example:

Policy allows only:

    "Action": "ec2:DescribeInstances"

The user can only view instance information.

All other EC2 actions remain denied automatically.

---

### Key Takeaways

AWS IAM permissions are controlled using Allow and Deny rules.

If an action is not explicitly allowed, it is denied by default (implicit deny).

Policies can explicitly allow actions using the Allow effect.

Policies can explicitly block actions using the Deny effect.

Explicit deny always overrides allow.

Understanding Allow vs Deny is critical for designing secure IAM policies and preventing unintended access.
