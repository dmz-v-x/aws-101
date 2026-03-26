## What is TLS/SSL? — Purpose, Handshake Overview, Encryption vs Authentication

### 1. Introduction to TLS/SSL
TLS (Transport Layer Security) and SSL (Secure Sockets Layer) are cryptographic protocols used to secure communication over a network, most commonly the internet.

- SSL is the older protocol (now deprecated).
- TLS is the modern, secure version used today.

When you see **HTTPS** in a browser, it means HTTP is running over TLS.

In simple words:
TLS = Secure communication between client (browser) and server

---

### 2. Why Do We Need TLS?
Without TLS, data travels in plain text over the internet.

That means:
- Anyone in the middle (attacker, ISP, hacker) can read your data
- Passwords, credit card numbers, and personal data can be stolen

TLS solves this by:
1. Encrypting the data → so no one can read it
2. Authenticating the server → so you know you’re talking to the real server

---

### 3. Core Goals of TLS
TLS provides three main guarantees:

1. Confidentiality → Data is encrypted
2. Integrity → Data is not tampered with
3. Authentication → You are talking to the correct server

---

### 4. Encryption (Confidentiality)
Encryption means converting readable data (plaintext) into unreadable data (ciphertext).

Example:
- Plaintext: hello123
- Ciphertext: X7#@9!kL

Only someone with the correct key can decrypt it back.

In TLS:
- Data sent between client and server is encrypted
- Even if intercepted, it cannot be understood

---

### 5. Authentication (Trust)
Authentication ensures that:
- You are connecting to the real server (not an attacker)

How?
- The server presents a **digital certificate**
- This certificate is issued by a trusted Certificate Authority (CA)
- Your browser verifies it

If valid:
→ Connection proceeds

If invalid:
→ Browser shows warning (e.g., "Your connection is not private")

---

### 6. TLS Handshake (Step-by-Step Overview)
Before secure communication starts, a process called the **TLS Handshake** happens.

This establishes:
- Encryption keys
- Trust between client and server

Let’s break it down:

#### Step 1: Client Hello
- Client sends:
  - Supported TLS versions
  - Supported cipher suites
  - Random data

#### Step 2: Server Hello
- Server responds with:
  - Chosen TLS version
  - Selected cipher suite
  - Server certificate
  - Server random data

#### Step 3: Certificate Verification
- Client checks:
  - Is the certificate valid?
  - Is it signed by a trusted CA?
  - Does the domain match?

#### Step 4: Key Exchange
- Client and server agree on a shared secret
- This is used to generate encryption keys

#### Step 5: Session Keys Created
- Both sides now have:
  - Same symmetric key

#### Step 6: Secure Communication Begins
- All further communication is encrypted

---

### 7. Symmetric vs Asymmetric Encryption (Important Concept)

TLS uses both types:

#### Asymmetric Encryption
- Uses public + private key
- Used during handshake
- Slower but secure for key exchange

#### Symmetric Encryption
- Uses a single shared key
- Used after handshake
- Fast and efficient for data transfer

---

### 8. Real-World Example
When you open:
https://amazon.com

Here’s what happens:
1. Browser sends request
2. TLS handshake occurs
3. Amazon sends certificate
4. Browser verifies it
5. Secure connection established
6. Data (like login/password) is encrypted

---

### 9. TLS vs SSL (Difference)
- SSL → Old, insecure, deprecated
- TLS → Modern, secure, actively used

In practice:
- People still say “SSL”
- But actual protocol used is TLS (like TLS 1.2 / TLS 1.3)

---

### 10. Why This Matters for AWS Certificate Manager (ACM)
ACM works with TLS certificates.

Understanding TLS helps you:
- Request certificates correctly
- Attach them to services (ALB, CloudFront, API Gateway)
- Debug HTTPS issues
- Manage secure communication in production systems

---

### 11. Key Takeaways
- TLS secures communication over the internet
- It provides encryption, authentication, and integrity
- Handshake establishes trust and keys
- Certificates prove server identity
- TLS is the foundation of HTTPS and AWS ACM usage
