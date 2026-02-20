## Common DNS Problems 

### 1. Absolute Zero — Why DNS Problems Feel Mysterious

At the simplest level:

DNS failures often show:

• “Site not reachable”  
• “Server not found”  
• “Connection timed out”

Yet infrastructure may be running perfectly.

DNS is a hidden dependency.

---

### 2. Critical Insight — DNS Is a Distributed System

DNS involves:

• Client cache  
• OS cache  
• Browser cache  
• Recursive resolver cache  
• ISP cache  
• Authoritative DNS servers

Many layers.

Many possible failure points.

---

## DNS NOT RESOLVING — MOST COMMON FAILURE

---

### 3. Absolute Zero — What “DNS Not Resolving” Means

Simplest definition:

**Domain name cannot be translated into IP address**

Example:

    example.com → ❌ No IP returned

---

### 4. How This Appears to Users

Common symptoms:

• “DNS_PROBE_FINISHED_NXDOMAIN”  
• “Server not found”  
• “This site can’t be reached”

---

### 5. Most Common Causes

Typical root causes:

• Missing DNS record  
• Typo in record  
• Wrong hosted zone  
• Incorrect nameservers  
• Domain not delegated  
• Expired domain  
• DNS propagation delay

---

### 6. Classic Beginner Mistake — Typo

Example:

Configured:

    exampl.com

Instead of:

    example.com

DNS silently fails.

---

### 7. Hosted Zone Mismatch — Very Common

Record created in:

Wrong hosted zone.

DNS system never sees it.

---

### 8. Nameserver Misconfiguration — Critical Issue

Registrar pointing to:

Incorrect nameservers.

DNS queries routed to wrong authority.

---

### 9. Domain Expiration — Frequently Overlooked

Expired domain:

DNS stops resolving entirely.

---

## WRONG IP ADDRESS — SUBTLE & DANGEROUS

---

### 10. Absolute Zero — What “Wrong IP” Means

DNS resolves successfully.

But returns:

Incorrect destination.

Example:

    example.com → Wrong Server

---

### 11. Symptoms of Wrong IP Issues

Common symptoms:

• Website loads incorrect content  
• Old version of site  
• Different application  
• SSL certificate mismatch  
• Unexpected server response

---

### 12. Most Common Causes

Typical causes:

• Incorrect A/AAAA record  
• Old IP still configured  
• Copy-paste errors  
• Environment mix-up  
• Elastic IP reassociation  
• Infrastructure migration errors

---

### 13. Classic Migration Mistake

Server changed.

DNS record not updated.

Traffic still flows to old server.

---

### 14. Dangerous Scenario — IP Reuse

Old IP now assigned to:

Different system.

DNS sends users to unintended target.

---

### 15. SSL Certificate Warning — Important Signal

Wrong IP often triggers:

Certificate mismatch warnings.

Valuable debugging clue.

---

## CACHING ISSUES — MOST CONFUSING PROBLEM

---

### 16. Absolute Zero — What DNS Caching Means

DNS caching stores:

Previous resolution results.

To improve performance.

---

### 17. Why Caching Exists

Without caching:

Every request → DNS lookup.

Internet would be slower & overloaded.

Caching improves efficiency.

---

### 18. Core Problem with Caching

Cached answers may become:

Outdated.

Leading to inconsistent behavior.

---

### 19. Classic Symptom — “Works for Me but Not Others”

Some users see:

New server.

Others see:

Old server.

Caching discrepancy.

---

### 20. TTL — Governing Control Mechanism

TTL (Time To Live) defines:

How long responses stay cached.

---

### 21. Why TTL Creates Delays

DNS changes propagate only after:

Caches expire.

No instant updates.

---

### 22. Classic Beginner Confusion

“I updated DNS but nothing changed.”

Reality:

Resolvers still using cached answers.

---

### 23. Multi-Layer Cache Complexity

Caches exist at:

• Browser  
• OS  
• Recursive resolver  
• ISP infrastructure

Flushing one cache may not help.

---

### 24. Negative Caching — Subtle Gotcha

Failed lookups may also be cached.

Record added later → Still fails temporarily.

---

### 25. Gotcha — Long TTL Mistakes

TTL set to:

Very high value.

Changes take hours/days to propagate.

---

### 26. Gotcha — Very Low TTL Trade-Off

Low TTL:

• Faster updates  
• Higher DNS query load  
• Increased resolver traffic

Balance required.

---

## DEBUGGING MENTAL MODEL

---

### 27. Step 1 — Ask: Resolution vs Connectivity?

Does domain resolve?

Yes → Likely wrong IP / caching / backend issue  
No → Likely DNS configuration problem

---

### 28. Step 2 — Check Authoritative DNS

Verify:

Correct records exist.

---

### 29. Step 3 — Consider Caching Effects

If records correct but behavior inconsistent:

Caching likely involved.

---

### 30. Step 4 — Check Nameserver Delegation

Ensure domain points to:

Correct DNS authority.

---

## COMMON BEGINNER MISCONCEPTIONS

---

### 31. Misconception #1 — DNS Changes Instant

Incorrect.

TTL & caching delays apply.

---

### 32. Misconception #2 — One Cache Exists

Incorrect.

Multiple caching layers.

---

### 33. Misconception #3 — Wrong IP Always Obvious

Incorrect.

Often subtle & confusing.

---

### 34. Misconception #4 — DNS Failure = Server Down

Incorrect.

Server may be perfectly healthy.

---

### 35. Misconception #5 — TTL Only Performance Setting

Incorrect.

TTL affects stability & change behavior.

---

## PRACTICAL GOTCHAS & RISKS

---

### 36. Gotcha — Editing Wrong Hosted Zone

Very common in multi-environment setups.

---

### 37. Gotcha — Mixing Staging & Production IPs

Leads to confusing wrong-IP behavior.

---

### 38. Gotcha — Ignoring Negative Cache Effects

New records may appear broken temporarily.

---

### 39. Gotcha — DNS vs CDN Cache Confusion

DNS cache ≠ CDN cache.

Separate systems.

---

## FINAL MENTAL MODEL

---

### 40. What DNS Problems Truly Represent

Most DNS issues fall into:

**Three fundamental categories**

• Resolution Failure  
• Incorrect Destination  
• Stale Information

---

### 41. DNS Not Resolving

Name → No IP mapping.

---

### 42. Wrong IP

Name → Incorrect mapping.

---

### 43. Caching Issues

Name → Correct mapping but outdated visibility.

---

### 44. Elegant Big Picture

DNS problems are rarely random.

They typically involve:

**Visibility delays + Configuration mismatches + Distributed caching behavior**

Understanding caching & delegation removes most mystery.

DNS failures become predictable.

Debuggable.

Rational.
