## VPC Endpoints — Gateway Endpoints vs Interface Endpoints (AWS PrivateLink)

When instances inside a VPC need to access AWS services such as:

```
Amazon S3
DynamoDB
Secrets Manager
SSM
CloudWatch
```

they normally reach those services through the **public internet**.

Example architecture without endpoints:

```
EC2
  |
Private Subnet
  |
NAT Gateway
  |
Internet Gateway
  |
Internet
  |
AWS Service (S3, DynamoDB, etc.)
```

Even though traffic stays within AWS infrastructure in many cases, it still goes through **public endpoints**.

AWS provides a feature called **VPC Endpoints** that allows services inside a VPC to communicate with AWS services **privately without using the internet**.

---

### 1. What Is a VPC Endpoint?

A **VPC Endpoint** enables private connectivity between a VPC and supported AWS services.

Definition:

> A VPC endpoint allows resources inside a VPC to communicate with AWS services privately without traversing the public internet.

With endpoints:

```
Traffic stays inside the AWS network backbone.
```

Architecture example:

```
EC2 Instance
     |
VPC Endpoint
     |
AWS Service (S3)
```

No NAT gateway or internet gateway is required.

---

### 2. Why VPC Endpoints Exist

Without endpoints:

```
Private subnet → NAT → Internet → AWS service
```

Problems:

```
extra cost (NAT gateway)
internet routing
additional latency
security exposure
```

Endpoints solve these problems by enabling **private service connectivity**.

---

### 3. Types of VPC Endpoints

AWS provides two main endpoint types:

```
Gateway Endpoints
Interface Endpoints
```

Each type serves different services and networking models.

---

### 4. Gateway Endpoints

Gateway endpoints are used specifically for:

```
Amazon S3
Amazon DynamoDB
```

Definition:

> A gateway endpoint is a route-table based endpoint that enables private access to S3 or DynamoDB.

Gateway endpoints are implemented using **route table entries**.

Architecture:

```
EC2
   |
Subnet
   |
Route Table
   |
Gateway Endpoint
   |
S3
```

Traffic never leaves the AWS network.

---

### 5. Gateway Endpoint Routing

When a gateway endpoint is created, AWS automatically updates the route table.

Example route:

```
Destination → S3 prefix list
Target → Gateway Endpoint
```

Example route table entry:

```
pl-xxxxxx → vpce-id
```

This directs traffic destined for S3 through the endpoint.

---

### 6. Gateway Endpoint Example

Example architecture:

```
EC2 (Private Subnet)
     |
Route Table
     |
Gateway Endpoint
     |
Amazon S3
```

Benefits:

```
no NAT gateway needed
no internet exposure
lower cost
```

---

### 7. Gateway Endpoint Characteristics

Key characteristics:

```
free to use
route-table based
supports S3 and DynamoDB only
highly scalable
```

These endpoints are very common in production environments.

---

### 8. Interface Endpoints

Interface endpoints use **AWS PrivateLink**.

Definition:

> An interface endpoint is a private endpoint that uses an Elastic Network Interface (ENI) to connect to an AWS service.

Architecture:

```
EC2
  |
VPC
  |
Interface Endpoint (ENI)
  |
AWS Service
```

The endpoint creates **a private IP inside the subnet**.

---

### 9. AWS PrivateLink

PrivateLink allows private connectivity to:

```
AWS services
partner services
customer services
```

Example supported services:

```
Secrets Manager
SSM
CloudWatch
KMS
ECR
SNS
SQS
```

Many AWS services support PrivateLink.

---

### 10. Interface Endpoint Architecture

Example:

```
Private Subnet
     |
EC2 Instance
     |
Interface Endpoint ENI
     |
AWS Service
```

Traffic stays entirely inside AWS.

---

### 11. DNS with Interface Endpoints

Interface endpoints support **private DNS resolution**.

Example request:

```
secretsmanager.amazonaws.com
```

DNS resolves to:

```
Private IP of endpoint ENI
```

Traffic stays within the VPC.

---

### 12. Security Groups for Interface Endpoints

Because interface endpoints create ENIs, they support:

```
security groups
```

Example security rule:

```
Allow HTTPS from application subnet
```

This provides fine-grained control.

---

### 13. Cost Differences

| Endpoint Type | Cost |
|------|------|
| Gateway Endpoint | Free |
| Interface Endpoint | Hourly + data processing |

Gateway endpoints are preferred when available.

---

### 14. Gateway vs Interface Endpoint Comparison

| Feature | Gateway Endpoint | Interface Endpoint |
|------|------|------|
| Services | S3, DynamoDB | Many AWS services |
| Implementation | Route table | ENI |
| Security groups | No | Yes |
| Cost | Free | Paid |
| DNS | Standard | Private DNS |

---

### 15. Example Without VPC Endpoint

Example architecture:

```
EC2
  |
Private Subnet
  |
NAT Gateway
  |
Internet Gateway
  |
S3
```

Costs include:

```
NAT hourly charges
data processing charges
```

---

### 16. Example With VPC Endpoint

Architecture:

```
EC2
  |
Private Subnet
  |
Gateway Endpoint
  |
S3
```

Advantages:

```
lower cost
more secure
simpler architecture
```

---

### 17. Enterprise Architecture Example

Example architecture using multiple endpoints:

```
Private Subnet
     |
Application Servers
     |
------------------------------------
|           |           |          |
S3 Endpoint SSM Endpoint KMS Endpoint
```

Applications access AWS services privately.

---

### 18. Removing NAT Gateway Using Endpoints

Many architectures eliminate NAT gateways by using endpoints.

Example:

```
Private Subnet
   |
Endpoints
   |
AWS Services
```

This reduces infrastructure cost significantly.

---

### 19. Mental Model

Think of VPC endpoints as **private service doors inside your network**.

Without endpoint:

```
Office → Public Road → Service
```

With endpoint:

```
Office → Private hallway → Service
```

No public road required.

---

### 20. Key Concepts Summary

| Concept | Meaning |
|------|------|
| VPC Endpoint | Private connection to AWS service |
| Gateway Endpoint | Route-based endpoint for S3/DynamoDB |
| Interface Endpoint | ENI-based endpoint using PrivateLink |
| PrivateLink | Private service connectivity |
| Private DNS | DNS resolving to endpoint IP |

---

### 21. Checkpoint Questions

You should now be able to answer these.

#### What is a VPC endpoint?

A private connection between a VPC and AWS services.

---

#### Which services support gateway endpoints?

```
S3
DynamoDB
```

---

#### What is an interface endpoint?

An ENI-based endpoint connecting a VPC to AWS services via PrivateLink.

---

#### Do interface endpoints support security groups?

```
Yes
```

---

#### Why use VPC endpoints?

To keep traffic private and avoid internet routing.
