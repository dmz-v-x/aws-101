## EC2 Instance Lifecycle

An EC2 instance does not simply exist in a single permanent state. Instead, it moves through several **states during its lifetime**, from creation to deletion. This sequence of states is called the **EC2 instance lifecycle**.

Understanding the EC2 lifecycle is important because different actions can only be performed in specific states. For example, you cannot connect to an instance before it starts running, and once an instance is terminated it cannot be recovered.

The main states in the EC2 lifecycle include:

- Launch
- Running
- Stop
- Start
- Reboot
- Terminate

Each of these states represents a stage in the life of an EC2 instance.

---

### 1. Launch

The lifecycle of an EC2 instance begins when you **launch the instance**.

Launching an instance means AWS creates a new virtual server based on the configuration you provide.

During the launch process, you typically choose:

- An **AMI** (operating system template)
- An **instance type** (hardware configuration)
- **Storage volumes**
- **Networking settings**
- **Security group rules**
- **Key pair for SSH access**

Once these configurations are selected, AWS allocates the necessary resources and begins creating the instance.

During this stage, the instance moves through an internal **pending state** while AWS prepares the server.

After the initialization is complete, the instance enters the **running state**.

---

### 2. Running

The **running state** means the EC2 instance is fully operational.

In this state:

- The operating system is running
- Applications can run on the server
- You can connect to the instance
- The instance can send and receive network traffic

For example:

- A Linux instance can be accessed through **SSH**
- A Windows instance can be accessed through **RDP**

While the instance is running, AWS charges you for the compute resources being used.

This is the state where your application actually runs.

---

### 3. Stop

Stopping an EC2 instance means **shutting down the virtual machine**, but **not deleting it**.

When an instance is stopped:

- The operating system shuts down
- The compute resources are released
- The instance is no longer running

However, several important things remain:

- The **EBS storage volumes remain attached**
- Your **data remains intact**
- The instance configuration remains saved

Because the instance is not running, you are **not charged for compute usage**, but you are still charged for any attached storage.

Stopping instances is useful when you temporarily do not need the server but want to preserve the environment.

---

### 4. Start

Starting an instance means **powering on a previously stopped instance**.

When you start the instance:

- AWS allocates compute resources again
- The operating system boots up
- The server returns to the **running state**

Your data and configuration remain the same because the storage volumes were preserved while the instance was stopped.

However, there is one important detail.

If the instance was using a **public IP address assigned automatically**, that IP may change when the instance is started again.

To maintain a constant public IP, an **Elastic IP** must be used.

---

### 5. Reboot

Rebooting an EC2 instance is similar to **restarting a normal computer**.

When you reboot an instance:

- The operating system restarts
- Applications restart
- The server temporarily becomes unavailable during the reboot process

Unlike stopping and starting an instance:

- The instance **remains on the same physical host**
- The **public IP address usually stays the same**
- The **storage remains unchanged**

Rebooting is typically used when:

- Applying system updates
- Restarting services
- Fixing temporary system issues

It is generally faster than stopping and starting the instance.

---

### 6. Terminate

Termination is the **final stage in the lifecycle of an EC2 instance**.

When an instance is terminated:

- The virtual server is permanently deleted
- The compute resources are released
- The instance cannot be recovered

In most cases:

- The **attached root EBS volume is also deleted**
- All data stored on that root volume is lost

Because termination is irreversible, AWS includes an optional feature called **termination protection** that prevents accidental deletion of important instances.

Termination is typically used when:

- The server is no longer needed
- A temporary environment has completed its purpose
- Infrastructure is being replaced

---

### 7. Lifecycle Summary

The EC2 instance lifecycle represents the journey of a virtual server from creation to deletion.

The main lifecycle states are:

| State | Meaning |
|------|------|
| Launch | Creating a new EC2 instance |
| Running | Instance is active and operational |
| Stop | Instance is powered off but preserved |
| Start | Restarting a previously stopped instance |
| Reboot | Restarting the operating system |
| Terminate | Permanently deleting the instance |

Understanding these lifecycle states helps you manage EC2 instances effectively, control costs, and maintain your cloud infrastructure properly.
