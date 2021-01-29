# openstack基础

IaaS\(openstack\),PaaS\(docker\),SaaS

**Object Storage: 代码名Swift:**分布式存储，基于RESTful的API实现非结构化数据对象的存储及检索。

**Block Storage: 代码名Cinder:** 为运行实例而提供的持久性块存储。

**image service:代码名字Glance:** 当nova创建一个虚拟机实例，需要下载磁盘映像的时候，会先找glance，glance会告诉你到哪里去下载。所以glance是作为swift前端，存储和检索虚拟机磁盘镜像。

**Identity: 代码名Key stone:** 为Openstack中所有服务提供认证和授权服务以及端点目录

新建并启动一个实例的流程：

通过dashboard/CLI 

