# 网络虚拟化技术基础

随着虚拟化技术的出现，网络也随之被虚拟化，相较于单一的物理网络，虚拟网络变得非常复杂，在一个主机系统里面，需要实现诸如**交换、路由、隧道、隔离、聚合等**多种网络功能。

**而实现这些功能的基本元素就是虚拟的网络设备，比如:**

**虚拟二层网卡(tap)** 只能处理二层的以太网数据帧，与其中的以太网（Ethernet）协议对应

**虚拟三层隧道网卡(tun)** 是一个点对点（Peer To Peer）的网络层设备，只能处理 IP 数据包，通常用于建立 IP 层隧道（Tunnel）

**虚拟网线(veth-pair)** 将 tap 之间，tap 与 Bridge 之间连接起来。veth pair 通常还与 Network namespace 一起配合，实现不同 Network namespace 中的网络设备传输。

```
网络虚拟化：
    桥接：将物理网卡添加到bridge上，而后让虚拟机网卡后半段也桥接到bridge上，从而使得虚拟机借助
         物理网卡与外部通信。
    隔离：构建内部专用的通信虚拟网络，这个虚拟网络中由一个虚拟交换机来实现，而这个虚拟机就是我们
         自己创建的网桥设备。
    路由：把隔离模型中的这个桥设备附着在有hypervisor的第零层上,打开hypervisor上的路由转发功能
         由此其内部虚拟机就可以通过某物理设备路由转发以后可以同外部通信，外部网络同内部虚拟机通信
         必须要有明确路由指明。
    NAT：在路由基础上添加地址转发规则。
一旦虚拟机部署到了某个物理机上，cpu和内存就只能使用该物理

管理节点两张网卡 内网网络网卡或者多一个可以访问互联网的网卡
计算节点两张网卡 一个用于实现管理操作，一个用于所有能建立虚拟机的物理机彼此通信
网络节点3张网卡，一个内部管理用，一个用于连接互联网，一个跟各物理机通信

管理节点跟虚拟化不在同一个概念中

VLAN间路由:
    路由器:
        访问链接：router为每个vlan提供一个接口
        汇聚链接：router只向交换机提供一个接口
    三层交换机：
        三层交换机既有二层功能又有三层功能。首先在2层能够在各个接口之间实现报文交换的功能，
在上面额外提供一个软件实现路由

# 容器级的虚拟化，就是利用linux内核提供的资源隔离功能来实现的。

虚拟化技术：
    IaaS:
    Paas: docker        
    linux内核：
        namespace：名称空间，完成特定类型资源的隔离
        cgroups：控制组，可以在已经隔离出来的的名称空间中，按比例把资源分配到名称空间中去
            假设一台服务器有16核，分成4个分支ABCD（A-8C,B-4C,C-2C,D-2C）A这个分支可以分成
            3个分支(A1-3C,A2-3C,A3-2C)。那么一个进程，可以绑定到根上，这16核，它想用多少用多
            少。也可以绑定到A分支上，最多能用8核。也可以绑定到A1分支上，最多能用3核。也可以绑定
            到B分支上，最多能用4核。
网络虚拟化技术
    复杂的虚拟化网络：
        netns
        OpenVSwitch
    openvswitch:基于C语言研发，特性：
        支持802.1q,trunk,access
        支持NIC绑定，
        NetFlow,sFlow
        QoS配置及策略
        GRE，Vxlan
        OpenFlow
        基于linux实现高性能转发
    openvswitch的组成部分：
        ovs-vswitchd: ovs daemon,实现数据报文交换功能，和linux内核兼容模块一同实现了基于流的
        交换技术
        ovsdb-server:轻量级的数据库服务，主要保存了整个OVS的配置信息
        ovs-vsctl: 用于获取或更改ovs-vswitchd的配置信息，修改操作会保存到ovsdb-server

```

二、netns

```
# 创建namespace
root@network:~# ip netns add r1
root@network:~# ip netns add r2
root@network:~# ip netns list
r2
r1
# 创建物理桥
root@network:~# brctl addbr br-ex
root@network:~# ip link set br-ex up
root@network:~# ip a
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether f8:bc:12:5a:d5:e1 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.97/24 brd 192.168.0.255 scope global enp1s0
       valid_lft forever preferred_lft forever
    inet6 fe80::fabc:12ff:fe5a:d5e1/64 scope link 
       valid_lft forever preferred_lft forever
6: br-ex: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 4a:37:86:4e:7a:44 brd ff:ff:ff:ff:ff:ff
    inet6 fe80::4837:86ff:fe4e:7a44/64 scope link 
       valid_lft forever preferred_lft forever
# 创建一对虚拟网卡
root@network:~# ip link add veth1.1 type veth peer name veth1.2
root@network:~# ip a
3: veth1.2@veth1.1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether e6:c6:2b:69:92:eb brd ff:ff:ff:ff:ff:ff
4: veth1.1@veth1.2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether e6:0b:ac:a8:d5:21 brd ff:ff:ff:ff:ff:ff
# 将这对网卡分别添加到r1,r2这两个名称空间
root@network:~# ip link set veth1.1 netns r1
root@network:~# ip link set veth1.2 netns r2
root@network:~# ip netns exec r1 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
4: veth1.1@if3: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether e6:0b:ac:a8:d5:21 brd ff:ff:ff:ff:ff:ff link-netnsid 1
root@network:~# ip netns exec r2 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: veth1.2@if4: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether e6:c6:2b:69:92:eb brd ff:ff:ff:ff:ff:ff link-netnsid 0
#修改名称空间r1,r2中这对网卡ip地址及启用这对网卡
root@network:~# ip netns exec r1 ip addr add 172.16.0.1/24 dev veth1.1
root@network:~# ip netns exec r2 ip addr add 172.16.0.2/24 dev veth1.2
root@network:~# ip netns exec r1 ip link set veth1.1 up
root@network:~# ip netns exec r2 ip link set veth1.2 up
root@network:~# ip netns exec r1 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
4: veth1.1@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether e6:0b:ac:a8:d5:21 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet 172.16.0.1/24 scope global veth1.1
       valid_lft forever preferred_lft forever
    inet6 fe80::e40b:acff:fea8:d521/64 scope link 
       valid_lft forever preferred_lft forever
root@network:~# ip netns exec r2 ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: veth1.2@if4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether e6:c6:2b:69:92:eb brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.16.0.2/24 scope global veth1.2
       valid_lft forever preferred_lft forever
    inet6 fe80::e4c6:2bff:fe69:92eb/64 scope link 
       valid_lft forever preferred_lft forever
root@network:~# ip netns exec r1 ping 172.16.0.2
PING 172.16.0.2 (172.16.0.2) 56(84) bytes of data.
64 bytes from 172.16.0.2: icmp_seq=1 ttl=64 time=0.037 ms



# 
ip addr del 192.168.0.97/24 dev enp1s0; ip addr add 192.168.0.97/24 dev br-ex; brctl addif br-ex enp1s0

```
