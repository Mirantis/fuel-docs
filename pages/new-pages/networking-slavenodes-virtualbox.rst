Networking Notes for Slave Nodes on VirtualBox
----------------------------------------------

Helper scripts for VirtualBox create network adapters eth0, eth1, eth2 
assigned to vboxnet0, vboxnet1, vboxnet2 correspondingly. vboxnet0 is 
dedicated for **Fuel** network so it is impossible to use it for any other 
untagged networks.

Also these scripts assign IP addresses for adapters: vboxnet0 - 
10.20.0.1/24, vboxnet1 - 172.16.1.1/24, vboxnet2 - 172.16.0.1/24.

To access guest VMs from host machine **Public** and/or **Management** 
networks must be untagged and assigned to vboxnet1 and vboxnet2 adapters 
with IP addresses from ranges specified earlier.

During installation the Slave nodes access the Internet via Master node. But 
when installation is done Slave nodes on guest VMs shall access the Internet 
via the **Public** network. To make it happen the following command must be 
executed on host:

::
sudo iptables -t nat -A POSTROUTING -s 172.16.1.0/24 \! -d 172.16.1.0/24 -j 
MASQUERADE

To access VMs managed by OpenStack it is needed to provide IP addresses from 
floating IP range. When OpenStack cluster is deployed it is needed to create 
security groups to provide access to guest VMs using following commands:

::
nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0
nova secgroup-add-rule default tcp 22 22 0.0.0.0/0

You can also add these security groups from Horizon UI.

IP ranges for Public and Management networks (172.16.*.*) defined in 
config.sh script. You can reassign these IP ranges before running VirtualBox 
scripts only. 
