## Password Policies

In AWS IAM, password policies define the **rules that control how IAM user passwords must be created and managed**.

These policies help enforce **strong authentication practices** and protect AWS accounts from unauthorized access.

Without strong password policies, attackers may be able to gain access through:

- Weak passwords
- Password guessing attacks
- Credential stuffing
- Brute force attempts

Password policies are therefore an important part of **account-level security**.

They apply specifically to **IAM users who log in to the AWS Management Console**.

---

### 1. What Is an IAM Password Policy

An **IAM password policy** is a set of rules that determine:

- How strong passwords must be
- How often passwords must change
- Whether users can reuse old passwords
- Whether passwords expire automatically

This policy is configured **at the AWS account level** and applies to all IAM users.

Administrators configure password policies using the **IAM service**.

---

### 2. Example Password Policy Settings

AWS allows administrators to configure several password requirements.

Common password policy settings include:

Minimum password length  
Require uppercase letters  
Require lowercase letters  
Require numbers  
Require special characters  
Prevent password reuse  
Password expiration period

These rules ensure users create **strong and difficult-to-guess passwords**.

---

### 3. Password Complexity

Password complexity rules ensure that passwords are **not simple or easily guessable**.

Example requirements might include:

Minimum length of 12 characters

Must include:

- Uppercase letters
- Lowercase letters
- Numbers
- Special characters

Example strong password:

    P@ssw0rdSecure!2024

Example weak password:

    password123

Weak passwords are dangerous because attackers can easily guess them using automated tools.

Complex passwords significantly reduce the success of brute-force attacks.

---

### 4. Enforcing Password Complexity in AWS

Administrators can configure password complexity requirements in IAM.

Example configuration:

Minimum password length: 12

Require at least one:

- Uppercase letter
- Lowercase letter
- Number
- Special character

Prevent reuse of the last 5 passwords.

This ensures that users cannot reuse weak or previously compromised passwords.

---

### 5. Password Rotation

Password rotation requires users to **change their passwords periodically**.

For example:

Passwords must be changed every 90 days.

This reduces the risk that stolen or leaked credentials remain valid for long periods.

Example policy rule:

Password expiration: 90 days

When the password expires, the user must change it before accessing the console again.

---

### 6. Why Password Rotation Matters

Password rotation helps reduce risks such as:

Credential leaks  
Long-term credential exposure  
Undetected compromises

Example scenario:

An attacker obtains a password through a phishing attack.

If the password never expires, the attacker could access the system indefinitely.

But if passwords rotate every 90 days, the compromised credential eventually becomes useless.

---

### 7. Preventing Password Reuse

AWS password policies can also prevent users from reusing old passwords.

Example rule:

Prevent reuse of last 10 passwords.

This ensures users cannot rotate through a set of old passwords repeatedly.

Without this rule, users might simply alternate between two passwords.

---

### 8. Account Security Implications

Weak password policies can create major security risks.

If IAM users use weak passwords, attackers may gain access through:

Brute-force attacks  
Password spraying  
Credential stuffing from leaked databases

Once attackers gain access to an AWS account, they may:

- Launch expensive resources
- Steal sensitive data
- Delete infrastructure
- Modify security settings

Strong password policies significantly reduce these risks.

---

### 9. Combining Password Policies with MFA

Password policies should always be combined with **Multi-Factor Authentication (MFA)**.

MFA adds an additional security layer.

Example login flow:

Step 1  
User enters password.

Step 2  
User must provide MFA code from a device.

Even if an attacker steals the password, they cannot access the account without the MFA code.

This dramatically improves account security.

---

### 10. Real AWS Best Practice

AWS security best practices recommend:

Minimum password length of at least 12 characters.

Require uppercase, lowercase, numbers, and symbols.

Enable password expiration (for IAM users).

Prevent password reuse.

Enable MFA for all privileged users.

Avoid using IAM users where possible — prefer IAM roles and federated access.

---

### 11. Mental Model

Think of password policies as **the security rules for the front door of your AWS account**.

Weak passwords make the door easy to break into.

Strong password policies make the door significantly harder to breach.

When combined with MFA, the system becomes much more secure.

---

### Key Takeaways

Password policies control how IAM user passwords are created and managed.

They enforce password complexity requirements such as length, character types, and uniqueness.

Password rotation forces users to change passwords periodically to reduce credential exposure.

Preventing password reuse ensures users do not recycle old passwords.

Weak password policies can expose AWS accounts to brute-force and credential theft attacks.

Strong password policies combined with MFA significantly improve AWS account security.
