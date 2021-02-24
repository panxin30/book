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
  *-network
       description: Ethernet interface
       product: MT26448 [ConnectX EN 10GigE, PCIe 2.0 5GT/s]
       vendor: Mellanox Technologies
       physical id: 0
       bus info: pci@0004:01:00.0
       logical name: eth4
       version: b0
       serial: e4:1d:2d:67:83:56
       slot: U78CB.001.WZS09KB-P1-C6-T1
       size: 10Gbit/s
       capacity: 10Gbit/s
       width: 64 bits
       clock: 33MHz
       capabilities: pm vpd msix pciexpress bus_master cap_list ethernet physical fibre 10000bt-fd
       configuration: autonegotiation=off broadcast=yes driver=mlx4_en driverversion=4.0-0 duplex=full firmware=2.9.1326 ip=192.168.1.1 latency=0 link=yes multicast=yes port=fibre speed=10Gbit/s
       resources: iomemory:24000-23fff irq:481 memory:3fe200000000-3fe2000fffff memory:240000000000-240007ffffff
```

#### Ethernet Interface Logical Names

Interface logical names can also be configured via a netplan configuration. If you would like control which interface receives a particular logical name use the _match_ and _set-name_ keys. The match key is used to find an adapter based on some criteria like MAC address, driver, etc. Then the set-name key can be used to change the device to the desired logial name.

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

## 二、IP Addressing

## 三、Name Resolution

## 四、Bridging

## 五、Resources

```
$ give me super-powers 
```



