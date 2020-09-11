#Index

### Packstack

### Ceph 

### Integration. 

#### Packstack Installation:

0) Set **Hostname** and **ip** addess. 

~~~
hostnamectl set-hostname packstack-queens.example.com
~~~

~~~
DEVICE="eth0"
BOOTPROTO="none"
ONBOOT="yes"
TYPE="Ethernet"
IPADDR=192.168.100.150
PREFIX=24
GATEWAY=192.168.100.1
DNS1=192.168.100.1
DNS2=8.8.8.8
~~~

1) Set **System Setting** 

~~~
systemctl disable NetworkManager.service
systemctl enable network
systemctl disable firewalld
~~~

2) Enable **EPEL** repo

~~~
yum  install epel-release.noarch -y
yum clean all
yum update -y 
reboot
~~~

3) After rebbot, enable **openstack** repo

~~~
sudo yum install -y https://www.rdoproject.org/repos/rdo-release.rpm
sudo yum install -y centos-release-openstack-queens
yum install yum-utils  -y
~~~

4) Install **Packstack** utility. 

~~~
yum install -y openstack-packstack
~~~

5) Generate **answer** file. 

~~~
packstack --gen-answer-file=packstack 
~~~

6) Edit **answer** file, and update with below details:

~~~
CONFIG_CEILOMETER_INSTALL=n
~~~

~~~
CONFIG_NEUTRON_OVS_BRIDGE_MAPPINGS=datacentre:br-ex
CONFIG_NEUTRON_OVS_BRIDGE_IFACES=
CONFIG_NEUTRON_OVS_BRIDGES_COMPUTE=
CONFIG_NEUTRON_OVS_EXTERNAL_PHYSNET=extnet
CONFIG_NEUTRON_OVS_TUNNEL_IF=
CONFIG_NEUTRON_OVS_TUNNEL_SUBNETS=
CONFIG_NEUTRON_OVS_VXLAN_UDP_PORT=4789
~~~

~~~
CONFIG_NEUTRON_ML2_MECHANISM_DRIVERS=openvswitch
~~~

~~~
CONFIG_NEUTRON_L2_AGENT=openvswitch
~~~

~~~
CONFIG_PROVISION_DEMO=n
~~~

7) Remove **OVN**

8) Here is my [answer file](https://github.com/NileshChandekar/ceph_openstack-packstack-/blob/master/images/answer.txt)

9) Create a **bridge** network

~~~
DEVICE=eth0
ONBOOT=yes
TYPE=OVSPort
DEVICETYPE=ovs
OVS_BRIDGE=br-ex



DEVICE=br-ex
BOOTPROTO=static
ONBOOT=yes
TYPE=OVSBridge
DEVICETYPE=ovs
USERCTL=yes
PEERDNS=yes
IPV6INIT=no
IPADDR=192.168.100.150
NETMASK=255.255.255.0
GATEWAY=192.168.100.1
DNS1=10.75.5.25
DNS2=8.8.8.8
~~~


