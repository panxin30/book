# 大规模网站集群架构

## **常见的网页资源有三种，分别是静态网页，动态网页，伪静态** <a href="chang-jian-de-wang-ye-zi-yuan-you-san-zhong-fen-bie-shi-jing-tai-wang-ye-dong-tai-wang-ye-wei-jing-t" id="chang-jian-de-wang-ye-zi-yuan-you-san-zhong-fen-bie-shi-jing-tai-wang-ye-dong-tai-wang-ye-wei-jing-t"></a>

静态网页就是没有后台数据库，不含php，jsp，asp等程序，不可交互的，开发者编写的是啥，显示的就是啥，不会有任何改变

动态网页，有后台数据库，支持更多的功能，如用户注册，登录，发帖，订单，博客等，动态网页并不独立存在于服务器上的网页文件，而是当用户请求服务器上的动态程序时，服务器解析这些程序，并调用数据库来返回一个完整的网页内容，它跟静态网页的url不同，它的url中包含？、&等特殊符号，搜索引擎收录的时候存在一定的问题。

动态网页为了方便收录，常常会利用rewrite技术，把动态网页的URL伪装成静态网页URL，这就是伪静态。

## **不同的网页资源，打开的流程不一样** <a href="bu-tong-de-wang-ye-zi-yuan-da-kai-de-liu-cheng-bu-yi-yang" id="bu-tong-de-wang-ye-zi-yuan-da-kai-de-liu-cheng-bu-yi-yang"></a>

客户端会通过http协议，下载服务器上的html文件，然后去读这个html文件，根据html页面中的链接，自上而下的请求，每一个请求是一个链接，如果是图片的话，会边下载边渲染，遇到js，就会加载js，当js比较内容较复杂时，浏览器就会等待，鼠标在转圈，我们称这个为js阻塞，当js下载完毕并执行完成之后，才会显示我们看到的网页。

当我们访问的是一个动态网页时，首先用户发出一个请求，服务器收到这个请求之后，这里假设服务器使用的是nginx，nginx会把这个请求转给php，php就会去查询数据库，根据数据库返回的值，生成一个完整的网页内容，发送给用户，用户收到之后，也是边下载边渲染，加载js，执行完毕之后，才会显示我们看到的网页

当服务器的访问量达到亿级PV时，这个访问的过程就更复杂了，用户的请求会先访问全国的CDN节点，通过CDN挡住全国80%的请求，

当CDN上没有时，再访问服务器集群，这个集群一般都有一个4层的代理，这个4层的代理，使用软件来完成的话，就是LVS，使用硬件就是F5，

后面才是7层的负载均衡，常用的是haproxy,nginx，然后才是多台web服务器，

web服务器比较多的时候，就有两个问题，一个是用户数据的一致性，不能因为不同的web服务器提供服务，而导致数据不同步，这时候，我们就需要使用NFS共享存储，

## session

**第二个问题是session，不能因为不同的web服务器提供服务，session找不到了**，这时候，我们就需要使用memcached来存放并共享session。

由于用户访问量太大，这时候的瓶颈就是数据库的压力，我们一般都是使用分布式缓存memcache,redis等，另外数据库还需要做读写分离等优化，后面的过程与访问动态网页类似

### 为什么要使用Session？

\
因为很多第三方可以获取到这个Cookie，服务器无法判断Cookie是不是真实用户发送的，所以Cookie可以伪造，伪造Cookie实现登录进行一些HTTP请求。如果从安全性上来讲，Session比Cookie安全性稍微高一些，我们先要知道一个概念--SessionID。SessionID是什么？客户端第一次请求服务器的时候，服务器会为客户端创建一个Session，并将通过特殊算法算出一个session的ID，下次请求资源时（Session未过期），浏览器会将sessionID(实质是Cookie)放置到请求头中，服务器接收到请求后就得到该请求的SessionID，服务器找到该id的session返还给请求者使用。

### Session的生命周期？

\
根据需求设定，一般来说，半小时。举个例子，你登录一个服务器，服务器返回给你一个sessionID，登录成功之后的半小时之内没有对该服务器进行任何HTTP请求，半小时后你进行一次HTTP请求，会提示你重新登录。
