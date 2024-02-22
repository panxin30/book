# mtr

**网络诊断工具 例如 ping traceroute mtr 都使用的 "ICMP" 包来测试 Internet 两点之间的网络连接状况**

## 使用Ping检查连通性有六个步骤： <a href="ping_2" id="ping_2"></a>

* 使用ifconfig观察本地网络设置是否正确;
* Ping127.0.0.1，127.0.0.1回送地址Ping回送地址是为了检查本地的TCP/IP协议有没有设置好;
* Ping本机IP地址，这样是为了检查本机的IP地址是否设置有误;
* Ping本网网关或本网IP地址，这样的是为了检查硬件设备是否有问题，也可以检查本机与本地网络连接是否正常;(在非局域网中这一步骤可以忽略)
* Ping本地DNS地址，这样做是为了检查DNS是否能够将IP正确解析。\
  注： `/etc/resolv.conf`文件，“nameserver 10.0.0.211”指定了dns服务器的地址
* Ping远程IP地址，这主要是检查本网或本机与外部的连接是否正常

## MTR测试结果分析 <a href="mtr_12" id="mtr_12"></a>

引用：[https://cloud.tencent.com/developer/article/1035350](https://cloud.tencent.com/developer/article/1035350)\
Best 、Avg、Worst、Last：分别是到相应节点延迟的最小值、平均值、最大值和最后一次值。

### 1.1 <a href="11_15" id="11_15"></a>

在分析 MTR 输出时，需要寻找两件事情：**丢包和延迟。** 首先让我们来谈谈丢包。如果您在任何特定跳点看到一定百分比的丢失，这可能表明该特定路由器存在问题。然而，**一些服务提供商通常的做法是限制 MTR 使用的ICMP流量**。**这实际上没有真正的丢包**，但是给出丢包的错觉。要确定您看到的丢包是真实的还是由于速率限制的，可查看随后的一跳，如果该跳丢包率是0.0％，那么您可以确定您看到的是 ICMP 速率限制导致，而不是实际丢包。

### 1.2 <a href="12_17" id="12_17"></a>

对链路测试结果进行分析时，需要关注如下要点。\
网络区域\
链路负载均衡\
结合Avg（平均值）和 StDev（标准偏差）综合判断\
Loss%（丢包率）的判断\
延迟\
在这种情况下，第1跳和第4跳之间报告的丢包可能是由于第2和3跳速率限制。虽然剩下的跳数的流量都触及第2和3跳，但是第4跳没有丢包。**如果丢失持续多于一个跳，则可能存在一些丢包或路由问题**。请记住，速率限制和实际丢失可能同时发生。在这种情况下，**将序列中的最低损失百分比作为实际损失。**

**然而，一些损失是由于速率限制，因为最终的跳数只有10％的损失。报告不同数量的损失时，请始终信任来自后期的报告。**

一些损失也可以通过返回路线中的问题来解释。数据包将无错误地到达目的地，但很难做出回程。这在报告中很明显，**但可能难以从 MTR 的输出中推断出来。因此，当您遇到问题时，通常最好双向收集 MTR 报告。**

另外，**抵制调查或报告您的连接中所有丢包发生的诱惑。** 互联网协议被设计为对一些网络退化具有弹性，并且数据跨 Internet 的路由可以响应于负载，简短的维护事件和其他路由问题而波动。**如果您的 MTR 报告显示10％附近的少量损失，则没有任何理由引起真正的关注，因为应用层将补偿可能短暂的丢包。**

\