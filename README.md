### Packstack_with_Ceph

##### *Index

#### Packstack

#### Ceph 

#### Integration. 

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

10) **reboot** the node. 

11) After **reboot** , check **service** details. 

~~~
source keystonerc_admin
~~~

~~~
[root@packstack-queens ~(keystone_admin)]# openstack service list
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
[root@packstack-queens ~(keystone_admin)]# 
~~~

~~~
[root@packstack-queens ~(keystone_admin)]# openstack compute service list
+----+----------------+------------------------------+----------+---------+-------+----------------------------+
| ID | Binary         | Host                         | Zone     | Status  | State | Updated At                 |
+----+----------------+------------------------------+----------+---------+-------+----------------------------+
|  2 | nova-conductor | packstack-queens.example.com | internal | enabled | up    | 2020-09-11T10:07:50.000000 |
|  3 | nova-scheduler | packstack-queens.example.com | internal | enabled | up    | 2020-09-11T10:07:49.000000 |
|  5 | nova-compute   | packstack-queens.example.com | nova     | enabled | up    | 2020-09-11T10:07:46.000000 |
+----+----------------+------------------------------+----------+---------+-------+----------------------------+
[root@packstack-queens ~(keystone_admin)]# 
~~~

~~~
[root@packstack-queens ~(keystone_admin)]# openstack volume service list
+------------------+----------------------------------+------+---------+-------+----------------------------+
| Binary           | Host                             | Zone | Status  | State | Updated At                 |
+------------------+----------------------------------+------+---------+-------+----------------------------+
| cinder-scheduler | packstack-queens.example.com     | nova | enabled | up    | 2020-09-11T10:08:02.000000 |
| cinder-backup    | packstack-queens.example.com     | nova | enabled | up    | 2020-09-11T10:08:05.000000 |
| cinder-volume    | packstack-queens.example.com@lvm | nova | enabled | up    | 2020-09-11T10:08:07.000000 |
+------------------+----------------------------------+------+---------+-------+----------------------------+
[root@packstack-queens ~(keystone_admin)]# 
~~~

~~~
[root@packstack-queens ~(keystone_admin)]# openstack network agent list
+--------------------------------------+--------------------+------------------------------+-------------------+-------+-------+---------------------------+
| ID                                   | Agent Type         | Host                         | Availability Zone | Alive | State | Binary                    |
+--------------------------------------+--------------------+------------------------------+-------------------+-------+-------+---------------------------+
| 1b71f13b-0701-4dd5-b3ce-87337e13897b | Open vSwitch agent | packstack-queens.example.com | None              | :-)   | UP    | neutron-openvswitch-agent |
| 281b4481-a6de-4162-ad5d-62400122752e | L3 agent           | packstack-queens.example.com | nova              | :-)   | UP    | neutron-l3-agent          |
| 507f8431-9ceb-44b6-a564-739f9ffb99dc | Metadata agent     | packstack-queens.example.com | None              | :-)   | UP    | neutron-metadata-agent    |
| 8bfd16ec-9ccb-42ce-88e2-f8ae2a53fb98 | DHCP agent         | packstack-queens.example.com | nova              | :-)   | UP    | neutron-dhcp-agent        |
| be148363-179f-435b-900c-f00478937d07 | Metering agent     | packstack-queens.example.com | None              | :-)   | UP    | neutron-metering-agent    |
+--------------------------------------+--------------------+------------------------------+-------------------+-------+-------+---------------------------+
[root@packstack-queens ~(keystone_admin)]# 
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

|NodeName|IP|Role|RAM|RootDisk|DataDisk|
|----|----|----|
|Packstack|192.168.100.150|MON_Node|6GB|50GB|NA|
|cepn_node0|192.168.100.160|OSD_0_Node|1GB|10GB|5GB|
|cepn_node1|192.168.100.161|OSD_1_Node|1GB|10GB|5GB|
|cepn_node2|192.168.100.162|OSD_2_Node|1GB|10GB|5GB|
