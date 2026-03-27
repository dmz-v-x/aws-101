## Origin Access Control (OAC) — Modern Access Control for CloudFront

### 1. What is Origin Access Control (OAC)?

#### 1.1 Definition

**Origin Access Control (OAC)** is the modern mechanism used to:

> Securely allow CloudFront to access private S3 content using signed requests

---

#### 1.2 Core Idea

Instead of using a static identity (like OAI):

- CloudFront **signs each request**
- S3 verifies the signature
- Only valid CloudFront requests are allowed

---

#### 1.3 Key Insight

> OAC uses **AWS Signature Version 4 (SigV4)** for request authentication

---

### 2. Why OAC Exists

---

#### 2.1 Problems with OAI

- Static identity  
- Limited region support  
- No strong request-level authentication  
- Poor integration with modern AWS features  

---

#### 2.2 What OAC Improves

- Per-request authentication  
- Full region support  
- Better security model  
- Supports encryption (SSE-KMS)  

---

### 3. How OAC Works

---

#### 3.1 Request Flow

```
User → CloudFront → (Signed Request) → S3 → Response
```

---

#### 3.2 Step-by-Step Flow

1. User sends request to CloudFront  
2. CloudFront prepares request to S3  
3. CloudFront signs request using SigV4  
4. S3 validates signature  
5. If valid → returns object  
6. Response cached and returned to user  

---

#### 3.3 Key Insight

> Every request from CloudFront to S3 is **authenticated and verifiable**

---

### 4. AWS Signature Version 4 (SigV4)

---

#### 4.1 What is SigV4?

SigV4 is a signing process that:

- Adds cryptographic signature to requests  
- Ensures request integrity  
- Verifies identity of requester  

---

#### 4.2 What Gets Signed

- HTTP method  
- Headers  
- Path  
- Query parameters  
- Payload (optional)

---

#### 4.3 Why It Matters

- Prevents tampering  
- Ensures only authorized access  
- Enables fine-grained security  

---

### 5. Key Features of OAC

---

#### 5.1 Strong Authentication

- Every request is signed  
- No anonymous access  

---

#### 5.2 Supports SSE-KMS

- Works with KMS-encrypted S3 objects  
- Secure access to encrypted data  

---

#### 5.3 Full Region Support

- Works across all AWS regions  
- No legacy limitations  

---

#### 5.4 Better Security Model

- Dynamic authentication  
- Fine-grained control  
- Future-proof  

---

### 6. OAC vs OAI (Quick Comparison)

| Feature | OAI | OAC |
|-------|-----|-----|
| Type | Legacy | Modern |
| Auth method | Static identity | SigV4 signed requests |
| Region support | Limited | All regions |
| SSE-KMS support | Limited | Fully supported |
| Security | Basic | Strong |

---

### 7. Creating an OAC (Step-by-Step Concept)

---

#### 7.1 Step 1: Create OAC

Define:
- Name  
- Description  
- Signing behavior  

---

#### 7.2 Signing Behavior Options

- **Sign requests (recommended)**  
- Sign requests + include additional headers  
- Do not sign (not recommended)  

---

#### 7.3 Key Insight

> Always choose **"Sign requests"** for secure setups

---

### 8. Attaching OAC to CloudFront Distribution

---

#### 8.1 Step 1: Open Distribution Settings

- Navigate to your CloudFront distribution  
- Go to **Origins**

---

#### 8.2 Step 2: Select Origin

- Choose your S3 origin  

---

#### 8.3 Step 3: Attach OAC

- Select created OAC  
- Save changes  

---

#### 8.4 Step 4: Deploy Changes

- Wait for CloudFront deployment  

---

### 9. Updating S3 Bucket Policy for OAC

---

#### 9.1 Why This is Required

S3 must allow:
> Only CloudFront (via OAC) to access objects

---

#### 9.2 Example Bucket Policy

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::ACCOUNT_ID:distribution/DISTRIBUTION_ID"
        }
      }
    }
  ]
}
```

---

#### 9.3 Key Components Explained

- `Service: cloudfront.amazonaws.com` → allows CloudFront  
- `SourceArn` → restricts to specific distribution  
- `s3:GetObject` → read access  

---

#### 9.4 Key Insight

> This ensures **only your CloudFront distribution** can access S3

---

### 10. Blocking Public Access

---

#### 10.1 Required Step

- Enable **Block Public Access** in S3  
- Remove any public policies  

---

#### 10.2 Result

- Direct S3 access blocked  
- Only CloudFront can fetch objects  

---

### 11. End-to-End Secure Architecture

```
User → CloudFront → (SigV4 signed request) → S3 (private)
```

---

### 12. Common Issues & Debugging

---

#### 12.1 403 Access Denied

Causes:
- Incorrect bucket policy  
- Wrong distribution ARN  
- Missing permissions  

---

#### 12.2 Signature Errors

Causes:
- Misconfigured OAC  
- Incorrect origin setup  

---

#### 12.3 Public Access Still Enabled

- S3 bypass possible  
- Security risk  

---

### 13. Best Practices

---

#### 13.1 Always Use OAC

- For all new systems  
- Avoid OAI completely  

---

#### 13.2 Restrict by SourceArn

- Prevent unauthorized distributions  

---

#### 13.3 Use HTTPS Only

- Secure origin communication  

---

#### 13.4 Combine with Other Security Layers

- WAF  
- Signed URLs  
- IAM policies  

---

### 14. Real-World Use Case

---

#### Static Website (Secure Setup)

- S3 bucket (private)  
- CloudFront distribution  
- OAC attached  
- ACM certificate for HTTPS  

Result:
- Fast  
- Secure  
- Scalable  

---

### 15. Migration from OAI to OAC

---

#### Steps

1. Create OAC  
2. Update distribution to use OAC  
3. Update bucket policy  
4. Remove OAI  

---

#### Key Insight

> Migration improves security and future compatibility

---

### 16. Final Mental Model

> OAC transforms CloudFront into a **secure, authenticated client of S3**, where every request is signed, verified, and controlled.

It provides:
- Strong security  
- Modern AWS compatibility  
- Fine-grained access control  
- Support for encrypted data  
