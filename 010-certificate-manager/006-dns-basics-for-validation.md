## DNS Basics for Validation — A, CNAME, TXT Records and TTLs

### 1. What is DNS?
DNS (Domain Name System) is like the **phonebook of the internet**.

It translates:
- Human-readable domain → example.com  
into  
- Machine-readable IP → 192.0.2.1

Without DNS:
→ Browsers wouldn’t know where to send your request

---

### 2. Why DNS Matters for Certificate Validation
When you request a certificate (especially in AWS ACM), the CA must verify:

"Do you actually own this domain?"

DNS is used as **proof of ownership**.

How?
→ You create a specific DNS record  
→ If it exists, CA trusts that you control the domain

---

### 3. Types of DNS Records (Core for ACM)

We will focus on:
1. A Record  
2. CNAME Record  
3. TXT Record  

---

### 4. A Record (Address Record)

Purpose:
Maps a domain → IP address

Example:
```txt
example.com → 192.0.2.1
```

Use case:
- Point domain to a server or load balancer

Important:
- Used for routing traffic
- NOT used for certificate validation

---

### 5. CNAME Record (Canonical Name)

Purpose:
Maps one domain → another domain

Example:
```txt
www.example.com → example.com
```

In ACM:
Used for **DNS validation**

Example:
```txt
_name.example.com → _random.acm-validations.aws
```

Meaning:
- You prove ownership by pointing to ACM’s validation domain

---

### 6. TXT Record (Text Record)

Purpose:
Stores arbitrary text data

Example:
```txt
example.com → "verification-code-123"
```

In certificates:
- Used by some CAs for domain validation
- Also used for:
  - SPF (email security)
  - DKIM
  - Domain ownership verification

---

### 7. DNS Validation Flow (Very Important)

When requesting a certificate:

1. ACM gives you a record like:
```txt
_name.example.com → _xyz.acm-validations.aws
```

2. You add it in your DNS provider (Route53, GoDaddy, etc.)

3. ACM checks:
- Does this record exist?

4. If yes:
→ Domain ownership verified  
→ Certificate issued

---

### 8. TTL (Time To Live)

TTL defines:
How long a DNS record is cached

Example:
```txt
TTL = 300 seconds (5 minutes)
```

Meaning:
- DNS resolvers cache this record for 5 minutes

---

### 9. Why TTL Matters

Lower TTL:
- Faster updates
- Useful during validation/debugging

Higher TTL:
- Better performance
- Less DNS traffic

For certificate validation:
→ Use low TTL (e.g., 60–300 seconds)

---

### 10. DNS Propagation (Common Confusion)

After adding a DNS record:
→ It is NOT instantly visible everywhere

Why?
- DNS servers cache old values

Time taken:
- Few seconds → few minutes → sometimes hours

---

### 11. How to Check DNS Records

Use tools like:

#### Check CNAME
```bash
dig CNAME _name.example.com
```

#### Check TXT
```bash
dig TXT example.com
```

#### Using nslookup
```bash
nslookup -type=TXT example.com
```

---

### 12. Real Example (ACM DNS Validation)

ACM gives:
```txt
_abcd.example.com → _xyz.acm-validations.aws
```

You add CNAME:
```txt
_abcd.example.com CNAME _xyz.acm-validations.aws
```

ACM verifies:
→ Certificate becomes "Issued"

---

### 13. Route53 (AWS DNS Service)
If you use Route53:

- ACM can automatically create validation records
- No manual DNS setup needed

This is the easiest setup in AWS

---

### 14. Common Mistakes

- Adding wrong record type (TXT instead of CNAME)
- Missing underscore (_) in record name
- Wrong domain/subdomain
- High TTL → slow validation
- Not waiting for propagation

---

### 15. Mental Model

Think of DNS validation like this:

CA says:
"Put this secret note at your domain"

You:
"Done"

CA:
"I can see it → you own the domain"

---

### 16. Why This Matters for ACM Mastery

You will use DNS validation everywhere:

- Requesting certificates
- Auto-renewal
- Multi-domain certificates
- Debugging failures

Without DNS understanding:
→ ACM will feel confusing and unpredictable

---

### 17. Key Takeaways

- DNS maps domain → IP or other domains
- A → IP mapping
- CNAME → domain to domain mapping
- TXT → text-based verification
- TTL controls caching duration
- DNS validation proves domain ownership
- Essential for ACM and TLS workflows
