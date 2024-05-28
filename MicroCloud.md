# Install and initialise LXD
## Complete the following steps to install and initialise LXD:
### 1.Install snapd:
### Run snap version to find out if snap is installed on your system:
```
snap version
```
### If the version for snapd is earlier than 2.59, or if the snap version command returns an error, run the following commands to install the latest version of snapd:
```
sudo apt update
sudo apt install snapd
```
### 2.Enter the following command to install LXD:
```
sudo snap install lxd
```
### 3.Enter the following command to initialise LXD:
```
lxd init
```
### Accept the default values except for the following questions:
*Size in GiB of the new loop device (1GiB minimum)*
***Enter 40GiB***

*Would you like the LXD server to be available over the network? (yes/no)*
***Enter yes***

#  Provide storage disks

## Create a ZFS storage pool called disks
```
lxc storage create disks zfs size=100GiB
```
## Configure the default volume size for the disks pool:
```
lxc storage set disks volume.size 10GiB
```
## Create four disks to use for local storage:
```
lxc storage volume create disks local1 --type block
lxc storage volume create disks local2 --type block
lxc storage volume create disks local3 --type block
lxc storage volume create disks local4 --type block
```
## Create three disks to use for remote storage:
```
lxc storage volume create disks remote1 --type block size=20GiB
lxc storage volume create disks remote2 --type block size=20GiB
lxc storage volume create disks remote3 --type block size=20GiB
```

# Create a network

## Create a bridge network without any parameters:
```
lxc network create microbr0
```
## Enter the following commands to find out the assigned IPv4 and IPv6 addresses for the network and note them down:
```
lxc network get microbr0 ipv4.address
lxc network get microbr0 ipv6.address
```

# Create and configure your VMs

## Create the VMs, but don’t start them yet
```
lxc init ubuntu:22.04 micro1 --vm --config limits.cpu=2 --config limits.memory=2GiB
lxc init ubuntu:22.04 micro2 --vm --config limits.cpu=2 --config limits.memory=2GiB
lxc init ubuntu:22.04 micro3 --vm --config limits.cpu=2 --config limits.memory=2GiB
lxc init ubuntu:22.04 micro4 --vm --config limits.cpu=2 --config limits.memory=2GiB
```
## Attach the disks to the VMs
```
lxc storage volume attach disks local1 micro1
lxc storage volume attach disks local2 micro2
lxc storage volume attach disks local3 micro3
lxc storage volume attach disks local4 micro4
lxc storage volume attach disks remote1 micro1
lxc storage volume attach disks remote2 micro2
lxc storage volume attach disks remote3 micro3
```
## Create and add network interfaces that use the dedicated MicroCloud network to each VM:
```
lxc config device add micro1 eth1 nic network=microbr0 name=eth1
lxc config device add micro2 eth1 nic network=microbr0 name=eth1
lxc config device add micro3 eth1 nic network=microbr0 name=eth1
lxc config device add micro4 eth1 nic network=microbr0 name=eth1
```
## Start the VMs:
```
lxc start micro1
lxc start micro2
lxc start micro3
lxc start micro4
```

# Install MicroCloud on each VM

## Complete the following steps on each VM (micro1, micro2, micro3, and micro4):
## Access the shell in the VM. For example, for micro1:
```
lxc exec micro1 -- bash
```
## Configure the network interface connected to microbr0 to not accept any IP addresses (because MicroCloud requires a network interface that doesn’t have an IP address assigned):
```
echo 0 > /proc/sys/net/ipv6/conf/enp6s0/accept_ra
```
## Bring the network interface up:
```
ip link set enp6s0 up
```
## Install the required snaps:
```
snap install microceph --channel=quincy/stable --cohort="+"
snap install microovn --channel=22.03/stable --cohort="+"
snap install microcloud --channel=latest/stable --cohort="+"
```
## The LXD snap is already installed. Refresh it to the latest version:
```
snap refresh lxd --channel=5.21/stable --cohort="+"
```

# Initialise MicroCloud

## After installing all snaps on all VMs, you can initialise MicroCloud. This initialisation is done on one of the machines only. We use micro1, but you can choose another machine.

## Access the shell in micro1:
```
lxc exec micro1 -- bash
```
## Start the initialisation process:
```
microcloud init
```
## Answer the questions:

+ As the address for MicroCloud’s internal traffic, select the listed IPv4 address.
+ Select yes to limit the search for other MicroCloud servers to the local subnet.
+ Select all listed servers (these should be micro2, micro3, and micro4).
+ Select yes to set up local storage.
+ Select the listed local disks (local1, local2, local3, and local4).
+ You don’t need to wipe any disks (because we just created them).
+ Select yes to set up distributed storage.
+ Select yes to confirm that there are fewer disks available than machines.
+ Select all listed disks (these should be remote1, remote2, and remote3).
+ You don’t need to wipe any disks (because we just created them).
+ Select yes to optionally configure the CephFS distributed file system.
+ Select yes to configure distributed networking.
+ Select all listed network interfaces (these should be enp6s0 on the four different VMs).
+ Specify the IPv4 address that you noted down for your microbr0 network as the IPv4 gateway.
+ Specify an IPv4 address in the address range as the first IPv4 address. For example, if your IPv4 gateway is 192.0.2.1/24, the first address could be 192.0.2.100.
+ Specify a higher IPv4 address in the range as the last IPv4 address. As we’re setting up four machines only, the range must contain a minimum of four addresses, but setting up a bigger range is more fail-safe. For example, if your IPv4 gateway is 192.0.2.1/24, the last address could be 192.0.2.254.
+ Specify the IPv6 address that you noted down for your microbr0 network as the IPv6 gateway.

# Inspect your MicroCloud setup

## Inspect the cluster setup:
```
lxc cluster list
microcloud cluster list
microceph cluster list
microovn cluster list
```

## Inspect the storage setup:
```
lxc storage list
lxc storage info local
lxc storage info remote
lxc storage info remote-fs
```

## Inspect the network setup:
```
lxc network list
lxc network show default
```

## Make sure that you can ping the virtual router within OVN. You can find the IPv4 and IPv6 addresses of the virtual router under volatile.network.ipv4.address and volatile.network.ipv6.address, respectively, in the output of lxc network show default.

```
ping <volatile.network.ipv4.address>
ping <volatile.network.ipv6.address>
```

## Inspect the default profile:
```
lxc profile show default
```

# Launch some instances

## Now that your MicroCloud cluster is ready to use, let’s launch a few instances:
## Launch an Ubuntu container with the default settings:
```
lxc launch ubuntu:22.04 u1
```
## Launch another Ubuntu container, but use the local storage instead of the remote storage that is the default:
```
lxc launch ubuntu:22.04 u2 --storage local
```
## Launch an Ubuntu VM:
```
lxc launch ubuntu:22.04 u3 --vm
```
## Check the list of instances. You will see that the instances are running on different cluster members.
```
lxc list
```
## Check the storage. You will see that the instance volumes are located on the specified storage pools.
```
lxc storage volume list remote
lxc storage volume list local
```


# Inspect your networking
## The instances that you have launched are all on the same subnet. You can, however, create a different network to isolate some instances from others.
## Check the list of instances that are running:
```
lxc list
```
## Access the shell in u1:
```
lxc exec u1 -- bash
```
## Ping the IPv4 address of u2:
```
ping <IPv4_address_of_u2>
```
## Confirm that the instance has connectivity to the outside world:
```
ping google.com
```
## Log out of the u1 shell:
```
exit
```
## Create an OVN network with the default settings:
```
lxc network create isolated --type=ovn
```
## Show information about the new network:
```
lxc network show isolated
```
## Check that you can ping the volatile.network.ipv4.address:
```
ping <volatile.network.ipv4.address>
```

## Launch an Ubuntu container that uses the new network:
```
lxc launch ubuntu:22.04 u4 --network isolated
```
## Access the shell in u4:
```
lxc exec u4 -- bash
```
## Confirm that the instance has connectivity to the outside world:
```
ping google.com
```
## Ping the IPv4 address of u2:
```
ping <IPv4_address_of_u2>
```

# Access the UI

## Check the LXD cluster list to determine the IP addresses of the cluster members in our case use public IP address of instance
```
https://<PublicIP>:8443
```


