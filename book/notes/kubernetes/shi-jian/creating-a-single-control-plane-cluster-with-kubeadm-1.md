---
description: k8s1.20后使用runtime做为pod的运行
---

# Creating a single control-plane cluster with kubeadm

参考：[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

## 环境

ubuntu 18.04,k8s 1.22, runtime

## 部署

### 一、Installing kubeadm

参考：[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

#### **1.Letting iptables see bridged traffic**

Make sure that the `br_netfilter` module is loaded. This can be done by running `lsmod | grep br_netfilter`. 

#### 2.Installing runtime

参考：[https://kubernetes.io/docs/setup/production-environment/container-runtimes/\#containerd](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

This section contains the necessary steps to use containerd as CRI runtime.

Use the following commands to install Containerd on your system:

Install and configure prerequisites:

```text
cat <<EOF | sudo tee /etc/modules-load.d/containerd.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```



#### 3.Installing kubeadm, kubelet and kubectl

#### 4.Configuring a cgroup driver[ ](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configuring-a-cgroup-driver)

Both the container runtime and the kubelet have a property called ["cgroup driver"](https://kubernetes.io/docs/setup/production-environment/container-runtimes/), which is important for the management of cgroups on Linux machines.

>

