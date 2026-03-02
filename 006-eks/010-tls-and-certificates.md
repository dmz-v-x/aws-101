## TLS & Certificates

### 1. Introduction

In any real-world distributed system, security is one of the most critical requirements. Applications communicate over networks, users access services through the internet, and sensitive data travels between components. Without proper encryption, this data could be intercepted or modified by attackers.

This is where **TLS (Transport Layer Security)** and **certificates** come into play.

In Kubernetes environments such as Amazon EKS, TLS certificates are used to secure communication between:

Users and applications  
Applications and other services  
Ingress controllers and external clients  
Pods communicating internally  

When you run Kubernetes locally using tools like **kind**, TLS is usually hidden or automatically configured. However, in production environments like EKS, certificate management becomes a critical operational responsibility.

---

### 2. What is TLS

TLS stands for **Transport Layer Security**.

TLS is a cryptographic protocol that encrypts communication between two systems.

Example without TLS:

User → Website

Data is transmitted as plain text. Anyone intercepting network traffic could read the data.

Example with TLS:

User → Encrypted channel → Website

All communication is encrypted.

This encryption prevents:

Data interception  
Data tampering  
Identity spoofing

TLS is the technology behind HTTPS.

---

### 3. What is a TLS Certificate

A TLS certificate is a digital file that proves the identity of a server.

The certificate contains:

Domain name  
Public key  
Certificate authority signature  
Expiration date  

Example certificate fields:

Common Name

```
api.example.com
```

Issuer

```
Amazon Trust Services
```

Validity

```
Jan 1 2026 → Jan 1 2027
```

Certificates allow clients to verify they are communicating with the correct server.

---

### 4. Public Key Cryptography Basics

TLS uses **asymmetric encryption**.

Two keys exist:

Public key  
Private key

Public key

Shared with everyone.

Private key

Stored securely on the server.

Encryption process:

Client encrypts data using public key  
↓

Server decrypts data using private key

This ensures only the server can read the encrypted message.

---

### 5. Certificate Authorities

A certificate authority (CA) is a trusted organization that issues TLS certificates.

Examples include:

Amazon Trust Services  
Let's Encrypt  
DigiCert  

The CA verifies the domain ownership before issuing a certificate.

Browsers trust certificates signed by recognized certificate authorities.

---

### 6. Why TLS is Important in Kubernetes

In Kubernetes environments, applications are often exposed through **Ingress controllers or load balancers**.

Example architecture:

User → Load Balancer → Kubernetes Ingress → Service → Pod

Without TLS:

User traffic is unencrypted.

With TLS:

Traffic is encrypted using HTTPS.

TLS ensures:

Secure communication with external users  
Protection against man-in-the-middle attacks  
Compliance with security standards

---

### 7. Why Local Kubernetes (kind) Feels Simpler

In local clusters like kind or minikube:

Applications are usually accessed via:

```
http://localhost
```

These environments are used for development and testing.

TLS configuration is often skipped because:

No public internet exposure  
Short-lived environments  
Lower security requirements

Production environments like EKS expose services to the internet, so TLS becomes mandatory.

---

### 8. TLS Termination in Kubernetes

TLS termination means decrypting encrypted traffic at a specific point in the architecture.

Common TLS termination locations:

Load balancer  
Ingress controller  
Application server

Example architecture:

User → HTTPS → Load Balancer → HTTP → Kubernetes pods

The load balancer decrypts traffic and forwards it internally.

---

### 9. AWS Certificate Manager (ACM)

AWS provides a service called **AWS Certificate Manager (ACM)**.

ACM manages TLS certificates for AWS services.

Features include:

Automatic certificate issuance  
Automatic certificate renewal  
Integration with AWS load balancers  
Managed security

ACM removes the need to manually generate certificates.

---

### 10. Creating a Certificate in ACM

Navigate to AWS Certificate Manager.

Request certificate.

Provide domain:

```
api.example.com
```

Choose validation method:

DNS validation (recommended)

AWS generates a DNS record for validation.

Once the DNS record is added, ACM issues the certificate.

---

### 11. Using ACM Certificates with EKS

ACM certificates are commonly used with AWS load balancers.

Example architecture:

User → HTTPS → AWS ALB → Kubernetes Ingress → Pods

Steps:

Create ACM certificate

Configure AWS Load Balancer Controller

Create Kubernetes ingress resource referencing certificate

Example ingress annotation:

```
alb.ingress.kubernetes.io/certificate-arn
```

The load balancer attaches the certificate automatically.

---

### 12. Installing AWS Load Balancer Controller

First install the controller.

Add Helm repository:

```
helm repo add eks https://aws.github.io/eks-charts
```

Install controller:

```
helm install aws-load-balancer-controller eks/aws-load-balancer-controller
```

This controller allows Kubernetes ingress resources to create AWS ALBs.

---

### 13. Example HTTPS Ingress with ACM

Example ingress manifest.

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account:certificate/id
spec:
  ingressClassName: alb
  rules:
    - host: api.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-service
                port:
                  number: 80
```

The ALB automatically handles TLS termination.

---

### 14. What is cert-manager

cert-manager is a Kubernetes controller that automates certificate management inside Kubernetes clusters.

It can:

Request certificates  
Renew certificates  
Store certificates as Kubernetes secrets  

cert-manager integrates with certificate authorities such as:

Let's Encrypt  
HashiCorp Vault  
Custom certificate authorities

---

### 15. Why cert-manager is Needed

ACM works well for AWS load balancers, but sometimes certificates must exist **inside the Kubernetes cluster**.

Examples include:

Mutual TLS between services  
Internal service communication  
Custom ingress controllers

cert-manager handles these scenarios.

---

### 16. Installing cert-manager on EKS

Add Jetstack Helm repository.

```
helm repo add jetstack https://charts.jetstack.io
```

Install cert-manager.

```
helm install cert-manager jetstack/cert-manager \
--namespace cert-manager \
--create-namespace \
--set installCRDs=true
```

Verify installation.

```
kubectl get pods -n cert-manager
```

---

### 17. Creating a ClusterIssuer

A ClusterIssuer defines how certificates are obtained.

Example using Let's Encrypt.

```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: admin@example.com
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: nginx
```

This configuration allows cert-manager to obtain certificates automatically.

---

### 18. Creating Certificates

Example certificate resource.

```
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: web-cert
spec:
  secretName: web-tls
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  commonName: api.example.com
  dnsNames:
    - api.example.com
```

cert-manager requests the certificate and stores it as a Kubernetes secret.

---

### 19. Certificate Lifecycle

TLS certificates have expiration dates.

Typical lifecycle:

Certificate requested  
↓

Certificate issued  
↓

Certificate stored as Kubernetes secret  
↓

Application uses certificate  
↓

Certificate expires after 90 days or 1 year  
↓

cert-manager automatically renews certificate

Automatic renewal prevents service disruptions.

---

### 20. Observing Certificate Status

Check certificates.

```
kubectl get certificates
```

Inspect certificate.

```
kubectl describe certificate web-cert
```

Check stored secret.

```
kubectl get secret web-tls
```

---

### 21. Common TLS Problems in Kubernetes

Certificate expiration

If renewal fails, applications become inaccessible.

Domain validation failures

DNS misconfiguration prevents certificate issuance.

Incorrect ingress configuration

Ingress not properly referencing TLS secret.

Controller misconfiguration

cert-manager not properly installed.

---

### 22. Production Best Practices

Use AWS ACM for external load balancer TLS.

Use cert-manager for internal certificate management.

Enable automatic renewal.

Monitor certificate expiration.

Store certificates securely.

