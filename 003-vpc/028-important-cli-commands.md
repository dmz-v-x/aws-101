## AWS Console → CLI — Core VPC Tasks Using the AWS CLI

When working with AWS networking, many engineers start with the **AWS Console (UI)**. However, in real infrastructure environments (DevOps, automation, CI/CD, IaC), engineers often interact with AWS using the **AWS CLI**.

Learning to translate **Console actions → CLI commands** is an important skill.

This guide walks through core VPC tasks using the AWS CLI:

```
create VPC
create subnets
create internet gateway
create route tables
associate route tables
```

These commands represent the **most important networking operations** you will perform when building VPC infrastructure.

---

### 1. Verify AWS CLI Configuration

Before running VPC commands, confirm the CLI is configured.

Run:

```
aws configure
```

You will be prompted for:

```
AWS Access Key ID
AWS Secret Access Key
Default region
Default output format
```

Example:

```
AWS Access Key ID: AKIA...
AWS Secret Access Key: ********
Default region: ap-south-1
Default output format: json
```

Verify identity:

```
aws sts get-caller-identity
```

Example output:

```
{
  "UserId": "AIDA...",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/admin"
}
```

---

### 2. Create a VPC

Console equivalent:

```
VPC → Create VPC
```

CLI command:

```
aws ec2 create-vpc \
  --cidr-block 10.0.0.0/16
```

Example output:

```
{
  "Vpc": {
    "VpcId": "vpc-0abc123",
    "CidrBlock": "10.0.0.0/16"
  }
}
```

Save the **VpcId**.

Example:

```
vpc-0abc123
```

---

### 3. Enable DNS Hostnames for the VPC

Important for services like:

```
EC2
ALB
Private DNS
```

Command:

```
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-0abc123 \
  --enable-dns-hostnames
```

Enable DNS resolution:

```
aws ec2 modify-vpc-attribute \
  --vpc-id vpc-0abc123 \
  --enable-dns-support
```

---

### 4. Create Subnets

Console equivalent:

```
VPC → Subnets → Create subnet
```

Example creating a subnet in AZ-a:

```
aws ec2 create-subnet \
  --vpc-id vpc-0abc123 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone ap-south-1a
```

Example output:

```
{
  "Subnet": {
    "SubnetId": "subnet-0abc123"
  }
}
```

Create another subnet:

```
aws ec2 create-subnet \
  --vpc-id vpc-0abc123 \
  --cidr-block 10.0.2.0/24 \
  --availability-zone ap-south-1b
```

This creates a **multi-AZ network**.

---

### 5. Create an Internet Gateway

Console equivalent:

```
VPC → Internet Gateways → Create
```

CLI command:

```
aws ec2 create-internet-gateway
```

Example output:

```
{
  "InternetGateway": {
    "InternetGatewayId": "igw-0abc123"
  }
}
```

Save:

```
igw-0abc123
```

---

### 6. Attach the Internet Gateway to the VPC

Internet gateways must be attached to a VPC.

Command:

```
aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-0abc123 \
  --vpc-id vpc-0abc123
```

Architecture now becomes:

```
VPC
 |
Internet Gateway
 |
Internet
```

---

### 7. Create a Route Table

Console equivalent:

```
VPC → Route Tables → Create
```

CLI command:

```
aws ec2 create-route-table \
  --vpc-id vpc-0abc123
```

Example output:

```
{
  "RouteTable": {
    "RouteTableId": "rtb-0abc123"
  }
}
```

Save:

```
rtb-0abc123
```

---

### 8. Add Internet Route

Add a default route to the internet gateway.

Command:

```
aws ec2 create-route \
  --route-table-id rtb-0abc123 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-0abc123
```

Route meaning:

```
All internet traffic → IGW
```

---

### 9. Associate Route Table With Subnet

Console equivalent:

```
Route Table → Subnet Associations
```

CLI command:

```
aws ec2 associate-route-table \
  --route-table-id rtb-0abc123 \
  --subnet-id subnet-0abc123
```

Now instances in this subnet can reach the internet.

---

### 10. Enable Auto Public IP for Subnet (Optional)

For public subnets:

```
aws ec2 modify-subnet-attribute \
  --subnet-id subnet-0abc123 \
  --map-public-ip-on-launch
```

This ensures EC2 instances receive public IPs automatically.

---

### 11. Useful CLI Commands for VPC Inspection

List VPCs:

```
aws ec2 describe-vpcs
```

List subnets:

```
aws ec2 describe-subnets
```

List route tables:

```
aws ec2 describe-route-tables
```

List internet gateways:

```
aws ec2 describe-internet-gateways
```

These commands help inspect existing networking resources.

---

### 12. Example Complete Network Setup

Example architecture created by the above commands:

```
VPC (10.0.0.0/16)
 |
--------------------------------
|                              |
Subnet AZ-a                Subnet AZ-b
10.0.1.0/24                10.0.2.0/24
 |
Route Table
 |
0.0.0.0/0 → Internet Gateway
 |
Internet
```

This forms a basic **public VPC network**.

---

### 13. Important CLI Commands Summary

| Task | Command |
|------|------|
| Create VPC | `create-vpc` |
| Create Subnet | `create-subnet` |
| Create IGW | `create-internet-gateway` |
| Attach IGW | `attach-internet-gateway` |
| Create Route Table | `create-route-table` |
| Add Route | `create-route` |
| Associate Route Table | `associate-route-table` |

These commands represent the **core networking operations** in AWS CLI.

---

### 14. Mental Model

Console actions map directly to CLI commands.

Example:

```
Console: Create VPC
CLI: aws ec2 create-vpc

Console: Create Subnet
CLI: aws ec2 create-subnet

Console: Add Route
CLI: aws ec2 create-route
```

The CLI simply exposes the same operations as the console.
