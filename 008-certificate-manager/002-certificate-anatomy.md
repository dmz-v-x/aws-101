## X.509 Certificate Anatomy — Subject, Issuer, Validity, SANs, Key Usage

### 1. What is an X.509 Certificate?
An X.509 certificate is a standard format used to represent a public key along with identity information.

In simple terms:
It is a digital identity card for a server (or entity) that proves:
- Who you are (identity)
- Your public key
- Who verified you (Certificate Authority)

This is the certificate used in TLS/HTTPS.

---

### 2. High-Level Structure of a Certificate
An X.509 certificate contains three main parts:

1. Certificate Data (who, what, validity, etc.)
2. Public Key (used for encryption)
3. Digital Signature (from the issuer/CA)

---

### 3. Subject (Who the Certificate Belongs To)
The **Subject** field identifies the owner of the certificate.

Common attributes inside Subject:
- CN (Common Name) → main domain (e.g., example.com)
- O (Organization) → company name
- OU (Organizational Unit) → department (optional)
- C (Country)
- ST (State)
- L (Locality/City)

Example:
CN=example.com, O=Example Inc, C=US

Important:
- Earlier, CN was used for domain validation
- Now, SAN (Subject Alternative Name) is used instead (more on that below)

---

### 4. Issuer (Who Issued the Certificate)
The **Issuer** is the Certificate Authority (CA) that signed the certificate.

Examples:
- DigiCert
- Let’s Encrypt
- Amazon (ACM private/public CA)

Why this matters:
- Your browser trusts certain CAs
- If the issuer is trusted → certificate is trusted

---

### 5. Validity Period (When the Certificate is Valid)
Every certificate has a time window:

- Not Before → start date
- Not After → expiry date

Example:
- Not Before: Jan 1, 2026
- Not After: Apr 1, 2026

Important:
- If expired → browser shows security warning
- Modern certificates usually last ~90 days (Let’s Encrypt) or up to ~1 year

---

### 6. Subject Alternative Names (SANs)
SANs define **all domains this certificate is valid for**.

Example:
- example.com
- www.example.com
- api.example.com

Why SANs matter:
- Modern browsers ignore CN and rely on SANs
- You can secure multiple domains with one certificate

Types:
- Multiple domains (multi-domain cert)
- Wildcard domains (*.example.com)

Example SAN:
DNS:example.com, DNS:www.example.com

---

### 7. Public Key (Core of Encryption)
The certificate contains a **public key**.

This key:
- Is shared openly
- Used during TLS handshake
- Helps establish secure communication

Important:
- Public key → in certificate
- Private key → kept secret on server

---

### 8. Key Usage (What This Certificate Can Be Used For)
Key Usage defines **what operations the certificate is allowed to perform**.

Common usages:
- Digital Signature
- Key Encipherment
- Data Encipherment

For TLS server certificates:
- Used for encrypting data
- Used for verifying server identity

---

### 9. Extended Key Usage (EKU)
More specific version of key usage.

Examples:
- TLS Web Server Authentication
- TLS Web Client Authentication
- Code Signing
- Email Protection

For HTTPS:
- Must include: Server Authentication

---

### 10. Certificate Signature (Trust Layer)
The certificate is signed by the Issuer (CA).

This signature:
- Ensures certificate is not tampered
- Links it to a trusted CA

Process:
- CA signs certificate using its private key
- Browser verifies using CA’s public key

---

### 11. Certificate Chain (Important Concept)
A certificate is usually part of a chain:

Root CA → Intermediate CA → Your Certificate

Why?
- Root CA is trusted by browsers
- Intermediate signs your certificate
- Builds a chain of trust

---

### 12. Real Example (Simplified View)
A typical certificate looks like:

- Subject: example.com
- Issuer: Let's Encrypt Authority X3
- Validity: Jan 1 → Apr 1
- SAN: example.com, www.example.com
- Public Key: RSA 2048-bit
- Key Usage: Digital Signature, Key Encipherment

---

### 13. Why This Matters for AWS Certificate Manager (ACM)
When using ACM:
- You request certificates for domains (Subject + SANs)
- ACM handles Issuer (Amazon CA or external)
- ACM manages validity (auto-renewal)
- You attach certificates to services (ALB, CloudFront)

Understanding X.509 helps you:
- Debug certificate errors
- Choose correct SANs
- Understand validation failures
- Work with imported certificates

---

### 14. Key Takeaways
- X.509 certificate = identity + public key + CA signature
- Subject = who the certificate is for
- Issuer = who signed it
- Validity = time range
- SAN = all valid domains
- Key Usage = what it can be used for
- Certificate chain = trust hierarchy
