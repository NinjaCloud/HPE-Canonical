# Configure namespaces

### Let's create the Namespace object for this pods.
```
kubectl create ns policy-demo
```

## Create demo pods

### Create some nginx pods in the policy-demo namespace.
```
kubectl create deployment --namespace=policy-demo nginx --image=nginx
```

### Expose them through a service.
```
kubectl expose --namespace=policy-demo deployment nginx --port=80
```

### Ensure the nginx service is accessible.
```
kubectl run --namespace=policy-demo access --rm -ti --image busybox /bin/sh
```

### From inside the access pod, attempt to reach the nginx service.
```
wget -q nginx -O -
```

### exit from pod
```
exit
```

## Enable isolation

### Let's turn on isolation in our policy-demo namespace. Calico will then prevent connections to pods in this namespace.
### Running the following command creates a NetworkPolicy which implements a default deny behavior for all pods in the policy-demo namespace.
```
kubectl create -f - <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: default-deny
  namespace: policy-demo
spec:
  podSelector:
    matchLabels: {}
EOF
```

# Test Isolation

### This will prevent all access to the nginx service. We can see the effect by trying to access the service again.
```
kubectl run --namespace=policy-demo access --rm -ti --image busybox /bin/sh
```

### Now from within the busybox access pod execute the following command to test access to the nginx service.
```
wget -q --timeout=5 nginx -O -
```

### By enabling isolation on the namespace, we've prevented access to the service.

### exit from pod
```
exit
```

# Allow access using a network policy

## Now, let's enable access to the nginx service using a NetworkPolicy. This will allow incoming connections from our access pod, but not from anywhere else.

### Create a network policy access-nginx with the following contents:
```
kubectl create -f - <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: access-nginx
  namespace: policy-demo
spec:
  podSelector:
    matchLabels:
      app: nginx
  ingress:
    - from:
      - podSelector:
          matchLabels:
            run: access
EOF
```

### We should now be able to access the service from the access pod.
```
kubectl run --namespace=policy-demo access --rm -ti --image busybox /bin/sh
```

### Now from within the busybox access pod execute the following command to test access to the nginx service.
```
wget -q --timeout=5 nginx -O -
```

### However, we still cannot access the service from a pod without the label run-access

### Now create a pod with label attatched and check the connectivity 
```
kubectl run --namespace=policy-demo access --rm -ti --image busybox -l run=access /bin/sh
```
```
wget -q --timeout=5 nginx -O -
```



