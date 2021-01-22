# shell脚本调试



#### Shell脚本调试就是发现引发脚本错误的原因以及在脚本源代码中定位发生错误的行，常用的手段包括分析输出的错误信息，通过在脚本中加入调试语句，输出调试信息来辅助诊断错误，利用调试工具等

set -eux

#### Shell脚本的错误可分为两类：

1.语法错误（syntax error），脚本无法执行到底。  
2.Shell脚本能够执行完毕，但是并不是按照我们所期望的方式运行，即存在逻辑错误

#### **trap命令**

#### **tee命令**

用于调试管道，tee -a 追加结果到debug.txt

```text
localIP=`cat /etc/sysconfig/network-scripts/ifcfg-eth0 | tee debug.txt | grep 'IPADDR' | tee -a debug.txt | cut -d= -f2 | tee -a debug.txt`
echo "The local IP is: $localIP"
```



