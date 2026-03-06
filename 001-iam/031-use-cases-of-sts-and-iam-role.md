## Use Cases of STS and IAM Roles

AWS Security Token Service (STS) and IAM roles are widely used to provide **temporary credentials and secure access** to AWS resources.

Instead of using permanent access keys, systems and users obtain **temporary credentials through role assumption**.

This improves security and removes the need to store long-term secrets.

Some of the most common real-world use cases include:

Cross-account access  
EC2 instance roles  
Lambda execution roles  
Federated user access

Each of these scenarios uses **STS and IAM roles to provide temporary permissions**.

---

### 1. Cross-Account Access

Cross-account access allows identities in **one AWS account to access resources in another AWS account**.

This is commonly used in organizations with multiple AWS accounts.

Example structure:

    Account A → Developers
    Account B → Production infrastructure

Instead of creating separate users in both accounts, developers in Account A can **assume a role in Account B**.

Flow:

Step 1  
Developer logs into Account A.

Step 2  
Developer calls the AssumeRole API for a role in Account B.

Step 3  
STS verifies that the developer is allowed to assume the role.

Step 4  
STS returns temporary credentials.

Step 5  
Developer uses these credentials to access resources in Account B.

Example CLI command:

    aws sts assume-role \
        --role-arn arn:aws:iam::222222222222:role/DeploymentRole \
        --role-session-name DeploySession

Benefits of cross-account access:

No duplicate users across accounts  
Better account isolation  
Temporary access instead of permanent credentials

This pattern is widely used in **multi-account AWS environments**.

---

### 2. EC2 Instance Roles

Applications running on EC2 instances often need access to AWS services such as:

S3  
DynamoDB  
SQS  
CloudWatch

A common mistake is embedding **access keys in application code**.

Example (bad practice):

    ACCESS_KEY=AKIA123EXAMPLE
    SECRET_KEY=abc123secret

This creates serious security risks.

Instead, EC2 instances should use **IAM roles**.

---

### Example EC2 Role Setup

Step 1  
Create an IAM role.

Step 2  
Attach permissions to the role.

Example role policy:

    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::app-logs/*"
    }

Step 3  
Attach the role to the EC2 instance.

Step 4  
The application automatically receives temporary credentials.

AWS provides these credentials through the **instance metadata service**.

The application can now upload logs without storing access keys.

Benefits:

No permanent credentials  
Automatic credential rotation  
Improved security

---

### 3. Lambda Execution Roles

AWS Lambda functions also need permission to access other AWS services.

Examples include:

Reading from DynamoDB  
Writing logs to CloudWatch  
Accessing S3 buckets  
Sending messages to SQS

Lambda functions do not use access keys.

Instead, they use **execution roles**.

---

### Example Lambda Execution Role

A Lambda function writes logs to CloudWatch and reads from S3.

Role policy example:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "logs:CreateLogGroup",
            "logs:CreateLogStream",
            "logs:PutLogEvents"
          ],
          "Resource": "*"
        },
        {
          "Effect": "Allow",
          "Action": "s3:GetObject",
          "Resource": "arn:aws:s3:::app-data/*"
        }
      ]
    }

When the Lambda function runs:

Step 1  
Lambda assumes the execution role.

Step 2  
STS generates temporary credentials.

Step 3  
Lambda uses those credentials to access AWS services.

This ensures credentials are **temporary and managed by AWS automatically**.

---

### 4. Federated Users

Federated access allows users to log in to AWS using **external identity providers** instead of IAM users.

Examples of identity providers include:

Corporate SSO systems  
Active Directory  
Google  
Azure AD  
Okta

Instead of creating IAM users for every employee, organizations allow employees to log in through their existing identity system.

---

### Example Federated Login Flow

Step 1  
User logs into corporate identity provider.

Step 2  
Identity provider verifies credentials.

Step 3  
Identity provider sends authentication token to AWS.

Step 4  
STS exchanges the token for temporary AWS credentials.

Step 5  
User accesses AWS console or APIs.

This approach provides several benefits:

No need to manage IAM users for every employee  
Centralized identity management  
Temporary credentials for AWS access

Federated access is commonly used in **large enterprises**.

---

### 5. Mental Model

All these use cases rely on the same principle:

Instead of giving permanent credentials, AWS generates **temporary credentials through STS**.

Examples:

Cross-account access → AssumeRole.

EC2 instances → Instance roles.

Lambda functions → Execution roles.

External users → Federated identity.

In each case:

Identity assumes role → STS generates temporary credentials → credentials expire automatically.

---

### Key Takeaways

STS and IAM roles are used widely across AWS to provide temporary credentials.

Cross-account access allows identities in one account to access resources in another.

EC2 instance roles allow applications to access AWS services without storing access keys.

Lambda execution roles provide permissions for serverless functions.

Federated users allow external identity providers to authenticate users into AWS.

These mechanisms improve security by eliminating long-term credentials and using temporary access instead.
