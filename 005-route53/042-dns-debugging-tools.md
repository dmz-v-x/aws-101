## DNS Debugging Tools

### 1. Absolute Zero — Why DNS Debugging Feels Difficult

At the simplest level:

DNS is:

• Distributed  
• Cached  
• Layered  
• Indirect

Failures rarely produce obvious errors.

---

### 2. Critical Insight — DNS Failures Are Usually Visibility Problems

Most DNS issues involve:

• Wrong configuration  
• Stale cache  
• Delegation mistakes  
• Resolver differences

Debugging tools reveal hidden state.

---

## ABSOLUTE ZERO — WHAT ARE DNS DEBUGGING TOOLS?

---

### 3. Simplest Definition

DNS debugging tools allow you to ask:

**"What does the DNS system actually know right now?"**

Instead of guessing.

---

### 4. Why These Tools Matter

Without tools:

DNS troubleshooting becomes speculation.

With tools:

DNS becomes observable.

Deterministic.

---

## dig — THE PROFESSIONAL DNS TOOL

---

### 5. Absolute Zero — What is dig?

dig = **Domain Information Groper**

It is the most precise DNS diagnostic tool.

Used by engineers.

---

### 6. Simplest Mental Model

dig asks DNS servers directly:

"Give me raw DNS truth."

---

### 7. Basic dig Query

Example:

    dig example.com

What this does:

Queries resolver → Requests DNS records → Displays response details.

---

### 8. What dig Reveals

dig shows:

• Returned IPs  
• TTL values  
• Record types  
• DNS server used  
• Response flags  
• Authority information

Extremely rich data.

---

### 9. Querying Specific Record Types

Example:

    dig example.com A
    dig example.com AAAA
    dig example.com MX

Why this matters:

Different problems affect different records.

---

### 10. Critical Debugging Skill — Checking Authoritative Servers

Example:

    dig example.com NS

Then query nameserver directly:

    dig example.com @ns-123.awsdns.com

Removes resolver/cache ambiguity.

---

### 11. Why This Is Important

Resolvers may cache stale answers.

Authoritative servers show ground truth.

---

### 12. Gotcha — dig Output Looks Overwhelming

Beginners often panic seeing:

• Flags  
• Sections  
• Timers  
• Metadata

Focus initially on:

ANSWER SECTION.

---

### 13. Gotcha — Resolver vs Authority Confusion

Default dig queries:

Recursive resolver.

Not authoritative server.

Important distinction.

---

## nslookup — THE CLASSIC TOOL

---

### 14. Absolute Zero — What is nslookup?

nslookup = **Name Server Lookup**

Older, simpler DNS diagnostic tool.

Still widely used.

---

### 15. Simplest Mental Model

nslookup asks:

"What IP does this name resolve to?"

Less verbose than dig.

---

### 16. Basic nslookup Query

Example:

    nslookup example.com

Output:

Cleaner & simpler than dig.

Good for quick checks.

---

### 17. Querying Specific Record Types

Example:

    nslookup -type=MX example.com
    nslookup -type=TXT example.com

---

### 18. Querying Specific DNS Server

Example:

    nslookup example.com 8.8.8.8

Why this matters:

Different resolvers may return different answers.

---

### 19. dig vs nslookup — Key Differences

dig:

• Rich details  
• Precise diagnostics  
• Engineering-grade visibility

nslookup:

• Simple & fast  
• Beginner-friendly  
• Less metadata

Both useful.

---

### 20. Gotcha — nslookup Can Hide Critical Clues

Because it omits:

TTL, flags, authority chains.

Use dig for deep analysis.

---

## ROUTE 53 CONSOLE CHECKS — INFRASTRUCTURE VIEW

---

### 21. Absolute Zero — Why Check Route 53 Console?

Because DNS failures often originate from:

Configuration mistakes.

Console shows intended state.

---

### 22. What Console Checks Reveal

Console shows:

• Record definitions  
• Record types  
• TTL settings  
• Alias targets  
• Routing policies  
• Health check associations

---

### 23. Critical Insight — Console = Intended Truth

dig/nslookup = Observed Truth

Mismatch between them = Root cause clue.

---

### 24. Common Console Debugging Steps

Verify:

• Correct hosted zone  
• Correct domain name  
• Correct record type  
• Correct IP/target  
• TTL sanity  
• Alias configuration  
• Health check state

---

### 25. Gotcha — Editing Wrong Hosted Zone

Very common mistake.

Especially with:

• Multiple environments  
• Similar domain names  
• Staging vs production confusion

---

### 26. Gotcha — Record Exists But DNS Still Wrong

Often caused by:

Caching.

dig reveals TTL & propagation status.

---

### 27. Gotcha — Alias vs Record Type Confusion

Alias records behave differently from:

Traditional A/CNAME.

Check target resource carefully.

---

## CORE DEBUGGING MENTAL MODEL

---

### 28. Step 1 — Does DNS Resolve?

Use:

    nslookup example.com

If fails → Resolution problem  
If works → Continue analysis

---

### 29. Step 2 — Verify Returned Data

Use:

    dig example.com

Check:

• IP correctness  
• TTL values  
• Unexpected answers

---

### 30. Step 3 — Check Authoritative Truth

Use:

    dig example.com @authoritative-server

Removes cache ambiguity.

---

### 31. Step 4 — Compare With Console

Console config vs dig output.

Mismatch indicates:

• Propagation delay  
• Delegation issue  
• Cache interference  
• Wrong zone edits

---

## ADVANCED DEBUGGING INSIGHTS

---

### 32. Resolver Differences Are Normal

Example:

Google DNS vs ISP DNS may differ temporarily.

Not always error.

Often caching.

---

### 33. TTL Explains Most “Weird Behavior”

High TTL → Slow updates  
Low TTL → Fast changes

dig exposes TTL directly.

---

### 34. Negative Cache Effects

Previously failed lookups may persist.

Even after record creation.

---

### 35. Delegation Problems Require NS Checks

Use:

    dig example.com NS

Wrong nameservers → Entire DNS chain broken.

---

### 36. Authoritative vs Recursive Queries — Critical Distinction

Recursive resolver:

May cache stale answers.

Authoritative server:

Always source of truth.

---

## COMMON BEGINNER MISCONCEPTIONS

---

### 37. Misconception #1 — DNS Tools Always Agree

Incorrect.

Caching layers cause temporary differences.

---

### 38. Misconception #2 — Console Configuration = Live DNS

Incorrect.

Propagation & caching delays apply.

---

### 39. Misconception #3 — nslookup Enough for Debugging

Incorrect.

dig required for deep visibility.

---

### 40. Misconception #4 — DNS Problems Random

Incorrect.

Usually systematic & explainable.

---

### 41. Misconception #5 — TTL Irrelevant During Debugging

Incorrect.

TTL often explains everything.

---

## FINAL MENTAL MODEL

---

### 42. What DNS Debugging Tools Truly Provide

They convert DNS from:

Invisible system → Observable system.

---

### 43. dig Role

Deep diagnostic visibility.

---

### 44. nslookup Role

Quick resolution checks.

---

### 45. Route 53 Console Role

Infrastructure configuration verification.

---

### 46. Elegant Big Picture

DNS debugging is not magic.

It is:

**Comparing Intended State vs Observed State across a Cached Distributed System**

Tools reveal:

Where mismatch exists.

Why failures occur.

How resolution truly behaves.
