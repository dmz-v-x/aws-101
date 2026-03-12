## EC2 Troubleshooting & Real-World Scenarios

Operating EC2 in real production environments means that issues will inevitably occur. Being able to quickly identify and fix these problems is a critical skill for cloud engineers, DevOps engineers, and system administrators.

Troubleshooting EC2 requires understanding several components such as:

- instance state
- networking configuration
- storage
- IAM permissions
- system performance
- security settings

The goal of troubleshooting is to **identify the root cause of a problem and restore normal operation as quickly as possible**.

Below are several common real-world EC2 issues and how they are typically diagnosed and resolved.

---

### 1. EC2 Won’t Start

Sometimes an EC2 instance may fail to start or may remain stuck in the **pending** or **stopping** state.

#### Possible Causes

- corrupted operating system
- insufficient instance capacity in the AZ
- EBS volume issues
- incorrect AMI configuration
- underlying hardware failure

#### Troubleshooting Steps

1. **Check Instance State**

   Verify whether the instance is:

   - pending
   - running
   - stopping
   - stopped

2. **Check System Status Checks**

   AWS performs two health checks:

   - **System status check** (AWS infrastructure)
   - **Instance status check** (OS level)

   If the system check fails, the issue is usually with AWS infrastructure.

3. **Review EC2 System Logs**

   System logs may show:

   - kernel panic
   - boot failure
   - configuration errors

4. **Detach and Inspect Root Volume**

   If the OS is corrupted:

   - stop the instance
   - detach the root EBS volume
   - attach it to another instance
   - repair configuration files

5. **Launch New Instance from Snapshot**

   If the root volume is damaged beyond repair, restore from an EBS snapshot.

---

### 2. EC2 Unreachable

An EC2 instance may be running but cannot be accessed through SSH or RDP.

#### Possible Causes

- incorrect security group rules
- missing internet gateway
- wrong route table configuration
- firewall configuration inside the OS
- incorrect key pair or password

#### Troubleshooting Steps

1. **Check Security Group Rules**

   Verify that required ports are open:

   - SSH → port 22
   - RDP → port 3389
   - HTTP → port 80
   - HTTPS → port 443

2. **Verify Public IP Address**

   Ensure the instance has:

   - a public IP
   - an Elastic IP if needed

3. **Check Route Table**

   Ensure the subnet has a route to the internet gateway.

4. **Verify Network ACLs**

   Ensure inbound and outbound rules allow required traffic.

5. **Check OS Firewall**

   Linux firewall tools such as:

   - `iptables`
   - `ufw`

   may block connections.

6. **Use EC2 Instance Connect or SSM**

   If SSH fails, try connecting using:

   - EC2 Instance Connect
   - Systems Manager Session Manager

---

### 3. Disk Full

A full disk is a common operational issue that can cause applications to fail or services to stop responding.

#### Symptoms

- application crashes
- log files stop writing
- database failures
- system performance degradation

#### Troubleshooting Steps

1. **Check Disk Usage**

```
df -h
```

2. **Identify Large Files**

```
du -sh /*
```

3. **Clean Temporary Files**

Common directories:

- `/tmp`
- `/var/tmp`

4. **Rotate Logs**

Log files often consume large amounts of disk space.

Example directories:

- `/var/log`

5. **Increase EBS Volume Size**

If the disk is genuinely too small:

- modify the EBS volume size
- extend the filesystem

6. **Move Data to Additional Volumes**

Large data directories can be moved to a separate EBS volume.

---

### 4. High CPU Usage

High CPU usage can cause application slowdown or system instability.

#### Possible Causes

- inefficient application code
- infinite loops
- excessive traffic
- background processes
- insufficient instance size

#### Troubleshooting Steps

1. **Check CPU Usage**

```
top
```

or

```
htop
```

2. **Identify CPU-Intensive Processes**

Look for processes consuming the highest CPU.

3. **Review Application Logs**

Errors or loops may appear in logs.

4. **Scale Infrastructure**

Possible solutions:

- upgrade instance type
- enable Auto Scaling
- distribute workload across multiple instances

5. **Optimize Application Code**

High CPU may be caused by inefficient algorithms or memory issues.

---

### 5. Security Misconfigurations

Incorrect security settings can expose EC2 instances to unauthorized access or block legitimate traffic.

#### Common Misconfigurations

- open security groups (`0.0.0.0/0`)
- unused ports exposed
- overly permissive IAM policies
- outdated software

#### Troubleshooting Steps

1. **Review Security Groups**

Ensure only necessary ports are open.

Example:

| Port | Purpose |
|-----|------|
| 22 | SSH access |
| 80 | HTTP |
| 443 | HTTPS |

2. **Check Network ACLs**

Ensure rules are correctly configured.

3. **Audit IAM Permissions**

Ensure policies follow the **least privilege principle**.

4. **Use AWS Security Tools**

Examples include:

- AWS Config
- GuardDuty
- Security Hub

5. **Regularly Patch Systems**

Use Systems Manager Patch Manager.

---

### 6. IAM Permission Errors

Applications running on EC2 often interact with AWS services such as S3, DynamoDB, or CloudWatch. If permissions are misconfigured, operations may fail.

#### Common Symptoms

- "AccessDenied" errors
- failed API calls
- inability to read or write AWS resources

Example error:

```
AccessDenied: User is not authorized to perform s3:PutObject
```

#### Troubleshooting Steps

1. **Check IAM Role Attached to EC2**

Verify the instance has the correct IAM role.

2. **Review IAM Policy**

Ensure required actions are allowed.

Example policy permission:

```
s3:GetObject
s3:PutObject
```

3. **Verify Resource ARN**

Ensure the policy references the correct resource.

4. **Check Policy Conditions**

Conditions may restrict access based on:

- IP address
- time
- tags

5. **Test Using AWS CLI**

Run commands manually from the instance to verify permissions.

---

### Summary

Troubleshooting EC2 requires systematic investigation across several layers of the cloud environment.

| Issue | Common Causes |
|------|------|
| EC2 Won’t Start | OS corruption, EBS issues |
| EC2 Unreachable | Security groups, networking configuration |
| Disk Full | Logs, temporary files, insufficient storage |
| High CPU Usage | heavy workload, inefficient processes |
| Security Misconfigurations | open ports, weak policies |
| IAM Permission Errors | incorrect role or policy configuration |
