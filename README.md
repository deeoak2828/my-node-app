
# Node.js Application with Kubernetes and NGINX Ingress

## Overview
This project demonstrates the deployment of a Node.js application in Kubernetes using an NGINX Ingress Controller. It is designed to route traffic through a custom subdomain, `nodejs.my.node.app.deepak2828.com`, hosted on AWS and managed via GoDaddy for domain registration.

## Prerequisites
1. **Domain Name**: Ensure you own a registered domain (e.g., `deepak2828.com`) and have DNS management access.
2. **Kubernetes Cluster**: A working Kubernetes cluster set up (e.g., EKS, GKE, or Minikube).
3. **NGINX Ingress Controller**: Installed and running in your Kubernetes cluster.
4. **AWS Load Balancer**: Configured to handle external traffic to your cluster.
5. **AWS CLI**: Installed and configured with credentials.

## Steps to Set Up

### 1. DNS Configuration
- Log in to GoDaddy or your domain registrar.
- Create a CNAME record for the subdomain:
  - **Name**: `nodejs.my.node.app`
  - **Points To**: AWS Load Balancer DNS (e.g., `k8s-ingressn-ingressn-529eec9849-5c7aa9b6b0bbd22b.elb.us-east-2.amazonaws.com`)
- Save the record and allow time for propagation.

### 2. Kubernetes Ingress Configuration
Apply the following `Ingress` YAML file:
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
To deploy the Ingress, run:

bash
kubectl apply -f example-ingress.yaml
3. Verify Ingress and Backend Service
Check the Ingress resource:

bash
kubectl get ingress -n dev
Verify the backend service:

bash
kubectl get svc -n dev
4. Local Testing (Optional)
If testing locally without DNS registration, update your /etc/hosts file:

18.189.56.62 nodejs.my.node.app.deepak2828.com
Flush your DNS cache:

bash
ipconfig /flushdns
5. Test the Setup
Use curl to test connectivity:

bash
curl -H "Host: nodejs.my.node.app.deepak2828.com" http://k8s-ingressn-ingressn-529eec9849-5c7aa9b6b0bbd22b.elb.us-east-2.amazonaws.com
Or access the domain in a browser:

http://nodejs.my.node.app.deepak2828.com
6. Enable HTTPS (Optional)
Add an SSL certificate using AWS Certificate Manager and update the Ingress resource:

yaml
spec:
  tls:
  - hosts:
    - nodejs.my.node.app.deepak2828.com
    secretName: your-tls-secret
Expected Output
You should see the response:

Hello from Node.js!
Troubleshooting
Check NGINX Ingress Logs:

bash
kubectl logs -n ingress-nginx <ingress-controller-pod-name>
Verify DNS Propagation: Use tools like nslookup to ensure the domain resolves correctly:

bash
nslookup nodejs.my.node.app.deepak2828.com
Save the file and exit. In the terminal, you can save and close the file in nano by pressing CTRL + O, then Enter, and finally CTRL + X.


