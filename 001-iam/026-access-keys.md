## Access Keys (Very Sensitive)

Access keys are one of the most sensitive credentials in AWS.

They allow **programmatic access** to AWS services through tools such as:

- AWS CLI  
- SDKs (Python, JavaScript, Java, etc.)  
- Automation scripts  
- Applications running on servers  

Unlike passwords, which are used for **console login**, access keys are used by **programs and systems** to communicate with AWS APIs.

Because access keys provide direct access to AWS resources, **leaked access keys are one of the most common causes of AWS security breaches**.

---

### 1. What Access Keys Are

An **access key** is a credential used for programmatic authentication with AWS.

Each access key consists of two parts:

Access Key ID  
Secret Access Key

Example structure:

    Access Key ID: AKIAIOSFODNN7EXAMPLE
    Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

These credentials allow tools such as the AWS CLI to sign API requests.

Example CLI configuration:

    aws configure

You will be asked to provide:

    AWS Access Key ID
    AWS Secret Access Key
    Default Region
    Output Format

After configuration, the CLI can make authenticated API requests.

---

### 2. Example CLI Request Using Access Keys

Example command:

    aws s3 ls

This command lists S3 buckets.

Behind the scenes:

- The CLI signs the request using the access key and secret key
- AWS verifies the signature
- AWS checks IAM permissions
- AWS returns the result

If the access key has permission to list buckets, the command succeeds.

If not, AWS returns an **AccessDenied** error.

---

### 3. Why Access Keys Are Very Sensitive

Access keys provide **direct API access to AWS resources**.

If attackers obtain valid access keys, they can perform any actions that the associated IAM identity is allowed to perform.

Example attacker actions:

- Launch expensive EC2 instances  
- Delete databases  
- Exfiltrate sensitive data  
- Disable security logging  
- Create backdoor accounts  

Because access keys do not require interactive login, attackers can automate these actions quickly.

---

### 4. Why Access Keys Leak

Access key leaks are very common in cloud environments.

The most frequent causes include:

Hardcoding keys in source code.

Example:

    const accessKey = "AKIA123456EXAMPLE";
    const secretKey = "abc123secretkey";

If this code is pushed to GitHub, the credentials become public.

Uploading keys to public repositories.

Attackers actively scan GitHub for exposed AWS credentials.

Storing keys in configuration files.

Example:

    ~/.aws/credentials

If a machine is compromised, attackers may retrieve the credentials.

Logging keys accidentally.

If logs contain access keys, they may be exposed in monitoring systems.

Sharing keys between developers.

Shared credentials create accountability and security problems.

---

### 5. Real Example of an Access Key Breach

Imagine a developer accidentally commits AWS credentials to GitHub.

Attackers monitoring public repositories detect the key within minutes.

The attacker uses the key to run commands like:

    aws ec2 run-instances

They launch hundreds of GPU instances for cryptocurrency mining.

The result:

The company receives a **massive AWS bill**.

This type of incident happens frequently in real-world environments.

---

### 6. Access Key Rotation Best Practices

Access keys should **never remain active indefinitely**.

Rotation means periodically replacing old keys with new ones.

Typical best practice:

Rotate access keys every **90 days**.

Steps for rotating keys:

Step 1  
Create a new access key.

Step 2  
Update applications or scripts to use the new key.

Step 3  
Test that the system works with the new key.

Step 4  
Disable and delete the old key.

This process ensures compromised keys become useless over time.

---

### 7. Limiting Access Key Permissions

Access keys should always follow the **least privilege principle**.

Bad practice:

    "Action": "*"
    "Resource": "*"

This grants full administrator access.

Better practice:

    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::app-logs/*"
    }

This limits the key to uploading logs only.

Even if the key leaks, the attacker can do **very limited damage**.

---

### 8. Monitoring Access Key Usage

AWS provides tools to monitor access key usage.

Examples include:

IAM credential reports

AWS CloudTrail

IAM Access Analyzer

These tools help identify:

Unused access keys  
Suspicious API activity  
Compromised credentials

Unused access keys should be **disabled or deleted immediately**.

---

### 9. When NOT to Use Access Keys

In modern AWS architectures, access keys should be avoided whenever possible.

Better alternatives include:

IAM Roles  
Temporary credentials  
Federated identity access

Example scenario:

Instead of embedding access keys in an EC2 server, the server should assume an **IAM role**.

The role provides **temporary credentials automatically**.

This eliminates the need to store secrets on the server.

---

### 10. Example Using IAM Role Instead of Access Keys

Bad practice:

Application stores access keys in configuration.

    ACCESS_KEY=AKIA123...
    SECRET_KEY=abc123...

Better approach:

Attach an IAM role to the EC2 instance.

Example role policy:

    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::app-logs/*"
    }

Now the application automatically receives temporary credentials from AWS.

No access keys are stored in the application.

---

### 11. Mental Model

Think of access keys as **permanent API passwords for AWS**.

If someone obtains them, they can interact with your AWS environment programmatically.

Because of this, access keys must be:

Protected carefully  
Rotated regularly  
Used only when necessary  

Whenever possible, replace them with **temporary credentials from IAM roles**.

---

### Key Takeaways

Access keys are credentials used for programmatic access to AWS services.

They consist of an access key ID and a secret access key.

Leaked access keys are one of the most common causes of AWS security breaches.

Keys often leak through source code, public repositories, configuration files, and logs.

Best practices include rotating keys regularly, limiting permissions, and monitoring usage.

Modern AWS architectures prefer IAM roles and temporary credentials instead of long-lived access keys.
