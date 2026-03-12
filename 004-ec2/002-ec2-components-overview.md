## EC2 Components Overview

Before launching or managing EC2 instances, it is important to understand the **core components that make up EC2**. EC2 is not just a single thing. Instead, it is a combination of multiple pieces that work together to create a fully functional virtual server.

When you launch an EC2 instance, you are actually configuring several components at the same time. These components define **what the server runs, how powerful it is, how it stores data, how it communicates, and how it stays secure**.

The major EC2 components include:

- Instance
- AMI
- Instance Type
- Storage
- Network
- Security

Each of these plays a specific role in creating and operating an EC2 server.

---

### 1. Instance

An **EC2 instance** is the actual **virtual server** that runs inside AWS.

When people say they are “launching an EC2”, they actually mean they are launching an **EC2 instance**.

An instance behaves like a normal computer. It has:

- CPU
- Memory (RAM)
- Storage
- Network connectivity
- An operating system

Once an instance is running, you can connect to it and use it like any other server.

For example, you might:

- Install a web server
- Run a backend API
- Host a database
- Run batch processing jobs

Every EC2 instance runs inside an AWS data center but appears to you as a **remote computer that you control**.

Instances also have a lifecycle. They can be:

- Launched
- Running
- Stopped
- Restarted
- Terminated

When an instance is terminated, the virtual server is permanently removed.

---

### 2. AMI

An **AMI (Amazon Machine Image)** is the **template used to create an EC2 instance**.

It contains everything required to start the server, including:

- The operating system
- System libraries
- Preinstalled software
- Configuration settings

When you launch an instance, AWS uses an AMI to create the machine.

For example, if you choose an Ubuntu AMI, the EC2 instance will start with the **Ubuntu operating system already installed**.

You can also create your own AMIs. This is useful when you want to reuse a preconfigured server setup.

For instance, you might create a custom AMI that already contains:

- Node.js
- Docker
- Your application dependencies

This allows you to launch new servers quickly with the same environment.

---

### 3. Instance Type

The **instance type** defines the **hardware configuration** of the EC2 instance.

It determines how powerful the server will be.

Instance types define:

- Number of CPUs
- Amount of RAM
- Network performance
- Storage capabilities

AWS provides many instance types optimized for different workloads.

For example:

- **General purpose instances** balance CPU and memory
- **Compute optimized instances** provide more CPU power
- **Memory optimized instances** provide large amounts of RAM
- **Storage optimized instances** provide fast disk performance

Choosing the correct instance type is important because it affects:

- Performance
- Cost
- Scalability

---

### 4. Storage

Storage is where the instance stores its **data, operating system files, and application files**.

EC2 instances typically use **Amazon Elastic Block Store (EBS)** as their primary storage.

EBS acts like a **virtual hard disk** attached to the instance.

It is used to store:

- Operating system files
- Application code
- Databases
- Logs

One important characteristic of EBS is that it **persists even if the instance stops**, unless explicitly deleted.

AWS also offers another type of storage called **instance store**, which is temporary storage attached directly to the host machine.

However, instance store data is **lost if the instance stops or terminates**.

Because of this, most applications use **EBS volumes for persistent data**.

---

### 5. Network

Networking allows an EC2 instance to **communicate with other systems and with the internet**.

Every EC2 instance runs inside a **Virtual Private Cloud (VPC)**.

A VPC is essentially a **private network within AWS**.

Networking components determine:

- Whether the instance is public or private
- How traffic enters or leaves the instance
- Which IP addresses it receives

An EC2 instance may have:

- A **private IP address** used within the VPC
- A **public IP address** used to communicate with the internet

Networking configuration also includes components such as:

- Subnets
- Internet Gateways
- Route Tables

These determine how network traffic flows to and from the instance.

---

### 6. Security

Security controls **who can access the EC2 instance and how traffic is allowed**.

AWS provides multiple security layers for EC2.

The most important one is the **Security Group**.

A security group acts like a **firewall** for the instance.

It defines rules such as:

- Which ports are open
- Which IP addresses are allowed
- Which protocols are permitted

For example, a web server might allow:

- Port 80 for HTTP
- Port 443 for HTTPS
- Port 22 for SSH

Security is also managed through **IAM (Identity and Access Management)**, which controls who can launch, modify, or terminate EC2 instances.

Proper security configuration is essential to prevent unauthorized access.

---

### 7. How These Components Work Together

Launching an EC2 instance involves combining all these components.

A typical instance launch includes:

1. Selecting an **AMI** to define the operating system
2. Choosing an **instance type** to determine hardware power
3. Attaching **storage** to hold the operating system and data
4. Configuring **network settings** to enable communication
5. Applying **security rules** to control access

Once all these components are configured, AWS creates the **EC2 instance**, which becomes your running virtual server.

---

### 8. Summary

EC2 is built from several important components that work together to create a complete cloud server environment.

The key components include:

- **Instance** — the virtual server itself
- **AMI** — the template used to create the server
- **Instance Type** — the hardware configuration
- **Storage** — where data and system files are stored
- **Network** — how the server communicates
- **Security** — how access to the server is controlled

Understanding these components is essential because every EC2 deployment is built by combining them in the right way.
