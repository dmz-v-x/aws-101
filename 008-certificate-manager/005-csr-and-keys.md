## CSRs & Keys — How to Generate CSR and Private Key (OpenSSL Commands)

### 1. What is a CSR (Certificate Signing Request)?
A CSR is a file you send to a Certificate Authority (CA) to request a certificate.

It contains:
- Your public key
- Domain name (CN / SAN)
- Organization details
- Signature created using your private key

Important:
CSR does NOT contain the private key  
Private key ALWAYS stays with you

---

### 2. What is a Private Key?
A private key is a secret cryptographic key that:

- Stays on your server
- Is used to decrypt data
- Is used during TLS handshake

Key rule:
If private key is leaked → your security is compromised

---

### 3. Relationship Between CSR, Public Key, and Private Key

Flow:
1. You generate a private key
2. From that, a public key is derived
3. CSR is created using:
   - Public key
   - Identity info
   - Signed with private key

Then:
CA → verifies → issues certificate

---

### 4. OpenSSL — The Tool We Use
OpenSSL is a command-line tool used for:
- Generating keys
- Creating CSRs
- Inspecting certificates
- Debugging TLS

---

### 5. Step 1 — Generate Private Key
We first generate a private key.

#### Command (RSA 2048-bit)
```bash
openssl genrsa -out private.key 2048
```

Explanation:
- genrsa → generate RSA key
- -out private.key → file to store key
- 2048 → key size (secure default)

Output:
- private.key (your secret key)

---

### 6. Step 2 — Generate CSR from Private Key

#### Command:
```bash
openssl req -new -key private.key -out request.csr
```

You will be prompted to enter:

- Country Name (C)
- State (ST)
- Locality (L)
- Organization (O)
- Organizational Unit (OU)
- Common Name (CN) → IMPORTANT (your domain)
- Email

Example:
CN = example.com

Output:
- request.csr

---

### 7. One-Step Command (Key + CSR Together)

You can generate both in one command:

```bash
openssl req -new -newkey rsa:2048 -nodes \
  -keyout private.key \
  -out request.csr
```

Explanation:
- -newkey rsa:2048 → generate new key
- -nodes → no passphrase (important for servers)
- -keyout → private key file
- -out → CSR file

---

### 8. Adding SANs (Very Important)
Modern certificates use SAN (Subject Alternative Names), not just CN.

To include SANs, create a config file.

#### Step 1: Create config file (san.cnf)
```ini
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
req_extensions = req_ext

[dn]
C = IN
ST = Karnataka
L = Bangalore
O = MyCompany
CN = example.com

[req_ext]
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = api.example.com
```

#### Step 2: Generate CSR with SAN
```bash
openssl req -new -key private.key \
  -out request.csr \
  -config san.cnf
```

---

### 9. Verify CSR Contents

To inspect CSR:

```bash
openssl req -in request.csr -noout -text
```

Check for:
- Subject
- SAN entries
- Public key info

---

### 10. Extract Public Key (Optional)

```bash
openssl req -in request.csr -noout -pubkey
```

---

### 11. Security Best Practices

1. Never share private.key
2. Store it securely (permissions 600)
   ```bash
   chmod 600 private.key
   ```
3. Use strong key size (2048 or 4096)
4. Avoid passphrase for servers (or handle automation properly)
5. Rotate keys periodically

---

### 12. Real-World Flow

1. Generate private key + CSR
2. Send CSR to CA (or ACM / Let’s Encrypt)
3. CA verifies domain
4. CA issues certificate
5. Install certificate + private key on server

---

### 13. AWS ACM Context

Important difference:

When using ACM:
- You usually DO NOT generate CSR manually
- ACM generates key + CSR for you internally

But you need this knowledge when:
- Importing certificates into ACM
- Using external CAs
- Debugging TLS issues

---

### 14. Common Mistakes

- Wrong CN or missing SAN → certificate invalid
- Losing private key → cannot use certificate
- Including passphrase → breaks automation
- Not verifying CSR before sending

---

### 15. Key Takeaways

- CSR = request sent to CA
- Private key = secret, never shared
- Public key = derived from private key
- OpenSSL is used to generate and inspect
- SANs are mandatory in modern certificates
- Always verify before sending CSR
