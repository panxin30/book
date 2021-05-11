# openstack基础

IaaS\(openstack\),PaaS\(docker\),SaaS

**Object Storage: 代码名Swift:**分布式存储，基于RESTful的API实现非结构化数据对象的存储及检索。

**Block Storage: 代码名Cinder:** 为运行实例而提供的持久性块存储。

**image service:代码名字Glance:** 当nova创建一个虚拟机实例，需要下载磁盘映像\(\)的时候，会先找glance，glance会告诉你那些可用，到哪里去下载。所以glance是作为swift前端，存储和检索虚拟机磁盘镜像。

**Identity: 代码名Key stone:** 为Openstack中所有服务提供认证和授权服务以及端点目录

## 新建并启动一个实例的流程：

通过dashboard/CLI发起创建实例 --&gt;先到**keystone**请求认证，认证通过则访问**nova-api**\(能收到用户请求的调用接口,监听在某个**套接字上**, tcp连接的端点叫套接字（socket）,根据RFC 793的定义,**端口号拼接到IP地址即构成了套接字** \)

--&gt;**nova-api**收到请求后，再查询**keystone**这个帐号有没有相应权限，有则下一步**nova-api**先查询**novaDB**\(存放此前创建过的实例的各种结果实例名,虚拟核心数,内存大小等。大概新建实例的规格也存放在这?\)--&gt;**novaDB**把待建实例的规格发送给**nova-api**--&gt;**nova-api**把这个信息生成一个特定的请求交给**nova-cumpute**，但是**nova-api和nova-compute**之间的**交流是异步**的，因此**nova-api**把这个特定的请求丢给**Queue**\(队列\)--&gt;**nova-scheduler**在队列中读取到这个新建请求，根据此新建请求和调度策略选择一个计算节点--&gt;**nova-scheduler**更新选择信息到**novaDB**中，确认保存后--&gt;**nova-scheduler**将此调度结果丢到**Queue**\(队列\)--&gt;被选中的**compute**节点从**Queue**队列中读取到调度结果，在本地运行实例\(为了避免给**novaDB**造成太大压力,**nova-compute**把新建实例的运行等相关信息扔到**Queue**队列中\)--&gt;**nova-conductor**负责从**Queue**队列中取出与数据库更新相关的信息--&gt;**novaDB**存储信息--&gt;返回存储信息给**nova-conductor**--&gt;扔给**Queue**队列--&gt;**nova-compute**获取自己更新实例信息存储到数据库的结果，下一步，这个实例需要运行除了需要CPU,内存之外的其他资源,磁盘镜像，网络服务，磁盘服务等

--&gt;**nova-compute**向**glance-api**发起请求，检索存在**glace-registry**中自己需要的磁盘镜像文件相关信息，并到image store下载相关镜像，在此之前**glance-api**会先去**keystone**验证这个帐号有没有相关权限

--&gt;**nova-compute**连接**neutron-server**\(创建虚拟网络，网桥，把接口关联到网桥上,创建虚拟路由器，在虚拟路由上生成对应的规则以实现路由转发或地址转换\)

--&gt;**neutron-server**是个很繁忙的服务，因此收到请求后不会直接构建网络，而是扔到**Queue**队列中异步与其他相关**neutron-plugins**协作，根据查询**neutronDB**来完成对对应实例的虚拟网络的创建，其中有一部分需要在真正运行了此实例的compute节点上的**neutron-agent**来实现\(网桥等\)--&gt;因此把需求扔到**Queue**队列中，待建实例所在的计算节点的**neutron-agent**收到请求，给待建实例创建好网桥，网卡等，把创建虚拟网络的相关信息保存到**neutronDB**，然后通知**neutron-server**通知**nova-compute**。在此之前**neutron-server**会先到**keystone**验证权限。 

--&gt;**nova-compute**向**cinder-api**发起请求块存储，**cinder-api**先向**keystone**认证通过后--&gt;**cinder-volume**分析请求，查询**cinderDB**，确定没有问题后扔到**Queue**队列中，由**cinder**负责具体生成卷及分配卷--&gt;保存结果到**cinderDB**--&gt;反馈给**cinder-api**--&gt;反馈给**nova-compute**，然后开始启动实例。



三节点openstack架构 controller node,network node,compute node

controller node: mysql+rabbitmq,keystone,

