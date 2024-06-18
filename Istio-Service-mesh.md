# On Microk8s Cluster

### Enable the community add-on repository in MicroK8s.
```
microk8s enable community 
```

### Verify the status of MicroK8s to ensure all components are running correctly.
```
microk8s status
```

### Install the Istio service mesh add-on.
```
microk8s enable istio
```

### Verify the status after enabling Istio
```.
microk8s status
```

### Get an overview of all resources in the cluster across all namespaces
```
microk8s kubectl get all --all-namespaces
```

### Set up MetalLB for load balancing and configure the IP address range.
```
microk8s enable metallb 
```
### When you enable this add on you will be asked for an IP address pool that MetalLB will hand out IPs from:

```
192.168.5.1-192.168.5.100  
```


### Label the default namespace to enable automatic Istio sidecar injection.
```
microk8s kubectl label namespace default istio-injection=enabled
```

### verify the istio injection
### Deploy an Nginx application and expose it via a NodePort service.
```
microk8s kubectl create deployment nginx --image=nginx
microk8s kubectl expose deployment nginx --port=80 --type=NodePort
```

### List all pods and services to ensure the Nginx deployment is running and exposed.
```
microk8s kubectl get pods
microk8s kubectl get svc
```

### Get detailed information about the Nginx pod.
```
microk8s kubectl describe pod <POD_NAME>
```


### verify the istio injection with different namespace
```
microk8s kubectl create ns dev
microk8s kubectl run webapp1 --image httpd --port 80 -n dev
microk8s kubectl -n dev get pods
microk8s kubectl label namespace dev istio-injection=enabled
microk8s kubectl run webapp2 --image httpd --port 80 -n dev
microk8s kubectl -n dev get pods
```
