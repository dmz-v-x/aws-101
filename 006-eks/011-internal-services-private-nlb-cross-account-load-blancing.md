## Internal services, private NLBs, cross-account load balancing patterns.

### 1. Introduction

In Kubernetes environments, applications are composed of multiple services that communicate with each other. Some services must be accessible to the public internet, while others should only be accessible internally within the cluster or within a private network.

In Amazon EKS, exposing services internally or externally involves integration with AWS networking components such as:

Network Load Balancers (NLB)  
Application Load Balancers (ALB)  
VPC networking  
Private subnets  

---

### 2. Kubernetes Services Recap

In Kubernetes, a **Service** is an abstraction that provides a stable network endpoint for accessing a group of pods.

Pods are ephemeral and can be recreated at any time. Their IP addresses change frequently. Services provide a stable IP address and DNS name that route traffic to the correct pods.

Example service types include:

ClusterIP  
NodePort  
LoadBalancer  

Each service type provides a different method for accessing applications.

---

### 3. ClusterIP Services (Internal by Default)

The default service type in Kubernetes is **ClusterIP**.

Example service definition:

    apiVersion: v1
    kind: Service
    metadata:
      name: api-service
    spec:
      type: ClusterIP
      selector:
        app: api
      ports:
        - port: 80
          targetPort: 3000

ClusterIP services are only accessible **inside the Kubernetes cluster**.

Other pods communicate with this service using DNS.

Example request:

    http://api-service.default.svc.cluster.local

This is sufficient for **internal microservice communication**.

---

### 4. Why ClusterIP is Not Enough in EKS

ClusterIP works well for communication **within the Kubernetes cluster**, but production architectures often require communication outside the cluster.

Examples:

A backend API running in EKS accessed by EC2 instances

A private internal API accessed by services in another VPC

A shared platform cluster used by multiple AWS accounts

ClusterIP does not expose services outside Kubernetes. For these cases we need AWS networking integration.

---

### 5. Service Type LoadBalancer in EKS

In EKS, when a service is created with type LoadBalancer, Kubernetes integrates with AWS to provision a load balancer automatically.

Example:

    apiVersion: v1
    kind: Service
    metadata:
      name: api-service
    spec:
      type: LoadBalancer
      selector:
        app: api
      ports:
        - port: 80
          targetPort: 3000

When this service is created, AWS provisions a load balancer and attaches worker nodes as targets.

This load balancer can be:

Internet-facing  
Internal (private)

---

### 6. Internet-Facing vs Internal Load Balancers

An internet-facing load balancer has a public IP address.

Example endpoint:

    api.example.com

Traffic flow:

User → Internet → Load Balancer → Kubernetes Service → Pods

An internal load balancer only has **private IP addresses inside the VPC**.

Traffic flow:

Internal system → Private Load Balancer → Kubernetes → Pods

Internal load balancers are used for secure internal services.

---

### 7. Private Network Load Balancers

A **Network Load Balancer (NLB)** operates at Layer 4 (TCP/UDP).

NLBs are commonly used in EKS for:

High performance workloads  
Low latency applications  
Static IP requirements  
Internal services

Private NLBs are created by setting a service annotation.

Example:

    apiVersion: v1
    kind: Service
    metadata:
      name: internal-api
      annotations:
        service.beta.kubernetes.io/aws-load-balancer-internal: "true"
    spec:
      type: LoadBalancer
      selector:
        app: api
      ports:
        - port: 80
          targetPort: 3000

This annotation instructs AWS to create a **private NLB** instead of a public load balancer.

---

### 8. Verifying the Private Load Balancer

After creating the service, check the external endpoint.

    kubectl get svc internal-api

Example output:

    EXTERNAL-IP: internal-abc123.elb.amazonaws.com

This DNS name resolves to **private VPC IP addresses**.

Only resources inside the VPC can access it.

---

### 9. Why Internal Services Are Important

Many services should not be exposed to the public internet.

Examples include:

Internal APIs  
Database access layers  
Authentication services  
Event processing pipelines

Exposing these services publicly creates security risks.

Internal load balancers allow controlled access within private networks.

---

### 10. Accessing Internal Services from EC2

Example architecture:

EKS cluster → Private NLB → EC2 instance

Steps:

Deploy service with internal load balancer

Connect from EC2 instance

Example test:

    curl internal-api-123.us-east-1.elb.amazonaws.com

The request reaches pods running inside EKS.

---

### 11. Multi-VPC Architectures

In large organizations, different systems run in separate VPCs.

Example:

VPC A

EKS cluster

VPC B

Analytics platform

VPC C

Machine learning services

These systems still need to communicate.

This introduces the need for **cross-VPC connectivity**.

---

### 12. VPC Peering

One solution is **VPC peering**.

VPC peering allows two VPC networks to communicate privately.

Architecture:

VPC A → VPC Peering → VPC B

If an EKS cluster exposes a private NLB, services in the peer VPC can access it.

Example request from VPC B:

    curl internal-api.vpc-a.internal

---

### 13. AWS PrivateLink

Another common solution is **AWS PrivateLink**.

PrivateLink exposes services privately across AWS accounts and VPCs.

Architecture:

Service provider VPC  
↓

Network Load Balancer  
↓

PrivateLink endpoint service  
↓

Consumer VPC endpoint

PrivateLink allows secure access without exposing services publicly.

---

### 14. Cross-Account Load Balancing

In many enterprises, services run across multiple AWS accounts.

Example:

Account A

Shared EKS platform

Account B

Customer applications

Account C

Data processing pipelines

Cross-account load balancing allows services in one account to expose APIs to other accounts.

---

### 15. Cross-Account Pattern Using NLB

Steps:

Create private NLB in EKS account

Create endpoint service for the NLB

Allow other AWS accounts to connect

Consumers create VPC endpoints

Traffic flow:

Consumer VPC → PrivateLink Endpoint → NLB → Kubernetes Service

This enables secure service sharing.

---

### 16. Example Architecture

Production architecture may look like this:

EKS Cluster (Account A)

↓

Private NLB

↓

PrivateLink Endpoint Service

↓

Consumer VPC (Account B)

↓

Application calls internal service

This architecture allows centralized platform clusters.

---

### 17. Observing NLB Targets

To inspect NLB target groups:

Navigate to AWS console → EC2 → Target Groups.

Targets usually include worker node instances.

Health checks determine whether pods are reachable.

---

### 18. Common Gotchas

Incorrect subnet configuration

Private NLBs must be deployed in private subnets.

Security group restrictions

Worker node security groups must allow traffic from load balancer.

Health check failures

Improper health check configuration may cause load balancer to mark targets unhealthy.

---

### 19. Differences from Local Kubernetes

In local clusters like kind or minikube:

Networking runs on a single machine.

There is no concept of VPCs or cloud load balancers.

Services are typically accessed using:

    NodePort
    localhost port forwarding

Production clusters operate in distributed cloud infrastructure.

This requires integration with cloud networking components.

---

### 20. Production Best Practices

Use private load balancers for internal services.

Avoid exposing internal APIs to the public internet.

Design subnet CIDR ranges carefully.

Use PrivateLink for cross-account service sharing.

Monitor load balancer health checks.

Apply least privilege security group rules.
