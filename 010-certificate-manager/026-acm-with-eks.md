## ACM + EKS / Ingress Controllers — AWS Load Balancer Controller (SSL Termination)

### 1. What Are We Doing Here?
We will:
- Use an ACM certificate with Kubernetes (EKS)
- Configure Ingress to expose services over HTTPS
- Use AWS Load Balancer Controller to create ALB with SSL

Goal:
Client → HTTPS → ALB (ACM cert) → Kubernetes Service → Pods

---

### 2. Key Components (Understand First)

1. EKS (Elastic Kubernetes Service) → Kubernetes cluster  
2. Ingress → Rules to expose services  
3. AWS Load Balancer Controller → Creates ALB automatically  
4. ACM Certificate → Provides HTTPS  

---

### 3. Where TLS Termination Happens

In this setup:

→ TLS is terminated at ALB (edge)

Client → HTTPS  
        ↓  
ALB (uses ACM cert)  
        ↓  
HTTP  
        ↓  
K8s Service → Pods  

---

### 4. Prerequisites

You need:

- EKS cluster running  
- AWS Load Balancer Controller installed  
- IAM roles configured (IRSA)  
- ACM certificate (Status = ISSUED)  
- Domain name + DNS access  

---

### 5. Step 1 — Install AWS Load Balancer Controller

Using Helm:

    helm repo add eks https://aws.github.io/eks-charts
    helm repo update

    helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
      -n kube-system \
      --set clusterName=<CLUSTER_NAME>

---

### 6. Step 2 — Request ACM Certificate

    aws acm request-certificate \
      --domain-name example.com \
      --subject-alternative-names www.example.com \
      --validation-method DNS \
      --region <EKS_REGION>

Validate via DNS → wait until ISSUED

---

### 7. Step 3 — Create Kubernetes Service

    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      selector:
        app: my-app
      ports:
        - port: 80
          targetPort: 3000
      type: ClusterIP

---

### 8. Step 4 — Create Ingress Resource (Important)

    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: my-ingress
      annotations:
        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
        alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
        alb.ingress.kubernetes.io/ssl-redirect: "443"
    spec:
      ingressClassName: alb
      rules:
        - host: example.com
          http:
            paths:
              - path: /
                pathType: Prefix
                backend:
                  service:
                    name: my-service
                    port:
                      number: 80

---

### 9. Step 5 — Apply Ingress

    kubectl apply -f ingress.yaml

Controller will:
- Create ALB  
- Attach ACM certificate  
- Configure listeners  

---

### 10. Step 6 — Update DNS

Point domain to ALB:

example.com → <ALB DNS name>

---

### 11. What AWS Load Balancer Controller Does

Automatically:

- Creates ALB  
- Configures listeners (HTTP + HTTPS)  
- Attaches ACM certificate  
- Routes traffic to services  

---

### 12. Important Annotations (Core Concepts)

- certificate-arn → Attach ACM cert  
- listen-ports → Enable HTTP/HTTPS  
- ssl-redirect → Force HTTPS  
- scheme → Internal or internet-facing  

---

### 13. Multiple Domains / Hosts

You can define multiple hosts:

    rules:
      - host: api.example.com
      - host: app.example.com

Certificate must include all domains (SANs)

---

### 14. TLS Options (Advanced)

You can:
- Use HTTPS only  
- Enable HTTP → HTTPS redirect  
- Customize SSL policies  

---

### 15. Common Mistakes

Mistake 1: Wrong Region  
- ACM cert must be in same region as EKS/ALB  

Mistake 2: Wrong Certificate ARN  
- Typo or incorrect ARN  

Mistake 3: DNS Not Pointing to ALB  
- Domain won’t resolve  

Mistake 4: Missing IAM Permissions  
- Controller fails to create ALB  

---

### 16. Verification

    kubectl get ingress

Open browser:
https://example.com

Verify:
- HTTPS works  
- Certificate is valid  

---

### 17. Real-World Flow

1. Deploy app in Kubernetes  
2. Create service  
3. Create ingress with ACM cert  
4. ALB created automatically  
5. Domain points to ALB  

Result:
Secure HTTPS application  

---

### 18. Why This Matters for ACM Mastery

Used in:
- Kubernetes production systems  
- Microservices architecture  
- Scalable cloud apps  

---

### 19. Mental Model

Ingress = Rules  
Controller = Executor  
ACM = Certificate provider  
ALB = Traffic gateway  

---

### 20. Key Takeaways

- ACM cert is attached via Ingress annotations  
- AWS Load Balancer Controller creates ALB automatically  
- TLS termination happens at ALB  
- DNS must point to ALB  
- Region alignment is critical  
