# 1. Install and initialise LXD
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
