# nc

## 传输文件

**方法1**，-----先启动接收命命令，后开启接发送命令 \
步骤1，先在B机器上启动一个接收文件的监听，格式如下

意思是把在10086端口接收到的数据都写到file文件里（这里文件名随意取）

nc -l port >file

栗子：nc -l 10086 >zabbix.rpm

步骤2，在A机器上往B机器的10086端口发送数据，把下面rpm包发送过去

nc 192.168.0.2 10086 < zabbix.rpm

B机器接收完毕，它会自动退出监听，文件大小和A机器一样，md5值也一样

**方法2**，-----先启动发送命命令，后开启接受命令\
步骤1，先在B机器上，启动发送文件命令

下面命令表示通过本地的10086端口发送abc.txt文件

nc -l 10086

\
