---
description: k8s1.20后使用runtime做为pod的运行
---

# Creating a single control-plane cluster with kubeadm v1.22

参考：[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

## 环境

ubuntu 18.04,k8s 1.22, runtime

## 部署

### 一、Installing kubeadm

参考：[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

#### **1.Letting iptables see bridged traffic**

Make sure that the `br_netfilter` module is loaded. This can be done by running `lsmod | grep br_netfilter`.&#x20;

#### 2.Installing runtime

参考：[https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

This section contains the necessary steps to use containerd as CRI runtime.

Use the following commands to install Containerd on your system:

**Install and configure prerequisites:**

```
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

**Install containerd:**

1. Install the `containerd.io` package from the official Docker repositories. Instructions for setting up the Docker repository for your respective Linux distribution and installing the `containerd.io` package can be found at [Install Docker Engine](https://docs.docker.com/engine/install/#server).  (因为系统是ubuntu，所以点击ubuntu选项)
2.  Configure containerd:

    ```
    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    ```
3.  Restart containerd:

    ```
    sudo systemctl restart containerd
    ```

#### 3.Installing kubeadm, kubelet and kubectl

#### 4.Configuring a cgroup driver[ ](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#configuring-a-cgroup-driver)

Both the container runtime and the kubelet have a property called ["cgroup driver"](https://kubernetes.io/docs/setup/production-environment/container-runtimes/), which is important for the management of cgroups on Linux machines.

>
