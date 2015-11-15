## Introduction

Openstack is a collection of open source software which work together to "controls large pools of compute, storage, and networking resources throughout a datacenter".

There are three major parts of a cloud which openstack manages - Compute, Storage and networking. Openstack has a set of services which run on the nodes of the cloud and interact with each other. Openstack components consist of nova which manages all the compute nodes(more details on it [here](http://docs.openstack.org/kilo/install-guide/install/apt/content/ch_nova.html#compute-service) and [here]()), cinder and swift for storage, and neutron for networking. There are also shared services like keystone for identity management, glance for image management, and a web interface called horizon.

For the most part, [OpenStack Kilo Installtion guide for Ubuntu 14.04](http://docs.openstack.org/kilo/install-guide/install/apt/content/) provides good documentation on how to set up openstack. We have mostly followed steps given in the guide with a few modifications. Hence, wherever required we will explicitly provide the steps or else provide links from the guide.

## Environment

We will set up openstack on four nodes. Three of these will act as compute nodes, where the VM instances will be scheduled, while one node will be controller and network node which will run the nova, keystone, glance and horizon services. All the four nodes have Ubuntu 14.04 LTS installed

All the compute nodes have a single network interface(NIC) and the controller node has two NICs (we added a USB ehternet port to the CPU). The NIC of the compute nodes are and one NIC of the controller node are connected to a Cisco L2 switch. This will act as the Management Network for the cloud(more on this later). The second NIC of the controller node is connected to the external network(the CSE department network in our case).


## Preliminaries

### Assigning IPs
On the controller node, add ip addresses for management network and external network as follows. Here we assume that eth0 is the interface connected to the switch i.e. the management network and eth1 is connected to the external network.

Edit `/etc/network/interfaces file` and add

```
auto eth0
iface eth0 inet static
  address 10.0.0.11
  netmask 255.255.255.0

# following settings are relevant to CSE department only. Find out you own IP, subnet, gateway and DNS  if in a different setting
auto eht1
iface eth1 inet static
  address 172.27.24.145
  netmask 255.255.0.0
  gateway 172.27.26.254
  dns-nameservers 172.31.1.130 172.31.1.1
```

On the first compute node, edit `etc/network/interfaces` file and add

```
auto eht0
iface eth0 inet static
  address 10.0.0.12
  netmask 255.255.255.0
```

Similarly on other compute nodes, add other IPs in the management networks.
Restart to apply the configurations.

### Assigning Hostnames

On each of the nodes, edit `etc/hosts` file and add hostname entries for each of the host

```
10.0.0.11 controller
10.0.0.12 compute1
10.0.0.13 compute2
```

It is recommended that the hostname of the system is same as the network hostname we have given it. So, for the controller node, system hostname should be controller and for first compute node, compute1. Hostname on ubuntu can be changed by the `hostnamectl` command(see `man hostnamectl` for its usage).

Make sure that every node is pingable from every other node.

### Setting up NTP

Run `apt-get install ntp` to install ntp on all nodes

### Adding Openstack Kilo Repo

On all the nodes, run

```bash
apt-get install ubuntu-cloud-keyring

echo "deb http://ubuntu-cloud.archive.canonical.com/ubuntu" \
  "trusty-updates/kilo main" > /etc/apt/sources.list.d/cloudarchive-kilo.list

sudo apt-get update
```

Make sure that the above commands complete without any error on all the nodes, otherwise wrong versions of packages will be installed on the nodes.

### Installing MySQL

Follow instructions given [here](http://docs.openstack.org/kilo/install-guide/install/apt/content/ch_basic_environment.html#basics-database) to setup mysql database.

### Installing RabbitMQ

Follow instructions given [here](http://docs.openstack.org/kilo/install-guide/install/apt/content/ch_basic_environment.html#basics-message-queue) to setup RabbitMQ.

## Keystone

Follow all the steps given [here](http://docs.openstack.org/kilo/install-guide/install/apt/content/ch_keystone.html)

## Glance (Openstack Image Service)

Follow all the steps given [here](http://docs.openstack.org/kilo/install-guide/install/apt/content/ch_glance.html)

To add a Ubuntu image, download Ubuntu Cloud Image for 14.04 and add it using following commands

```bash
wget -P /tmp/images https://cloud-images.ubuntu.com/trusty/current/trusty-server-cloudimg-amd64-disk1.img

glance image-create --name "ubuntu-14.04-amd64" --file /tmp/images/trusty-server-cloudimg-amd64-disk1.img \
  --disk-format qcow2 --container-format bare --visibility public --progress
```

## Compute

Follow all the steps given [here](http://docs.openstack.org/kilo/install-guide/install/apt/content/ch_nova.html)

Note that `MANAGEMENT_INTERFACE_IP_ADDRES` referred to in the above guide will be the IP addresses given to the nodes from 10.0.0.0/24 IP address range. For controller it will be `10.0.011` , for compute1 it will be `10.0.0.12` and so on for all the other nodes.

## Networking


### Installing Neutron

We will configure Neutron networking since Nova networking is deprecated. Since we do not have a dedicated network node, all the network node service will be installed on the controller node itself.

Follow the steps given [here](http://docs.openstack.org/kilo/install-guide/install/apt/content/section_neutron-networking.html).

The instructions given in the guide for the network node are to be performed on the controller node. While configuring the network services, we will have to make some changes in the configuration to make it work with our setup. While configuring the Open vSwitch (OVS) service, the interface which is to be added to the bridge `br-ex` is `eth1`. This is the interface which connects the tenant network of the VMs and the external network. Generally, this interface has no IP assigned and connected to a switch which acts as a gateway to the external network for the cloud. We have no switch and this interface will have to act as a gateway.

TBD

Make sure not to miss the optional configuration given in the network node configuration guide for configuring the MTU in the DHCP agent. Reducing the MTU size is necessary for large packets to be transmitted in the network.

### Creating Networks

* __External Network__ - Follow the steps given [here](http://docs.openstack.org/kilo/install-guide/install/apt/content/neutron_initial-external-network.html). In our case, external network is the network to which `eth1` interface of the controller node is connected to i.e. the CSE department network. So, `EXTERNAL_NETWORK_CIDR` will be 172.27.0.0/16 and `EXTERNAL_NETWORK_GATEWAY` will be 172.27.16.254 . Floating Network IP Range will be the set of IP addresses from 172.27.0.0/16 range that can be assigned to the VMs. In our case, we have set it to 172.27.3.10 to 172.27.3.60 since these IP addresses are currently unassigned.

* __Tenant Network__ - Follow the steps given [here](http://docs.openstack.org/kilo/install-guide/install/apt/content/neutron_initial-tenant-network.html). This will create a network for the demo tenant. All the VMs created will get IPs from the range 192.168.1.0/24 . To access the VMs from the external network, the VM will have to be assigned a floating IP which will give it an IP in the 172.16.0.0/24 range.


## Setting up NFS

One of the requirements of the setup is live migration of VMs across compute nodes. This can be achieved in two ways - having a shared storage or enabling block migrations. In Block Migration, the disk of the VM along with its memory is migrated across the hosts while in shared storage setup, the disks are shared across the hosts. Block migrations are slow and have a greater performance penalty, hence we will configure shared storage through NFS.

In a typical Openstack deployment, every compute node manages its instances locally in a dedicated directory (for example, /var/lib/nova/instances/) but for shared storage live migration, this folder has to be in a centralized location and shared across all the compute nodes. We will share the /var/lib/nova/instances directory on controller node and mount it on all the compute nodes. This is an arbitrary choice and any other directory on any other node could have been chosen as well.

_We have to make sure that the UID of the owner of this directory i.e. the 'nova' user is same on all the nodes. Otherwise there will be problems in the permissions of files in this directory. This can be ensured by changing the UID of the 'nova' user on all node to make them same. Refer to https://muffinresearch.co.uk/linux-changing-uids-and-gids-for-user/ for information on how to change UIDs in linux_

Steps to install NFS server on Controller:

```bash
apt-get install nfs-kernel-server
```
In the `etc/exports` file, add the following lines

```
/var/lib/nova/images *(rw,sync,no_root_squash)
/var/lib/nova/instances *(rw,sync,no_root_squash)
```

On the compute nodes:

 ```bash
 apt-get install nfs-common
 ```

 To mount the directory, run

 ```bash
 mount -t nfs -o proto=tcp,port=2049 controller:/var/lib/nova/instances /var/lib/nova/instances
 ```

## Configuring Live Migrations

For live migration, the libvirt daemon on each of the compute nodes should listen to TCP. Do the following on the compute nodes:

* Edit the `/etc/libvirt/libvirtd.conf` on each node and add
  ```
  before : #listen_tls = 0
  after : listen_tls = 0

  before : #listen_tcp = 1
  after : listen_tcp = 1

  add: auth_tcp = "none"
  ```

* Modify `/etc/default/libvirt-bin`
  ```
  before :libvirtd_opts=" -d"
  after :libvirtd_opts=" -d -l"
  ```

* Restart libvirtd
  ```
  stop libvirt-bin && start libvirt-bin
  ```
