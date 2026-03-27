## Email Validation — How It Works, Email Addresses Checked, Pros/Cons

### 1. What is Email Validation?
Email validation is a method used by Certificate Authorities (like ACM) to verify:

"Do you control this domain?"

Instead of DNS:
→ Verification is done via **email approval**

---

### 2. When is Email Validation Used?
You choose email validation when:
- You select **Email validation** during certificate request
- You cannot modify DNS records
- You prefer manual approval

---

### 3. How Email Validation Works (Step-by-Step)

1. You request a certificate  
2. ACM identifies domain  
3. It sends validation emails to specific addresses  
4. You click approval link in email  
5. Certificate gets issued  

---

### 4. Which Email Addresses Are Used? (Very Important)

ACM sends emails to **standard domain-based addresses**

For domain: example.com

It tries:

- admin@example.com  
- administrator@example.com  
- hostmaster@example.com  
- postmaster@example.com  
- webmaster@example.com  

Also:
- WHOIS registered email (if available)

---

### 5. Subdomain Behavior

If you request:

```txt
api.example.com
```

Emails are sent to:

- admin@api.example.com  
- admin@example.com  

(and similar variants)

---

### 6. What You Receive in Email

The email contains:

- Validation link  
- Certificate details  
- Expiry of request  

You must:
→ Click link and approve

---

### 7. Time Limit for Approval

- Validation email expires (usually within days)
- If not approved:
  → Request fails
  → You must request again

---

### 8. What Happens After Approval

Once approved:

- ACM verifies domain ownership  
- Certificate status → **ISSUED**  
- Ready to use  

---

### 9. Renewal Process (Important Limitation)

For email validation:

- ACM sends renewal emails before expiry  
- You MUST approve again  

If you don’t:
→ Certificate expires  

---

### 10. Pros of Email Validation

- Simple concept  
- No DNS access required  
- Useful in restricted environments  
- Works without technical DNS knowledge  

---

### 11. Cons of Email Validation (Very Important)

- Manual process (not automated)  
- Renewal requires human action  
- Emails may be missed or go to spam  
- Not suitable for production systems  
- Slower compared to DNS validation  

---

### 12. DNS vs Email Validation (Quick Comparison)

| Feature          | DNS Validation        | Email Validation        |
|------------------|----------------------|--------------------------|
| Automation       | Yes                  | No                       |
| Renewal          | Automatic            | Manual                   |
| Speed            | Fast                 | Slower                   |
| Reliability      | High                 | Medium                   |
| Recommended      | Yes                  | No (for production)      |

---

### 13. Real-World Scenario

Case:
You request certificate for example.com

ACM sends email to:
- admin@example.com

You:
- Open email
- Click approval link

Result:
→ Certificate issued

---

### 14. Common Problems

#### Problem 1: No access to email
→ Cannot validate domain

#### Problem 2: Email not received
- Check spam folder
- Check WHOIS email

#### Problem 3: Expired validation link
→ Request certificate again

---

### 15. When Should You Use Email Validation?

Use ONLY when:
- DNS access is not available
- Temporary/testing setup
- Legacy systems

Avoid in:
- Production systems
- Automated pipelines

---

### 16. AWS Best Practice

AWS strongly recommends:
→ Use **DNS validation**

Because:
- Fully automated
- Supports auto-renewal
- No manual intervention

---

### 17. Mental Model

Think like this:

CA says:
"Reply from official email to prove ownership"

You:
"Clicked approval link"

CA:
"Verified → you own domain"

---

### 18. Why This Matters for ACM Mastery

You must know:
- Both validation methods
- When to use each
- Limitations of email validation

This helps in:
- Debugging issues
- Working in restricted environments
- Designing robust systems

---

### 19. Key Takeaways

- Email validation uses domain-based email approval
- Sent to standard addresses (admin, hostmaster, etc.)
- Manual process → not scalable
- Renewal also requires approval
- DNS validation is preferred for production
