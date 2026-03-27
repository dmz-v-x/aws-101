### 1. Introduction

Kubernetes allows users to run containers with a wide range of privileges. While this flexibility is powerful, it can also be dangerous if not properly controlled.

Example risks:

Containers running as root  
Access to host filesystem  
Privileged containers accessing kernel  
Containers escaping isolation  

To prevent these risks, Kubernetes provides **Pod Security controls**.

Earlier, Kubernetes used:

PodSecurityPolicies (PSP) (now deprecated)

Modern Kubernetes uses:

Pod Security Standards (PSS)  
Pod Security Admission (PSA)

This guide explains these concepts from absolute basics to advanced production practices, including how to migrate from PSP to PSA.

---

### 2. Why Pod Security is Required

By default, Kubernetes allows very permissive pod configurations.

Example:

    privileged: true

This gives the container almost full control over the host machine.

Without restrictions, malicious or misconfigured workloads can:

Access sensitive data  
Modify system configurations  
Break cluster isolation  

Pod security ensures workloads run with **safe configurations**.

---

### 3. What Was PodSecurityPolicy (PSP)

PSP was an older Kubernetes feature used to enforce security rules.

Example rules:

Run as non-root  
Disallow privileged containers  
Restrict volume types  

However, PSP had several problems:

Complex to configure  
Hard to understand  
Difficult to maintain  
Poor usability  

Because of these issues, PSP was **removed in Kubernetes 1.25**.

---

### 4. Introduction to Pod Security Standards (PSS)

Pod Security Standards define **three predefined security levels**.

These are:

Privileged  
Baseline  
Restricted  

These standards define what a pod is allowed to do.

---

### 5. Privileged Policy

This is the least restrictive policy.

Allows:

Privileged containers  
Host access  
Root access  

Use case:

System-level workloads  
CNI plugins  
Monitoring agents  

Example:

VPC CNI DaemonSet in EKS runs under privileged mode.

---

### 6. Baseline Policy

This provides moderate security.

Prevents:

Privileged containers  
Some host access  

Allows:

Standard application workloads  

Use case:

Most general applications

---

### 7. Restricted Policy

This is the most secure policy.

Enforces:

Run as non-root  
No privilege escalation  
Limited volume types  
Strict security context  

Use case:

Production workloads  
Highly secure environments  

---

### 8. What is Pod Security Admission (PSA)

PSA is the mechanism that enforces Pod Security Standards.

It works using **namespace labels**.

Instead of writing complex policies, you label namespaces.

Example:

    kubectl label namespace dev \
    pod-security.kubernetes.io/enforce=baseline

This enforces baseline security for all pods in the namespace.

---

### 9. PSA Modes

PSA supports three modes.

enforce

Blocks non-compliant pods.

audit

Logs violations but allows pods.

warn

Shows warnings to users.

Example:

    pod-security.kubernetes.io/audit=restricted

This helps detect issues before enforcing policies.

---

### 10. Hands-On: Applying PSA

Step 1: Create namespace

    kubectl create namespace secure-app

Step 2: Apply PSA label

    kubectl label namespace secure-app \
    pod-security.kubernetes.io/enforce=restricted

Now all pods must comply with restricted policy.

---

### 11. Example Pod That Fails PSA

Example pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: insecure
spec:
  containers:
  - name: app
    image: nginx
    securityContext:
      privileged: true
```

This pod will be rejected under restricted policy.

---

### 12. Fixing Pod for Restricted Policy

Secure configuration:

```
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
```

This complies with restricted policy.

---

### 13. Security Context Basics

SecurityContext defines security settings for containers.

Key fields include:

runAsUser

Run container as specific user.

runAsNonRoot

Prevents root execution.

privileged

Grants elevated privileges.

allowPrivilegeEscalation

Controls privilege escalation.

These fields are enforced by PSA.

---

### 14. Why PSA is Simpler Than PSP

PSP required:

Custom policy definitions  
RBAC integration  
Complex configurations  

PSA simplifies this by:

Using predefined standards  
Using namespace labels  
Reducing configuration complexity  

---

### 15. PSP to PSA Migration Strategy

Step 1

Identify existing PSP policies.

Step 2

Map PSP rules to PSS levels.

Example:

Strict PSP → Restricted  
Moderate PSP → Baseline  

Step 3

Apply PSA in audit mode.

    pod-security.kubernetes.io/audit=restricted

Step 4

Fix non-compliant workloads.

Step 5

Switch to enforce mode.

---

### 16. Example Migration Flow

Old PSP:

Disallow privileged containers  
Require non-root users  

Equivalent PSA:

    pod-security.kubernetes.io/enforce=restricted

This replaces complex PSP configuration.

---

### 17. Exceptions and Special Workloads

Some workloads require elevated privileges.

Examples include:

CNI plugins  
CSI drivers  
Monitoring agents  

Solution:

Run them in separate namespace.

Example:

    kube-system namespace → privileged

Application namespaces → restricted

---

### 18. PSA in EKS Context

In EKS:

PSA works out of the box

No additional setup required

You simply label namespaces

EKS add-ons such as VPC CNI already run in privileged namespaces.

---

### 19. Common Gotchas

Applying restricted policy too early

Breaks existing workloads.

Not testing in audit mode

Leads to production failures.

Forgetting system workloads

System components may require privileged access.

---

### 20. Production Best Practices

Use restricted policy for production namespaces.

Use baseline for development environments.

Use privileged only for system workloads.

Apply audit mode before enforce mode.

Separate system and application namespaces.

---

### 21. Real Production Example

Namespace setup:

kube-system → privileged

dev → baseline

production → restricted

This ensures:

System components work correctly  
Applications remain secure  

---

### 22. Final Summary

Pod Security Standards define three levels of security controls for Kubernetes workloads: privileged, baseline, and restricted.

Pod Security Admission enforces these standards using namespace labels, replacing the older PodSecurityPolicy system.

PSA simplifies security management by providing predefined policies and easy configuration, making it suitable for modern Kubernetes environments.

Migrating from PSP to PSA involves mapping existing policies to appropriate security levels, testing in audit mode, and gradually enforcing restrictions to ensure secure and stable workloads.
