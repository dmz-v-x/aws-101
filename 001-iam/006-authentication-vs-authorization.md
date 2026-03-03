## Authentication vs Authorization 

### 1. The Core Idea

Security systems always answer **two different questions**:

Who are you?  
What are you allowed to do?

These two questions correspond to two separate processes:

Authentication → Who are you?  
Authorization → What are you allowed to do?

Both are required for secure systems.

---

### 2. What Is Authentication?

Authentication is the process of **verifying identity**.

It answers the question:

Who is trying to access the system?

The goal of authentication is to confirm that the person or system **is who they claim to be**.

---

### 3. How Identity Is Proven

Identity can be proven using several methods.

Some common examples include:

Password  
Security key  
Access keys  
Biometric verification  
Multi-factor authentication

In AWS, authentication often happens using things like:

- Username and password
- Access keys
- Temporary credentials
- Multi-factor authentication (MFA)

Once the system verifies the identity, the user is considered **authenticated**.

---

### 4. Real-World Example of Authentication

Imagine entering an office building.

At the entrance, security checks your ID card.

The security guard verifies:

- Your name
- Your employee badge
- Your identity

If the badge is valid, you are allowed into the building.

That process is **authentication**.

It proves **who you are**, but it does not yet decide **where you can go inside the building**.

---

### 5. What Is Authorization?

Authorization happens **after authentication**.

Once the system knows who you are, the next question is:

What are you allowed to do?

Authorization determines:

- Which resources you can access
- Which actions you can perform
- Which services you can use

It defines **permissions**.

---

### 6. Real-World Example of Authorization

Continuing the office building example:

After entering the building, you may try to access different areas.

Examples:

- Office floor
- Meeting rooms
- Server room
- Finance department

Your employee badge determines **which doors open**.

Some doors may open for you.

Some doors may remain locked.

That process is **authorization**.

It controls **what you are allowed to access**.

---

### 7. Authentication vs Authorization (Side-by-Side)

Authentication answers:

Who are you?

Authorization answers:

What are you allowed to do?

Authentication happens **first**, because the system must know **who you are** before deciding what permissions you have.

---

### 8. Simple Mental Model

Think of a website login.

Step 1: Login  
You enter your email and password.

The system verifies your credentials.

This is **authentication**.

Step 2: Access Control  
Once logged in, the system decides:

- Can you edit data?
- Can you delete records?
- Can you access admin panels?

This is **authorization**.

---

### 9. How IAM Handles Authentication

In AWS, IAM handles authentication by verifying identities using credentials.

Examples of IAM authentication methods include:

- IAM user login with username and password
- Access keys used by applications
- Temporary credentials issued to services
- Multi-factor authentication (MFA)

When a request is sent to AWS, AWS first verifies the identity.

If the identity is valid, authentication succeeds.

---

### 10. How IAM Handles Authorization

After authentication, IAM evaluates **permissions**.

Permissions are defined using **policies**.

Policies describe:

- Allowed actions
- Allowed resources
- Conditions for access

When an identity attempts an action, AWS evaluates the policies attached to that identity.

If the policies allow the action, the request succeeds.

If the policies deny the action, the request is blocked.

---

### 11. What Happens When Someone Makes an AWS Request

Every AWS action goes through the same security flow.

Step 1  
A user or system sends a request to AWS.

Step 2  
AWS performs authentication to verify the identity.

Step 3  
AWS evaluates IAM policies.

Step 4  
AWS determines whether the action is authorized.

Step 5  
The request is either allowed or denied.

This process happens **for every AWS API request**.

---

### 12. Example AWS Scenario

Imagine a developer trying to start a server.

Step 1  
The developer logs into AWS.

Authentication verifies the developer’s identity.

Step 2  
The developer attempts to start a server.

IAM checks authorization policies.

If the policy allows starting servers, the action succeeds.

If the policy does not allow it, AWS blocks the request.

---

### 13. Why This Difference Is Extremely Important

Many security systems rely on this separation.

Authentication confirms identity.

Authorization controls permissions.

If either step is missing, security breaks.

For example:

If authentication is weak → attackers can impersonate users.

If authorization is weak → users may gain excessive privileges.

IAM protects AWS by handling **both processes together**.

---

### 14. Final Mental Model

Authentication proves identity.

Authorization decides permissions.

AWS IAM handles both by:

- Verifying identities through credentials
- Evaluating policies to determine permissions

Every request made to AWS passes through these two stages before it is allowed to execute.

---

### Key Takeaways

Authentication verifies who you are.

Authorization determines what you are allowed to do.

Authentication always happens before authorization.

IAM manages authentication using credentials and identity verification.

IAM manages authorization using policies that define permissions.

Both processes are essential for securing AWS infrastructure.
