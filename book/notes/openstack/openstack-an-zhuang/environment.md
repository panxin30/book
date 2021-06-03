---
description: '参考：https://docs.openstack.org/install-guide/'
---

# 2、Environment

#### openstack 查看版本

```text
root@cn-office-public-ops01:~# nova-manage version
19.0.1
root@cn-office-public-ops01:~# nova --version
13.0.0
```

然后在[https://releases.openstack.org/teams/nova.html](https://releases.openstack.org/teams/nova.html) 的列表做对比 根据列表可知道这个版本是**Stein**

## 0. 安全

## 1. 架构：一个控制节点，一个计算节点，一个网络节点

os:Ubuntu 18.04.5 LTS \(GNU/Linux 4.15.0-117-generic x86\_64\)

flat+vxlan

### 一、控制节点：192.168.0.95 root@controller 2C8G

#### 安装组件：mysql,rabbitmq,memcache,etcd,keystone,glance,nova-api,nova-scheduler,nova-conductor,nova-novncproxy,neutron-server,neutron-metadata-agent,dashboard

可选组件：cinder、swift

### 二、计算节点：192.168.0.96 root@compute 12C32G

#### 安装组件：nova-compute,neutron-linuxbridge-agent

### 三、网络节点：192.168.0.97 root@network 2C4G

#### 安装组件：neutron-plugin-ml2，neutron-linuxbridge-agent，neutron-l3-agent，neutron-dhcp-agent，neutron-metadata-agent

> Note: 该实验环境是单网卡物理机，需求一块物理网卡能够绑定多个 IP 以及多个 MAC 地址，绑定多个 IP 很容易，但是这些 IP 会共享物理网卡的 MAC 地址,所以用物理网卡虚拟化macvlan。该实验instance接口和外网接口用同一个接口 tun 设备用来实现三层隧道（三层 ip 数据报），tap 设备用来实现二层隧道（二层以太网数据帧）

三台服务器确认是否支持硬件虚拟化 `egrep -c '(vmx|svm)' /proc/cpuinfo` 

## 2. 主机网络：

按照你选择的架构，完成各个节点操作系统安装后，**必须配置网络接口**。**推荐禁用自动网络管理**，手动编辑你相应版本的配置文件。 在大部分情况下，**节点应该通过管理网络接口访问互联网**。为了更好的突出网络隔离的重要性，示例架构中为管理网络使用`private address space` 更多关于如何配置你版本网络信息内容，参考 [documentation](https://ubuntu.com/server/docs) 。

#### self-service networks

增加了启用 self-service\`覆盖分段方法的layer-3（路由）服务，比如 : VXLAN。本质上，它使用 : NAT路由虚拟网络到物理网络。另外，这个选项也提供高级服务的基础，比如LBaas和FWaaS。 管理接口不需要网关 

**controller：管理接口10.0.0.11/24,外网接口192.168.0.95/24,\(网络组件neuron server,ML2 Plug-in\)**

**compute：管理接口10.0.0.21/24，instance 通道10.0.0.21/24\(实例之间构建隧道,GRE,VXLAN隧道\),\(网络组件ML2 Plug-in,layer 2 Agent\(ovs创建桥，创建vlan，构建GRE与network node节点通信\)\)，外网接口192.168.0.96/24**

**network：管理接口10.0.0.31/24，instance 通道10.0.0.31/24\(实例之间构建隧道,GRE,VXLAN隧道\),\(网络组件ML2 Plug-in,layer 2 Agent\(ovs\),layer 3 Agent\(主要作用创建netns,在netns中生成规则,在netns中生写iptables的DNAT/SNAT规则\),DHCP Agent\)，外网接口192.168.0.97/24**

## 配置网络接口

服务器一般都是多网卡,\(**多网卡聚合把多个网络端口绑定到一个IP地址**，可以提高网络总带宽和容错能力。\)

> Note: 该实验环境是单网卡物理机，需求一块物理网卡能够绑定多个 IP 以及多个 MAC 地址，绑定多个 IP 很容易，但是这些 IP 会共享物理网卡的 MAC 地址,所以用物理网卡虚拟化macvlan。该实验instance接口和外网接口用同一个接口 tun 设备用来实现三层隧道（三层 ip 数据报），tap 设备用来实现二层隧道（二层以太网数据帧）

**重启会失效**

```text
   41  ip link add link enp1s0 dev enp1s0.01 type macvlan
   42  ip link set enp1s0.01 up
   43  ip addr add 10.0.0.11/24 dev enp1s0.01
   44  ip a
```

## 3. 网络时间协议ntp

安装参考：[https://docs.openstack.org/install-guide/environment-ntp-controller.html](https://docs.openstack.org/install-guide/environment-ntp-controller.html)

**服务器集群之间的系统时间同步**

### 方法一、

在生产环境中，其网络都是内网结构，那么内网如何保证服务器之间的时间同步呢？其实这个问题很简单，只需要搭建一台内网时间服务器，然后让所有计算机都到服务端（10.28.204.66）去同步时间即可。 具体操作：**在服务端注释以下内容**：

```text
#server 0.centos.pool.ntp.org iburst
#server 1.centos.pool.ntp.org iburst
#server 2.centos.pool.ntp.org iburst
#server 3.centos.pool.ntp.org iburst
```

并添加以下内容：（表示与本机同步时间）

server 10.28.204.66 iburst

这样我们需求的一台内网时间服务器已经配置完毕。

**同样在客户端注释掉其他server**，并在客户端（10.28.204.65）添加以下：

server 10.28.204.66 iburst

### 方法二、作为所有其他本地系统的服务器与ntp服务器同步

此模型的好处包括减少外部连接的负载，减少远程NTP服务器的负载，并在外部连接或服务器出现故障时使本地系统彼此同步。 要在配置文件中启用本地服务器，请指定允许连接的网络和子网

```text
root@controller:~# ip route show
default via 192.168.0.2 dev enp1s0 proto static 
10.0.0.0/24 dev enp1s0.01 proto kernel scope link src 10.0.0.11 
192.168.0.0/24 dev enp1s0 proto kernel scope link src 192.168.0.95 
#######增加配置
cat /etc/chrony/chrony.conf
allow 192.168.0.0/24
allow 10.0.0.0/24
# 重启
systemctl restart chronyd.service
```

**在客户端，请更新chrony配置以指向新系统，然后重新启动chrony**。 例如，我的服务器位于192.168.2.12，我可以通过添加以下内容来更新配置文件以使用它进行同步： `server 192.168.2.12 iburst` **在客户端验证**

```text
root@test-network-node01:~# chronyc activity
200 OK
1 sources online
0 sources offline
0 sources doing burst (return to online)
0 sources doing burst (return to offline)
0 sources with unknown address
root@test-network-node01:~# chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* test-controller-node01        2   6   377    48  +1004us[+1097us] +/-  125ms
```

**在服务器上，使用以下clients命令验证客户端列表**

```text
root@test-controller-node01:~# sudo chronyc clients
Hostname                      NTP   Drop Int IntL Last     Cmd   Drop Int  Last
===============================================================================
test-network-node01             9      0   6   -    25       0      0   -     -
test-compute-node01             9      0   6   -     7       0      0   -     -
```

## 4. **openstack包 for ubuntu18.04**

> Note: Disable or remove any automatic update services because they can impact your OpenStack environment

### **4.1 Enable the repository for Ubuntu Cloud Archive**

**OpenStack Stein for Ubuntu 18.04 LTS:** `add-apt-repository cloud-archive:stein`

### **4.2 Finalize the installation**

```text
#Upgrade packages on all nodes
apt update && apt dist-upgrade
#Install the OpenStack client
apt install python3-openstackclient
```

## 5. SQL database for Ubuntu

As of Ubuntu 18.04 or 16.04, install the packages: `apt install mysql-server python-pymysql` Create and edit the **/etc/mysql/mysql.conf.d/mysqld.cnf** file and complete the following actions:

```text
[mysqld]
bind-address = 10.0.0.11 #控制节点的管理网络IP地址以使得其它节点可以通过管理网络访问数据库

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

Secure the database service by running the mysql\_secure\_installation script. In particular, choose a suitable password for the database root account:

 `systemctl enable mysql.service && systemctl restart mysql.service`

## 6. 消息队列

```text
apt install rabbitmq-server
rabbitmqctl add_user openstack RABBIT_PASS
# Creating user "openstack" ...
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
# Setting permissions for user "openstack" in vhost "/" ...
```

`systemctl enable rabbitmq-server && systemctl restart rabbitmq-server`

## 7. memcached

在生产部署中，推荐联合启用防火墙、认证和加密保证它的安全。 Install the packages:

```text
apt install memcached python-memcache
### Edit the /etc/memcached.conf file and configure the service to use the management IP address of the controller node. This is to enable access by other nodes via the management network:

-l 10.0.0.11
```

`systemctl enable memcached && systemctl restart memcached`

## 8. Etcd

`apt install etcd`

> Note As of Ubuntu 18.04, the etcd package is no longer available from the default repository. To install successfully, enable the Universe repository on Ubuntu.

Edit the /etc/default/etcd file and set the ETCD\_INITIAL\_CLUSTER, ETCD\_INITIAL\_ADVERTISE\_PEER\_URLS, ETCD\_ADVERTISE\_CLIENT\_URLS, ETCD\_LISTEN\_CLIENT\_URLS to the management IP address of the controller node to enable access by other nodes via the management network:

```text
ETCD_NAME="controller"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER="controller=http://10.0.0.11:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://10.0.0.11:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://10.0.0.11:2379"
ETCD_LISTEN_PEER_URLS="http://0.0.0.0:2380"
ETCD_LISTEN_CLIENT_URLS="http://10.0.0.11:2379"
```

`systemctl enable etcd && systemctl restart etcd`

#### Object Storage Installation Guide for Stein

[https://docs.openstack.org/swift/stein/install/](https://docs.openstack.org/swift/stein/install/)

