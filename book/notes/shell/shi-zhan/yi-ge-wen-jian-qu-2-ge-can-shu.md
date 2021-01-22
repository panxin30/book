# 一个文件取2个参数



```text
root@cn-office-crm-qa-all-k8s01:~# cat 1.txt 
registry-hk-tools.lwork.com/bw-account:master-31
registry-hk-tools.lwork.com/bw-copy-trade:master-3
```

**对k8s集群中服务的镜像升级**  
同时需要服务名字和镜像地址链

```text
root@cn-office-crm-qa-all-k8s01:~# cat 2.sh 
#!/bin/bash
for i in $(cat 1.txt)
do
	k=$(echo $i | awk -F ':' '{print $1}' | awk -F '/' '{print $2}')
	kubectl set image deploy $k $k=$i -n crm-prod
	sleep 120
done
```

