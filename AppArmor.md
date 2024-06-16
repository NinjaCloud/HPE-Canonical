# Task 1

## set up an AppArmor profile on your Kubernetes cluster, enforcing a denial rule on file writes within the secure pod.

### List the current status of apparmor
```
sudo aa-status
```

### If apparmor is not enabled by default then enable it 
```
sudo systemctl enable apparmor && sudo systemctl start apparmor
```

### Reload all profiles
```
sudo service apparmor reload
```

### On the Kubernetes Worker Node:

### create the AppArmor profile configuration file:

```
sudo vi  /etc/apparmor.d/deny-profile
```

### Add the following profile configuration to deny all file writes:

```
#include <tunables/global>
profile k8s-apparmor-deny-write flags=(attach_disconnected) {
  #include <abstractions/base>
  file,
  # Deny all file writes.
  deny /** w,
}
```

### Parse/Load the AppArmor profile into the kernel:

sudo apparmor_parser -r /etc/apparmor.d/deny-profile

### Check the status of the AppArmor profile to confirm the denial rule:

```
sudo aa-status 
```


### Create a YAML file named secure-pod.yaml and edit it:
```
vi secure-pod.yaml
```

### Add the following configuration to the YAML file to define a secure pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: secure-app1
  annotations:
    container.apparmor.security.beta.kubernetes.io/ctr-1: localhost/k8s-apparmor-deny-write
spec:
  nodeName: <worker-node-name>
  containers:
  - name: ctr-1
    image: busybox
    command: ["sh", "-c", "sleep 5000"]
```

### Replace <worker-node-name> with the name of your Kubernetes worker node.

### Apply the YAML file to create the secure pod:
```
kubectl apply -f secure-pod.yaml
```

### Access the shell of the secure pod:
```
kubectl exec -it secure-app1 -- sh
```

### Attempt to create a file inside the pod:

```
echo hello > demo.txt
```

### Expected output "sh: can't create demo.txt: Permission denied"

### You should receive a "Permission denied" error due to the AppArmor profile denying file writes.

### Exit the shell of the pod:
```
exit
```
