## What is AWS Certificate Manager (ACM)? — Public vs Private Certificates; Managed vs Imported

### 1. What is AWS Certificate Manager (ACM)?
AWS Certificate Manager (ACM) is a service that helps you:

- Provision (create) SSL/TLS certificates
- Manage them (renew, track, deploy)
- Use them with AWS services (ALB, CloudFront, API Gateway)

In simple terms:
ACM = A managed service to handle certificates so you don’t have to do it manually

---

### 2. Why ACM Exists
Without ACM, you would need to:

- Generate private keys manually
- Create CSR
- Send to CA
- Install certificates on servers
- Renew certificates manually
- Handle expiry and downtime risks

ACM automates all of this.

---

### 3. Types of Certificates in ACM

There are two main types:

1. Public Certificates  
2. Private Certificates  

---

### 4. Public Certificates (ACM Public CA)

These are certificates issued by Amazon’s public Certificate Authority.

#### Key Characteristics:
- Trusted by browsers globally
- Used for public websites
- Free of cost
- Automatically renewed

#### Use Cases:
- Public websites (https://example.com)
- APIs exposed to internet
- CloudFront distributions
- Application Load Balancers

---

### 5. Private Certificates (ACM Private CA)

These are certificates issued using AWS Private Certificate Authority.

#### Key Characteristics:
- NOT trusted by default
- Used for internal systems
- Requires AWS Private CA (paid service)
- Full control over certificate policies

#### Use Cases:
- Internal services (microservices)
- mTLS (mutual TLS)
- Internal load balancers
- Corporate networks

---

### 6. Public vs Private (Quick Comparison)

| Feature        | Public Certificate        | Private Certificate         |
|---------------|--------------------------|-----------------------------|
| Trust         | Globally trusted         | Internal trust only         |
| Cost          | Free                     | Paid (Private CA)           |
| Use Case      | Public apps              | Internal systems            |
| Validation    | Domain validation needed | No public validation needed |
| Control       | Limited                  | Full control               |

---

### 7. Managed Certificates (Amazon-Issued)

These are certificates that ACM creates and manages for you.

#### Features:
- ACM generates private key internally
- You never see or handle the key
- Auto-renewal handled by ACM
- Easy integration with AWS services

#### Key Idea:
Fully managed → minimal effort

---

### 8. Imported Certificates (Bring Your Own)

These are certificates you generate outside AWS and upload to ACM.

#### Features:
- You generate:
  - Private key
  - CSR
  - Certificate from external CA
- Then import into ACM

#### Important:
- ACM does NOT auto-renew imported certificates
- You must manually re-import on renewal

---

### 9. Managed vs Imported (Comparison)

| Feature            | Managed (ACM-Issued) | Imported Certificate       |
|--------------------|----------------------|----------------------------|
| Key generation     | Done by ACM          | Done by you                |
| Private key access | Not accessible       | Fully accessible           |
| Renewal            | Automatic            | Manual                     |
| Effort             | Very low             | Higher                     |
| Flexibility        | Limited              | High                       |

---

### 10. When to Use Managed Certificates

Use ACM-managed certificates when:
- You want simplicity
- You use AWS services (ALB, CloudFront)
- You don’t need access to private key
- You want automatic renewal

---

### 11. When to Use Imported Certificates

Use imported certificates when:
- You already have a certificate from another CA
- You need access to private key
- You use special certificate types (EV, custom policies)
- You need to use same cert outside AWS

---

### 12. Important Limitation (Very Important)

ACM-managed certificates:
- Cannot export private key

Imported certificates:
- You already have private key → usable anywhere

---

### 13. AWS Integration (Where ACM is Used)

ACM certificates are commonly attached to:

- Application Load Balancer (ALB)
- CloudFront (CDN)
- API Gateway
- Elastic Beanstalk

---

### 14. Region Behavior (Critical Detail)

- Certificates are region-specific
- Exception:
  - CloudFront requires cert in us-east-1

---

### 15. Real-World Flow

#### Using ACM Managed Cert:
1. Request certificate
2. Validate domain (DNS)
3. ACM issues cert
4. Attach to ALB/CloudFront
5. ACM auto-renews

#### Using Imported Cert:
1. Generate key + CSR
2. Get cert from external CA
3. Import into ACM
4. Attach to service
5. Manually renew later

---

### 16. Why This Matters for Mastery

To master ACM, you must:
- Know public vs private use cases
- Decide managed vs imported correctly
- Understand limitations (like no key export)
- Design systems based on these constraints

---

### 17. Key Takeaways

- ACM simplifies certificate management in AWS
- Public certs = for internet-facing apps
- Private certs = for internal systems
- Managed certs = automatic, no key access
- Imported certs = manual, full control
- Choosing the right type is critical in real-world architecture
