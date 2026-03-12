## EC2 with SSM

Managing EC2 instances traditionally required connecting to servers using **SSH (for Linux)** or **RDP (for Windows)**. However, this approach has several challenges such as managing SSH keys, opening inbound ports, and maintaining bastion hosts.

To simplify and secure instance management, AWS provides **AWS Systems Manager (SSM)**.

**AWS Systems Manager** is a service that allows administrators to **manage EC2 instances without directly connecting to them through SSH or RDP**.

With SSM, you can:

- securely access instances
- run commands remotely
- automate patch updates
- manage configuration

EC2 instances communicate with Systems Manager using the **SSM Agent**, which runs inside the instance.

This allows administrators to manage servers through the AWS console or CLI without exposing them to the internet.

---

### 1. Session Manager

**Session Manager** is a feature of AWS Systems Manager that allows you to **securely connect to EC2 instances without using SSH or opening inbound ports**.

Instead of connecting through SSH, you open a session directly from the AWS console or CLI.

The connection happens through the Systems Manager service.

Benefits of Session Manager include:

- no need to open port 22 (SSH) or 3389 (RDP)
- no need to manage SSH keys
- improved security
- centralized access control through IAM
- automatic session logging

Example architecture:

```
Administrator
      ↓
AWS Console / CLI
      ↓
Systems Manager
      ↓
SSM Agent on EC2 Instance
```

This method significantly improves security because the EC2 instance can remain inside a **private subnet with no public IP address**.

Session activity can also be logged for auditing purposes.

---

### 2. Patch Management

Keeping servers updated with security patches is an important operational task.

AWS Systems Manager provides **Patch Manager**, which automates the process of updating operating systems and software on EC2 instances.

Patch Manager allows administrators to:

- define patch baselines
- schedule patch updates
- automatically install security patches
- track patch compliance

For example, an organization might configure a patch policy such as:

- install security updates every Sunday at midnight
- apply patches to all production instances

This ensures that servers remain secure and up-to-date without requiring manual intervention.

Patch Manager supports multiple operating systems including:

- Amazon Linux
- Ubuntu
- Red Hat
- Windows Server

Automated patching helps reduce vulnerabilities caused by outdated software.

---

### 3. Command Execution

Another powerful feature of Systems Manager is **Run Command**.

Run Command allows administrators to **execute commands on multiple EC2 instances remotely without logging into each server individually**.

For example, you could run commands such as:

- installing software packages
- restarting services
- collecting logs
- updating configuration files

Example use case:

An administrator needs to install a monitoring agent on 50 EC2 instances.

Instead of connecting to each server manually, they can run a command through Systems Manager that installs the agent on all instances at once.

Example architecture:

```
Administrator
      ↓
Systems Manager Run Command
      ↓
SSM Agent
      ↓
Commands executed on EC2 instances
```

Run Command improves operational efficiency by allowing centralized management of large numbers of servers.

---

### 4. Summary

AWS Systems Manager provides powerful tools for managing EC2 instances securely and efficiently.

It removes the need for direct SSH or RDP access while providing centralized administration capabilities.

Key features include:

| Feature | Purpose |
|------|------|
| Session Manager | Securely connect to EC2 instances without SSH or RDP |
| Patch Manager | Automate operating system and security updates |
| Run Command | Execute commands remotely across multiple instances |

By integrating EC2 with Systems Manager, organizations can achieve **more secure, automated, and scalable server management** in AWS environments.
