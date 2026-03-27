## Revocation Basics and CRL/OCSP for Private CAs

### 1. What is Certificate Revocation?
Revocation means:

Invalidating a certificate **before its expiry date**

Why?
Because a certificate might become unsafe even if it hasn’t expired.

---

### 2. Why Do We Need Revocation?

Common reasons:

- Private key compromised  
- Domain ownership lost  
- Certificate issued incorrectly  
- Insider/security breach  

Without revocation:
→ Certificate remains trusted  
→ Security risk  

---

### 3. What Happens After Revocation?

Once revoked:

- Certificate should NOT be trusted
- Clients (browsers/services) must reject it
- Even if expiry date is in future

---

### 4. How Do Clients Know a Cert is Revoked?

Two main mechanisms:

1. CRL (Certificate Revocation List)  
2. OCSP (Online Certificate Status Protocol)  

---

### 5. CRL (Certificate Revocation List)

#### What is CRL?
A CRL is a **list of revoked certificates** published by a CA.

Think:
→ A blacklist of certificates

---

#### How CRL Works:

1. CA revokes certificate  
2. Adds it to CRL  
3. Publishes CRL at a URL  
4. Client downloads CRL  
5. Checks if cert is in list  

---

#### Example Flow:

- Client connects to server  
- Server sends certificate  
- Client fetches CRL  
- If cert serial number is in CRL → reject  

---

### 6. CRL Characteristics

- Stored as a file (periodically updated)
- Can grow large over time
- Cached by clients

---

### 7. OCSP (Online Certificate Status Protocol)

#### What is OCSP?
OCSP is a **real-time query system** for certificate status.

Instead of downloading a list:
→ Client asks CA directly

---

#### How OCSP Works:

1. Client sends request to OCSP server  
2. "Is this certificate valid?"  
3. CA responds:
   - GOOD
   - REVOKED
   - UNKNOWN  

---

### 8. OCSP vs CRL (Comparison)

| Feature       | CRL                          | OCSP                        |
|---------------|------------------------------|-----------------------------|
| Method        | Download full list           | Query per certificate       |
| Speed         | Slower (large lists)         | Faster (real-time)          |
| Data usage    | Higher                       | Lower                       |
| Freshness     | Depends on update interval   | Real-time                   |

---

### 9. OCSP Stapling (Advanced Concept)

Problem:
- OCSP requires extra network call

Solution:
→ OCSP Stapling

How it works:
- Server fetches OCSP response
- "Staples" it to TLS handshake
- Client doesn’t need to query CA

---

### 10. AWS Private CA Context

When using AWS Private CA:

You can configure:

- CRL distribution
- OCSP support

---

### 11. CRL in AWS Private CA

You must:

1. Configure S3 bucket  
2. Store CRL there  
3. Provide CRL URL in certificates  

Flow:
- CA updates CRL in S3
- Clients fetch from S3

---

### 12. Revoking a Certificate in AWS

Using CLI:

```bash
aws acm-pca revoke-certificate \
  --certificate-authority-arn <CA_ARN> \
  --certificate-serial <SERIAL_NUMBER> \
  --revocation-reason KEY_COMPROMISE
```

---

### 13. Revocation Reasons

Common reasons:

- KEY_COMPROMISE  
- CA_COMPROMISE  
- AFFILIATION_CHANGED  
- SUPERSEDED  
- CESSATION_OF_OPERATION  

---

### 14. Important Limitation (Very Important)

Revocation only works if:

→ Clients actually CHECK CRL or OCSP

If not:
→ Revoked cert may still be accepted

---

### 15. Real-World Scenario

Case:
- Private key leaked

Steps:
1. Revoke certificate  
2. Update CRL  
3. Issue new certificate  
4. Deploy new certificate  

---

### 16. Best Practices

1. Always enable CRL for Private CA  
2. Use OCSP where possible  
3. Automate revocation workflows  
4. Monitor compromised keys  
5. Rotate certificates after revocation  

---

### 17. Common Mistakes

- Not configuring CRL  
- Assuming revocation is automatic  
- Not updating clients to check revocation  
- Ignoring OCSP setup  

---

### 18. Mental Model

Think:

Expiry = time-based invalidation  
Revocation = emergency invalidation  

---

### 19. Why This Matters for ACM Mastery

In real systems:

- Security incidents happen  
- Keys get compromised  

Revocation is your:
→ Emergency shutdown mechanism  

---

### 20. Key Takeaways

- Revocation invalidates certificates before expiry  
- CRL = list of revoked certificates  
- OCSP = real-time status check  
- AWS Private CA supports both  
- Revocation only works if clients verify it  
- Critical for security-sensitive systems  
