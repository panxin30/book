# netstat

**netstat**

```
[root@linux ~]# netstat -[rn]       <==与路由有关的参数
[root@linux ~]# netstat -[antulpc]  <==与网络接口有关的参数
参数：
与路由 (route) 有关的参数说明：
-r  ：列出路由表(route table)，功能如同 route 这个命令；
-n  ：不适用主机名称与服务名称，只使用IP和Port NUmber
-a  ：列出所有的联机状态，包括 tcp/udp/unix socket 等；
-t  ：仅列出 TCP 封包的联机；
-u  ：仅列出 UDP 封包的联机；
-l  ：仅列出有在 Listen (监听) 的服务之网络状态；
-p  ：列出程序PID和程序名；
-c  ：可以配置几秒钟后自动升级一次，例如 -c 5 每五秒升级一次网络状态的显示；
```

范例一：**列出目前的路由表状态，且以 IP 及 port number 显示：**

```
[root@linux ~]# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
192.168.10.0    0.0.0.0         255.255.255.0   U         0 0          0 eth0
169.254.0.0     0.0.0.0         255.255.0.0     U         0 0          0 eth0
0.0.0.0         192.168.10.30   0.0.0.0         UG        0 0          0 eth0
```

其实这个参数就跟 route -n 一模一样，这不是 netstat 的主要功能！

范例二：**列出目前的所有网络联机状态，使用 IP 与 port number**

```
[root@linux ~]# netstat -an
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State
tcp        0      0 0.0.0.0:25                  0.0.0.0:*                   LISTEN
tcp        0      0 :::22                       :::*                        LISTEN
tcp        0      0 ::ffff:192.168.10.100:25    ::ffff:192.168.10.200:57509 TIME_WAIT
tcp        0     52 ::ffff:192.168.10.100:22    ::ffff:192.168.10.210:1504  ESTABLISHED
udp        0      0 127.0.0.1:53                0.0.0.0:*
Active UNIX domain sockets (servers and established)
Proto RefCnt Flags       Type       State         I-Node Path
unix  2      [ ACC ]     STREAM     LISTENING     4792   public/cleanup
unix  2      [ ACC ]     STREAM     LISTENING     4799   private/rewrite
......(底下省略)......
```

netstat 的输出主要分为两大部分，分别是 TCP/IP 的网络接口部分，以及传统的 Unix socket 部分。\
建议加上『 -n 』这个参数的，因为可以避过主机名与服务名称的反查，直接以 IP 及端口号码 (port number) 来显示，显示的速度上会快很多！至于在输出的信息当中

* **Proto**：该联机的封包协议，主要为 TCP/UDP 等封包；
* **Recv-Q**：非由用户程序连接所复制而来的总 bytes 数；
* **Send-Q**：由远程主机所传送而来，但不具有 ACK 标志的总 bytes 数， 意指主动联机 SYN 或其他标志的封包所占的 bytes 数；
* **Local Address**：本地端的地址，可以是 IP (-n 参数存在时)， 也可以是完整的主机名。如上表我们看到的 IP 格式有两种，一种是 IPv4 的标准， 亦即是四组十进制的数字后面加上冒号『:』后，接着 port number 。一种是 IPv6 ， 前面的 IP 加上很多冒号『:』的格式。我们可以由这个显示的数据看出这个服务是开放在哪一个接口， 例如上表当中， port 22 是开放在 0.0.0.0 ，亦即是所有接口都可以连到 port 22 ， 至于 port 53 则仅开放在本机的 127.0.0.1 这个接口而已，所以是不对外部接口开放的意思。
* **Foreign Address**：远程的主机 IP 与 port number
* **stat**：状态栏，主要的状态含有：
* ESTABLISED：已创建联机的状态；
* SYN\_SENT：发出主动联机 (SYN 标志) 的联机封包；
* SYN\_RECV：接收到一个要求联机的主动联机封包；
* FIN\_WAIT1：该插槽服务(socket)已中断，该联机正在断线当中；
* FIN\_WAIT2：该联机已挂断，但正在等待对方主机响应断线确认的封包；
* TIME\_WAIT：该联机已挂断，但 socket 还在网络上等待结束；
* LISTEN：通常用在服务的监听 port ！可使用『 -l 』参数查阅。

**范例三：列出TCP和UDP在Listen的服务，同时显示PID和程序名**

```
[root@linux ~]# netstat -unplt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address   Foreign Address  State    PID/Program name
tcp        0      0 0.0.0.0:25      0.0.0.0:*        LISTEN   2141/master
tcp        0      0 :::22           :::*             LISTEN   1924/sshd
tcp        0      0 :::25           :::*             LISTEN   2141/master
udp        0      0 127.0.0.1:53    0.0.0.0:*                 1911/named
# 上面最重要的其实是那个 -l 的参数，因为可以仅列出有在 Listen 的 port
```

**范例四：观察本机上头所有的网络联机状态**

```
[root@linux ~]# netstat -atunp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address     Foreign Address     State       PID/Program name
tcp        0      0 0.0.0.0:25        0.0.0.0:*           LISTEN      2141/master
tcp        0      0 :::22             :::*                LISTEN      1924/sshd
tcp        0      0 :::25             :::*                LISTEN      2141/master
tcp        0      1 192.168.5.233:34994     10.26.44.42:9200        SYN_SENT    11029/sidekiq 4.2.1
tcp        0      0 127.0.0.1:9090          127.0.0.1:53328         TIME_WAIT   -                         
tcp        0      0 192.168.5.233:80        118.112.177.3:53235   ESTABLISHED 3516/nginx: worker         
tcp        0      1 192.168.5.233:34992     10.26.44.42:9200        SYN_SENT    11029/sidekiq 4.2.1        
tcp        0      0 192.168.5.233:36850     119.23.141.98:65398     ESTABLISHED 6498/java       
tcp        0      0 192.168.5.233:80        118.112.177.3:53396   ESTABLISHED 3521/nginx: worker           
tcp        0      0 192.168.5.233:34048     100.100.30.26:80        ESTABLISHED 16406/AliYunDun          
tcp        0   3260 192.168.5.233:22        118.112.177.3:42662   ESTABLISHED 1693/0                    
tcp        0      0 192.168.5.233:33586     52.231.18.241:443       ESTABLISHED 6498/java 
```

**必须要想起来的是：『Client 端是随机取一个大于 1024 以上的 port 进行联机』，此外『只有 root 可以启动小于 1023 以下的 port 』**
