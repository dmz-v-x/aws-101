## Bastion Hosts

In secure cloud architectures, it is considered a bad practice to allow **direct SSH access to all EC2 instances from the internet**. Doing so increases the attack surface and exposes internal servers to potential security risks.

To solve this problem, organizations use a **Bastion Host**.

A **Bastion Host** is a special EC2 instance that acts as a **secure gateway for accessing other servers inside a private network**.

Instead of connecting directly to every EC2 instance, administrators first connect to the **bastion host**, and then from there connect to the internal servers.

Because the bastion host is the only server exposed to the internet for administrative access, it helps **protect the rest of the infrastructure**.

---

### 1. Why Bastion Exists

In many AWS architectures, servers are placed inside **private subnets** to prevent direct internet access.

A private subnet means:

- the instance does not have a public IP
- the instance cannot be accessed directly from the internet

This design improves security, but it creates a challenge:

Administrators still need a way to connect to those servers for tasks such as:

- system maintenance
- software installation
- debugging
- monitoring

This is where a bastion host is used.

A bastion host is placed in a **public subnet** and has a **public IP address**. Administrators connect to this instance first.

From the bastion host, they can then connect to instances in **private subnets** using their private IP addresses.

This creates a controlled entry point into the private network.

The architecture typically looks like this:

    Administrator → Bastion Host (Public Subnet) → Private EC2 Instances


By using a bastion host, organizations avoid exposing all internal servers to the internet.

---

### 2. Secure SSH Access

The bastion host enables **secure SSH access** to internal servers.

The process usually works like this:

1. An administrator connects to the bastion host using SSH.
2. The bastion host authenticates the user using a key pair.
3. Once inside the bastion host, the administrator uses SSH again to connect to private EC2 instances.

For example:

First connection to bastion host:

    ssh -i my-key.pem ec2-user@bastion-public-ip

Then connection from bastion to a private server:

    ssh ec2-user@private-instance-ip


This two-step access model provides an extra layer of security.

---

### 3. Security Advantages of Bastion Hosts

Using a bastion host provides several security benefits.

#### Reduced Attack Surface

Only one server (the bastion host) is exposed to the internet.

All other servers remain protected inside private subnets.

---

#### Centralized Access Control

All administrative access goes through a single server.

This makes it easier to:

- monitor login activity
- enforce security policies
- manage access permissions

---

#### Easier Logging and Monitoring

Because all access passes through the bastion host, it becomes a central point for:

- audit logs
- connection monitoring
- security tracking

---

#### Stronger Security Configuration

The bastion host can be hardened with:

- strict security group rules
- multi-factor authentication
- limited user accounts
- intrusion detection tools

---

### 4. Bastion Host vs Direct Access

The difference between direct access and bastion access can be summarized as follows:

| Approach | Description |
|------|------|
| Direct SSH Access | Every EC2 instance has a public IP and can be accessed from the internet |
| Bastion Host | Only one instance has public access, and all internal servers are accessed through it |

The bastion host approach is significantly more secure and is commonly used in production environments.

---

### 5. Summary

A **bastion host** is a dedicated EC2 instance used as a secure gateway to access servers inside private networks.

Instead of exposing all servers to the internet, administrators connect to the bastion host first and then access internal servers from there.

Key characteristics include:

| Feature | Description |
|------|------|
| Purpose | Secure gateway for accessing private EC2 instances |
| Location | Usually placed in a public subnet |
| Access Method | SSH connection from administrator to bastion |
| Security Benefit | Protects internal servers from direct internet exposure |

Bastion hosts are an important component of secure cloud architectures and help enforce controlled and monitored administrative access to infrastructure.
