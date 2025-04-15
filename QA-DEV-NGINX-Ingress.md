
# **QA and DEV Implementation of NGINX Ingress in Kubernetes**

## **Introduction**
NGINX Ingress Controller is widely used in Kubernetes to manage HTTP(S) traffic to applications inside a cluster. In a multi-environment setup, such as **QA (Quality Assurance)** and **DEV (Development)**, separate Ingress configurations ensure **isolated traffic management, controlled access, and distinct testing workflows**.

---

## **1. Setting Up NGINX Ingress Controller**
Before configuring Ingress rules for QA and DEV, install the **NGINX Ingress Controller** in the cluster:

### **Install via Helm**
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace
```

Verify installation:
```bash
kubectl get pods -n ingress-nginx
kubectl get svc -n ingress-nginx
```

---

## **2. Creating Separate Namespaces**
Each environment should have its own namespace for isolation:

```bash
kubectl create namespace dev
kubectl create namespace qa
```

Verify:
```bash
kubectl get namespaces
```

---

## **3. Defining Services for DEV and QA**
Each environment should have a service exposing the application.

### **DEV Service Configuration**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-node-app-service
  namespace: dev
spec:
  selector:
    app: my-node-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
```

### **QA Service Configuration**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-node-app-service
  namespace: qa
spec:
  selector:
    app: my-node-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP
```

Apply:
```bash
kubectl apply -f dev-service.yaml
kubectl apply -f qa-service.yaml
```

---

## **4. Configuring Ingress for DEV**
Define the Ingress resource to route requests for the DEV environment.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: dev-ingress
  namespace: dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: dev.nodejs.my.node.app.deepak2828.com
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: my-node-app-service
            port:
              number: 3000
```

Apply:
```bash
kubectl apply -f dev-ingress.yaml
```

---

## **5. Configuring Ingress for QA**
Define a similar Ingress resource for the QA environment.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: qa-ingress
  namespace: qa
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - host: qa.nodejs.my.node.app.deepak2828.com
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: my-node-app-service
            port:
              number: 3000
```

Apply:
```bash
kubectl apply -f qa-ingress.yaml
```

---

## **6. Validating Ingress Setup**
Check if the Ingress configurations are applied:
```bash
kubectl get ingress -n dev
kubectl get ingress -n qa
```

Test access using `curl`:
```bash
curl -H "Host: dev.nodejs.my.node.app.deepak2828.com" http://k8s-ingressn-ingressn-529eec9849-5c7aa9b6b0bbd22b.elb.us-east-2.amazonaws.com/app1
```
```bash
curl -H "Host: qa.nodejs.my.node.app.deepak2828.com" http://k8s-ingressn-ingressn-529eec9849-5c7aa9b6b0bbd22b.elb.us-east-2.amazonaws.com/app1
```

---

## **7. Managing DNS for DEV and QA**
Each environment requires a unique DNS entry:
- **DEV Domain:** `dev.nodejs.my.node.app.deepak2828.com`
- **QA Domain:** `qa.nodejs.my.node.app.deepak2828.com`

Update DNS in **GoDaddy** or **Route 53** with CNAME records pointing to the Load Balancer.

---

## **Conclusion**
By implementing **NGINX Ingress** separately for **QA and DEV**, 

you achieve:
✅ **Environment Isolation**: Prevents conflicts between testing and development workloads.  
✅ **Controlled Routing**: Ensures traffic reaches the correct service per environment.  
✅ **Scalability & Flexibility**: Allows easy modification of paths and domain names.  

