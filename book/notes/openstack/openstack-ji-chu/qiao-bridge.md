# 桥bridge

引用: [https://www.ibm.com/developerworks/cn/linux/1310\_xiawc\_networkdevice/](https://www.ibm.com/developerworks/cn/linux/1310_xiawc_networkdevice/)

引用: [https://blog.csdn.net/sld880311/article/details/77840343?ops\_request\_misc=%257B%2522request%255Fid%2522%253A%2522161473663516780357211805%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request\_id=161473663516780357211805&biz\_id=0&utm\_medium=distribute.pc\_search\_result.none-task-blog-2~all~sobaiduend~default-1-77840343.first\_rank\_v2\_pc\_rank\_v29&utm\_term=linux+bridge](https://blog.csdn.net/sld880311/article/details/77840343?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161473663516780357211805%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161473663516780357211805&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-77840343.first_rank_v2_pc_rank_v29&utm_term=linux+bridge)

## 基本概念

    bridge是一个虚拟网络设备，具有网络设备的特性（可以配置IP、MAC地址等）；**而且bridge还是一个虚拟交换机**，和物理交换机设备功能类似。**bridge是一种在链路层实现中继，对帧进行转发的技术，根据MAC分区块，可隔离碰撞，将网络的多个网段在数据链路层连接起来的网络设备。**

    对于普通的物理设备来说，只有两端，从一端进来的数据会从另一端出去，比如物理网卡从外面网络中收到的数据会转发到内核协议栈中，而从协议栈过来的数据会转发到外面的物理网络中。**而bridge不同，bridge有多个端口，数据可以从任何端口进来，进来之后从哪个口出去原理与物理交换机类似，需要看mac地址。**

    Bridge 设备实例可以和 Linux 上其他网络设备实例连接（物理设备、虚拟设备、vlan设备等，即attach一个从设备，**类似于现实世界中的交换机和一个用户终端之间连接了一根网线**），

    并且可以为bridge配置一个IP（参考LinuxBridge MAC地址行为），这样该主机就可以通过这个bridge设备与网络中的其他主机进行通信了。

    当有数据到达时，Bridge 会根据报文中的 MAC 信息进行广播、转发、丢弃处理。另外它的从设备被虚拟化为端口port，它们的IP及MAC都不再可用，且它们被设置为接受任何包，最终由bridge设备来决定数据包的去向：接收到本机、转发、丢弃、广播。

## **通过Linux bridge来实现打通容器网络**

**vm：** 虚拟机通过tun/tap或者其它类似的虚拟网络设备，将虚拟机内的网卡同br0连接起来，这样就达到了和真实交换机一样的效果，虚拟机发出去的数据包先到达br0,然后由br0交给eth0\(物理机网卡\)发送出去，数据包都不需要经过host机器的协议栈

**docker：** 由于容器运行在自己单独的network namespace，所以都有自己单独的协议栈 容器中配置网关88.1,发出去的数据包先到达br0,然后交给host机器的协议栈，由于目的IP是外网IP，且host机器开启了IP forward转发功能，于是数据包会通过eth0发送出去，由于88.1是内网ip，所以发出去之前会做NAT转换\(NAT转换和IP forward功能需要自己配置\)。 性能没有上面的好，但是处于内网中，安全性相对高点。

## 工作原理

    bridge和route比较相似，都可以用来分发网络数据包，它们的本质不同在于：**route在L3网络层，使用路由协议、bridge在L2数据链路层，通过学习和缓存在链路上传输的数据包中的源地址以及物理层的输入端口**： 

* 收到新数据包时，记录源MAC地址和输入端口
* 根据数据包中的目的MAC地址查找本地缓存，如果能找到对应的MAC地址记录
* 若发现记录不在本地网络，直接丢弃数据包
* 若发现记录存在对应的端口，则将数据包直接从该端口转发出去
* 如果本地缓存中不存在任何记录，则在本网段中进行广播。



   

