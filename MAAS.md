# Build a MAAS and LXD environment on Ubuntu

## You will need:
+ Ubuntu 18.04 LTS or higher OR Windows with Hyper-V
+ 16 GB of RAM
+ A quad core CPU with virtualisation support (Intel VT or AMD-V)
+ Virtualisation support enabled in the BIOS
+ 30 GB of free disk space

## Check whether virtualisation is working

## We now need to check whether virtualisation is working correctly. This is a relatively simple process. In your terminal, run:
```
sudo apt install cpu-checker
kvm-ok
```
## Run the following commands on ec2 instance
```
sudo apt-get update
sudo apt-get install jq
sudo snap install --channel=3.0/stable maas
sudo snap install --channel=latest/stable lxd
sudo snap refresh --channel=latest/stable lxd
sudo snap install maas-test-db
```

## Fetch IPv4 address from the device, setup forwarding and NAT
```
export IP_ADDRESS=$(ip -j route show default | jq -r '.[].prefsrc')
export INTERFACE=$(ip -j route show default | jq -r '.[].dev')
sed -i 's/#net.ipv4.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf
sysctl -p
iptables -t nat -A POSTROUTING -o $INTERFACE -j SNAT --to $IP_ADDRESS
```

## Persist NAT configuration
```
echo iptables-persistent iptables-persistent/autosave_v4 boolean true | sudo debconf-set-selections
echo iptables-persistent iptables-persistent/autosave_v6 boolean true | sudo debconf-set-selections
```

## LXD init

## Create a lxd.cfg file on ec2 instance and copy following content inside the file

```
vi lxd.cfg
```

```
config:
  core.https_address: '[::]:8443'
  core.trust_password: password
networks:
- config:
    ipv4.address: 10.10.10.1/24
    ipv6.address: none
  description: ""
  name: lxdbr0
  type: ""
  project: default
storage_pools:
- config:
    size: 24GB
  description: ""
  name: default
  driver: zfs 
profiles:
- config: {}
  description: ""
  devices:
    eth0:
      name: eth0
      network: lxdbr0
      type: nic 
    root:
      path: /
      pool: default
      type: disk
  name: default
projects: []
cluster: null
```

## Now init lxd with config file
```
cat lxd.cfg | lxd init --preseed
```

## Initialise MAAS
```
maas init region+rack --database-uri maas-test-db:/// --maas-url http://${PUBLIC_IP_ADDRESS}:5240/MAAS
```

## Create MAAS admin and grab API key
```
maas createadmin --username admin --password admin --email admin
export APIKEY=$(maas apikey --username admin)
```

## MAAS admin login
```
maas login admin 'http://${PUBLIC_IP_ADDRESS}:5240/MAAS/' $APIKEY
```

## Configure MAAS networking (set gateways, vlans, DHCP on etc)
```
export SUBNET=10.10.10.0/24
export FABRIC_ID=$(maas admin subnet read "$SUBNET" | jq -r ".vlan.fabric_id")
export VLAN_TAG=$(maas admin subnet read "$SUBNET" | jq -r ".vlan.vid")
export PRIMARY_RACK=$(maas admin rack-controllers read | jq -r ".[] | .system_id")
maas admin subnet update $SUBNET gateway_ip=10.10.10.1
maas admin ipranges create type=dynamic start_ip=10.10.10.200 end_ip=10.10.10.254
maas admin vlan update $FABRIC_ID $VLAN_TAG dhcp_on=True primary_rack=$PRIMARY_RACK
maas admin maas set-config name=upstream_dns value=8.8.8.8
```

## Add LXD as a VM host for MAAS
```
maas admin vm-hosts create  password=password  type=lxd power_address=https://${Public_IP_ADDRESS}:8443 project=maas
```

## Automatically create and add ssh keys to MAAS
```
ssh-keygen -q -t rsa -N "" -f "/home/ubuntu/.ssh/id_rsa"
chown ubuntu:ubuntu /home/ubuntu/.ssh/id_rsa /home/ubuntu/.ssh/id_rsa.pub
chmod 600 /home/ubuntu/.ssh/id_rsa
chmod 644 /home/ubuntu/.ssh/id_rsa.pub
maas admin sshkeys create key="$(cat /home/ubuntu/.ssh/id_rsa.pub)"
```

## # Wait for images to be synced to MAAS
