# Use NFS for Persistent Volumes on MicroK8s

## Setup an NFS server on local host

### If you don’t have a suitable NFS server already, you can simply create one on a local machine with the following commands on Ubuntu:
```
sudo apt-get install nfs-kernel-server
```

### Create a directory to be used for NFS:
```
sudo mkdir -p /srv/nfs
sudo chown nobody:nogroup /srv/nfs
sudo chmod 0777 /srv/nfs
```

### Edit the /etc/exports file. Make sure that the IP addresses of all your MicroK8s nodes are able to mount this share. For example, to allow all IP addresses in the 10.0.0.0/24 subnet: in our case we will use private ip of our host/subnet
```
sudo mv /etc/exports /etc/exports.bak
echo '/srv/nfs {privateIP/subnet}(rw,sync,no_subtree_check)' | sudo tee /etc/exports
```

### Finally, restart the NFS server:
```
sudo systemctl restart nfs-kernel-server
```

## Install the CSI driver for NFS

### First, we will deploy the NFS provisioner using the official Helm chart. Enable the Helm3 addon (if not already enabled) and add the repository for the NFS CSI driver:
```
microk8s enable helm3
microk8s helm3 repo add csi-driver-nfs https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/charts
microk8s helm3 repo update
```

### Then, install the Helm chart under the kube-system namespace with:
```
microk8s helm3 install csi-driver-nfs csi-driver-nfs/csi-driver-nfs \
    --namespace kube-system \
    --set kubeletDir=/var/snap/microk8s/common/var/lib/kubelet
```

### After deploying the Helm chart, wait for the CSI controller and node pods to come up using the following kubectl command …
```
microk8s kubectl wait pod --selector app.kubernetes.io/name=csi-driver-nfs --for condition=ready --namespace kube-system
```

### At this point, you should also be able to list the available CSI drivers in your Kubernetes cluster …
```
microk8s kubectl get csidrivers
```

## Create a StorageClass for NFS

### Next, we will need to create a Kubernetes Storage Class that uses the nfs.csi.k8s.io CSI driver. Assuming you have configured an NFS share /srv/nfs and the address of your NFS server is <privateIPofHost>, create the following file:

```
vi sc-nfs.yaml
```

```
---
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-csi
provisioner: nfs.csi.k8s.io
parameters:
  server: <PrivateIPofHost>
  share: /srv/nfs
reclaimPolicy: Delete
volumeBindingMode: Immediate
mountOptions:
  - hard
  - nfsvers=4.1
```

### Then apply it on your MicroK8s cluster with:
```
microk8s kubectl apply -f - < sc-nfs.yaml
```

## Create a new PVC

### The final step is to create a new PersistentVolumeClaim using the nfs-csi storage class. This is as simple as specifying storageClassName: nfs-csi in the PVC definition, for example:

```
vi pvc-nfs.yaml
```

```
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: nfs-csi
  accessModes: [ReadWriteOnce]
  resources:
    requests:
      storage: 5Gi
```

### Then create the PVC with:
```
microk8s kubectl apply -f - < pvc-nfs.yaml
```

### If everything has been configured correctly, you should be able to check the PVC…

```
microk8s kubectl describe pvc my-pvc
```


	
