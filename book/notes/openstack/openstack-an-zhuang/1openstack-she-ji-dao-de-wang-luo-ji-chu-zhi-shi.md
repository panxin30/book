# 1、openstack涉及到的网络基础知识

参考：[https://access.redhat.com/documentation/zh-cn/red\_hat\_enterprise\_linux\_openstack\_platform/7/html/networking\_guide/networking\_overview](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux_openstack_platform/7/html/networking_guide/networking_overview)

## openstack-networking和虚拟网络

    ****软件定义的网络（**Software-defined Networking，简称 SDN**）是一个用来描述虚拟网络功能的术语。虚拟环境中的服务器和物理服务器一样，仍然需要网络连接来接收和发送数据。SDN 通过把网络设备（如路由器和交换机）移到相同的虚拟空间来满足服务器对网络连接功能的要求。

    **需要在云环境中管理虚拟路由器和交换机，并需要考虑 IP 地址分配、VLAN 隔离以及子网等与网络相关的问题。**

    当把您的虚拟网络看做为物理网络的一个扩展部分时，以上问题可能就会变得简单。**默认网关、路由、子网这些概念的含义和作用都是相同的，虚拟网络仍然使用 TCP/IP、VLAN 和 MAC 地址。通常情况下，虚拟交换机是基于物理交换机上的 VLAN 配置的，它们只是物理网络的一个扩展。**

    **OpenStack 节点需要使用物理网络架构、IP 地址需要被分配、物理交换机的接口需要进行配置来支持 VLAN**。另外，除了故障排除外，在其它一些情况下也需要虚拟系统团队和网络团队的合作。例如，在调整虚拟机的 MTU 大小时，包括虚拟和物理交换机和路由器在内的所有端点都需要进行设置

## 网络基础

### **1.1 网络如何工作**

    Networking,一个计算机和另外一个计算机间进行的信息传递,通过在分别安装了网卡（NIC）的两个机器间的网线上实现 如果你需要把多于2台计算机进行通信，则需要一个**交换机**。现在，就有了一个"局域网" \(Local Area Network，简称 LAN\)

    交换机位于 OSI 模型的第 2 层，它会比第 1 层更智能一些：每个 NIC 都有一个与硬件关联的唯一 MAC 地址，连接到同一个交换机的设备可以通过 MAC 地址相互进行通讯。交换机会管理一个记录了哪些 MAC 地址连接到哪个端口的信息列表。

### **1.2 连接两个 LAN**

有2种选择实现这个功能 

选择二：使用网线把所有交换机连接到一个路由设备，从而可以使路由器知道每个交换机的配置。 连接到交换机的一端会被分配一个ip地址，它被称为默认网关（网关就是路由，路由就是网关）。当发送的数据目的地不在同一个LAN，会把数据包发给默认网关，路由器会处理以后再发送到目的地，路由器知道哪些网络位于哪些端口的信息。

#### **1.2.1 VLAN**

    可以通过配置端口来把交换机上的网络进行逻辑分割，从而形成多个“迷你” LAN，达到通过分隔网络数据实现数据安全的目的，它们将无法直接进行通讯。如果需要通信的话，则需要一个路由器。在这种情况下，防火墙可以用来管理VLAN间的网络通信。

#### **1.2.2 以太网**

    当NIC收到以太网帧时，默认情况下，NIC会检查目标MAC地址是否与NIC地址（或广播地址）匹配，如果MAC地址不匹配，则丢弃以太网帧。For a compute host\(计算节点\)，此行为是不希望有的，因为该帧可能用于其中一个实例。**可以将NIC配置为混杂模式\(promiscuous mode\)**，即使MAC地址不匹配，它们也可以将所有以太网帧传递给操作系统。**Compute hosts 应始终为混杂模式配置适当的NIC。**

**1.2.3 子网subnet**

{% hint style="info" %}
注意**:**

在OpenStack环境中不能使用创建包含多播地址或环回地址的CIDR子网。例如，不支持使用224.0.0.0/16或创建子网127.0.1.0/24。
{% endhint %}

    有时我们想引用一个子网，而不是子网中的任何特定IP地址。通用约定是将主机标识符设置为全零以引用子网。例如，如果主机的IP地址是192.0.2.24/24，那么我们说**子网是192.0.2.0/24。** 192.0.2.7首次尝试与192.0.2.8通信，未知192.0.2.8的MAC地址--&gt;于是在子网内发起**广播ARP**,所有人，我在寻找地址为192.0.2.8的计算机，我的MAC99：47：49：d4：a0--&gt;192.0.2.8响应我的地址192.0.2.8,MAC54：78：1a：86：00：a5

**1.2.4 DHCP**

    OpenStack使用名为dnsmasq的第三方程序 来实现DHCP服务器。Dnsmasq写入系统日志，您可以在其中观察DHCP请求并回复：

```text
Apr 23 15:53:46 c100-1 dhcpd: DHCPDISCOVER from 08:00:27:b9:88:74 via eth2
Apr 23 15:53:46 c100-1 dhcpd: DHCPOFFER on 192.0.2.112 to 08:00:27:b9:88:74 via eth2
Apr 23 15:53:48 c100-1 dhcpd: DHCPREQUEST for 192.0.2.112 (192.0.2.131) from 08:00:27:b9:88:74 via eth2
Apr 23 15:53:48 c100-1 dhcpd: DHCPACK on 192.0.2.112 to 08:00:27:b9:88:74 via eth2
```

对无法通过网络访问的实例进行故障排除时，检查此日志以验证是否对所讨论的实例执行了DHCP协议的所有四个步骤，可能会有所帮助。

#### **1.2.5 IP**

    **IP依赖于称为路由器或网关的特殊网络主机**。**路由器是连接到至少两个局域​​网的主机**，可以将IP数据包从一个局域网转发到另一个。**路由器具有多个IP地址：一个用于路由器的IP地址**。 在Linux计算机上，以下任何命令都会显示路由表：

```text
$ ip route show
$ route -n
$ netstat -rn
tony@z6:~$ ip route show
default via 192.168.2.1 dev wlp4s0 proto dhcp metric 600 
192.168.2.0/24 dev wlp4s0 proto kernel scope link src 192.168.2.38 metric 600
```

第1行指定默认路由的位置（默认网关） 

第2行指定192.168.2.0/24子网中的IP与网络接口wlp4s0关联的本地网络

**1.2.6 TCP/UDP**

    由于网络主机可能正在运行多个基于TCP的应用程序，因此TCP使用称为端口的寻址方案来唯一标识基于TCP的应用程序。 : TCP客户端应用程序的操作系统会自动为客户端分配端口号。客户端拥有该端口号，直到TCP连接终止为止，此后操作系统将收回该端口号。这些类型的端口称为临时端口。 : **DHCP, the Domain Name System \(DNS\), the Network Time Protocol \(NTP\), and Virtual extensible local area network \(VXLAN\)** are examples of UDP-based protocols used in OpenStack deployments.

UDP支持一对多通信：将单个数据包发送到多个主机。通过将接收方IP地址设置为特殊IP广播地址，应用程序可以向本地网络上的所有网络主机广播UDP数据包255.255.255.255。应用程序还可以使用IP多播将UDP数据包发送到一组接收器。预期的接收方应用程序通过将UDP套接字绑定到特殊IP地址（该有效IP地址是有效的多播组地址之一）来加入多播组。**接收主机不必与发送方位于同一本地网络上，但必须将中间路由器配置为支持IP多播路由。VXLAN是使用IP多播的基于UDP协议的示例。**

**1.2.7 网络组件**

**Switches** : 使数据包能够从一个节点传播到另一个节点。交换机连接属于同一第二层网络的主机，它们根据**数据包头中的目标以太网地址转发流量**。

**Routers** : 路由器使不同第3层网络上彼此不直接连接的两个节点之间能够进行通信。它们根据**数据包头中的目标IP地址路由**流量。

**Firewalls** : 防火墙用于调节与主机或网络之间的通信。防火墙可以是连接两个网络的专用设备，也可以是在操作系统上实现的基于软件的过滤机制。防火墙用于根据主机上定义的规则将流量限制到主机。他们可以根据几种标准（例如源IP地址，目标IP地址，端口号，连接状态等）过滤数据包。它主要用于保护主机免受未经授权的访问和恶意攻击。

**Load balancers** : 负载均衡器可以是基于软件的设备，也可以是基于硬件的设备，它们可以将流量平均分配到多个服务器上。通过在多台服务器之间分配流量，可以避免单台服务器过载，从而防止产品出现单点故障。这进一步提高了服务器的性能，网络吞吐量和响应时间。负载平衡器通常用于3层体系结构。

**1.2.8 Overlay \(tunnel\) protocols**

隧道传输是一种机制，可通过不兼容的传输网络实现有效载荷的传输。**它允许网络用户访问被拒绝或不安全的网络。**可以采用数据加密来传输有效负载，以确保封装的用户网络数据即使是私有的也可以轻松通过冲突的网络，也显示为公共的。 **Generic routing encapsulation \(GRE\) 通用路由封装** : 是一种在IP上运行的协议，当传递协议和有效负载协议兼容但有效负载地址不兼容时使用。

**Virtual extensible local area network \(VXLAN\) 虚拟可扩展局域网** : VXLAN的目的是提供可扩展的网络隔离。VXLAN是第3层网络上的第2层覆盖方案。它允许覆盖第2层网络分布在多个底层3层网络域中。每个覆盖称为VXLAN段。仅同一VXLAN网段内的VM可以通信。

**Generic Network Virtualization Encapsulation \(GENEVE\) 通用网络虚拟化封装** : Geneve旨在识别和适应网络虚拟化中不同设备不断变化的功能和需求。它提供了一个隧道框架，而不是对整个系统进行说明。Geneve灵活地定义了在封装过程中添加的元数据的内容，并尝试适应各种虚拟化方案。它使用UDP作为其传输协议，并使用可扩展的选项标头来动态调整大小。Geneve支持单播，多播和广播。

**1.2.9 Network namespaces**

**名称空间是确定一组特定标识符的一种方式。使用名称空间，您可以在不同的名称空间中多次使用相同的标识符。您还可以限制对特定进程可见的标识符集。**

 **Linux network namespaces** : 因此给定的网络设备（例如eth0）存在于特定的名称空间中。Linux会使用默认的网络名称空间启动，因此，如果您的操作系统没有做任何特别的事情，那么它将是所有网络设备所在的位置。但是也可以创建其他非默认名称空间，**并在这些名称空间中创建新设备**，或将现有设备从一个名称空间移动到另一个名称空间。

**每个网络名称空间也都有其自己的路由表，实际上，这是名称空间存在的主要原因**。路由表以目标IP地址为关键字，因此，如果您希望同一目标IP地址在不同时间表示不同的内容，则需要网络名称空间-**这是OpenStack Networking要求的，因为它具有在不同时间提供重叠IP地址的功能虚拟网络。**

每个网络名称空间还具有自己的一组iptables（用于IPv4和IPv6）。因此，您可以将不同的安全性应用于在不同名称空间和不同路由中具有相同IP地址的流。

**Virtual routing and forwarding \(VRF\)虚拟路由和转发** 虚拟路由和转发是一项IP技术，它允许路由表的多个实例同时在同一路由器上共存。它是上述网络名称空间功能的另一个名称。

**1.2.10 Network address translation解读**

网络地址转换（NAT）是一种在IP数据包传输过程中修改IP数据包标头中的源地址或目标地址的过程。通常，发送方和接收方应用程序不知道IP数据包正在被操纵。

**NAT通常由路由器实现**，因此我们将执行NAT的主机称为NAT路由器。但是，**在OpenStack部署中，通常是Linux服务器实现NAT功能**，而不是硬件路由器。**这些服务器使用 iptables 软件包来实现NAT功能**。

NAT有多种变体，在这里我们描述OpenStack部署中常见的三种。

**SNAT**

在源网络地址转换（SNAT）中，NAT路由器**在IP数据包中修改发送方的IP地址**。SNAT通常用于**私有地址的主机能够与公共Internet上的服务器进行通信**。

以OpenStack部署使用的形式，发送方和接收方之间路径上的NAT路由器将数据包的源IP地址替换为路由器的公共IP地址。**路由器还将源TCP或UDP端口修改为另一个值**，并且路由器会保留发送者的真实IP地址和端口以及修改后的IP地址和端口的记录。

由于NAT路由器会修改端口以及IP地址，因此这种形式的SNAT有时称为端口地址转换 （PAT）。有时也称为NAT重载。OpenStack使用SNAT使实例内部运行的应用程序可以连接到公共Internet。

**DNAT** : 在目标网络地址转换（DNAT）中，NAT路由器在IP数据包头中修改目标的IP地址。

: OpenStack使用DNAT将数据包从实例路由到OpenStack元数据服务。**在实例内部运行的应用程序通过向IP地址**为169.254.169.254的Web服务器发出HTTP GET请求来访问OpenStack元数据服务。在OpenStack部署中，没有主机具有该IP地址。相反，OpenStack使用DNAT更改这些数据包的目标IP，以便它们到达元数据服务正在侦听的网络接口。

**One-to-one NAT** : 在一对一NAT中，NAT路由器在私有IP地址和公共IP地址之间维护一对一映射。OpenStack使用一对一NAT来实现浮动IP地址。

**1.2.11 OpenStack Networking**

代号为**neutron**的网络服务提供了一个API，可让您定义云中的网络连接和寻址。网络服务使运营商可以利用不同的网络技术来为其云网络提供支持.neutron提供了一个API，用于配置和管理从L3转发和网络地址转换（NAT）到负载平衡，外围防火墙和虚拟专用网络的各种网络服务。

It includes the following components:

**API server** : OpenStack网络API包括对第2层网络和IP地址管理（IPAM）的支持，以及对第3层路由器构造的扩展，该构造使第2层网络与网关之间可以路由到外部网络。OpenStack Networking包括越来越多的插件，这些插件可实现与各种商业和开源网络技术的互操作性，包括路由器，交换机，虚拟交换机和软件定义的网络（SDN）控制器。

**OpenStack Networking plug-in and agents插件和代理** : 插入和拔出端口，创建网络或子网，并提供IP寻址。根据特定云中使用的供应商和技术，所选的插件和代理会有所不同。值得一提的是，一次只能使用一个插件。

**Messaging queue消息队列** 消息传递指的是程序之间通过在消息中发送数据进行通信，而不是通过直接调用彼此来通信 : Accepts and routes RPC requests between agents **to complete API operations**,

"Message queue" is used in the "ML2 plug-in" for "RPC" between "the neutron server and neutron agents" that run on "each hypervisor", in the "ML2 mechanism drivers" for "Open vSwitch and Linux bridge".

在agents之间接受并路由RPC请求以完成API操作。 Message queue是用于Open vSwitch and Linux bridge的ML2机制驱动程序中的the neutron server和在每个虚拟机管理程序上运行的neutron agents之间的RPC的ML2插件中使用。

## 概念

要配置丰富的网络拓扑，**您可以创建和配置网络和子网，并指示其他OpenStack服务（如Compute）将虚拟设备连接到这些网络上的端口。** 网络有两种类型，project and provider networks。作为网络创建过程的一部分，可以在项目之间共享任何类型的网络。

## Provider networks

Provider networks通过对DHCP和元数据服务的可选支持，**提供了到实例的第二层连接。这些网络通常使用VLAN（802.1q）标记来识别或分隔它们，以连接或映射到数据中心中的现有第2层网络。**

Provider networks 通常以灵活性为代价提供简单性，性能和可靠性。**默认情况下，只有管理员才能创建或更新provider networks，因为他们需要配置物理网络基础结构。**可以使用以下参数更改允许创建或更新提供商网络的用户 policy.json

## Self-service networks

自助服务网络主要使一般（非特权）项目可以管理网络，而无需管理员参与。**这些网络完全是虚拟的，需要虚拟路由器与提供商和外部网络（例如Internet）进行交互**。自助服务网络通常还为实例提供DHCP和元数据服务。

在大多数情况下，自助服务网络使用**VXLAN**或**GRE**之类的覆盖协议

IPv4自助服务网络通常使用**私有地址**范围（RFC1918），并通过虚拟路由器上的源NAT与提供商网络进行交互（这里是与物理网络交互？）。浮动IP地址允许通过虚拟路由器上的目标NAT从提供商网络访问实例。

**网络服务使用通常驻留至少一个网络节点的第3层代理来实现路由器。** **自助服务网络必须遍历第3层代理**。因此，第3层代理或网络节点的超额预订或故障可能会影响大量自助服务网络和使用它们的实例。**考虑实现一项或多项高可用性功能，以提高自助服务网络的冗余性和性能。**

用户创建项目网络以实现项目内的连接。默认情况下，它们是完全隔离的，不会与其他项目共享。OpenStack Networking支持以下类型的网络隔离和覆盖技术。 **平面** : 所有实例都位于同一网络上，也可以与主机共享。没有VLAN标记或其他网络隔离发生。

**虚拟局域网** : 通过联网，用户可以使用与物理网络中存在的VLAN相对应的VLAN ID（标记为802.1Q）创建多个提供商或项目网络。这允许实例在整个环境中相互通信。他们还可以与同一第2层VLAN上的专用服务器，防火墙，负载平衡器和其他网络基础结构进行通信。

**GRE和VXLAN** : VXLAN和GRE是封装协议，它们创建覆盖网络来激活和控制计算实例之间的通信。需要使用网络路由器，以允许流量流向GRE或VXLAN项目网络之外。还需要路由器将直接连接的项目网络与包括Internet在内的外部网络连接。路由器提供了使用浮动IP地址直接从外部网络连接到实例的功能。

路由器¶ 路由器在自助服务和提供者网络之间或属于项目的自助服务网络之间提供虚拟第3层服务，例如路由和NAT。**网络服务使用第3层代理程序通过名称空间管理路由器。**

安全组¶ 安全组为虚拟防火墙规则提供了一个容器，用于在端口级别控制入口（实例的入站）和出口（实例的出站）网络流量。安全组使用默认的拒绝策略，并且仅包含允许特定流量的规则。每个端口可以加法引用一个或多个安全组。防火墙驱动程序将安全组规则转换为基础数据包筛选技术（例如）的配置iptables。

每个项目都包含一个default安全组，该安全组允许所有出口流量并拒绝所有入口流量。您可以在default安全组中更改规则 。如果在不指定安全组的情况下启动实例，则该default安全组将自动应用于该实例。同样，如果您在未指定安全组的情况下创建端口，则该default安全组将自动应用于该端口 。

还有一些其他概念 [https://docs.openstack.org/neutron/stein/admin/intro-os-networking.html](https://docs.openstack.org/neutron/stein/admin/intro-os-networking.html)

**1.3. OpenStack 中的网络** : 以上概念在 OpenStack 中完全适用，只是在 OpenStack 中被称为**软件定义的网络**（SDN,Software Defined Network）。**虚拟交换机**（使用 Open vSwitch）和**虚拟路由器**（l3-agent,小写的L）提供了在实例间进行通讯的功能，并可以为实例提供与外部网络进行通讯的能力。Open vSwitch 网桥会为实例分配虚拟端口，并可以使入站和出站的网络数据分散到相关的物理网络中。

**1.4.高级openstack网络概念**

1.4.1.第3层高可用性 : openstack networking 在专门用于运行虚拟网络组件的物理服务器上运行虚拟路由器。通过实现业界标准的VRRP协议来保护**虚拟路由器和浮动ip地址**。在启用第3层高可用性功能后，租户的虚拟路由器会**被随机地在多个网络节点**上进行分配，其中一个路由器会被指定为"活跃的"路由器，其他路由器作为备份路由器。当运行活跃路由器的网络节点出现故障时，备份路由器会作为活跃路由器使用。 如需了解更多相关信息，请参阅"配置第 3 层高可用性"一章。

1.4.2.负载均衡既服务LBaaS 负载均衡即服务（Load Balancing-as-a-Service，简称 LBaaS）使 OpenStack Networking 可以在相关的实例中平均分配入站的网络流量。 1.4.3IPv6 1.4.4.CIDR格式 常规形式：子网地址的传统表示方法是使用网络地址和子网掩码。例如： 网络地址：192.168.100.0 子网掩码：255.255.255.0 CIDR 格式：这种格式把子网掩码缩短为使用实际的位数进行代表。例如，192.168.100.0/24。其中的 /24 代表了 255.255.255.0（这个子网掩码的二进制形式共有 24 位 1）。在 ifcfg-xxx 脚本中可以使用 CIDR 格式来替代 NETMASK 值： `#NETMASK=255.255.255.0 PREFIX=24`

## 未整理

**Modular Layer 2（ML2）** ML2是OpenStack Havana发行版中引入的OpenStack Networking核心插件。取代以前的单片插件模型，ML2模块化设计支持混合网络技术的并发运行。单一的Open vSwitch和Linux Bridge插件已被弃用并删除；它们的功能现在由ML2机制驱动程序实现。 **ML2 网络类型** Enable Type drivers in the ML2 section of the ml2\_conf.ini file. For example:

```text
[ml2]
type_drivers = local,flat,vlan,gre,vxlan,geneve
```

**L2 Population** L2 Population 驱动支持使用广播、多播和单播方式在覆盖网络中进行扩展。在默认情况下，Open vSwitch GRE 和 VXLAN 会复制广播到包括那些没有运行目标网络的每个代理。这种设计导致了大量的网络和处理负载。而 L2 Population 驱动使用了另外一种设计观念，它为 ARP 解析以及 MAC 地址学习功能提供了一个部分的网状网络（mesh），网络数据会被打包作为目标单波只发送到相关的代理上。为了启用 L2 population，您需要把它添加到机制驱动列表中。另外，您还需要最少启用一个隧道（tunneling）驱动（GRE、VXLAN 或两个都启用）。在 ml2\_conf.ini 文件中添加适当的配置：

\[ml2\] type\_drivers = local,flat,vlan,gre,vxlan mechanism\_drivers = openvswitch,linuxbridge,l2population 在 ovs\_neutron\_plugin.ini 文件中启用 L2 population。这需要在所有运行 L2 agent 的节点上都启用：

\[agent\] l2\_population = True



## Openstack的网络模式有5种 

Local模式：一般测试时使用，只需一台物理机即可。 

GRE模式：隧道模式， VLAN数量没有限制，性能有点问题。 

Vlan模式：vlan数量有4096的限制 

VXlan模式：vlan数量没有限制，性能比GRE好。 

Flat模式：管理员创建，租户直接到外网的一种网络模式，不需要NAT。 

