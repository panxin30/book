# Page 1

## 环境

ubuntu 20.04,k8s 1.23.3,docker 20.10.12 数据盘使用xfs格式

**从1.24后就不能用docker了**

****[**https://kubernetes.io/blog/2020/12/02/dockershim-faq/**](https://kubernetes.io/blog/2020/12/02/dockershim-faq/) **弃用说明**

## 一、服务器优化和更新已安装软件包

```
apt update && apt upgrade
# 更新已安装软件包
cat /etc/sysctl.conf
net.core.somaxconn = 40480
net.core.rmem_default = 262144
net.core.wmem_default = 262144
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 4096 160777216
net.ipv4.tcp_wmem = 4096 4096 160777216
net.ipv4.tcp_mem = 786432 2097152 300145728
net.ipv4.tcp_max_syn_backlog = 46384
net.core.netdev_max_backlog = 50000
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_max_syn_backlog = 56384
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_max_orphans = 131072
net.ipv4.ip_local_port_range = 1024 65535
net.netfilter.nf_conntrack_max = 2097152
fs.aio-max-nr = 524288
fs.file-max = 6590202
net.ipv4.ip_forward = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.all.rp_filter = 0
net.ipv4.tcp_keepalive_time = 600
net.ipv4.tcp_timestamps = 0
```



## 二、Installing kubeadm

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

## 三、安装docker

#### Install using the repository

Before you install Docker Engine for the first time on a new host machine, you need to set up the Docker repository. Afterward, you can install and update Docker from the repository.

**Set up the repository**

1.  Update the `apt` package index and install packages to allow `apt` to use a repository over HTTPS:

    ```
    $ sudo apt-get update

    $ sudo apt-get install \
        ca-certificates \
        curl \
        gnupg \
        lsb-release
    ```
2.  Add Docker’s official GPG key:

    ```
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```
3.  Use the following command to set up the **stable** repository. To add the **nightly** or **test** repository, add the word `nightly` or `test` (or both) after the word `stable` in the commands below. [Learn about **nightly** and **test** channels](https://docs.docker.com/engine/install/).

    ```
    $ echo \
      "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
      $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

**Install Docker Engine**

1.  Update the `apt` package index, and install the _latest version_ of Docker Engine and containerd, or go to the next step to install a specific version:

    ```
     $ sudo apt-get update
     $ sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```

**优化**

cat /etc/docker/daemon.json

```
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "1000m"
  },
  "storage-driver": "overlay2"
}
```

