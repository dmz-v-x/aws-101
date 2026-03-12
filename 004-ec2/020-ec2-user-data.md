## EC2 User Data

When launching EC2 instances, it is common to perform certain **setup tasks automatically** as soon as the server starts. Instead of logging into the instance and manually installing software or configuring the system, AWS provides a feature called **EC2 User Data**.

**User data** allows you to provide a script that runs automatically **when the EC2 instance launches for the first time**.

This script can install software, configure the system, download application code, or perform any setup required for the server.

In simple terms:

User data is **a startup script that automatically configures an EC2 instance when it boots**.

This capability is extremely useful when launching multiple instances or when building automated infrastructure.

---

### 1. What User Data Is

User data is a **script or set of commands that AWS runs during the instance's first boot process**.

When you launch an EC2 instance, the AWS console allows you to provide user data in a text field.

The commands placed in that field are executed automatically by the instance.

For Linux instances, user data is usually written as a **shell script**.

Example:

    #!/bin/bash
    sudo apt update
    sudo apt install -y nginx
    sudo systemctl start nginx
    sudo systemctl enable nginx

This script performs several actions automatically when the server starts:

- updates the package manager
- installs the Nginx web server
- starts the Nginx service
- enables Nginx to start on boot

Instead of manually installing the software after connecting to the instance, the server is already configured during startup.

---

### 2. Bootstrapping Servers

The process of automatically configuring a server during startup is called **bootstrapping**.

Bootstrapping means **preparing a system so it is ready to run an application immediately after it starts**.

When an EC2 instance boots for the first time, the user data script runs and performs the setup tasks needed to prepare the server.

Bootstrapping can include actions such as:

- installing required software
- downloading application code
- configuring system settings
- setting environment variables
- starting services

This approach is extremely useful for automated cloud environments.

For example, in systems that use **Auto Scaling**, new instances are created automatically when traffic increases. These instances must be ready to run the application immediately.

User data scripts ensure that every new instance is automatically configured during startup.

---

### 3. Real-World Examples

User data scripts are widely used in real-world cloud deployments.

#### Example: Automatically Installing a Web Server

A simple user data script can install and start a web server when the instance launches.

    #!/bin/bash
    sudo yum update -y
    sudo yum install -y httpd
    sudo systemctl start httpd
    sudo systemctl enable httpd

This ensures that every instance launched with this script immediately becomes a running web server.

---

#### Example: Deploying an Application

User data can also be used to deploy application code.

    #!/bin/bash
    sudo apt update
    sudo apt install -y git
    git clone https://github.com/example/app.git
    cd app
    npm install
    npm start

This script installs Git, downloads the application, installs dependencies, and starts the application.

---

#### Example: Configuring Monitoring Agents

Organizations often install monitoring tools automatically.

For example, a user data script could install a monitoring agent to send metrics and logs to a monitoring system.

This ensures that every server launched in the environment is automatically monitored.

---

### 4. Summary

EC2 user data allows administrators to run **automatic startup scripts during instance launch**.

This feature is commonly used to configure servers without manual intervention.

Key characteristics include:

| Feature | Description |
|------|------|
| Purpose | Automatically configure EC2 instances at launch |
| Execution Time | Runs during the first boot of the instance |
| Common Usage | Installing software, configuring systems, deploying applications |
| Automation Benefit | Enables automated infrastructure and scaling |

User data plays an important role in modern cloud infrastructure because it allows servers to be **automatically prepared and ready for workloads immediately after launch**.
