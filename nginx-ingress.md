
# Ingress Controller Setup with Example

## Enabling Required Add-ons

### Step 1: Enable MicroK8s Community Add-ons
Ensure that MicroK8s community add-ons are enabled.
```bash
microk8s enable community
```

### Step 2: Enable Istio
Enable Istio if it is not already enabled.
```bash
microk8s enable ingress
```

### Step 3: Set Up MetalLB for Load Balancing
Enable MetalLB for load balancing and configure the IP address range.
```bash
microk8s enable metallb
```
When prompted, specify an IP address pool that MetalLB will use to allocate IPs:
```bash
192.168.5.1-192.168.5.100
```

## Deploying Applications and Exposing Services

### Step 4: Deploy and Expose an HTTPD Application
Deploy an `httpd` application and expose it via a ClusterIP service.
```bash
microk8s kubectl run httpd --image httpd --port 80
microk8s kubectl expose pod httpd --name httpd-svc --port 80 --target-port 80
```

### Step 5: Deploy and Expose an NGINX Application
Deploy an `nginx` application and expose it via a ClusterIP service.
```bash
microk8s kubectl run nginx --image nginx --port 80
microk8s kubectl expose pod nginx --name nginx-svc --port 80 --target-port 80
```

## Creating an Ingress Resource

### Step 6: Create an Ingress Object
Create an Ingress object to route traffic to the `httpd` and `nginx` services.
```bash
vi ingress.yaml
```
Add the following content to `ingress.yaml`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: httpd-svc
            port:
              number: 80
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80
```

### Step 7: Apply the Ingress Configuration
Apply the Ingress configuration to the cluster.
```bash
microk8s kubectl apply -f ingress.yaml
```

### Step 8: Retrieve the Ingress Object
Run the following command to get details of the Ingress object in the default namespace.
```bash
microk8s kubectl get ingress
```

### Step 9: Access the Applications via External IP
Copy the external IP assigned by MetalLB and access the applications using `curl`.
```bash
curl <ExternalIP>
curl <ExternalIP>/testpath

