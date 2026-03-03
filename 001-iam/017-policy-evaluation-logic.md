## Policy Evaluation Logic (How AWS Decides Allow or Deny)

When a request is made to AWS — for example starting a server, reading a file from S3, or creating a database — AWS must decide whether the request should be **allowed or denied**.

This decision is not random.

AWS follows a **very specific evaluation process** to determine permissions.

This process is called **IAM Policy Evaluation Logic**.

---

### 1. Every AWS Action Is an API Request

Before understanding evaluation logic, it is important to understand one thing.

Every operation in AWS is actually an **API request**.

Examples:

Opening the console and starting an EC2 instance.

    StartInstances API call

Uploading a file to S3.

    PutObject API call

Listing EC2 instances.

    DescribeInstances API call

Whenever one of these requests is made, AWS must decide whether the identity making the request is allowed to perform that action.

To make that decision, AWS evaluates **all relevant policies**.

---

### 2. The High-Level Permission Decision Flow

Whenever AWS receives a request, the following process occurs.

Step 1  
Authenticate the identity.

Step 2  
Collect all applicable policies.

Step 3  
Evaluate deny rules.

Step 4  
Evaluate allow rules.

Step 5  
Return allow or deny decision.

Let’s examine each step.

---

### 3. Step 1 — Authentication

Before evaluating permissions, AWS must first confirm **who is making the request**.

This process is called **authentication**.

Examples of authentication:

IAM user logging into console with password.

CLI request signed using access keys.

Application assuming an IAM role.

If authentication fails, AWS immediately rejects the request.

No policy evaluation happens.

---

### 4. Step 2 — Collect All Relevant Policies

Once the identity is authenticated, AWS gathers **all policies that apply to the request**.

These may include:

Identity-based policies  
Resource-based policies  
Permissions boundaries  
Service control policies (in organizations)

For a typical IAM user, this might include:

User-attached policies  
Group policies  
Role policies (if a role is assumed)

AWS collects all these policies before making a decision.

---

### 5. Step 3 — Check for Explicit Deny

The first rule AWS checks is whether **any policy explicitly denies the action**.

Example deny policy:

    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "*"
    }

If the request matches this rule, AWS immediately denies the request.

This happens **even if another policy allows the action**.

This is because:

Explicit Deny always overrides Allow.

---

### 6. Step 4 — Check for Explicit Allow

If no deny rule applies, AWS then checks whether **any policy explicitly allows the action**.

Example allow policy:

    {
      "Effect": "Allow",
      "Action": "ec2:StartInstances",
      "Resource": "*"
    }

If the request matches an allow rule, the action is permitted.

If no allow rule exists, the request remains denied.

---

### 7. Step 5 — Implicit Deny

If AWS does not find any matching allow rule, the request is denied automatically.

This is called **implicit deny**.

Example scenario:

User attempts:

    ec2:TerminateInstances

But policy only allows:

    ec2:DescribeInstances

Since there is no allow rule for termination, AWS denies the request.

Even though no explicit deny exists, the request still fails.

---

### 8. Real AWS Example

Imagine a developer attempts to delete a file in an S3 bucket.

The request is:

    s3:DeleteObject

The developer has two policies.

Policy 1:

    {
      "Effect": "Allow",
      "Action": "s3:*",
      "Resource": "*"
    }

Policy 2:

    {
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "*"
    }

Evaluation result:

Step 1  
Request matches allow rule.

Step 2  
Request also matches deny rule.

Because explicit deny overrides allow, AWS denies the request.

---

### 9. Full Policy Evaluation Flow

The complete evaluation process looks like this.

Request received.

Identity authenticated.

AWS collects all policies.

AWS checks for explicit deny.

If deny exists → request denied.

If no deny exists → AWS checks allow rules.

If allow rule exists → request allowed.

If no allow rule exists → implicit deny applies.

---

### 10. Visual Mental Model

Think of AWS policy evaluation like a security checkpoint.

Security officer checks three things:

First rule:

Is there a rule that blocks this action?

If yes → access denied immediately.

Second rule:

Is there a rule that allows this action?

If yes → access granted.

Third rule:

If no rule allows the action → access denied.

This logic ensures AWS remains **secure by default**.

---

### 11. Why This Matters in Real Systems

Understanding policy evaluation logic helps with:

Debugging permission errors.

Designing secure IAM architectures.

Preventing privilege escalation.

Avoiding accidental over-permission.

Many IAM troubleshooting problems happen because engineers forget:

Explicit deny overrides allow.

---

### Key Takeaways

AWS evaluates IAM permissions using a strict decision process.

Every AWS action is an API request that must be evaluated.

AWS first authenticates the identity making the request.

Then it collects all applicable policies.

If any policy explicitly denies the request, the request is denied.

If no deny exists and an allow rule matches, the request is allowed.

If no allow rule exists, implicit deny applies.

This evaluation logic ensures AWS security decisions remain consistent and predictable.
