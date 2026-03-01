## EKS Add Ons and Managed Components

### 1. Introduction

When you run Kubernetes locally using tools like kind or minikube, many internal components are installed automatically and run in the background. These components are part of Kubernetes itself and are often not noticed because the local environment installs and manages them for you.

However, in Amazon EKS, these components become more visible because AWS exposes them as **EKS add-ons or managed components**.

Some of the most important components include:

coreDNS  
kube-proxy  
aws-node (VPC CNI)  

These components are responsible for critical cluster functions such as:

Service discovery  
Networking  
Pod communication  
Cluster DNS resolution  

In Amazon EKS, these components can be managed in two different ways:

Self-managed add-ons  
EKS managed add-ons

Understanding how these components work is essential for operating production EKS clusters.

---

### 2. What Are Kubernetes Add-ons

Kubernetes add-ons are components that extend the core functionality of a Kubernetes cluster.

They run inside the cluster as normal Kubernetes resources such as:

Pods  
DaemonSets  
Deployments  

These components perform essential operational tasks.

Examples include:

DNS resolution  
Network routing  
Cluster metrics  
Logging  
Storage integration

Without these add-ons, the Kubernetes cluster would not function correctly.

---

### 3. Why These Components Are Less Visible in Local Kubernetes

When using local cluster tools like kind, minikube, or Docker Desktop Kubernetes, these components are automatically installed during cluster initialization.

Because everything runs on a single machine, the complexity is hidden.

For example:

kind installs coreDNS automatically  
kube-proxy runs automatically  
Networking plugins are preconfigured

In Amazon EKS, however, networking and cluster services must integrate with AWS infrastructure, so these components become more visible and configurable.

---

### 4. Where Add-ons Run in an EKS Cluster

Most add-ons run in a special namespace called:

kube-system

This namespace contains cluster-level services required for Kubernetes to function.

To see them, run:

    kubectl get pods -n kube-system

Typical output may include:

    coredns
    kube-proxy
    aws-node

Each of these performs a specific infrastructure task inside the cluster.

---

### 5. Understanding coreDNS

coreDNS is the DNS server used inside Kubernetes clusters.

It provides service discovery for Kubernetes services and pods.

In simple terms, coreDNS allows containers to communicate using service names instead of IP addresses.

Without DNS, services would need to know the exact IP addresses of other services, which constantly change in Kubernetes.

---

### 6. Why DNS is Required in Kubernetes

Consider two microservices.

Service A  
Service B

If Service A wants to call Service B, it needs a way to locate it.

Instead of using IP addresses, Kubernetes uses service names.

Example:

    http://user-service
    http://payment-service

coreDNS resolves these names to the correct pod IP addresses.

---

### 7. How coreDNS Works Internally

When a pod performs a DNS lookup:

Pod sends DNS query  
↓  
coreDNS receives query  
↓  
coreDNS checks Kubernetes service registry  
↓  
coreDNS returns IP address of service

This process allows service discovery inside the cluster.

---

### 8. Inspecting coreDNS in EKS

Check coreDNS pods:

    kubectl get pods -n kube-system -l k8s-app=kube-dns

Check the deployment:

    kubectl get deployment coredns -n kube-system

Inspect configuration:

    kubectl get configmap coredns -n kube-system -o yaml

This configuration controls how DNS resolution works inside the cluster.

---

### 9. Scaling coreDNS

If the cluster grows large, DNS traffic increases.

You may need to scale coreDNS.

Example:

    kubectl scale deployment coredns \
    --replicas=4 \
    -n kube-system

In large clusters, insufficient DNS capacity can cause application failures.

---

### 10. Understanding kube-proxy

kube-proxy is responsible for networking inside Kubernetes services.

It runs on every worker node as a **DaemonSet**.

DaemonSet means one pod runs on every node.

Verify this:

    kubectl get daemonset kube-proxy -n kube-system

kube-proxy manages networking rules that allow services to route traffic to pods.

---

### 11. How Kubernetes Services Work

Suppose you create a service:

    kubectl expose deployment nginx \
    --port=80 \
    --type=ClusterIP

This service receives a cluster IP such as:

    10.100.50.10

kube-proxy creates networking rules that route traffic from that IP to the actual pods.

---

### 12. kube-proxy Networking Mechanism

kube-proxy typically uses **iptables** or **IPVS**.

The workflow looks like this:

Request arrives at service IP  
↓  
iptables rule intercepts request  
↓  
kube-proxy forwards request  
↓  
Pod receives traffic

This mechanism allows load balancing between pods.

---

### 13. Inspecting kube-proxy

Check pods:

    kubectl get pods -n kube-system -l k8s-app=kube-proxy

Check logs:

    kubectl logs <kube-proxy-pod> -n kube-system

Troubleshooting kube-proxy is important when service networking fails.

---

### 14. Understanding aws-node (Amazon VPC CNI)

aws-node is the most important networking component in EKS.

It is part of the **Amazon VPC CNI plugin**.

CNI stands for:

Container Network Interface

This plugin allows Kubernetes pods to receive IP addresses from the AWS VPC.

---

### 15. Why EKS Needs a Special Networking Plugin

In most Kubernetes environments, pods receive IP addresses from an internal overlay network.

Example:

Flannel  
Calico  
Weave

These networks create virtual overlays on top of existing infrastructure.

However AWS chose a different approach.

In EKS:

Pods receive **real VPC IP addresses**.

This allows pods to communicate directly with other AWS services.

---

### 16. How VPC CNI Works

When a node starts:

The aws-node daemon runs on the node.

This daemon allocates IP addresses from the VPC subnet.

When a pod is created:

Pod requests IP address  
↓  
aws-node assigns VPC IP  
↓  
Pod receives network interface

This means each pod becomes a first-class citizen in the VPC network.

---

### 17. Inspecting aws-node

Check daemonset:

    kubectl get daemonset aws-node -n kube-system

Check pods:

    kubectl get pods -n kube-system -l k8s-app=aws-node

Logs can help diagnose networking issues:

    kubectl logs <aws-node-pod> -n kube-system

---

### 18. Important VPC CNI Limitations

Each EC2 instance can support only a limited number of network interfaces and IP addresses.

Example:

t3.medium supports limited IP capacity.

This means a node can run only a limited number of pods.

This is one of the most common scaling issues in EKS.

---

### 19. EKS Managed Add-ons

Amazon EKS allows AWS to manage cluster add-ons automatically.

These are called **EKS managed add-ons**.

Instead of manually installing components, AWS manages their lifecycle.

Supported managed add-ons include:

coreDNS  
kube-proxy  
VPC CNI  
EBS CSI driver  
EFS CSI driver  
AWS Load Balancer Controller

---

### 20. Advantages of Managed Add-ons

Automatic version compatibility

AWS ensures add-ons match Kubernetes version.

Simplified upgrades

Add-ons can be upgraded using EKS APIs.

Reduced operational complexity

AWS manages patching and updates.

---

### 21. Viewing Add-ons in a Cluster

List installed add-ons:

    aws eks list-addons \
    --cluster-name production-cluster

Example output:

    coredns
    kube-proxy
    vpc-cni

Describe add-on:

    aws eks describe-addon \
    --cluster-name production-cluster \
    --addon-name coredns

---

### 22. Installing an EKS Managed Add-on

Example:

    aws eks create-addon \
    --cluster-name production-cluster \
    --addon-name vpc-cni

This installs the AWS managed version of the VPC CNI plugin.

---

### 23. Upgrading Add-ons

List versions:

    aws eks describe-addon-versions \
    --addon-name coredns

Upgrade add-on:

    aws eks update-addon \
    --cluster-name production-cluster \
    --addon-name coredns \
    --addon-version v1.11.1

This ensures compatibility with the cluster version.

---

### 24. Common Production Gotchas

DNS failures

If coreDNS pods crash, services cannot resolve each other.

Service networking failures

If kube-proxy is misconfigured, services cannot route traffic.

Pod IP exhaustion

If VPC subnet IPs are exhausted, pods cannot start.

Addon version mismatches

Running incompatible addon versions can break cluster networking.
