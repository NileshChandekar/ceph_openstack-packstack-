### Packstack_with_Ceph

##### Index

#### 1) Packstack
#### 2) Ceph
#### 3) Integration.

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

8) Here is my [answer file](https://github.com/NileshChandekar/ceph_openstack-packstack-/blob/master/images/answer.txt) , Now perform a install of **Packstack**.

~~~
packstack --answer-file=packstack.txt
~~~


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

10) **reboot** the node.

11) After **reboot** , check **service** details.

~~~
source keystonerc_cephuser1
~~~

~~~
[root@packstack-queens ~(keystone_cephuser1)]# openstack service list
+----------------------------------+-----------+--------------+
| ID                               | Name      | Type         |
+----------------------------------+-----------+--------------+
| 2e22de865f214d28beadc094b1e22bf3 | nova      | compute      |
| 32119abccb294bf1a129b7c27bd6201e | placement | placement    |
| 6984cb1a33de4d2bb97a71ee66dbdbba | swift     | object-store |
| ab5c3fca3bc2435f90b1a4757c896027 | glance    | image        |
| c9d869a8b02a4fd0b46b395a6d5a63c6 | cinderv3  | volumev3     |
| d26b60f2bc994253bcbfab2dc2318608 | neutron   | network      |
| e3c8f69f83e3484dace9f99edc51fe2e | cinderv2  | volumev2     |
| f8c55cef9d684ff6951ae62edea0b2b3 | keystone  | identity     |
+----------------------------------+-----------+--------------+
[root@packstack-queens ~(keystone_cephuser1)]#
~~~

~~~
[root@packstack-queens ~(keystone_cephuser1)]# openstack compute service list
+----+----------------+------------------------------+----------+---------+-------+----------------------------+
| ID | Binary         | Host                         | Zone     | Status  | State | Updated At                 |
+----+----------------+------------------------------+----------+---------+-------+----------------------------+
|  2 | nova-conductor | packstack-queens.example.com | internal | enabled | up    | 2020-09-11T10:07:50.000000 |
|  3 | nova-scheduler | packstack-queens.example.com | internal | enabled | up    | 2020-09-11T10:07:49.000000 |
|  5 | nova-compute   | packstack-queens.example.com | nova     | enabled | up    | 2020-09-11T10:07:46.000000 |
+----+----------------+------------------------------+----------+---------+-------+----------------------------+
[root@packstack-queens ~(keystone_cephuser1)]#
~~~

~~~
[root@packstack-queens ~(keystone_cephuser1)]# openstack volume service list
+------------------+----------------------------------+------+---------+-------+----------------------------+
| Binary           | Host                             | Zone | Status  | State | Updated At                 |
+------------------+----------------------------------+------+---------+-------+----------------------------+
| cinder-scheduler | packstack-queens.example.com     | nova | enabled | up    | 2020-09-11T10:08:02.000000 |
| cinder-backup    | packstack-queens.example.com     | nova | enabled | up    | 2020-09-11T10:08:05.000000 |
| cinder-volume    | packstack-queens.example.com@lvm | nova | enabled | up    | 2020-09-11T10:08:07.000000 |
+------------------+----------------------------------+------+---------+-------+----------------------------+
[root@packstack-queens ~(keystone_cephuser1)]#
~~~

~~~
[root@packstack-queens ~(keystone_cephuser1)]# openstack network agent list
+--------------------------------------+--------------------+------------------------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host                         | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------------------------+-------------------+-------+-------+---------------------------+
| 1b71f13b-0701-4dd5-b3ce-87337e13897b | Open vSwitch agent | packstack-queens.example.com | None              | :-)   | UP    | neutron-openvswitch-agent |
| 281b4481-a6de-4162-ad5d-62400122752e | L3 agent           | packstack-queens.example.com | nova              | :-)   | UP    | neutron-l3-agent          |
| 507f8431-9ceb-44b6-a564-739f9ffb99dc | Metadata agent     | packstack-queens.example.com | None              | :-)   | UP    | neutron-metadata-agent    |
| 8bfd16ec-9ccb-42ce-88e2-f8ae2a53fb98 | DHCP agent         | packstack-queens.example.com | nova              | :-)   | UP    | neutron-dhcp-agent        |
| be148363-179f-435b-900c-f00478937d07 | Metering agent     | packstack-queens.example.com | None              | :-)   | UP    | neutron-metering-agent    |
+--------------------------------------+--------------------+------------------------------+-------------------+-------+-------+---------------------------+
[root@packstack-queens ~(keystone_cephuser1)]#
~~~

12) Let check ![Horizon](https://github.com/NileshChandekar/ceph_openstack-packstack-/tree/master/images/1.png)  dashboard.

13) Let check ![Log into Horizon](https://github.com/NileshChandekar/ceph_openstack-packstack-/tree/master/images/2.png)  dashboard.


### Ceph Deployment

* Ceph is a distributed file system supporting block, object and file based storage.

* It consists of:

	* MON nodes,
	* OSD nodes and optionally an
	* MDS node.

* The **MON** node is for monitoring the cluster and there are normally multiple monitor nodes to prevent a single point of failure.

* The **OSD** nodes house ceph Object Storage Daemons which is where the user data is held.

* The **MDS** node is the Meta Data Node and is only used for file based storage. It is not necessary if block and object storage is only needed.

* The **diagram** below is taken from the ceph web site and shows that all nodes have access to a front end Public network, [Storage Management]

* Optionally there is a backend Cluster Network which is only used by the OSD nodes. [Storage Data Network]

* The cluster network takes replication traffic away from the front end network and may improve performance.

* By default a backend cluster network is not created and needs to be manually configured in ceph’s configuration file (ceph.conf).

* The ceph clients are part of the cluster.

* The Client nodes know about monitors, OSDs and MDS’s but have no knowledge of object locations.

* Ceph clients communicate directly with the OSDs rather than going through a dedicated server.

* The OSDs (Object Storage Daemons) store the data. They can be up and in the map or can be down and out if they have failed.

* An OSD can be down but still in the map which means that the PG has not yet been remapped. When OSDs come on line they inform the monitor.

* The Monitors store a master copy of the cluster map.

### Diagram ###

#### The architectural model of ceph is shown below.

* **RADOS**  stands for Reliable Autonomic Distributed Object Store and it makes up the heart of the scalable object storage service.

* In addition to accessing RADOS via the defined interfaces, it is also possible to access RADOS directly via a set of library calls as shown above.

#### Ceph Replication

* By default three copies of the data are kept, although this can be changed!

* Ceph can also use Erasure Coding, with Erasure Coding objects are stored in k+m chunks where :

	* k = # of data chunks and
	* m = # of recovery or coding chunks

* Example k=7, m=2 would use 9 OSDs
	* 7 for data storage and
	* 2 for recovery

* Pools are created with an appropriate replication scheme.

#### CRUSH (Controlled Replication Under Scalable Hashing)

* The CRUSH map knows the topology of the system and is location aware. Objects are mapped to Placement Groups and Placement Groups are mapped to OSDs.

* It allows dynamic rebalancing and controls which Placement Group holds the objects and which of the OSDs should hold the Placement Group.

* A CRUSH map holds a list of OSDs, buckets and rules that hold replication directives.

* CRUSH will try not to shuffle too much data during rebalancing whereas a true hash function would be likely to cause greater data movement

* The CRUSH map allows for different resiliency models such as:

	#0 for a 1-node cluster.

	#1 for a multi node cluster in a single rack

	#2 for a multi node, multi chassis cluster with multiple hosts in a chassis

	#3 for a multi node cluster with hosts across racks, etc.

* osd crush chooseleaf type = {n}

### Installation of Ceph.

#### Prepare OSD Nodes First.

|NodeName|IP [Public Network]|Role|RAM|RootDisk|DataDisk|InternalNetwork [Data Replication]|
|----|----|----|----|----|----|----|
|Packstack|192.168.100.150|MON_Node|6GB|50GB|NA|NA|
|cepn_node0|192.168.100.160|OSD_0_Node|1GB|10GB|5GB|10.0.0.160|
|cepn_node1|192.168.100.161|OSD_1_Node|1GB|10GB|5GB|10.0.0.161|
|cepn_node2|192.168.100.162|OSD_2_Node|1GB|10GB|5GB|10.0.0.162|

### Setting up **MON** node.

~~~
[root@packstack-queens ~]# ip a s br-ex
6: br-ex: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 8e:59:c2:27:47:46 brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.150/24 brd 192.168.100.255 scope global br-ex
       valid_lft forever preferred_lft forever
    inet6 fe80::8c59:c2ff:fe27:4746/64 scope link
       valid_lft forever preferred_lft forever
[root@packstack-queens ~]#
~~~

~~~
[root@packstack-queens ~]# cat  /etc/hosts | tail -1
192.168.100.150 packstack-queens.example.com packstack-queens mon.example.com mon
[root@packstack-queens ~]#
~~~

~~~
[root@packstack-queens ~]# ping -c 2 mon
PING packstack-queens.example.com (192.168.100.150) 56(84) bytes of data.
64 bytes from packstack-queens.example.com (192.168.100.150): icmp_seq=1 ttl=64 time=0.104 ms
64 bytes from packstack-queens.example.com (192.168.100.150): icmp_seq=2 ttl=64 time=0.057 ms

--- packstack-queens.example.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.057/0.080/0.104/0.025 ms
[root@packstack-queens ~]#
~~~

~~~
[root@packstack-queens ~]# ping -c 2 mon.example.com
PING packstack-queens.example.com (192.168.100.150) 56(84) bytes of data.
64 bytes from packstack-queens.example.com (192.168.100.150): icmp_seq=1 ttl=64 time=0.037 ms
64 bytes from packstack-queens.example.com (192.168.100.150): icmp_seq=2 ttl=64 time=0.039 ms

--- packstack-queens.example.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.037/0.038/0.039/0.001 ms
[root@packstack-queens ~]#
~~~

* Create a user with any name like **cephuser1** . **ON ALL NODES**

~~~
useradd cephuser1
echo cephuser1 | passwd --stdin cephuser1
echo "cephuser1 ALL=(root) NOPASSWD:ALL" | tee -a /etc/sudoers.d/cephuser1
chmod 0440 /etc/sudoers.d/cephuser1
~~~

* Centos disabling requiretty **ON ALL NODES**

~~~
[root@packstack-queens ~]# visudo

Defaults:cephuser1 !requiretty
~~~

### Setting up **OSD** nodes.

* Set up **hostname** and **Public Network** i.e *Management Network*

~~~
[root@localhost ~]# hostnamectl set-hostname ceph-osd-0.example.com
~~~

~~~
[root@ceph-osd-0 ~]# ip a s eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:9d:c8:3c brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.160/24 brd 192.168.100.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe9d:c83c/64 scope link
       valid_lft forever preferred_lft forever
[root@ceph-osd-0 ~]#
~~~

~~~
[root@localhost ~]# hostnamectl set-hostname ceph-osd-1.example.com
~~~

~~~
[root@ceph-osd-1 ~]# ip a s eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:fb:76:8a brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.161/24 brd 192.168.100.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fefb:768a/64 scope link
       valid_lft forever preferred_lft forever
[root@ceph-osd-1 ~]#
~~~

~~~
[root@localhost ~]# hostnamectl set-hostname ceph-osd-2.example.com
~~~

~~~
[root@ceph-osd-2 ~]# ip a s eth0
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:73:db:5c brd ff:ff:ff:ff:ff:ff
    inet 192.168.100.162/24 brd 192.168.100.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe73:db5c/64 scope link
       valid_lft forever preferred_lft forever
[root@ceph-osd-2 ~]#
~~~

* Set up **Data Network** i.e *Storage replication*

~~~
[root@ceph-osd-0 ~]# ip a s eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:a4:fe:ab brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.160/24 brd 10.0.0.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fea4:feab/64 scope link
       valid_lft forever preferred_lft forever
[root@ceph-osd-0 ~]#
~~~

~~~
[root@ceph-osd-1 ~]# ip a s eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:0d:db:3b brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.161/24 brd 10.0.0.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe0d:db3b/64 scope link
       valid_lft forever preferred_lft forever
[root@ceph-osd-1 ~]#
~~~

~~~
[root@ceph-osd-2 ~]# ip a s eth1
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:26:ae:ed brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.162/24 brd 10.0.0.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::5054:ff:fe26:aeed/64 scope link
       valid_lft forever preferred_lft forever
[root@ceph-osd-2 ~]#
~~~

* Check **PING** over **Data Replication** Network.

~~~
[root@ceph-osd-0 ~]# ping -c 2 10.0.0.161
PING 10.0.0.161 (10.0.0.161) 56(84) bytes of data.
64 bytes from 10.0.0.161: icmp_seq=1 ttl=64 time=0.606 ms
64 bytes from 10.0.0.161: icmp_seq=2 ttl=64 time=0.839 ms

--- 10.0.0.161 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.606/0.722/0.839/0.119 ms
[root@ceph-osd-0 ~]#
[root@ceph-osd-0 ~]# ping -c 2 10.0.0.162
PING 10.0.0.162 (10.0.0.162) 56(84) bytes of data.
64 bytes from 10.0.0.162: icmp_seq=1 ttl=64 time=0.997 ms
64 bytes from 10.0.0.162: icmp_seq=2 ttl=64 time=0.519 ms

--- 10.0.0.162 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.519/0.758/0.997/0.239 ms
[root@ceph-osd-0 ~]#
~~~

~~~
[root@ceph-osd-1 ~]# ping -c 2 10.0.0.162
PING 10.0.0.162 (10.0.0.162) 56(84) bytes of data.
64 bytes from 10.0.0.162: icmp_seq=1 ttl=64 time=0.928 ms
64 bytes from 10.0.0.162: icmp_seq=2 ttl=64 time=0.430 ms

--- 10.0.0.162 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.430/0.679/0.928/0.249 ms
[root@ceph-osd-1 ~]#
[root@ceph-osd-1 ~]# ping -c 2 10.0.0.160
PING 10.0.0.160 (10.0.0.160) 56(84) bytes of data.
64 bytes from 10.0.0.160: icmp_seq=1 ttl=64 time=0.616 ms
64 bytes from 10.0.0.160: icmp_seq=2 ttl=64 time=0.541 ms

--- 10.0.0.160 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.541/0.578/0.616/0.044 ms
[root@ceph-osd-1 ~]#
~~~

~~~
[root@ceph-osd-2 ~]# ping -c 2 10.0.0.160
PING 10.0.0.160 (10.0.0.160) 56(84) bytes of data.
64 bytes from 10.0.0.160: icmp_seq=1 ttl=64 time=0.377 ms
64 bytes from 10.0.0.160: icmp_seq=2 ttl=64 time=0.867 ms

--- 10.0.0.160 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.377/0.622/0.867/0.245 ms
[root@ceph-osd-2 ~]#
[root@ceph-osd-2 ~]# ping -c 2 10.0.0.161
PING 10.0.0.161 (10.0.0.161) 56(84) bytes of data.
64 bytes from 10.0.0.161: icmp_seq=1 ttl=64 time=0.788 ms
64 bytes from 10.0.0.161: icmp_seq=2 ttl=64 time=0.395 ms

--- 10.0.0.161 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.395/0.591/0.788/0.198 ms
[root@ceph-osd-2 ~]#
~~~

* On **MON** node.

~~~
[root@packstack-queens ~]# cat  /etc/hosts | tail -3
192.168.100.160 ceph-osd-0.example.com ceph-osd-0
192.168.100.161 ceph-osd-1.example.com ceph-osd-1
192.168.100.162 ceph-osd-2.example.com ceph-osd-2
[root@packstack-queens ~]#
~~~

~~~
[root@packstack-queens ~]# ping -c 1 ceph-osd-0
PING ceph-osd-0.example.com (192.168.100.160) 56(84) bytes of data.
64 bytes from ceph-osd-0.example.com (192.168.100.160): icmp_seq=1 ttl=64 time=3.02 ms

--- ceph-osd-0.example.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 3.020/3.020/3.020/0.000 ms
[root@packstack-queens ~]#
~~~

~~~
[root@packstack-queens ~]# ping -c 1 ceph-osd-1
PING ceph-osd-1.example.com (192.168.100.161) 56(84) bytes of data.
64 bytes from ceph-osd-1.example.com (192.168.100.161): icmp_seq=1 ttl=64 time=0.860 ms

--- ceph-osd-1.example.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.860/0.860/0.860/0.000 ms
[root@packstack-queens ~]#
~~~

~~~
[root@packstack-queens ~]# ping -c 1 ceph-osd-2
PING ceph-osd-2.example.com (192.168.100.162) 56(84) bytes of data.
64 bytes from ceph-osd-2.example.com (192.168.100.162): icmp_seq=1 ttl=64 time=1.44 ms

--- ceph-osd-2.example.com ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.44
~~~

#### Setting up passwordless login
* The ceph-deploy tool requires passwordless login with a non-root account, this can be achieved by performing the following steps:

* On **MON** node.

~~~
[cephuser1@packstack-queens ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/cephuser1/.ssh/id_rsa):
Created directory '/home/cephuser1/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/cephuser1/.ssh/id_rsa.
Your public key has been saved in /home/cephuser1/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:9O1X/yIoV5DIM46CzrheP3SfzfghDPN8JuHsgsg269Y cephuser1@packstack-queens.example.com
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|       ... .     |
|       .=.o.     |
|   .   =S+...   .|
|  . ....X ...  ..|
| +..+.o .O=*. . .|
|. +*.E .o+Boo.. .|
|oo+oo.. .+.. . ..|
+----[SHA256]-----+
[cephuser1@packstack-queens ~]$
~~~

* Now copy the key from **MON** to each of the **OSD** nodes in turn.

~~~
[cephuser1@packstack-queens ~]$ ssh-copy-id cephuser1@ceph-osd-0
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/cephuser1/.ssh/id_rsa.pub"
The authenticity of host 'ceph-osd-0 (192.168.100.160)' can't be established.
ECDSA key fingerprint is SHA256:4Lk2Z6PVfrRqheDAx3vHZByfPXPVX4B1W9XMcbxsZHE.
ECDSA key fingerprint is MD5:e7:3f:78:de:b6:ed:0b:c6:8b:e3:a7:f8:b0:c9:aa:99.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
cephuser1@ceph-osd-0's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'cephuser1@ceph-osd-0'"
and check to make sure that only the key(s) you wanted were added.

[cephuser1@packstack-queens ~]$
~~~

~~~
[cephuser1@packstack-queens ~]$ ssh-copy-id cephuser1@ceph-osd-1
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/cephuser1/.ssh/id_rsa.pub"
The authenticity of host 'ceph-osd-1 (192.168.100.161)' can't be established.
ECDSA key fingerprint is SHA256:A/dvUKCKyxlXMsflcdMluX3lG6XQ3pLm6iTaqA8fqK0.
ECDSA key fingerprint is MD5:23:05:ae:e2:f1:67:bb:19:38:b0:ae:ec:35:20:d5:d7.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
cephuser1@ceph-osd-1's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'cephuser1@ceph-osd-1'"
and check to make sure that only the key(s) you wanted were added.

[cephuser1@packstack-queens ~]$
~~~

~~~
[cephuser1@packstack-queens ~]$ ssh-copy-id cephuser1@ceph-osd-2
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/cephuser1/.ssh/id_rsa.pub"
The authenticity of host 'ceph-osd-2 (192.168.100.162)' can't be established.
ECDSA key fingerprint is SHA256:sz2Vbz0/JdoeZa2CzdWSSHWKYFerlFm57twYRIbB4ew.
ECDSA key fingerprint is MD5:d4:e3:ee:a5:35:82:68:58:de:8a:d5:71:21:67:c5:28.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
cephuser1@ceph-osd-2's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'cephuser1@ceph-osd-2'"
and check to make sure that only the key(s) you wanted were added.

[cephuser1@packstack-queens ~]$
~~~

* Lets try to **SSH** from  **MON** to **OSD** , it should not ask for to enter the password.

~~~
[cephuser1@packstack-queens ~]$ ssh cephuser1@ceph-osd-0
Last login: Sat Sep 12 03:10:54 2020 from 192.168.100.1
[cephuser1@ceph-osd-0 ~]$
~~~

~~~
[cephuser1@packstack-queens ~]$ ssh cephuser1@ceph-osd-1
Last login: Sat Sep 12 03:07:23 2020 from 192.168.100.1
[cephuser1@ceph-osd-1 ~]$
~~~

~~~
[cephuser1@packstack-queens ~]$ ssh cephuser1@ceph-osd-2
Last login: Sat Sep 12 03:07:43 2020 from 192.168.100.1
[cephuser1@ceph-osd-2 ~]$
~~~

### Configuring the ceph repositories on CentOS

~~~
[cephuser1@packstack-queens ~]$ cat  /etc/yum.repos.d/ceph.repo
[ceph-noarch]
name=Ceph noarch packages
baseurl=http://download.ceph.com/rpm-luminous/el7/noarch/
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
[cephuser1@packstack-queens ~]$
~~~

### Installing and configuring ceph

~~~
[cephuser1@packstack-queens ~]$ sudo yum update -y && sudo yum install ceph-deploy -y
~~~

~~~
[cephuser1@packstack-queens ~]$ mkdir  ~/cephcluster
[cephuser1@packstack-queens ~]$ cd cephcluster/
~~~

#### Setup the monitor node.

* The format of this command is ceph-deploy new <monitor1>, <monitor2>, . . <monitor n>.

**NOTE** : Note production environments will typically have a minimum of three monitor nodes to prevent a single node of failure

~~~
[cephuser1@packstack-queens cephcluster]$ ceph-deploy new mon
~~~

~~~
[cephuser1@packstack-queens cephcluster]$ ll -lhrt
total 12K
-rw-------. 1 cephuser1 cephuser1   73 Sep 12 03:39 ceph.mon.keyring
-rw-rw-r--. 1 cephuser1 cephuser1 3.6K Sep 12 03:39 ceph-deploy-ceph.log
-rw-rw-r--. 1 cephuser1 cephuser1  196 Sep 12 03:39 ceph.conf
[cephuser1@packstack-queens cephcluster]$
~~~

* Examine ceph.conf

~~~
[cephuser1@packstack-queens cephcluster]$ cat  ceph.conf
[global]
fsid = a2012d2a-640e-4069-91ef-24effd209e96
mon_initial_members = mon
mon_host = 192.168.100.150
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
[cephuser1@packstack-queens cephcluster]$
~~~

**NOTE**

a) The file *ceph.conf* is hugely important in ceph. This file holds the configuration details of the cluster.  

b) This is also the time to make any changes to the configuration file before it is pushed out to the other nodes.

c) One option that could be used is to lower the replication factor as shown following:

### Changing the replication factor in ceph.conf

* The following options can be used to change the replication factor:

~~~
osd pool default size = 2
osd pool default min size = 1
~~~

**NOTE** : In this case the default replication size is 2 and the system will run as long as one of the OSDs is up.

### Install ceph on all nodes

~~~
[cephuser1@packstack-queens cephcluster]$ ceph-deploy install mon ceph-osd-0 ceph-osd-1 ceph-osd-2
~~~
