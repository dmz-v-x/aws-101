## Key Pairs

When you launch an EC2 instance, you usually need a secure way to **connect to the server remotely**. AWS provides a mechanism called **Key Pairs** to enable secure authentication for accessing EC2 instances.

A **Key Pair** is a set of cryptographic keys used to securely log in to an EC2 instance.

A key pair consists of two parts:

- a **public key**
- a **private key**

AWS stores the **public key** on the EC2 instance, while you download and keep the **private key** on your local machine.

When you try to connect to the instance, these two keys work together to verify your identity.

Key pairs are primarily used when connecting to **Linux EC2 instances using SSH**.

---

### 1. What SSH Is

**SSH (Secure Shell)** is a network protocol used to securely connect to a remote computer over a network.

It allows you to control a server remotely through a command-line interface.

For example, when you connect to a Linux EC2 instance, you can run commands such as:

- installing software
- editing configuration files
- starting applications
- managing system resources

SSH provides several important security features:

- encrypted communication
- secure authentication
- protection against eavesdropping

Instead of using passwords, AWS recommends using **SSH key pairs** for authentication.

This method is considered more secure than traditional password-based login.

When connecting to an EC2 instance, the SSH command typically looks like:

    ssh -i my-key.pem ec2-user@public-ip-address


Here:

- `my-key.pem` is the private key
- `ec2-user` is the default username
- `public-ip-address` is the IP of the EC2 instance

---

### 2. Public vs Private Key

A key pair contains two keys that work together for authentication.

#### Public Key

The **public key** is stored on the EC2 instance.

It is placed inside a special file called `authorized_keys` on the server.

This key is used by the server to verify whether the connecting client has the correct private key.

The public key can be shared safely because it cannot be used alone to access the server.

---

#### Private Key

The **private key** is downloaded when the key pair is created.

This file usually has a `.pem` extension.

The private key must be stored securely on your local machine.

It is used by your SSH client to prove your identity when connecting to the EC2 instance.

If someone gains access to your private key, they may be able to access your server.

Because of this, private keys must:

- never be shared publicly
- be stored securely
- have restricted file permissions

---

### 3. Losing Keys Scenario

One important rule when working with EC2 key pairs is:

If you **lose the private key**, you cannot directly connect to the instance using SSH.

AWS does **not store your private key**, so it cannot be recovered once lost.

If the private key is lost, several recovery options may exist depending on the situation.

#### If You Still Have Instance Access

If you are still logged in to the instance, you can:

- create a new key pair
- manually add the new public key to the `authorized_keys` file

This restores SSH access.

---

#### If the Root Volume Is EBS-Based

You can recover access by:

1. Stopping the instance.
2. Detaching the root EBS volume.
3. Attaching the volume to another EC2 instance.
4. Editing the `authorized_keys` file.
5. Reattaching the volume to the original instance.

This method allows you to add a new public key.

---

#### If the Instance Store Is Used

If the instance uses **instance store storage** and you lose the private key, recovery may not be possible.

The instance may need to be recreated.

---

### 4. Summary

Key pairs provide a secure authentication method for accessing EC2 instances.

They use cryptographic keys instead of passwords to verify identity during SSH connections.

| Component | Purpose |
|------|------|
| SSH | Secure protocol used to connect to remote servers |
| Public Key | Stored on the EC2 instance for authentication |
| Private Key | Stored locally and used to prove identity |
| Key Pair | Combination of public and private keys |

Because the private key is essential for access, it must be stored securely and backed up properly.
