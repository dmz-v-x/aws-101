## CA Hierarchy Design — Root CA (Offline) vs Intermediate CAs

### 1. What is CA Hierarchy?
A CA hierarchy is how you structure your Certificate Authorities.

Instead of using a single CA, you create a chain:

Root CA  
   ↓  
Intermediate CA(s)  
   ↓  
Leaf Certificates (used by services)

This improves **security and scalability**

---

### 2. Why Not Use a Single CA?

If you use only one CA:

- If compromised → entire system is broken  
- No isolation  
- Hard to manage large systems  

So we use hierarchy:
→ Separation of trust and responsibility  

---

### 3. Components of CA Hierarchy

#### Root CA (Top Level)
- Highest authority  
- Self-signed  
- Trust anchor  

#### Intermediate CA(s)
- Signed by Root CA  
- Issues certificates  

#### Leaf Certificates
- Used by servers/services  

---

### 4. Root CA — What It Is

Root CA is:

- The ultimate authority  
- Trusted by all clients  
- Signs intermediate CAs  

Important:
→ Root CA signs ONLY intermediates (not end certificates)

---

### 5. Root CA Should Be OFFLINE (Very Important)

Best practice:
→ Keep Root CA **offline**

Meaning:
- Not exposed to network  
- Not used frequently  
- Stored securely  

---

### 6. Why Keep Root CA Offline?

If Root CA is compromised:

→ Entire PKI is compromised  

So:
- Minimize usage  
- Protect it heavily  

---

### 7. Intermediate CA — What It Is

Intermediate CA:

- Signed by Root CA  
- Issues certificates to services  
- Handles day-to-day operations  

---

### 8. Why Use Intermediate CAs?

Benefits:

- Protect Root CA  
- Allow multiple environments  
- Enable delegation  

Example:
- Prod CA  
- Dev CA  
- Staging CA  

---

### 9. Typical Hierarchy Design

```
Root CA (offline)
   ↓
Intermediate CA (online)
   ↓
Leaf Certificates (services)
```

---

### 10. Multi-Intermediate Design (Advanced)

You can have:

```
Root CA
   ↓
Intermediate CA - Prod
Intermediate CA - Dev
Intermediate CA - IoT
```

Each handles separate workloads

---

### 11. AWS Private CA Implementation

In AWS:

- Root CA → Created via Private CA  
- Intermediate CA → Also created via Private CA  
- Root signs intermediate  

Even though AWS manages keys:
→ You still design hierarchy  

---

### 12. How Signing Works

Flow:

1. Create Root CA  
2. Create Intermediate CA  
3. Generate CSR for Intermediate  
4. Root signs Intermediate  
5. Intermediate becomes ACTIVE  

---

### 13. Certificate Issuance Flow

```
Client → requests certificate
        ↓
Intermediate CA → issues certificate
        ↓
Certificate returned
```

Root CA is NOT involved here

---

### 14. Security Benefits of Hierarchy

- Root key rarely used  
- Compromise limited to intermediate  
- Easier revocation  
- Better audit control  

---

### 15. Revocation Strategy

If Intermediate is compromised:

- Revoke intermediate  
- Issue new intermediate  
- Root remains safe  

---

### 16. Validity Period Strategy

Typical:

- Root CA → long validity (10–20 years)  
- Intermediate CA → shorter (3–5 years)  
- Leaf cert → very short (days/months)  

---

### 17. Real-World Example

Company setup:

Root CA (offline)  
   ↓  
Intermediate CA (Prod)  
   ↓  
Certificates for services  

Intermediate CA (Dev)  
   ↓  
Certificates for test systems  

---

### 18. Common Mistakes

- Using Root CA directly to issue certs  
- Not separating environments  
- Not planning hierarchy early  
- Not rotating intermediates  

---

### 19. Mental Model

Root CA = King (never leaves castle)  
Intermediate CA = General (runs operations)  
Leaf cert = Soldiers (used in field)  

---

### 20. Why This Matters for ACM Mastery

In production:

- You MUST design secure PKI  
- Hierarchy is standard practice  
- Required in enterprise systems  

---

### 21. Key Takeaways

- Always use Root → Intermediate → Leaf structure  
- Keep Root CA offline and secure  
- Use Intermediate CA for issuing certificates  
- Design multiple intermediates for different environments  
- This structure improves security and scalability  
