# 一键安装gitlab后的备份和恢复

https://docs.gitlab.com/ce/README.html

Omnibus packages

rpm -qa | grep gitlab-ce

sudo apt-cache policy gitlab-ce | grep Installed

\*\*\*\*\*

/etc/gitlab/gitlab.rb

包含的nginx，redis，数据库等，要改配置生效的话，都从gitlab.rb里面改

gitlab-ctl tail #查看是否有错误

gitlab-ctl reconfigure 加载配置文件

gitlab  为安装一体化，内置nginx/redis/postgresql等



&#x9;		 gitlab    主程序目录：    /var/opt/gitlab/

&#x9;					   主配置文件：  /var/opt/gitlab/gitlab-shell/config.yml

&#x9;		 gitlab    认证文件：    /var/opt/gitlab/.ssh/authorized\_keys

&#x9;					  日志文件：   /var/log/gitlab/gitlab-shell/gitlab-shell.log &#x20;



&#x9;		 nginx 配置文件 ：  /var/opt/gitlab/nginx/conf/nginx.conf

&#x9;									  /var/opt/gitlab/nginx/conf/gitlab-http.conf

&#x20;                                       &#x20;



3、所有与gitlab相关服务统一通过gitlab来启动    &#x20;

&#x9;		gitlab:  启动、关闭、重启方式：    gitlab-ctl  start | stop | restart



4、备份，还原和迁移

Gitlab创建备份&#x20;

\## gitlab-rake gitlab:backup:create



使用以上命令会在/var/opt/gitlab/backups目录下创建一个名称类似为1393513186\_gitlab\_backup.tar的压缩包, 这个压缩包就是Gitlab整个的完整部分, 其中开头的1393513186是备份创建的日期。&#x20;

Gitlab恢复&#x20;

\## 停止相关数据连接服务&#x20;

gitlab-ctl stop unicorn&#x20;

gitlab-ctl stop sidekiq



\## 从1393513186编号备份中恢复&#x20;

gitlab-rake gitlab:backup:restore BACKUP=1393513186



\## 启动Gitlab&#x20;

sudo gitlab-ctl start



迁移如同备份与恢复的步骤一样, 只需要将老服务器/var/opt/gitlab/backups目录下的备份文件拷贝到新服务器上的/var/opt/gitlab/backups即可(如果你没修改过默认备份目录的话)。但是需要注意的是新服务器上的Gitlab的版本必须与创建备份时的Gitlab版本号相同. 比如新服务器安装的是最新的8.5版本的Gitlab, 那么迁移之前, 最好将老服务器的Gitlab 升级为8.5再进行备份。&#x20;

自动备份&#x20;

通过crontab使用备份命令实现自动备份:



sudo su -&#x20;

crontab -e



例如加入以下, 实现每天凌晨2点进行一次自动备份:



0 2 \* \* \* /opt/gitlab/bin/gitlab-rake gitlab:backup:create