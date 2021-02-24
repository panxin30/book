# 恢复阿里云物理备份

引用：https://help.aliyun.com/knowledge\_detail/41817.html

\# 恢复mysql，dump其中一部分数据

\#\#\# mysql数据库较大，下载解压4小时左右

1.在备份机root@ali-hk-lw-databackup上有每天的备份,已经安装了mysql5.7.28和mongo3.4.19,Percona XtraBackup2.4

2.下载备份

\`wget -c '&lt;数据备份文件外网或内网下载地址&gt;' -O &lt;自定义文件名&gt;.tar.gz\`

3.解压

\`tar -izxvf &lt;数据备份文件名&gt;.tar.gz -C /data/restore/mysql\`

4.执行如下命令，恢复解压好的备份文件。

\`innobackupex --defaults-file=/data/restore/mysql/backup-my.cnf --apply-log /data/restore/mysql\`

因为文件大，所以连开始都慢，不能直接停止，否则一个log文件会消失，就不能再成功执行了。只有重新解压缩包

5.自建数据库不支持下面的参数，注销\*\*backup-my.cnf\*\*中相关参数

\`\`\`

\#innodb\_log\_checksum\_algorithm

\#innodb\_fast\_checksum

\#innodb\_log\_block\_size

\#innodb\_doublewrite\_file

\#rds\_encrypt\_data

\#innodb\_encrypt\_algorithm

\#redo\_log\_version

\#master\_key\_id

\#server\_uuid

\`\`\`

6.修改文件所属

\`chown -R mysql:mysql /data/restore/mysql\`

7.执行如下命令，启动MySQL进程。

\`mysqld --defaults-file=/data/restore/mysql/backup-my.cnf --user=mysql --datadir=/data/restore/mysql &\`

\#\#\# 结果报错

\#\# Can't change dir to '/xxx' \(Errcode: 13\)，Permission denied，真是奇怪，所属已经改了的

\#\# 最后处理办法，\*\*将restore也修改所属为mysql\*\*

8.\*\*导出需要的数据，再恢复到生产环境\*\*。



或者用命令启动确认没有问题后，再修改/etc/my.cnf中数据库目录，直接使用这个配置启动

\# mongodb 恢复  

\#\#\# \*\*目前7天前的备份存储在oss中，需要先解冻再用 wget -c -O 下载，再恢复\*\*

\*\*最好用docker run一个mongo，想要什么版本就什么版本\*\*

使用mongo3.4.19

引用https://help.aliyun.com/document\_detail/58329.html?spm=a2c6r.11610030.0.0.63b64231DrgJiF

1.下载阿里云mongodb物理备份

\`wget -c '&lt;数据备份文件外网或内网下载地址&gt;' -O &lt;自定义文件名&gt;.tar.gz\`

2.解压到指定目录

\`tar zxf saas.owsc.20191126.tar.gz -C mongo/\`

3.以单节点模式恢复MongoDB物理备份的数据

引用WiredTiger 存储引擎的配置文件，修改mongodb所在目录

4.启动mongo

\`/usr/bin/mongod -f /etc/mongod.conf\`

5.等待启动完成后，可通过服务器的 mongo shell 登录 MongoDB 数据库。帐号密码就是线上数据库帐号密码

\`mongo 127.0.0.1:27017/owsc -uowsc -pDS3x8mHP\`

6，dump需要的数据，再恢复到生产。

