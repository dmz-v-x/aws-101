## IAM for CI/CD 

Modern software development relies heavily on **CI/CD pipelines** to automatically build, test, and deploy applications.

These pipelines often need to interact with AWS services such as:

S3  
ECR  
ECS  
Lambda  
CloudFormation  
EKS  

For example, a CI/CD pipeline might:

Build a Docker image  
Push it to Amazon ECR  
Deploy the image to ECS  
Update infrastructure with CloudFormation  

To perform these actions, the CI/CD system must **authenticate with AWS**.

Historically, pipelines used **long-lived access keys**, but this approach created serious security risks.

Modern AWS architectures now use **OIDC federation with IAM roles**, which eliminates the need to store AWS secrets.

This section explains the entire concept from **absolute basics to advanced architecture**.

---

### 1. What CI/CD Is

CI/CD stands for:

Continuous Integration  
Continuous Deployment (or Continuous Delivery)

These practices automate the process of building, testing, and deploying applications.

Typical CI/CD pipeline stages:

Code commit  
Build  
Test  
Package  
Deploy  

Example pipeline flow:

    Developer pushes code → CI pipeline builds app
    → Tests run automatically
    → Deployment to AWS infrastructure

Tools used for CI/CD include:

GitHub Actions  
GitLab CI  
Jenkins  
CircleCI  
Bitbucket Pipelines  

These tools run pipelines that need access to AWS.

---

### 2. The Authentication Problem in CI/CD

A CI/CD system needs permission to interact with AWS.

For example, a deployment pipeline may need to:

Push images to ECR  
Update Lambda functions  
Deploy CloudFormation stacks  
Upload artifacts to S3  

To do this, the pipeline must authenticate with AWS.

Historically, pipelines used **access keys**.

Example configuration:

    AWS_ACCESS_KEY_ID=AKIA123EXAMPLE
    AWS_SECRET_ACCESS_KEY=secret123

These keys were stored in:

GitHub secrets  
CI environment variables  
Configuration files  

This approach introduced major security risks.

---

### 3. Why Stored Secrets Are Dangerous

Long-term credentials stored in CI systems create several risks.

If the repository is compromised, attackers may access secrets.

If logs expose credentials, attackers can steal them.

If developers accidentally commit credentials to the repository, they become public.

If credentials are reused across systems, compromise spreads.

Because access keys are long-lived, attackers can use them indefinitely.

These risks led to the adoption of **secretless authentication using OIDC federation**.

---

### 4. Modern Solution: OIDC Federation

OIDC stands for **OpenID Connect**.

It is a standard authentication protocol built on top of OAuth.

OIDC allows external identity providers to authenticate users or systems.

In CI/CD pipelines, OIDC allows the pipeline to:

Authenticate with AWS  
Obtain temporary credentials  
Without storing AWS secrets

This approach uses:

IAM roles  
STS temporary credentials  
OIDC identity tokens

---

### 5. High-Level OIDC Authentication Flow

Modern CI/CD authentication works like this:

    GitHub Action
          ↓
    Requests OIDC token from GitHub
          ↓
    Sends token to AWS STS
          ↓
    STS validates token
          ↓
    STS issues temporary credentials
          ↓
    Pipeline deploys infrastructure

This means:

No AWS access keys stored in the repository.

All credentials are **temporary and automatically expire**.

---

### 6. What GitHub Actions Is

GitHub Actions is a CI/CD platform built into GitHub.

It allows developers to create automated workflows that run when events occur.

Examples of triggers:

Code pushed to repository  
Pull request created  
Scheduled workflow  
Manual deployment trigger  

Example workflow file:

    .github/workflows/deploy.yml

A workflow may contain steps such as:

Checkout code  
Build application  
Run tests  
Deploy infrastructure

When deploying to AWS, the workflow must authenticate with AWS.

---

### 7. GitHub OIDC Provider

GitHub supports OIDC tokens that allow workflows to authenticate with cloud providers.

When a workflow runs, GitHub can generate an **OIDC identity token**.

This token proves that the workflow:

Originated from GitHub  
Belongs to a specific repository  
Belongs to a specific branch or workflow

AWS can verify this token and allow the workflow to assume an IAM role.

---

### 8. Step-by-Step Setup of GitHub OIDC with AWS

Step 1  
Create an OIDC identity provider in AWS.

Example provider URL:

    https://token.actions.githubusercontent.com

This tells AWS to trust GitHub as an identity provider.

---

Step 2  
Create an IAM role for GitHub Actions.

This role will grant deployment permissions.

Example permissions:

    {
      "Effect": "Allow",
      "Action": [
        "ecr:PutImage",
        "ecs:UpdateService",
        "cloudformation:Deploy"
      ],
      "Resource": "*"
    }

---

Step 3  
Configure the trust policy.

The trust policy allows GitHub workflows to assume the role.

Example trust policy:

    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Principal": {
            "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
          },
          "Action": "sts:AssumeRoleWithWebIdentity",
          "Condition": {
            "StringEquals": {
              "token.actions.githubusercontent.com:sub": "repo:my-org/my-repo:ref:refs/heads/main"
            }
          }
        }
      ]
    }

This ensures only workflows from that repository and branch can assume the role.

---

Step 4  
Configure GitHub workflow.

Example workflow step:

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        role-to-assume: arn:aws:iam::123456789012:role/github-deploy-role
        aws-region: us-east-1

This step requests temporary credentials from AWS.

---

### 9. Example Full Deployment Flow

Example pipeline:

    Developer pushes code
        ↓
    GitHub Action runs workflow
        ↓
    GitHub issues OIDC token
        ↓
    Workflow calls AWS STS
        ↓
    STS verifies token
        ↓
    STS returns temporary credentials
        ↓
    Workflow deploys application

All credentials automatically expire after the workflow finishes.

---

### 10. Benefits of OIDC-Based CI/CD Authentication

Using OIDC instead of access keys provides major security advantages.

No AWS secrets stored in GitHub.

Temporary credentials instead of permanent keys.

Automatic credential expiration.

Fine-grained repository and branch restrictions.

Improved auditability through CloudTrail.

This approach significantly reduces credential leakage risks.

---

### 11. Advanced Security Controls

Organizations often add additional restrictions.

Examples include:

Restrict role assumption to specific repositories.

Restrict workflows to specific branches.

Require specific workflow names.

Limit permissions using least privilege policies.

Example condition restricting repository:

    "Condition": {
      "StringLike": {
        "token.actions.githubusercontent.com:sub": "repo:my-org/my-repo:*"
      }
    }

---

### 12. Real-World Enterprise Architecture

Large companies commonly use this pattern.

Example:

    GitHub Actions
         ↓
    OIDC Federation
         ↓
    IAM Role (Deployment Role)
         ↓
    Deploy to AWS infrastructure

This architecture allows secure deployments without managing secrets.

---

### 13. Mental Model

Think of OIDC CI/CD authentication like this:

The pipeline does not carry a permanent AWS key.

Instead, it presents a **temporary identity token** to AWS.

AWS verifies the token and issues a **temporary access badge**.

Once the deployment finishes, the badge expires.

No secrets remain stored anywhere.

---

### Key Takeaways

CI/CD pipelines need permissions to deploy infrastructure in AWS.

Traditional pipelines used long-lived access keys, which created security risks.

OIDC federation allows CI/CD systems to authenticate with AWS without storing secrets.

GitHub Actions can generate OIDC tokens that AWS trusts.

AWS STS exchanges these tokens for temporary credentials.

IAM roles define the permissions available to the pipeline.

This approach improves security by eliminating long-term credentials and using temporary access instead.
