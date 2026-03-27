### 1. Introduction

When you work with Kubernetes locally (like kind or minikube), accessing the cluster is simple. You install kubectl, and you can immediately run commands like:

    kubectl get pods

There is usually **no external authentication system** involved.

However, in Amazon EKS, things are very different.

EKS integrates Kubernetes authentication with **AWS IAM (Identity and Access Management)**. This means:

- You do not directly log in using Kubernetes users
- Instead, AWS IAM identities are used for authentication
- Kubernetes RBAC is used for authorization

This introduces two key components:

aws-iam-authenticator (built into EKS flow)  
IAM to Kubernetes role mapping  

Understanding this system is critical for:

Cluster security  
Access control  
Production-grade Kubernetes operations  

---

### 2. Authentication vs Authorization (Very Important)

Before going deeper, we must clearly understand two concepts.

Authentication

"Who are you?"

Authorization

"What are you allowed to do?"

In EKS:

Authentication is handled by **AWS IAM**

Authorization is handled by **Kubernetes RBAC**

This separation is extremely important.

---

### 3. How Local Kubernetes Differs

In local clusters:

Authentication is often disabled or simplified

kubectl uses local certificates

Example kubeconfig:

    user:
      client-certificate-data
      client-key-data

You are automatically trusted.

In EKS:

Clusters are exposed via AWS APIs

Security is critical

Therefore authentication must go through AWS IAM

---

### 4. High-Level EKS Authentication Flow

When you run:

    kubectl get pods

This is what happens internally.

kubectl reads kubeconfig  
↓

kubeconfig uses AWS CLI to generate token  
↓

Token is signed using IAM credentials  
↓

Request sent to EKS API server  
↓

EKS verifies IAM identity  
↓

Kubernetes RBAC decides access  

This flow replaces traditional Kubernetes user authentication.

---

### 5. What is aws-iam-authenticator

aws-iam-authenticator is a component that allows Kubernetes to authenticate users using AWS IAM.

In EKS:

You do not install it manually

It is integrated into the EKS control plane

It validates IAM tokens and maps them to Kubernetes users.

---

### 6. kubeconfig in EKS

When you run:

    aws eks update-kubeconfig \
    --region ap-south-1 \
    --name production-cluster

Your kubeconfig gets updated.

Example entry:

    users:
    - name: arn:aws:iam::123456:user/dev-user
      user:
        exec:
          command: aws
          args:
            - eks
            - get-token
            - --cluster-name
            - production-cluster

This means:

kubectl does not store credentials

Instead it calls AWS CLI to generate a token dynamically.

---

### 7. Token Generation Process

When kubectl runs a command:

AWS CLI generates a temporary token.

Command internally used:

    aws eks get-token --cluster-name production-cluster

This token:

Is short-lived  
Is signed using IAM credentials  
Is validated by EKS  

This makes authentication secure and temporary.

---

### 8. Mapping IAM Users to Kubernetes

After authentication, Kubernetes must decide:

"What can this user do?"

This is done using a special ConfigMap:

    aws-auth

Located in:

    kube-system namespace

---

### 9. aws-auth ConfigMap

View the ConfigMap:

    kubectl get configmap aws-auth -n kube-system -o yaml

Example:

    mapUsers:
      - userarn: arn:aws:iam::123456:user/dev-user
        username: dev-user
        groups:
          - system:masters

This maps an IAM user to a Kubernetes user and group.

---

### 10. Understanding system:masters

system:masters is a Kubernetes admin group.

If a user is mapped to this group:

They have full cluster access.

This is equivalent to admin privileges.

---

### 11. Mapping IAM Roles

Instead of users, production systems usually use IAM roles.

Example:

    mapRoles:
      - rolearn: arn:aws:iam::123456:role/dev-role
        username: dev-role
        groups:
          - developers

This allows EC2 instances or services to access the cluster.

---

### 12. Kubernetes RBAC Basics

After IAM mapping, Kubernetes RBAC controls permissions.

RBAC components:

Role

Defines permissions.

RoleBinding

Assigns role to users/groups.

Example Role:

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: pod-reader
    rules:
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["get", "list"]

---

### 13. Binding IAM User to Role

Example RoleBinding:

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: read-pods
    subjects:
      - kind: User
        name: dev-user
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io

Now this user can only read pods.

---

### 14. End-to-End Flow Example

Let’s combine everything.

Step 1

User runs:

    kubectl get pods

Step 2

AWS CLI generates token using IAM credentials.

Step 3

EKS validates IAM identity.

Step 4

aws-auth maps IAM user to Kubernetes user/group.

Step 5

RBAC checks permissions.

Step 6

Access granted or denied.

---

### 15. Hands-On Example

Step 1: Add IAM user to aws-auth

    kubectl edit configmap aws-auth -n kube-system

Add:

    mapUsers:
      - userarn: arn:aws:iam::123456:user/dev-user
        username: dev-user
        groups:
          - developers

Step 2: Create Role

    kubectl apply -f role.yaml

Step 3: Create RoleBinding

    kubectl apply -f rolebinding.yaml

Step 4: Test access

    kubectl get pods

User will have limited permissions.

---

### 16. IAM Roles for Service Accounts (IRSA)

EKS also allows pods to assume IAM roles.

This is called IRSA.

Use case:

Pod needs access to S3

Instead of storing AWS credentials, assign IAM role.

Flow:

Pod → ServiceAccount → IAM Role → AWS API

This improves security.

---

### 17. Common Gotchas

User cannot access cluster

Cause: Not added to aws-auth

kubectl works but access denied

Cause: RBAC not configured

Expired token

Solution: Run aws eks update-kubeconfig again

Using IAM users in production

Best practice is to use IAM roles instead.

---

### 18. Production Best Practices

Use IAM roles instead of users.

Grant least privilege access.

Avoid giving system:masters access widely.

Use RBAC for fine-grained control.

Use IRSA for pod-level AWS access.

Audit access regularly.

---

### 19. Why This Is Critical in EKS

Unlike local Kubernetes:

EKS clusters are exposed via AWS APIs

Multiple teams access the cluster

Security is critical

IAM integration ensures:

Centralized identity management

Secure authentication

Integration with AWS ecosystem

---

### 20. Final Summary

Amazon EKS uses AWS IAM for authentication and Kubernetes RBAC for authorization.

The aws-iam-authenticator (built into EKS) validates IAM tokens and maps identities to Kubernetes users using the aws-auth ConfigMap.

IAM users and roles are mapped to Kubernetes groups, and RBAC policies determine what actions they can perform inside the cluster.

This model provides a secure and scalable way to manage access in production Kubernetes environments.
