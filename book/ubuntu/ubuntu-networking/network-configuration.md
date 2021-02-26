---
description: '引用：https://help.ubuntu.com/18.04/serverguide/network-configuration.html'
---

# Network Configuration

## 一、Ethernet Interfaces

#### Identify Ethernet Interfaces识别以太网接口

To quickly identify all available Ethernet interfaces

```text
ip a
```

Another application that can help identify all network interfaces available to your system is the **lshw** command. This command provides greater details around the **hardware** capabilities of specific adapters. In the example below, lshw shows a single Ethernet interface with the logical name of eth0 along with bus information, driver details and all supported capabilities.

```text
sudo lshw -class network
```

#### Ethernet Interface Logical Names

Interface logical names can also be configured via a netplan configuration. If you would like control which interface receives a particular logical name use the _**match**_ and _**set-name**_ keys. The match key is used to find an adapter based on some criteria like MAC address, driver, etc. Then the set-name key can be used to change the device to the desired logial name.

```text
network:
  version: 2
  renderer: networkd
  ethernets:
    eth_lan0:
      dhcp4: true
	  match:
	    macaddress: 00:11:22:33:44:55
	  set-name: eth_lan0
```

#### Ethernet Interface Settings

_**ethtool**_ is a program that displays and changes Ethernet card settings such as auto-negotiation, port speed, duplex mode, and Wake-on-LAN. The following is an example of how to view supported features and configured settings of an Ethernet interface.

```text
sudo ethtool eth4(Ethernet card)
```

## 二、IP Addressing

The following section describes the process of configuring your systems IP address and default gateway needed for communicating on a local area network and the Internet.

#### Temporary IP Address Assignment

For temporary network configurations, you can use the ip command which is also found on most other GNU/Linux operating systems. The ip command allows you to configure settings which take effect immediately, however they are not persistent and will be lost after a reboot.

To temporarily configure an IP address, you can use the ip command in the following manner. Modify the IP address and subnet mask to match your network requirements.

```text
sudo ip addr add 10.102.66.200/24 dev enp0s25
```

The ip can then be used to set the link up or down.

```text
ip link set dev enp0s25 up
ip link set dev enp0s25 down
```

To verify the IP address configuration of enp0s25, you can use the ip command in the following manner.

```text
ip address show dev enp0s25
10: enp0s25: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:e2:52:42 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.102.66.200/24 brd 10.102.66.255 scope global dynamic eth0
       valid_lft 2857sec preferred_lft 2857sec
    inet6 fe80::216:3eff:fee2:5242/64 scope link
       valid_lft forever preferred_lft forever6
```

**To configure a default gateway**, you can use the ip command in the following manner. Modify the default gateway address to match your network requirements.

```text
sudo ip route add default via 10.102.66.1
```

To verify your default gateway configuration, you can use the ip command in the following manner.

```text
ip route show
default via 10.102.66.1 dev eth0 proto dhcp src 10.102.66.200 metric 100
10.102.66.0/24 dev eth0 proto kernel scope link src 10.102.66.200
10.102.66.1 dev eth0 proto dhcp scope link src 10.102.66.200 metric 100 
```

If you require DNS for your temporary network configuration, you can add DNS server IP addresses in the file /etc/resolv.conf. In general, editing /etc/resolv.conf directly is not recommanded, but this is a temporary and non-persistent configuration. The example below shows how to enter two DNS servers to /etc/resolv.conf, which should be changed to servers appropriate for your network. A more lengthy description of the proper persistent way to do DNS client configuration is in a following section.

```text
nameserver 8.8.8.8
nameserver 8.8.4.4
```

If you no longer need this configuration and wish to purge all IP configuration from an interface, you can use the ip command with the flush option as shown below.

```text
ip addr flush eth0
```

Flushing the IP configuration using the ip command does not clear the contents of /etc/resolv.conf. You must remove or modify those entries manually, or re-boot which should also cause /etc/resolv.conf, which is a symlink to /run/systemd/resolve/stub-resolv.conf, to be re-written.

#### Static IP Address Assignment静态地址分配

To configure your system to use static address assignment, create a netplan configuration in the file /etc/netplan/99\_config.yaml. The example below assumes you are configuring your first Ethernet interface identified as eth0. Change the addresses, gateway4, and nameservers values to meet the requirements of your network.

```text
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 10.10.10.2/24
      gateway4: 10.10.10.1
      nameservers:
          search: [mydomain, otherdomain]
          addresses: [10.10.10.1, 1.1.1.1]
```

The configuration can then be applied using the netplan command.

```text
sudo netplan apply
```

## 三、Name Resolution

## 四、Bridging

Bridging multiple interfaces is a more advanced configuration, but is very useful in multiple scenarios. One scenario is setting up a bridge with multiple network interfaces, then using a firewall to filter traffic between two network segments. Another scenario is using bridge on a system with one interface to allow virtual machines direct access to the outside network. The following example covers the latter scenario.

Configure the bridge by editing your netplan configuration found in /etc/netplan/:

```text
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: no
  bridges:
    br0:
      dhcp4: yes
      interfaces:
        - enp3s0
```

Enter the appropriate values for your physical interface and network.

Now apply the configuration to enable the bridge:

```text
sudo netplan apply
```

The new bridge interface should now be up and running. The brctl provides useful information about the state of the bridge, controls which interfaces are part of the bridge, etc. See man brctl for more information.

## 五、Resources



