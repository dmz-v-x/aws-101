## What Is a Policy

### 1. Policy = Permission Document

A **policy** is a document that contains rules describing:

- What actions are allowed or denied  
- Which AWS resources those actions apply to  
- Under what conditions the actions are allowed  

Think of a policy as a **set of instructions for AWS security**.

Example concept:

A policy might say:

- A developer can start EC2 servers  
- A developer can view logs  
- A developer cannot delete databases  

AWS reads this document and enforces the rules.

---

### 2. Where Policies Are Used in AWS

Policies can be attached to several IAM entities.

Examples include:

IAM Users  
IAM Groups  
IAM Roles  

Example structure:

User → belongs to group → group has policy  
or  
User → directly has policy  
or  
Role → has policy  

Example architecture:

    Developer User
            ↓
    Developers Group
            ↓
    Developer Policy

The policy defines what the developer can do.

---

### 3. Example AWS Scenario

Imagine a company where developers need permission to:

- Start EC2 servers  
- Stop EC2 servers  
- View EC2 instances  

But they should not be able to:

- Delete EC2 instances  
- Change networking  
- Modify IAM policies  

To implement this, an IAM policy is created.

Example policy conceptually allows:

- StartInstances  
- StopInstances  
- DescribeInstances  

But does not allow:

- TerminateInstances  

AWS will enforce these permissions automatically.

---

### 4. Policies Are Written in JSON

AWS policies are written using **JSON**.

JSON stands for **JavaScript Object Notation**.

It is a structured data format used to represent information.

AWS chose JSON because it is:

- Easy for humans to read  
- Easy for computers to process  
- Standard across many systems  

A policy is simply a **JSON document describing permissions**.

---

### 5. Example IAM Policy (Real AWS Example)

Here is a real example of an IAM policy that allows reading S3 objects.

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "s3:GetObject"
          ],
          "Resource": "arn:aws:s3:::my-company-data/*"
        }
      ]
    }

Let’s break this down.

Version

Specifies the policy language version.

Statement

Contains the permission rules.

Effect

Defines whether the action is allowed or denied.

Possible values:

Allow  
Deny  

Action

Defines the AWS operations that are permitted.

Example:

s3:GetObject → allows reading files from S3.

Resource

Specifies which AWS resources the policy applies to.

In this example:

    arn:aws:s3:::my-company-data/*

Means all objects inside the bucket **my-company-data**.

So this policy allows:

- Reading files from the bucket

But it does not allow:

- Uploading files  
- Deleting files  
- Changing bucket settings  

---

### 6. Example AWS Use Case (Developer Access)

Let’s imagine a real AWS setup.

Company architecture:

- EC2 servers running applications  
- S3 storing application assets  
- CloudWatch storing logs  

Developers need to:

- Deploy code  
- Read logs  
- Restart servers  

They should not be able to:

- Delete databases  
- Modify IAM  
- Change networking  

So an administrator creates a policy allowing:

- EC2 start and stop  
- CloudWatch log viewing  
- S3 read access  

That policy is attached to the **Developers IAM Group**.

Then all developers automatically receive those permissions.

---

### 7. Human-Readable vs Machine-Enforced

Policies are designed to be both:

Human-readable  
Machine-enforced  

Human-readable means engineers can **look at the policy and understand it**.

Example:

    "Action": "s3:GetObject"

Clearly means:

Allow reading objects from S3.

Machine-enforced means AWS services **strictly enforce the rules** defined in the policy.

When a request is made to AWS:

1. AWS authenticates the identity  
2. AWS evaluates the policies attached to that identity  
3. AWS decides whether the action is allowed or denied  

This happens **every time an AWS API request is made**.

---

### 8. What Happens When AWS Evaluates a Policy

Whenever an identity tries to perform an action.

Example request:

Developer tries to start an EC2 instance.

AWS performs the following steps.

Step 1

Identify who is making the request.

Step 2

Retrieve all policies attached to the identity.

Step 3

Check whether the action is allowed.

Step 4

Allow or deny the request.

If the policy allows:

The action succeeds.

If the policy does not allow:

AWS blocks the request.

---

### 9. Example AWS CLI Request

Example command a developer might run:

    aws ec2 start-instances --instance-ids i-1234567890abcdef0

AWS checks:

- Who is making the request  
- What permissions they have  
- Whether they are allowed to start instances  

If the policy allows **ec2:StartInstances**, the command succeeds.

If not, AWS returns an **AccessDenied error**.

---

### 10. Mental Model for IAM Policies

Think of policies as **security rulebooks**.

The rulebook contains instructions like:

- This person can start servers  
- This service can read files  
- This application cannot delete resources  

Whenever someone tries to perform an action, AWS consults the rulebook.

If the rule exists and allows the action, it proceeds.

Otherwise, AWS blocks it.

---

### Key Takeaways

A policy is a permission document that defines what actions are allowed or denied in AWS.

Policies are written in JSON and contain structured rules describing permissions.

Policies can be attached to IAM users, groups, and roles.

AWS reads and enforces policies whenever an action is requested.

Policies are designed to be human-readable so engineers can understand them, while also being machine-enforced so AWS can automatically apply security rules.
