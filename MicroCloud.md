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
