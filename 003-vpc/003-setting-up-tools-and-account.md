## Setting Up AWS Tools and Accounts (AWS Free Tier, AWS Console, AWS CLI)

---

### 1. What Is AWS?

AWS is a **cloud computing platform** provided by Amazon.

AWS provides services such as:

- Virtual servers (EC2)
- Networking (VPC)
- Storage (S3)
- Databases (RDS, DynamoDB)
- Security (IAM)

Instead of buying physical hardware, companies use AWS infrastructure on demand.

Example:

Instead of buying servers:

```
Company buys EC2 instances
```

Instead of building a physical network:

```
Company creates a VPC
```

Everything runs inside AWS data centers.

---

### 2. What Is an AWS Account?

An **AWS account** is your **identity inside AWS**.

It allows you to:

- create cloud resources
- manage services
- access the AWS Console
- use the AWS CLI
- run infrastructure in AWS regions

Think of it like your **personal cloud workspace**.

Example:

```
AWS Account
 ├── VPCs
 ├── EC2 Instances
 ├── S3 Buckets
 ├── Databases
 └── IAM Users
```

Every resource belongs to **one AWS account**.

---

### 3. What Is the AWS Free Tier?

The **AWS Free Tier** allows beginners to learn AWS without paying for most services.

The Free Tier includes limited usage of services such as:

| Service | Free Tier Limit |
|------|------|
| EC2 | 750 hours per month |
| S3 | 5 GB storage |
| Lambda | 1 million requests |
| DynamoDB | 25 GB storage |

This is enough to practice **most beginner labs**.

However, some services (like NAT Gateway or large instances) may still incur charges.

Always monitor usage.

---

### 4. Creating an AWS Account

To create an account:

1. Visit:

```
https://aws.amazon.com
```

2. Click **Create an AWS Account**

3. Enter:

- email
- password
- account name

4. Add billing information (credit/debit card)

5. Verify phone number

6. Choose **Basic Support Plan (Free)**

After registration you will receive an **AWS account ID**.

Example:

```
123456789012
```

This number uniquely identifies your AWS account.

---

### 5. What Is the AWS Management Console?

The AWS Management Console is the **web interface for AWS**.

It allows you to manage AWS resources through a graphical interface.

Example actions in the console:

- create EC2 instance
- create VPC
- configure security groups
- view logs
- manage IAM users

The console is accessed through a browser.

Example:

```
https://console.aws.amazon.com
```

---

### 6. AWS Regions

AWS infrastructure is divided into **regions**.

A region is a **geographical location containing multiple data centers**.

Example regions:

```
us-east-1
eu-west-1
ap-south-1
```

Example:

```
Mumbai → ap-south-1
```

Resources are always created inside a region.

Example:

```
VPC in ap-south-1
EC2 in ap-south-1
```

---

### 7. What Is the AWS CLI?

The AWS CLI is a **command-line tool for interacting with AWS services**.

CLI allows you to control AWS using terminal commands.

Example CLI command:

```
aws ec2 describe-instances
```

Instead of clicking in the console, you run commands.

This is useful for:

- automation
- scripting
- infrastructure management
- DevOps workflows

---

### 8. Installing AWS CLI

Download the AWS CLI from:

```
https://aws.amazon.com/cli/
```

Install depending on your OS.

#### Windows

Download:

```
AWSCLIV2.msi
```

Run installer.

---

#### macOS

Install using Homebrew:

```
brew install awscli
```

---

#### Linux

Install using package manager or pip.

Example:

```
sudo apt install awscli
```

---

### 9. Verify Installation

Check if AWS CLI is installed.

Run:

```
aws --version
```

Example output:

```
aws-cli/2.13.0 Python/3.11
```

If this appears, installation is successful.

---

### 10. How AWS Authentication Works

AWS CLI needs credentials to authenticate your requests.

AWS uses:

```
Access Key ID
Secret Access Key
```

These credentials belong to an **IAM user**.

Best practice:

```
Never use root credentials
Always use IAM users
```

IAM stands for **Identity and Access Management**.

---

### 11. Creating IAM User for CLI

Steps in AWS Console:

1. Open **IAM**
2. Click **Users**
3. Click **Create user**
4. Enable:

```
Programmatic access
```

This generates:

```
Access Key ID
Secret Access Key
```

Download or copy these keys.

You will need them for AWS CLI configuration.

---

### 12. Configuring AWS CLI

Run:

```
aws configure
```

You will see prompts.

Example:

```
AWS Access Key ID:
AWS Secret Access Key:
Default region name:
Default output format:
```

Example configuration:

```
AWS Access Key ID: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name: ap-south-1
Default output format: json
```

These values are stored locally.

Example location:

```
~/.aws/credentials
~/.aws/config
```

---

### 13. Understanding AWS CLI Profiles

AWS CLI supports multiple profiles.

Example:

```
dev
staging
production
```

You can configure different credentials.

Example command:

```
aws configure --profile dev
```

Use profile:

```
aws ec2 describe-instances --profile dev
```

---

### 14. What Is AWS STS?

It is a service used to verify identity and generate temporary credentials.

One useful command is:

```
aws sts get-caller-identity
```

This command returns information about the identity currently authenticated in AWS CLI.

---

### 15. Running the Verification Command

Run:

```
aws sts get-caller-identity
```

Example output:

```
{
 "UserId": "AIDABCDEFG123456789",
 "Account": "123456789012",
 "Arn": "arn:aws:iam::123456789012:user/admin"
}
```

Explanation:

| Field | Meaning |
|------|------|
| UserId | IAM user identifier |
| Account | AWS account ID |
| Arn | Amazon Resource Name |

This confirms your CLI authentication is working.

---

### 16. Understanding ARN

ARN stands for:

```
Amazon Resource Name
```

It uniquely identifies AWS resources.

Example:

```
arn:aws:iam::123456789012:user/admin
```

Structure:

```
arn:partition:service:region:account-id:resource
```

Example:

```
arn:aws:s3:::my-bucket
```

---

### 17. Example CLI Commands

List S3 buckets:

```
aws s3 ls
```

Describe EC2 instances:

```
aws ec2 describe-instances
```

List VPCs:

```
aws ec2 describe-vpcs
```

These commands query AWS APIs directly.

---

### 18. Lab Exercise

Perform the following steps.

1. Install AWS CLI
2. Create IAM user
3. Run:

```
aws configure
```

4. Enter:

```
Access Key
Secret Key
Region
Output format
```

5. Run verification command:

```
aws sts get-caller-identity
```

---

### 19. Expected Output

Example response:

```
{
 "UserId": "AIDABCDEFG123456789",
 "Account": "123456789012",
 "Arn": "arn:aws:iam::123456789012:user/dev-user"
}
```

This confirms:

- CLI works
- credentials work
- AWS account is accessible

---

### 20. Troubleshooting

If command fails:

#### Error: credentials not configured

Run again:

```
aws configure
```

---

#### Error: invalid credentials

Check:

- Access Key
- Secret Key

---

#### Error: region not set

Specify region:

```
aws configure
```

---

### 21. Checkpoint

Your setup is correct if this command works:

```
aws sts get-caller-identity
```

You should see:

```
Account ID
IAM user ARN
User ID
```

Example:

```
Account: 123456789012
```

If you can see this output, your AWS CLI setup is complete.
