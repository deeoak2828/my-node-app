
# Node.js Application with Kubernetes and NGINX Ingress

## Introduction
This project demonstrates deploying a Node.js application within a Kubernetes cluster hosted on **AWS EKS**, using **NGINX Ingress Controller** to manage traffic routing. The application is accessible via the custom subdomain: `nodejs.my.node.app.deepak2828.com`. The EKS cluster is named `slvr-us-east-2-eks-blue` and resides in the `us-east-2` AWS region.

---

## Prerequisites
Before getting started, ensure you have:
1. **A Registered Domain Name**: Example: `deepak2828.com`, with DNS management access (e.g., GoDaddy).
2. **AWS EKS Cluster**: A functioning Kubernetes cluster created in AWS, named `slvr-us-east-2-eks-blue`.
3. **NGINX Ingress Controller**: Installed and configured in your Kubernetes cluster.
4. **AWS Load Balancer**: Automatically provisioned via EKS for handling external traffic to your cluster.
5. **AWS CLI**: Installed and properly set up with credentials.

---

## AWS EKS Cluster Details
- **Cluster Name**: slvr-us-east-2-eks-blue
- **Region**: `us-east-2`
- **Status**: Active
- **Kubernetes Version**: 1.32
- **Support Period**: Standard support until March 21, 2026
- **Provider**: AWS EKS
- **Created On**: April 7, 2025, at 17:49 (UTC+05:30)

---

## Dockerfile for the Node.js Application

Here’s the `Dockerfile` used to containerize the Node.js application:

```dockerfile
# Base image
FROM node:16

# Set working directory
WORKDIR /usr/src/app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy application files
COPY . .

# Expose the application port
EXPOSE 3000

# Start the application
CMD ["npm", "start"]
```

### Build the Docker Image
To build the Docker image, run:
```bash
docker build -t my-node-app .
```

### Run the Docker Container Locally
To run the container locally, use:
```bash
docker run -p 3000:3000 my-node-app
```

---

## Setup Guide

### 1. Configure DNS for the Subdomain
Log in to your domain registrar (e.g., GoDaddy) and update the DNS records:
- **Type**: Add a CNAME record or A record.
  - **Name**: `nodejs.my.node.app`
  - **Points To**: AWS Load Balancer DNS name (e.g., `k8s-ingressn-ingressn-529eec9849-5c7aa9b6b0bbd22b.elb.us-east-2.amazonaws.com`).
- Allow sufficient time for DNS propagation.

---

### 2. Set Up Kubernetes Ingress
Define an Ingress resource to route traffic to the Node.js application. Use the following YAML configuration:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  namespace: dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: nodejs.my.node.app.deepak2828.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-node-app-service
            port:
              number: 80
```

Apply the configuration:
```bash
kubectl apply -f example-ingress.yaml
```

---

### 3. Verify Ingress Resource and Services
- Check the status of the Ingress resource:
  ```bash
  kubectl get ingress -n dev
  ```
- Ensure the backend service (`my-node-app-service`) is operational:
  ```bash
  kubectl get svc -n dev
  kubectl describe svc my-node-app-service -n dev
  ```

---

### 4. Local Testing (Optional)
If you’re testing locally:
1. Update your `/etc/hosts` file:
   ```
   18.189.56.62 nodejs.my.node.app.deepak2828.com
   ```
   Replace `18.189.56.62` with the IP address of your AWS Load Balancer.
2. Flush the DNS cache:
   ```bash
   ipconfig /flushdns
   ```

---

### 5. Test Connectivity
- Verify the setup using curl:
  ```bash
  curl -H "Host: nodejs.my.node.app.deepak2828.com" http://k8s-ingressn-ingressn-529eec9849-5c7aa9b6b0bbd22b.elb.us-east-2.amazonaws.com
  ```
- Or access the application directly in your browser:
  ```
  http://nodejs.my.node.app.deepak2828.com
  ```

---

### 6. Optional: Enable HTTPS
Secure your application using HTTPS:
1. Set up an SSL certificate via AWS Certificate Manager.
2. Update the Ingress resource:
   ```yaml
   spec:
     tls:
     - hosts:
       - nodejs.my.node.app.deepak2828.com
       secretName: your-tls-secret
   ```

---

## Expected Outcome
Once everything is set up, accessing your subdomain should display the response:
```
Hello from Node.js!
```

---

## Troubleshooting
1. **Inspect Logs for Errors**:
   - Check NGINX ingress logs:
     ```bash
     kubectl logs -n ingress-nginx <ingress-controller-pod-name>
     ```
   - Check application logs:
     ```bash
     kubectl logs <pod-name> -n dev
     ```
2. **Verify DNS Propagation**:
   - Use `nslookup` or `dig` to ensure your domain resolves correctly:
     ```bash
     nslookup nodejs.my.node.app.deepak2828.com

