## Origin Access Identity (OAI) — Legacy Access Control in CloudFront

### 1. What is Origin Access Identity (OAI)?

#### 1.1 Definition

**Origin Access Identity (OAI)** is a special CloudFront identity used to:

> Allow CloudFront to access private S3 bucket content securely

---

#### 1.2 Core Idea

Instead of making your S3 bucket public:

- You keep the bucket **private**
- Grant access **only to CloudFront (via OAI)**

---

#### 1.3 Why OAI Exists

Without OAI:

- S3 bucket must be public  
- Anyone can access content directly  

With OAI:

- Only CloudFront can access S3  
- Users must go through CloudFront  

---

#### 1.4 Key Insight

> OAI is used to enforce: **"Users cannot bypass CloudFront and directly access S3"**

---

### 2. Problem Without OAI

---

#### 2.1 Public Bucket Issue

If S3 bucket is public:

```
User → CloudFront (OK)
User → S3 directly (ALSO possible ❌)
```

---

#### 2.2 Risks

- No control over access  
- No caching benefits enforced  
- Security vulnerabilities  

---

### 3. How OAI Solves This

---

#### 3.1 Architecture

```
User → CloudFront → S3 (via OAI)
```

---

#### 3.2 Flow

1. User requests content via CloudFront  
2. CloudFront uses OAI identity  
3. S3 verifies permission  
4. Returns object only if request is from OAI  

---

#### 3.3 Result

- S3 bucket stays private  
- Only CloudFront can fetch content  

---

### 4. How OAI Works Internally

---

#### 4.1 OAI is a Virtual User

OAI behaves like:

- A special AWS identity  
- With its own canonical user ID  

---

#### 4.2 Bucket Policy Grants Access

You explicitly allow OAI in S3 bucket policy:

Example:

```
{
  "Effect": "Allow",
  "Principal": {
    "CanonicalUser": "OAI_CANONICAL_USER_ID"
  },
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```

---

#### 4.3 Key Insight

> OAI works through **S3 bucket policies**, not IAM roles

---

### 5. Setting Up OAI (Conceptual Steps)

---

#### 5.1 Steps

1. Create OAI in CloudFront  
2. Attach OAI to distribution  
3. Update S3 bucket policy  
4. Block public access on S3  

---

#### 5.2 Result

- S3 is private  
- Only CloudFront can read objects  

---

### 6. Limitations of OAI (VERY IMPORTANT)

---

#### 6.1 Legacy Feature

OAI is considered:

> **Legacy (older approach)**

AWS recommends using:
- **Origin Access Control (OAC)** instead

---

#### 6.2 Region Limitations

OAI:
- Does NOT support all AWS regions  
- Has compatibility issues with newer regions  

---

#### 6.3 No Support for Advanced Features

OAI does NOT support:

- AWS Signature Version 4 (SigV4) fully  
- Fine-grained request signing  
- Some encryption integrations  

---

#### 6.4 Limited Security Model

- Static identity (less flexible)  
- No per-request authentication  
- Cannot enforce advanced policies  

---

#### 6.5 No Support for SSE-KMS (Properly)

OAI has limitations when:
- Using KMS-encrypted S3 objects  

---

#### 6.6 Key Insight

> OAI is **static and limited**, while modern systems require dynamic and secure authentication

---

### 7. OAI vs Public Access

---

#### 7.1 Public Bucket

- Easy setup  
- Insecure  
- No access control  

---

#### 7.2 OAI

- Secure  
- Prevents direct access  
- Works with private buckets  

---

### 8. OAI vs OAC (Modern Replacement)

---

#### 8.1 OAI

- Legacy  
- Uses canonical user  
- Limited features  

---

#### 8.2 OAC (Origin Access Control)

- Modern  
- Uses SigV4 signing  
- Supports all regions  
- Better security  
- Supports SSE-KMS  

---

#### 8.3 Key Insight

> Always prefer **OAC over OAI** for new systems

---

### 9. When You Might Still See OAI

---

#### 9.1 Legacy Systems

- Older CloudFront distributions  
- Existing production setups  

---

#### 9.2 Migration Scenarios

- Gradually moving from OAI → OAC  

---

### 10. Migration Consideration

---

#### 10.1 Why Migrate?

- Better security  
- Future-proof  
- Full AWS support  

---

#### 10.2 Migration Approach

1. Create OAC  
2. Update distribution  
3. Modify bucket policy  
4. Remove OAI  

---

### 11. Common Mistakes

---

#### Mistake 1: Keeping bucket public

Defeats purpose of OAI

---

#### Mistake 2: Incorrect bucket policy

Leads to:
- 403 Access Denied  

---

#### Mistake 3: Using OAI in new projects

Leads to:
- Technical debt  

---

#### Mistake 4: Forgetting to block public access

Creates:
- Security loopholes  

---

### 12. Final Mental Model

> OAI is a **legacy mechanism** that allows CloudFront to securely access private S3 content by acting as a trusted identity.

However:
- It is limited  
- Not supported everywhere  
- Being replaced by OAC  

Modern systems should:
- Avoid OAI  
- Use OAC for better security and flexibility  
