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
```
sudo apparmor_parser -r /etc/apparmor.d/deny-profile
```

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


# TASK 2  

## set up an AppArmor profile on your Kubernetes cluster, enforcing a denial rule to write access to /tmp/ while allowing write access to other directories on your Kubernetes worker node

###  create the AppArmor profile configuration file:
```
sudo vi  /etc/apparmor.d/deny-tmp-profile
```

### Add the following profile configuration to deny write access to /tmp/ and allow write access to other directories:

```
#include <tunables/global>

profile k8s-apparmor-deny-test {
  #include <abstractions/base>
  file,
  # Deny write access to /tmp/
  deny /tmp/** w,

}
```

### Parse/Load the AppArmor profile into the kernel:
```
sudo apparmor_parser -r /etc/apparmor.d/deny-tmp-profile
```
### Check the status of the AppArmor profile to confirm the denial /tmp/ rule:
```
sudo aa-status 
```

### Create a YAML file named secure-pod2.yaml and edit it:
```
vi secure-pod2.yaml
```

### Add the following configuration to the YAML file to define a secure pod:

```
apiVersion: v1
kind: Pod
metadata:
  name: secure-app2
  annotations:
    container.apparmor.security.beta.kubernetes.io/ctr-1: localhost/k8s-apparmor-deny-tmp
spec:
  containers:
  - name: ctr-1
    image: busybox
    command: ["sh", "-c", "sleep 5000"]
```

### Replace <worker-node-name> with the name of your Kubernetes worker node.

### Apply the YAML file to create the secure pod:
```
kubectl apply -f secure-pod2.yaml
```

### Access the shell of the secure pod:
```
kubectl exec -it secure-app2 -- sh
```

### Attempt to create a file inside the pod:
```
vi /tmp/demo.txt
```

### You should receive a "Permission denied" error due to the AppArmor profile denying file writes into /tmp/.

### Try to create file other than /tmp dir .

### You should receive a "Permission denied" error due to the AppArmor profile denying file writes.

### Exit the shell of the pod:
```
exit
```

# For your Information

## Kubernetes does not allow you to directly remove or update AppArmor annotations on a running pod. This restriction is in place to ensure the security and stability of the system.

## But you can disable the apparmor profile by running following command and then redeploy you application.

```
sudo ln -s /etc/apparmor.d/profile.name /etc/apparmor.d/disable/
```
```
sudo apparmor_parser -R /etc/apparmor.d/profile.name
```

## AppArmor can be disabled, and the kernel module unloaded, by entering the following:

```
sudo systemctl stop apparmor.service
sudo systemctl disable apparmor.service
```

## To re-enable AppArmor, enter:
```
sudo systemctl enable apparmor.service
sudo systemctl start apparmor.service
```

## Set the Profile to Complaining Mode
```
sudo aa-complain /etc/apparmor.d/profile-name
```

## Check the Logs
### On the host machine, check the AppArmor logs to see the reported violations
```
sudo dmesg | grep -i apparmor
```

