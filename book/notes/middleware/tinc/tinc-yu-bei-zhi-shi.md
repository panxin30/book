# tinc 预备知识

## 预备知识 <a href="yu-bei-zhi-shi" id="yu-bei-zhi-shi"></a>

tinc 的几大的特点：&#x20;

加密/认证/压缩&#x20;

自动全网状路由&#x20;

易于扩展网络节点&#x20;

能够进行网络的桥接&#x20;

跨平台与IPv6 支持

* 每个 tinc VPN 必须有个名称，一个 VPN 可以包括很多主机；
* 每台主机必须有个名称，同时需要运行 tinc，一台主机可以通过运行多个 tinc 实例来加入多个 tinc VPN；
* tinc 启动时接受参数来指定要启动的网络，如**home\_vpn**，并定位到对应的网络配置目录**home\_vpn**读取配置；
* 启动后，读取**网络配置目录home\_vpn**中的主配置文件 **tinc.conf**，执行启动脚本（tinc-up），然后 ConnectTo 指定的主机，同时接受其他主机对本机的 ConnetTo；
* tinc 通过读取**/etc/tinc/home\_vpn/hosts目录下**主机描述文件来获得主机信息，当前主机和 ConnectTo 的目标主机上都必须有双方的主机描述文件；
* ConnectTo 成功（认证通过），则加入 tinc VPN；
* 当 tinc 结束的时候，执行关闭脚本（tinc-down）；

**tinc VPN 名称接受 a-z 0-9 \_ 中的字符，主机名称也是一样。**

## 配置文件说明 <a href="ru-he-pei-zhi-he-jiao-huan-mi-yao" id="ru-he-pei-zhi-he-jiao-huan-mi-yao"></a>

这是笔记本上的tinc配置

```
/etc/tinc                (主配置目录)
│
├── home_vpn            （home_vpn的网络配置目录，目录名和网络名保持一致。）
│   ├── hosts
│   │   ├── server    （主机 server 的描述文件）
│   │   └── notebook    （主机 notebook 的描述文件）
│   ├── rsa_key.priv    （本主机的 RSA 私钥）
│   ├── tinc.conf       （tinc 主配置文件）
│   ├── tinc-down       （当关闭 home_vpn 时，执行该脚本）
│   └── tinc-up            （当启动 home_vpn 时，执行该脚本）
/etc/tinc/home_vpn 目录下的文件都属于home_vpn这个网络
/etc/tinc/home_vpn/hosts 目录是存放其他用户或者说是其他网络的public key以及他们的 ip 地址
rsa_key.priv 本网络的私钥
tinc.conf 本网络的配置文件
tinc-down 本网络关闭时执行的脚本
tinc-up 本网络启动时执行的脚本
```

## **服务端配置/**核心主机配置

这台核心主机的 tinc 作为服务自动启动，启动后等待其他主机来连接，其他所有的主机都配置为主动连接核心主机。

|      |                     |                     |
| ---- | ------------------- | ------------------- |
| 操作系统 | Ubuntu Server 18.04 | Ubuntu Server 18.04 |
| 公网IP |                     |                     |
| 内网IP |                     |                     |
|      |                     |                     |
|      |                     |                     |

\
首先开启 Linux 转发，在/etc/sysctl.conf设置net.ipv4.ip_forward = 1，并通过sysctl -p来应用配置。????_\
__\
_修改tinc.conf配置文件_\
_Name = Server_\
_Interface = tinc_\
_Mode = switch_\
_Compression=9_\
_PrivateKeyFile=/etc/tinc/home\_vpn_/rsa\_key.priv\
\
Name 主机名称\
Interface 隧道所使用的网卡名称\
Mode 有三种模式，分别是 router / switch / hub ，相对应我们平时使用到的路由、交换机、集线器 (默认模式 router)\
Compression UDP 数据包压缩级别。可选有 0 (关闭)，1 (fast zlib) 至 9 (best zlib)，10 (fast lzo) 和 11 (best lzo)\
PrivateKeyFile 服务器私钥的位置\
\
\
网络名就是freeoa，网卡接口名vpn，使用switch模式，以及如下的加密方式：\
Name=freeoa\
Interface=vpn\
Mode=switch\
Cipher=aes-256-cbc\
Digest=sha256\
\
新建hosts文件，这个文件名必须和tinc.conf内的Name一致：\
vim /etc/tinc/freeoa/hosts/freeoa\
\
写入如下配置：\
Address = VPS公网IP\
Subnet = 10.0.0.1/32\
\
完成之后生成密匙对：\
tincd -n freeoa -K4096\
\
按两下回车全部保持默认配置，生成完成之后，对应的文件路径：\
/etc/tinc/freeoa/rsa\_key.priv # 私钥\
/etc/tinc/freeoa/hosts/freeoa # 公钥\
\
配置虚拟网卡tinc-up：\
vim /etc/tinc/freeoa/tinc-up\
\
\#!/bin/sh\
ifconfig $INTERFACE 10.0.0.1 netmask 255.255.255.0\
\
ip指令\
ip link set $INTERFACE up\
ip addr add 10.0.0.1/32 dev $INTERFACE\
ip route add 10.0.0.1/32 via 10.0.0.1\
\
配置虚拟网卡tinc-down：\
vim /etc/tinc/freeoa/tinc-down\
\
\#!/bin/sh\
ifconfig $INTERFACE down\
\
ip指令\
ip link set $INTERFACE down\
ip route del 10.0.0.1/32 via 10.0.0.1\
\
给执行权限：\
chmod +x /etc/tinc/freeoa/tinc-\*

## 客户端配置

\
客户端的tinc.conf与服务器的参数基本上相同，只需要修改Name\
\
在hosts文件夹内添加新的节点配置文件\
Subnet=10.0.0.2/32\
\
tinc-up和tinc-down跟服务器配置基本一样，只需要修改tinc-up的内网ip，Windows客户端无需这两个文件\
\
生成私钥和公钥\
tincd -n freeoa -K4096\
\
将服务端的节点配置文件放到客户端的hosts文件夹内，并将客户端的节点配置文件放到服务端的hosts文件夹内\
\
\
同样和之前一样新建一个freeoa的目录以及hosts目录：\
mkdir -p /etc/tinc/freeoa && mkdir -p /etc/tinc/freeoa/hosts\
\
新建tinc.conf配置文件：\
vim /etc/tinc/freeoa/tinc.conf\
\
写入如下配置，其中需要注意的是ConnectTo的值需要指定为服务端的网络名称，如果一样配置了加密，那么加密方式也需要和服务端对应：\
Name=node\_us\
Interface=vpn\
Mode=switch\
ConnectTo=freeoa\
Cipher=aes-256-cbc\
Digest=sha256\
\
新建hosts文件：\
vim /etc/tinc/freeoa/hosts/node\_us\
\
Subnet = 10.0.0.3/32\
\
生成密匙对：\
tincd -n freeoa -K4096\
\
配置虚拟网卡tinc-up：\
vim /etc/tinc/freeoa/tinc-up\
\
\#!/bin/sh\
ifconfig $INTERFACE 10.0.0.3 netmask 255.255.255.0\
\
配置虚拟网卡tinc-down：\
vim /etc/tinc/freeoa/tinc-down\
\
\#!/bin/sh\
ifconfig $INTERFACE down\
\
给执行权限：\
chmod +x /etc/tinc/freeoa/tinc-\*\
\
现在我们需要交换公钥，首先把服务端的公钥复制到客户端内，再把客户端的公钥复制到服务端内，这样两台Linux之间就实现了内网互通。\
\
接下来最主要的是如何把TincVPN放到Windows上运行(当作客户端)。其实配置起来大同小异，主要是参考了[官方文档](https://www.tinc-vpn.org/examples/windows-install/)。\
\
安装的时候一定要勾选TAP-Win64：进到安装软件的根目录，打开tap-win64目录，用管理员权限运行addtap.bat。\
\
安装驱动之后打开网络连接看看有没有新增加一个TAP设备，回到软件根目录，新建一个网络(文件夹)我这边建立文件夹名称为freeoa，然后在freeoa这个文件夹内再新建一个hosts文件夹。接着在freeoa这个文件夹内新建一个tinc.conf，配置内容如下：\
Name=node\_family\
Interface=vpn\
Mode=switch\
ConnectTo=freeoa\
Cipher=aes-256-cbc\
Digest=sha256\
\
在hosts文件夹内新建一个node\_family文件(名字必须和tinc.conf内的Name一致)写入如下配置：\
Subnet = 10.0.0.4/32\
\
之后用管理员权限打开CMD或者PowerShell，进入到软件根目录，生成密匙对：\
cd tinc\
./tincd -n freeoa -K4096\
\
最后和服务端(VPS)交换公钥，也就是把node\_family公钥文件上传到VPS的/etc/tinc/freeoa/hosts目录。同理，服务端上的freeoa公钥文件下载到本地的hosts文件夹下即可，文件夹下的结构与Linux上的一致即可。\
\
hosts文件夹下就是服务端的公钥文件/客户端公钥文件，注意到我们之前在tinc.conf配置的虚拟网卡接口名是：\
Interface=vpn\
\
现在需要打开Windows的网络连接界面，把TAP-Win32 Adapter V9这个设备的名称改为vpn，接着为这个设备配置IP，配置的IP必须和node\_family文件内的保持一致，创建服务：\
tincd -n freeoa\
\
最后在Windows的计算机管理，启动服务。
