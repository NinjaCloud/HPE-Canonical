# Deploying the application

## Start the application services

### The default Istio installation uses automatic sidecar injection. Label the namespace that will host the application with istio-injection=enabled

```
microk8s kubectl label namespace default istio-injection=enabled
```

### Deploy your application using the kubectl command

```
microk8s kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/platform/kube/bookinfo.yaml
```

### Confirm all services and pods are correctly defined and running:
```
microk8s kubectl get services
microk8s kubectl get pods
```
### Create Gateway and Virtual serives 
```
microk8s kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/bookinfo/networking/bookinfo-gateway.yaml
```

### To confirm that the Bookinfo application is running, send a request to it by a curl command from some pod, for example from ratings:

```
microk8s kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"
```

### To access the application from outside the word please edit productpage clusterIP service to NodePort

```
microk8s kubectl edit svc productpage
```

## Enable Kiali Dashboard 
```
microk8s kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/addons/kiali.yaml
```

### Kiali application is deployed in istio-system namespace to access it convert kiali clusterIP service to nodeport

```
microk8s kubectl edit svc kiali -n istio-system
```

## To collect traffic from application Kiali need Prometheus run the following command to install it
```
microk8s kubectl apply -f https://raw.githubusercontent.com/istio/istio/master/samples/addons/prometheus.yaml
```
