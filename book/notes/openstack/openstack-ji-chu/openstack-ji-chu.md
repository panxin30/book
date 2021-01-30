# openstack基础

IaaS\(openstack\),PaaS\(docker\),SaaS

**Object Storage: 代码名Swift:**分布式存储，基于RESTful的API实现非结构化数据对象的存储及检索。

**Block Storage: 代码名Cinder:** 为运行实例而提供的持久性块存储。

**image service:代码名字Glance:** 当nova创建一个虚拟机实例，需要下载磁盘映像的时候，会先找glance，glance会告诉你到哪里去下载。所以glance是作为swift前端，存储和检索虚拟机磁盘镜像。

**Identity: 代码名Key stone:** 为Openstack中所有服务提供认证和授权服务以及端点目录

新建并启动一个实例的流程：

通过dashboard/CLI发起创建实例 --&gt;先到keystone请求认证，认证通过则访问nova-api\(能收到用户请求的调用接口,监听在某个套接字上, tcp连接的端点叫套接字（socket）,根据RFC 793的定义,**端口号拼接到IP地址即构成了套接字** \)--&gt;nova-api收到请求后，再查询keystone这个帐号有没有相应权限，有则下一步nova-api先查询novaDB\(存放此前创建过的实例的各种结果实例名,虚拟核心数,内存大小等。大概新建实例的规格也存放在这?\)--&gt;novaDB把待建实例的规格发送给nova-api--&gt;nova-api把这个信息生成一个特定的请求交给nova-cumpute，但是nova-api和nova-compute之间的交流是异步的，因此nova-api把这个特定的请求丢给Queue\(队列\)--&gt;nova-scheduler在队列中读取到这个新建请求，根据此新建请求和调度策略选择一个compute节点--&gt;nova-scheduler更新选择信息到nova DB中，确认保存后--&gt;nova-scheduler将此调度结果丢到Queue\(队列\)--&gt;被选中的compute节点从Queue队列中读取到调度结果，在本地运行实例\(为了避免给novaDB造成太大压力,nova-compute把新建实例的运行等相关信息仍到Queue队列中\)--&gt;nova-conductor负责从Queue队列中取出与数据库更新相关的信息--&gt;novaDB存储信息--&gt;返回存储信息给nova-conductor--&gt;仍给Queue队列--&gt;nova-compute获取自己更新实例信息存储到数据库的结果，下一步，这个实例需要运行除了需要CPU,内存之外的其他资源,磁盘镜像，网络服务，磁盘服务等

--&gt;nova-compute连接glance-api在glace-registry中检索自己需要的磁盘镜像文件，并下载相关镜像，在此之前glance-api会先去keystone验证这个帐号有没有相关权限

--&gt;nova-compute连接neutron-server\(创建虚拟网络，网桥，把接口关联到网桥上,创建虚拟路由器，在虚拟路由上生成对应的规则以实现路由转发或地址转发\)

--&gt;neutron-server是个很繁忙的服务，因此收到请求后不会直接构建网络，而是仍到Queue队列中异步与其他相关neutron-plugins协作



