## Multi-Domain and Wildcard Certificates — Request Syntax and Limitations

### 1. What Are Multi-Domain Certificates (SAN Certificates)?
A **multi-domain certificate** (also called SAN certificate) allows you to secure **multiple domains/subdomains using a single certificate**.

Example:
```
example.com
www.example.com
api.example.com
```

All covered by ONE certificate.

---

### 2. What Are Wildcard Certificates?
A **wildcard certificate** secures all subdomains under a domain.

Example:
```
*.example.com
```

Covers:
- api.example.com
- app.example.com
- anything.example.com

---

### 3. Key Difference (Very Important)

| Feature            | Multi-Domain (SAN)         | Wildcard                    |
|--------------------|----------------------------|-----------------------------|
| Covers             | Specific domains           | All subdomains              |
| Flexibility        | Explicit control           | Broad coverage              |
| Example            | api, www, blog             | *.example.com               |

---

### 4. ACM CLI Syntax — Multi-Domain Certificate

```bash
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names www.example.com api.example.com blog.example.com \
  --validation-method DNS
```

---

### 5. ACM CLI Syntax — Wildcard Certificate

```bash
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names "*.example.com" \
  --validation-method DNS
```

---

### 6. Important Rule (Common Mistake)

Wildcard does NOT cover root domain.

```
*.example.com ❌ does NOT include example.com
```

So you must include both:

```bash
--domain-name example.com \
--subject-alternative-names "*.example.com"
```

---

### 7. Combining Multi-Domain + Wildcard

You can combine both:

```bash
aws acm request-certificate \
  --domain-name example.com \
  --subject-alternative-names "*.example.com" api.other.com \
  --validation-method DNS
```

---

### 8. SAN Limit (Very Important)

ACM limit:
- Up to **100 domain names per certificate**

Includes:
- Root domain
- All SAN entries
- Wildcards

---

### 9. Validation Requirement

Each domain (SAN) must be validated.

Example:
```
example.com
api.example.com
```

→ Each requires DNS validation

Good news:
- Often same DNS record works for subdomains

---

### 10. Wildcard Validation Insight

Validating:
```
*.example.com
```

Usually requires:
→ Validation for example.com

---

### 11. Limitations of Wildcard Certificates

#### 11.1 Only One Level Deep
```
*.example.com ✅
a.b.example.com ❌ (not covered)
```

---

#### 11.2 Cannot Cover Multiple Domains
```
*.example.com ≠ *.other.com
```

You need separate entries.

---

#### 11.3 Security Risk (Broad Scope)

If compromised:
→ ALL subdomains are affected

---

### 12. Limitations of Multi-Domain Certificates

- Must explicitly list each domain
- Validation required per domain
- Harder to manage at large scale (if many domains)

---

### 13. When to Use Multi-Domain

Use when:
- You have fixed known domains
- Need precise control
- Different domain names (example.com + example.org)

---

### 14. When to Use Wildcard

Use when:
- Many dynamic subdomains
- Microservices (service1.example.com, service2.example.com)
- SaaS apps (user1.example.com, user2.example.com)

---

### 15. Best Practice (Real-World)

Often combine both:

```
example.com
*.example.com
```

This covers:
- Root domain
- All subdomains

---

### 16. Performance & Cost Insight

- Using 1 SAN cert is better than many certs
- Reduces:
  - Quota usage
  - Management overhead

---

### 17. Common Mistakes

- Forgetting root domain with wildcard
- Exceeding 100 SAN limit
- Not validating all domains
- Using wildcard incorrectly for deep subdomains

---

### 18. Real-World Example

SaaS app:

Domains:
```
example.com
*.example.com
api.partner.com
```

Single certificate can cover all.

---

### 19. Why This Matters for ACM Mastery

You must understand:
- How to minimize certificates
- How to scale domain coverage
- How to design efficient certificate strategy

This is critical in:
- Multi-tenant apps
- Large systems
- Cost optimization

---

### 20. Key Takeaways

- Multi-domain = specific domains via SAN
- Wildcard = all subdomains
- Wildcard does NOT cover root domain
- Limit = 100 domains per cert
- Combine both for best coverage
- Choose based on use case and scale
