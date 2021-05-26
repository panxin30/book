# Creating a single control-plane cluster with kubeadm

## 环境

ubuntu 18.04,k8s 1.17.17,docker 19.03.15 数据盘使用xfs格式，亚马逊服务器，国内的话，相关源需要替换。

## 架构

ingress+k8s+minio+nginx

* K8S提供微服务
* minio提供前端
* nginx转发后端和前端服务
* ingress作为对外入口80/443，转发到nginx，172.31.27.142:800。使用ingress是因为可以使用cert-manager免费提供证书              

## 一、服务器优化和更新已安装软件包

```text
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

## 二、安装docker-ce,19.03.15

### 安装参考文档

* [Docker 官方 Ubuntu 安装文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/)

### 安装指定版本docker-ce

```text
apt-cache madison docker-ce
sudo apt-get install docker-ce=18.06.1~ce~3-0~ubuntu
```

### 根据需求修改docker驱动

```text
root@master01:~# cat /etc/docker/daemon.json 
{
  "storage-driver": "overlay2",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "1048m",
    "max-file": "2"
  }
}
```

## 三、安装kubelet kubeadm kubectl\(这里使用的是官方源，国内的话需要替换\)

```text
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-cache madison kubeadm
sudo apt-get install -y kubelet kubeadm kubectl
# 或者安装指定版本 
sudo apt -y install kubelet=1.17.17-00 kubeadm=1.17.17-00 kubectl=1.17.17-00
sudo apt-mark hold kubelet kubeadm kubectl 指定服务不自动更新
```

## 四、kubelet优化

```text
root@saas-test:# cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf 
# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS --allowed-unsafe-sysctls=kernel.msg*,net.core.somaxconn,net.ipv4.tcp_keepalive_time,net.ipv4.tcp_syncookies,net.ipv4.tcp_tw_reuse,net.ipv4.tcp_timestamps,net.ipv4.tcp_fin_timeout
```

### node默认最大pod数量为110个，所有node都达到110个时无法再调度

出现如下报错信息

`0/3 nodes are available: 1 node(s) had taints that the pod didn't tolerate, 2 Insufficient pods`

#### 解决办法：

#### 1、增加最大pod数量

```text
root@k8smaster ~]# vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf 
添加：
	Environment="KUBELET_NODE_MAX_PODS=--max-pods=600"
	ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS $KUBELET_NODE_MAX_PODS
# 将新添加的参数KUBELET_NODE_MAX_PODS，加入ExecStart
# 重启kubelet
```



#### 2、真的是内存不够了

```text
kubectl describe node slave01
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests       Limits
  --------           --------       ------
  cpu                2160m (54%)    22 (550%)
  memory             15450Mi (98%)  19964Mi (127%)
  ephemeral-storage  0 (0%)         0 (0%)
# pod的配置文件是从生产拷贝的，使用的内存都大，拷贝下来没有修改过。
```

## 五、初始化kubernets集群

### 初始化

`kubeadm init --pod-network-cidr=192.168.0.0/16`

`kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml`

{% hint style="info" %}
在k8s v1.17.X这个版本安装ingress只有安装calico这个网络插件才能成功
{% endhint %}

#### 修改端口范围 会自动重启

```text
cat /etc/kubernetes/manifests/kube-apiserver.yaml
新增参数
`- --service-node-port-range=80-65535`
```

#### svc上的nodeport会监听在所有的节点上\(如果不指定,即是随机端口,'30000-32767'\)

即使有1个pod,任意访问某台的nodeport都可以访问到这个服务

#### externalIPs 通过svc创建,在指定的node上监听端口

### worker节点加入集群

