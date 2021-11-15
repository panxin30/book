# IP

**ifconfig 已经失去维护，用 ip 替代。**

比如 ip addr 就能显示自己的IP地址。除此之外，创建网桥也能用 ip ，如下：\
sudo ip link add br0 type bridge\
ip a\
提示你已经创建了 br0 网桥。 tun/tap 也能用 ip 创建。 ArchLinux 是个现代化发行版，更新非常激进，默认是没有 ifconfig 的。

