---
description: k8s1.20后使用runtime做为pod的运行
---

# 单控制节点集群v1.23未验证通过

参考：[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/)

## 环境

ubuntu 20.04,k8s 1.23, runtime,

## 部署

### 一、Creating a cluster with kubeadm

### 1. Installing kubeadm on your hosts <a href="#installing-kubeadm-on-your-hosts" id="installing-kubeadm-on-your-hosts"></a>

**详细安装方法见第二条。**

### **2.** To initialize the control-plane node run:

使用kubeadm config print输出默认的配置文件，然后修改配置文件内容

kubeadm config print init-defaults > kubeadm-config.yaml

编辑配置文件kubeadm-config.yaml文件

```bash
kubeadm init <args>
# To make kubectl work for your non-root user, run these commands, which are also part of the kubeadm init output:
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### 错误一、kubeadm init，初始化的时候没有带\``--pod-network-cidr=192.168.0.0/16`\`

部署flannel的时候报错，根据提示就是没有分配cidr

`E0212 02:26:07.203360 1 main.go:325] Error registering network: failed to acquire lease: node "cn-office-tonytest-k8s-01" pod cidr not assigned`

**vim /etc/kubernetes/manifests/kube-controller-manager.yaml**\
**增加参数：**

```
--allocate-node-cidrs=true
--cluster-cidr=10.244.0.0/16
```

### 3. Installing a Pod network add-on

For Kubernetes v1.17+ `kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml`

### 4. Control plane node isolation

### 5. Joining your nodes <a href="#join-nodes" id="join-nodes"></a>

* ```bash
  kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
  ```

If you do not have the token, you can get it by running the following command on the control-plane node:

```bash
kubeadm token list
```

### 二、Installing kubeadm

参考：[https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)

### **1.Letting iptables see bridged traffic**

Make sure that the `br_netfilter` module is loaded. This can be done by running `lsmod | grep br_netfilter`. To load it explicitly call `sudo modprobe br_netfilter`.

As a requirement for your Linux Node's iptables to correctly see bridged traffic, you should ensure `net.bridge.bridge-nf-call-iptables` is set to 1 in your `sysctl` config, e.g.

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

### 2.Installing runtime

参考：[https://kubernetes.io/docs/setup/production-environment/container-runtimes/](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

**详细安装见第三条Container runtimes**

### 3. Installing kubeadm, kubelet and kubectl

1.  Update the `apt` package index and install packages needed to use the Kubernetes `apt` repository:

    ```shell
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl
    ```
2.  Download the Google Cloud public signing key:

    ```shell
    sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg
    ```
3.  Add the Kubernetes `apt` repository:

    ```shell
    echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
    ```
4.  Update `apt` package index, install kubelet, kubeadm and kubectl, and pin their version:

    ```shell
    sudo apt update
    sudo apt install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl
    ```



### 三、Container runtimes

### 1.有四个选项，这里选择安装**containerd**

* [containerd](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)
* [CRI-O](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-o)
* [Docker Engine](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#docker)
* [Mirantis Container Runtime](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#mcr)

### 2. 选择Cgroup drivers

参考：[https://kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/](https://kubernetes.io/zh/docs/tasks/administer-cluster/kubeadm/configure-cgroup-driver/)

由于 kubeadm 把 kubelet 视为一个系统服务来管理，所以对基于 kubeadm 的安装， 我们推荐使用 `systemd` 驱动，不推荐 `cgroupfs` 驱动。

在版本 1.22 及以后，如果用户没有在 `KubeletConfiguration` 中设置 `cgroupDriver` 字段， `kubeadm init` 会将它设置为默认值 `systemd`。

### 3.可选：Cgroup v2[ ](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cgroup-v2)(最终部署没有选择，作为了解)

Cgroup v2 是 cgroup Linux API 的下一个版本。与 cgroup v1 不同的是，每个控制器都有一个层次结构而不是不同的层次结构。

新版本对 cgroup v1 进行了多项改进，其中一些改进包括：

* 更干净，更易于使用的 API
* 安全的子树委托给容器
* 压力失速信息等新功能

### 4. CRI version support[ ](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#cri-versions)

Your container runtime must support at least v1alpha2 of the container runtime interface.

Kubernetes 1.23 defaults to using v1 of the CRI API. If a container runtime does not support the v1 API, the kubelet falls back to using the (deprecated) v1alpha2 API instead.

### 5. containerd[ ](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd)

This section contains the necessary steps to use containerd as CRI runtime.

Use the following commands to install Containerd on your system:

#### **Install and configure prerequisites**:

```shell
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

#### **Install containerd**

1. Install the `containerd.io` package from the official Docker repositories. Instructions for setting up the Docker repository for your respective Linux distribution and installing the `containerd.io` package can be found at [Install Docker Engine](https://docs.docker.com/engine/install/#server).
2.  Update the `apt` package index and install packages to allow `apt` to use a repository over HTTPS:

    ```
    sudo apt update

    sudo apt install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```
3.  Add Docker’s official GPG key:

    ```
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```
4.  Use the following command to set up the **stable** repository. To add the **nightly** or **test** repository, add the word `nightly` or `test` (or both) after the word `stable` in the commands below. [Learn about **nightly** and **test** channels](https://docs.docker.com/engine/install/).

    ```
      echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```
5.

    ```
     sudo apt update
     sudo apt install containerd.io
    ```
6.  Configure containerd:

    ```shell
    sudo mkdir -p /etc/containerd
    containerd config default | sudo tee /etc/containerd/config.toml
    ```
7.  Restart containerd:

    ```shell
    sudo systemctl restart containerd
    ```

#### **Using the `systemd` cgroup driver**[ **** ](https://kubernetes.io/docs/setup/production-environment/container-runtimes/#containerd-systemd)

To use the `systemd` cgroup driver in `/etc/containerd/config.toml` with `runc`, set

```
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
  ...
  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
```

If you apply this change make sure to restart containerd again:

```shell
sudo systemctl restart containerd
```







