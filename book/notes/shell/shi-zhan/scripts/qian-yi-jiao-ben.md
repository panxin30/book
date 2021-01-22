# 迁移脚本



```text
#!/bin/bash
PEXIT (){
    echo $1
    exit 9
}

#导出中间库生成的数据
dump_newdb (){
    id=$1
    echo -e "正在导出转化后的数据，请确认ID: $id 是否正确,10秒后继续,Ctrl+C 取消."
    sleep 10
    mysqldump -uroot -h120.76.46.55 -pnHH29kEA7 brokerwork_t"$id" > 1.sql
    mysqldump -uroot -h120.76.46.55 -pnHH29kEA7 brokerwork_t"$id"_mt4_1 > 2.sql
        sed -e 's/DEFINER[ ]*=[ ]*[^*]*\*/\*/ ' 1.sql > brokerwork_t"$id".sql
        sed -e 's/DEFINER[ ]*=[ ]*[^*]*\*/\*/ ' 2.sql > brokerwork_t"$id"_mt4_1.sql
    mysqldump -uroot -h120.76.46.55 -pnHH29kEA7 --complete-insert --no-create-info --skip-add-locks message_transfer_t"$id" >message_transfer_t"$id".sql
    mongodump -h120.76.46.55:27017 -dtransfer"$id" -umsc -pabc123 -o /data/scripts/
#    如果有实时返佣
#    mongodump -h120.76.46.55:27017 -drcr_t"$id" -uadmin --authenticationDatabase=admin -pWAer8R59G6 -o /root/
}

#第二步
#将转化后的数据导入到测试环境.
import_newdb_dev (){
    id=$1
    echo -e "正在将转化后的数据导入开发环境，请确认ID: $id 是否正确,10秒后继续,Ctrl+C 取消."
    sleep 10
    #导入到测试环境1,2不用再操作
    mysql -uroot -h120.76.46.55 -pnHH29kEA7 bw_message < message_transfer_t"$id".sql
    mongorestore -h120.76.46.55:27017 -dmsc-db -umsc -pabc123 transfer"$id"
}

#第三步
#导入到QA环境
#备份bw_message再导入这份数据(带上日期)
#这里的id如000128

import_newdb_qa (){
    id=$1
    #id2=$2
    echo -e "正在将转化后的数据导入开发环境，请确认ID: $id 是否正确,10秒后继续,Ctrl+C 取消."
    sleep 10
    backup_date=`date "+%Y-%m-%d-%H-%M"`
    #backup db bw_message
    echo -e "开始备份qa环境bw_message数据库\n"
    sleep 3
    mysqldump -ubwqa -hrm-j6c40s6glhqsd26cc.mysql.rds.aliyuncs.com -pGycHWb@qx9Ld --set-gtid-purged=OFF bw_message > qa_bw_message_$backup_date.sql
    mysqldump -ubwqa -hrm-j6c40s6glhqsd26cc.mysql.rds.aliyuncs.com -pGycHWb@qx9Ld --set-gtid-purged=OFF brokerwork_t$id > qa_bw_t$id_$backup_date.sql

    #import data to bw4.0 QA
    echo -e "开始将mysql数据导入到qa mysql\n"
    sleep 3
    mysql -ubwqa -hrm-j6c40s6glhqsd26cc.mysql.rds.aliyuncs.com -pGycHWb@qx9Ld -e "drop database brokerwork_t$id;" || PEXIT "drop database brokerwork_t$id 失败."
    mysql -ubwqa -hrm-j6c40s6glhqsd26cc.mysql.rds.aliyuncs.com -pGycHWb@qx9Ld -e "create database brokerwork_t$id;" || PEXIT "create database brokerwork_t$id 失败."
    mysql -ubwqa -hrm-j6c40s6glhqsd26cc.mysql.rds.aliyuncs.com -pGycHWb@qx9Ld brokerwork_t"$id" < brokerwork_t"$id".sql
    mysql -ubwqa -hrm-j6c40s6glhqsd26cc.mysql.rds.aliyuncs.com -pGycHWb@qx9Ld brokerwork_t"$id"_mt4_1 < brokerwork_t"$id"_mt4_1.sql
    mysql -ubwqa -hrm-j6c40s6glhqsd26cc.mysql.rds.aliyuncs.com -pGycHWb@qx9Ld bw_message < message_transfer_t"$id".sql
#执行QA生产导入完数据后
#use bw_message
#UPDATE t_user_message, 
#t_message 
#SET t_user_message.message = t_message.id 
#WHERE 
#t_message.tenant_id = "T000128" 
#AND t_user_message.tenant_id = t_message.tenant_id 
#AND t_user_message.message_no = t_message.entity_no 
#AND t_user_message.message_no IS NOT NULL;

    #备份QAmongo的brokerwork库，然后再导入这份数据
#####db.getCollection('t_tasks_item').remove({"tenantId":"T800078"})删除该租户的初始数据
    echo -e "开始备份qa环境 mongo库brokerwork.\n"
    sleep 3
    mongodump -h10.25.154.114:27017 -dbrokerwork -ubrokerwork -p4wUgjFA -o QA_brokerwork_$backup_date/
#    mongodump -h10.26.149.225:27017 -dbrokerwork -ubrokerwork -p4wUgjFA -o QA_brokerwork_$backup_date/
#    mongodump -h10.26.149.213:27017 -dbrokerwork -ubrokerwork -p4wUgjFA -o  QA_brokerwork_$backup_date/
    echo -e "开始将transfer库数据导入mongo：brokerwork\n"
    sleep 3
    #导入QA前删除taskitem
    echo "db.t_tasks_item.remove({'tenantId' : 'T$id','item.itemNo':'A00'});" > create_mongodb.js
    mongo 10.25.154.114:27017/brokerwork -ubrokerwork -p4wUgjFA create_mongodb.js
#    mongo 10.26.149.225:27017/brokerwork -ubrokerwork -p4wUgjFA create_mongodb.js
#    mongo 10.26.149.213:27017/brokerwork -ubrokerwork -p4wUgjFA create_mongodb.js
#测试上面的语句   
    mongorestore -h10.25.154.114:27017 -dbrokerwork -ubrokerwork -p4wUgjFA transfer"$id"
#    mongorestore -h10.26.149.225:27017 -dbrokerwork -ubrokerwork -p4wUgjFA transfer"$id"
#    mongorestore -h10.26.149.213:27017 -dbrokerwork -ubrokerwork -p4wUgjFA transfer"$id"
#   如果有实时返佣
#    mongorestore -h10.26.60.3:27017 -drcr_t"id" -uroot -pleanwork123 --authenticationDatabase=admin --drop /root/rcr_t"$id"
    echo -e "导入数据到qa环境完成\n"
    sleep 3
}


#第四步
#导入到生产环境
#备份bw_message再导入这份数据(带上日期)
#这里的id如000128
#db.getCollection('t_tasks_item').remove({'tenantId' : 'T000029'});

import_newdb_prod (){
    id=$1
    echo -e "正在将转化后的数据导入开发环境，请确认ID: $id 是否正确,10秒后继续,Ctrl+C 取消."
    sleep 10
    backup_date=`date "+%Y-%m-%d-%H-%M"`
    #/usr/bin/mysqldump -h127.0.0.1 -uroot -pGyc@HWb.qx9Ld --set-gtid-purged=OFF --all-databases > ali-hk-bw-db2.sql
    mysqldump -uroot -h10.28.184.253 -pGyc@HWb.qx9Ld --set-gtid-purged=OFF bw_message > prod_bw_message_$backup_date.sql
    mysqldump -uroot -h10.28.184.253 -pGyc@HWb.qx9Ld --set-gtid-purged=OFF brokerwork_t"$id" > brokerwork_t"$id"_$backup_date.sql
    echo -e "即将删除数据库:brokerwork_t$id,请核对是否正确!!!;10秒后继续，Ctrl+C 取消. \n"
    sleep 10
    mysql -uroot -h10.28.184.253 -pGyc@HWb.qx9Ld -e "drop database brokerwork_t$id;" || PEXIT "drop database brokerwork_t$id 失败."
    mysql -uroot -h10.28.184.253 -pGyc@HWb.qx9Ld -e "create database brokerwork_t$id;" || PEXIT "create database brokerwork_t$id 失败."

    mysql -uroot -h10.28.184.253 -pGyc@HWb.qx9Ld brokerwork_t"$id" < brokerwork_t"$id".sql
    mysql -uroot -h10.28.184.253 -pGyc@HWb.qx9Ld brokerwork_t"$id"_mt4_1 < brokerwork_t"$id"_mt4_1.sql
    mysql -uroot -h10.28.184.253 -pGyc@HWb.qx9Ld bw_message < message_transfer_t"$id".sql
#执行QA生产导入完数据后
#use bw_message
#UPDATE t_user_message, 
#t_message 
#SET t_user_message.message = t_message.id 
#WHERE 
#t_message.tenant_id = "T000128" 
#AND t_user_message.tenant_id = t_message.tenant_id 
#AND t_user_message.message_no = t_message.entity_no 
#AND t_user_message.message_no IS NOT NULL;
    #备份QAmongo的brokerwork库，然后再导入这份数据
    echo -e "开始备份生产mongo数据库brokerwork\n"
    sleep 3
    mongodump -h10.28.185.68:27017 -dbrokerwork -ubrokerwork -po6RJzAF1LELzutEv -o Prod_brokerwork_$backup_date/
    #mongodump -h10.47.43.22:27017 -dbrokerwork -ubrokerwork -po6RJzAF1LELzutEv -o Prod_brokerwork_$backup_date/
    #mongodump -h10.47.36.173:27017 -dbrokerwork -ubrokerwork -po6RJzAF1LELzutEv -o Prod_brokerwork_$backup_date/
    echo -e "开始将transfer数据导入mongo:brokerwork"
    #导入前删除taskitem
    echo "db.t_tasks_item.remove({'tenantId' : 'T$id','item.itemNo':'A00'});" > create_mongodb.js
    mongo 10.28.185.68:27017/brokerwork -ubrokerwork -po6RJzAF1LELzutEv create_mongodb.js
    mongorestore -h10.28.185.68:27017 -dbrokerwork -ubrokerwork -po6RJzAF1LELzutEv transfer"$id"
    #mongorestore -h10.47.43.22:27017 -dbrokerwork -ubrokerwork -po6RJzAF1LELzutEv transfer"$id"
    #mongorestore -h10.47.36.173:27017 -dbrokerwork -ubrokerwork -po6RJzAF1LELzutEv transfer"$id"
#   如果有返佣
#    mongorestore -h10.47.43.22:27017 -drcr_t"$id" -uadmin -p'Jp@2Cny&XIun' --authenticationDatabase=admin rcr_t"$id"
#    mongorestore -h10.28.185.68:27017 -drcr_t"$id" -uadmin -p'Jp@2Cny&XIun' --authenticationDatabase=admin rcr_t"$id"
    echo -e "导入数据到生产环境完成\n"
    sleep 3
}

#id如000128

echo -e "导出转化后的数据\n"
sleep 3
#dump_newdb 000136 || PEXIT "导出数据失败，请查看日志。"

echo -e "导入新数据到dev环境\n"
sleep 3
#import_newdb_dev id || PEXIT "导入数据到dev环境失败。"
echo -e "导入新数据到qa环境\n"
sleep 3

#import_newdb_qa 000136 || PEXIT "导入数据到qa环境失败."
echo -e "导入新数据到prod环境\n"
sleep 3
import_newdb_prod 000136 || PEXIT "导入数据到prod环境失败。"




exit 0
```

