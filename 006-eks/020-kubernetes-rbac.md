### 1. Introduction

In Kubernetes, multiple users, teams, and applications interact with the cluster. Some users should have full control, while others should only be able to perform limited actions.

For example:

A developer should only view logs  
A DevOps engineer can deploy applications  
A CI/CD pipeline can update deployments  
An admin can do everything  

To enforce these rules, Kubernetes uses **RBAC (Role-Based Access Control)**.

RBAC defines:

Who can access the cluster  
What actions they can perform  
Which resources they can access  

In Amazon EKS, RBAC works together with IAM authentication to provide secure access control.

---

### 2. Authentication vs Authorization Recap

Before RBAC, recall the two key steps.

Authentication

Who are you  
Handled by AWS IAM in EKS

Authorization

What can you do  
Handled by Kubernetes RBAC

RBAC comes into play **after authentication is successful**.

---

### 3. Why RBAC is Needed

Without RBAC, every authenticated user would have full access to the cluster.

This is dangerous because:

Users could delete production workloads  
Accidental changes could break systems  
Security boundaries would not exist  

RBAC ensures **controlled and secure access**.

---

### 4. Core RBAC Concepts

Kubernetes RBAC has four main components.

Role  
ClusterRole  
RoleBinding  
ClusterRoleBinding  

Each plays a specific role in defining permissions.

---

### 5. What is a Role

A Role defines permissions **within a specific namespace**.

It specifies:

Resources  
Actions (verbs)

Example:

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: dev
      name: pod-reader
    rules:
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["get", "list"]

This role allows reading pods in the **dev namespace only**.

---

### 6. Understanding Verbs

Verbs define actions that can be performed.

Common verbs include:

get

Read a single resource

list

List multiple resources

create

Create resource

update

Modify resource

delete

Delete resource

watch

Observe changes

---

### 7. What is a ClusterRole

A ClusterRole defines permissions **at the cluster level**.

It is not limited to a namespace.

Example:

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: cluster-reader
    rules:
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["get", "list"]

This role allows reading pods across all namespaces.

---

### 8. When to Use Role vs ClusterRole

Use Role when:

Access is limited to a namespace

Example:

Developer accessing dev environment only

Use ClusterRole when:

Access spans multiple namespaces  
Access to cluster-wide resources is required

Example:

Node access  
Persistent volumes  
Cluster-wide monitoring

---

### 9. What is a RoleBinding

A RoleBinding assigns a Role to a user, group, or service account.

Example:

    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: read-pods
      namespace: dev
    subjects:
      - kind: User
        name: dev-user
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io

This binds the Role to a user.

---

### 10. What is a ClusterRoleBinding

A ClusterRoleBinding assigns a ClusterRole to users or groups.

Example:

    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: cluster-read-access
    subjects:
      - kind: User
        name: dev-user
    roleRef:
      kind: ClusterRole
      name: cluster-reader
      apiGroup: rbac.authorization.k8s.io

This gives cluster-wide access.

---

### 11. Subjects in RBAC

Subjects define who receives permissions.

Types include:

User

IAM-mapped user

Group

IAM-mapped group

ServiceAccount

Used by pods

In EKS, users and groups come from **IAM mapping (aws-auth ConfigMap)**.

---

### 12. End-to-End Access Flow

Step-by-step flow:

User runs kubectl  
↓

IAM authenticates user  
↓

User mapped to Kubernetes user/group  
↓

RBAC checks Role or ClusterRole  
↓

Access granted or denied  

---

### 13. Hands-On Example: Namespace Isolation

Step 1: Create namespace

    kubectl create namespace dev

Step 2: Create Role

    kubectl apply -f role.yaml

Step 3: Create RoleBinding

    kubectl apply -f rolebinding.yaml

Step 4: Test

    kubectl get pods -n dev

User can access only dev namespace.

---

### 14. Service Accounts and RBAC

Pods use ServiceAccounts.

Example:

    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: app-sa

Bind role to service account:

    kind: RoleBinding

Pods using this service account inherit permissions.

---

### 15. Least Privilege Principle

Least privilege means:

Grant only the permissions required.

Example:

Bad:

    verbs: ["*"]

This gives full access.

Good:

    verbs: ["get", "list"]

Restrict access to only required actions.

---

### 16. Example: Least Privilege Role

    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: limited-access
    rules:
      - apiGroups: [""]
        resources: ["pods"]
        verbs: ["get", "list"]

This prevents deletion or modification.

---

### 17. Namespaced vs Cluster-wide Security

Namespaced roles isolate environments.

Example:

dev namespace  
staging namespace  
prod namespace  

Each team gets access only to their namespace.

Cluster roles control global resources.

---

### 18. Common RBAC Mistakes

Giving system:masters access widely

This grants full admin privileges.

Using wildcard permissions

Too permissive.

Not separating environments

Leads to accidental changes.

Not using service accounts properly

Pods may get excessive permissions.

---

### 19. Debugging RBAC Issues

Check permissions:

    kubectl auth can-i get pods --as dev-user

Check roles:

    kubectl get roles -A

Check bindings:

    kubectl get rolebindings -A

These commands help troubleshoot access issues.

---

### 20. RBAC in EKS Context

In EKS:

IAM handles authentication  
RBAC handles authorization  

IAM identity is mapped to Kubernetes users/groups  
RBAC defines permissions  

Both must be configured correctly.

---

### 21. Production Best Practices

Use namespaces for isolation.

Use least privilege roles.

Avoid using system:masters.

Use service accounts for applications.

Audit RBAC regularly.

Use separate roles for different teams.

---

### 22. Real Production Example

Example:

Dev team

Access only dev namespace

Ops team

Full cluster access

CI/CD pipeline

Access only deployments

Each role is defined separately using RBAC.

---

### 23. Final Summary

Kubernetes RBAC is the authorization system that controls what actions users and applications can perform inside a cluster.

Roles define permissions within a namespace, while ClusterRoles define permissions across the entire cluster.

RoleBindings and ClusterRoleBindings assign these permissions to users, groups, or service accounts.

Following least privilege principles ensures that users and applications have only the access they need, improving security and preventing accidental changes in production environments.
