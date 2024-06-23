# AppArmor Profile Loader

## The example DaemonSet demonstrating how the profile loader can be deployed onto a cluster to automatically load AppArmor profiles from a ConfigMap.

### Running the AppArmor Profile Loader

### Create the following files on host

```
vi namespace.yaml
```

```
apiVersion: v1
kind: Namespace
metadata:
  name: apparmor
```

```
vi configmap.yaml
```

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: apparmor-profiles
  namespace: apparmor
data:
  # Filename k8s-nginx maps to the definition of the nginx profile.
  k8s-nginx: |-
    #include <tunables/global>

    # From https://github.com/jfrazelle/bane/blob/master/docker-nginx-sample
    profile k8s-nginx flags=(attach_disconnected,mediate_deleted) {
      #include <abstractions/base>

      network inet tcp,
      network inet udp,
      network inet icmp,

      deny network raw,

      deny network packet,

      file,
      umount,

      deny /bin/** wl,
      deny /boot/** wl,
      deny /dev/** wl,
      deny /etc/** wl,
      deny /home/** wl,
      deny /lib/** wl,
      deny /lib64/** wl,
      deny /media/** wl,
      deny /mnt/** wl,
      deny /opt/** wl,
      deny /proc/** wl,
      deny /root/** wl,
      deny /sbin/** wl,
      deny /srv/** wl,
      deny /tmp/** wl,
      deny /sys/** wl,
      deny /usr/** wl,

      audit /** w,

      /var/run/nginx.pid w,

      /usr/sbin/nginx ix,

      deny /bin/dash mrwklx,
      deny /bin/sh mrwklx,
      deny /usr/bin/top mrwklx,

      capability chown,
      capability dac_override,
      capability setuid,
      capability setgid,
      capability net_bind_service,

      deny @{PROC}/{*,**^[0-9*],sys/kernel/shm*} wkx,
      deny @{PROC}/sysrq-trigger rwklx,
      deny @{PROC}/mem rwklx,
      deny @{PROC}/kmem rwklx,
      deny @{PROC}/kcore rwklx,
      deny mount,
      deny /sys/[^f]*/** wklx,
      deny /sys/f[^s]*/** wklx,
      deny /sys/fs/[^c]*/** wklx,
      deny /sys/fs/c[^g]*/** wklx,
      deny /sys/fs/cg[^r]*/** wklx,
      deny /sys/firmware/efi/efivars/** rwklx,
      deny /sys/kernel/security/** rwklx,
    }
```


```
vi daemon.yaml
```

```
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: apparmor-loader
  # Namespace must match that of the ConfigMap.
  namespace: apparmor
  labels:
    daemon: apparmor-loader
spec:
  selector:
    matchLabels:
      daemon: apparmor-loader
  template:
    metadata:
      name: apparmor-loader
      labels:
        daemon: apparmor-loader
    spec:
      containers:
      - name: apparmor-loader
        image: google/apparmor-loader:latest
        args:
          # Tell the loader to pull the /profiles directory every 30 seconds.
          - -poll
          - 30s
          - /profiles
        securityContext:
          # The loader requires root permissions to actually load the profiles.
          privileged: true
        volumeMounts:
        - name: sys
          mountPath: /sys
          readOnly: true
        - name: apparmor-includes
          mountPath: /etc/apparmor.d
          readOnly: true
        - name: profiles
          mountPath: /profiles
          readOnly: true
      volumes:
      # The /sys directory must be mounted to interact with the AppArmor module.
      - name: sys
        hostPath:
          path: /sys
      # The /etc/apparmor.d directory is required for most apparmor include templates.
      - name: apparmor-includes
        hostPath:
          path: /etc/apparmor.d
      # Map in the profile data.
      - name: profiles
        configMap:
          name: apparmor-profiles
```


```
vi pod.yaml
```

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-apparmor
  # Note that the Pod does not need to be in the same namespace as the loader.
  labels:
    app: nginx
  annotations:
    # Tell Kubernetes to apply the AppArmor profile "k8s-nginx".
    # Note that this is ignored if the Kubernetes node is not running version 1.4 or greater.
    container.apparmor.security.beta.kubernetes.io/nginx: localhost/k8s-nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```
### It is recommended to run the Daemon and ConfigMap in a separate, restricted namespace:

```
kubectl create -f namespace.yaml
kubectl create -f configmap.yaml 
kubectl create -f daemon.yaml
```
### Check that the profile was loaded:

```
export POD=$(kubectl --namespace apparmor get pod -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace apparmor logs $POD
```

### Trying running a pod with the loaded profile

```
kubectl create -f pod.yaml
```

### Verify that it's running with the new profile:

```
kubectl exec nginx-apparmor cat /proc/1/attr/current
```

### Test the profile
```
kubectl exec nginx-apparmor touch /tmp/foo
```





