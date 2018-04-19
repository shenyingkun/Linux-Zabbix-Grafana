## Zabbix监控MySQL服务

   1.创建监控脚本 chk_mysql.sh
   
    cd  /usr/local/zabbix/share/zabbix/alertscripts
    touch chk_mysql.sh
    
   2.添加脚本内容
   
    #!/bin/bash

    # 用户名
    MYSQL_USER='zabbix'
 
    # 密码
    MYSQL_PWD='zabbix'
 
    # 主机地址/IP
    MYSQL_HOST='127.0.0.1'
 
    # 端口
    MYSQL_PORT='3306'
 
    # 数据连接
    MYSQL_CONN="/usr/bin/mysqladmin -u${MYSQL_USER} -p${MYSQL_PWD} -h${MYSQL_HOST} -P${MYSQL_PORT}"
 
    # 参数是否正确
    if [ $# -ne "1" ];then 
        echo "arg error!" 
    fi 
 
    # 获取数据
    case $1 in 
      Uptime) 
        result=`${MYSQL_CONN} status|cut -f2 -d":"|cut -f1 -d"T"` 
        echo $result 
        ;; 
    Com_update) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_update"|cut -d"|" -f3` 
        echo $result 
        ;; 
    Slow_queries) 
        result=`${MYSQL_CONN} status |cut -f5 -d":"|cut -f1 -d"O"` 
        echo $result 
        ;; 
    Com_select) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_select"|cut -d"|" -f3` 
        echo $result 
                ;; 
    Com_rollback) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_rollback"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Questions) 
        result=`${MYSQL_CONN} status|cut -f4 -d":"|cut -f1 -d"S"` 
                echo $result 
                ;; 
    Com_insert) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_insert"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Com_delete) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_delete"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Com_commit) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_commit"|cut -d"|" -f3` 
                echo $result 
                ;; 
    Bytes_sent) 
        result=`${MYSQL_CONN} extended-status |grep -w "Bytes_sent" |cut -d"|" -f3` 
                echo $result 
                ;; 
    Bytes_received) 
        result=`${MYSQL_CONN} extended-status |grep -w "Bytes_received" |cut -d"|" -f3` 
                echo $result 
                ;; 
    Com_begin) 
        result=`${MYSQL_CONN} extended-status |grep -w "Com_begin"|cut -d"|" -f3` 
                echo $result 
                ;; 
                        
        *) 
        echo "Usage:$0(Uptime|Com_update|Slow_queries|Com_select|Com_rollback|Questions|Com_insert|Com_delete|Com_commit|Bytes_sent|Bytes_received|Com_begin)" 
        ;; 
     esac
     
   3.修改zabbix_agentd.conf 文件末尾增加自定义key
   
       # 获取mysql版本
       UserParameter=mysql.version,mysql -V
       # 获取mysql性能指标,这个是上面定义好的脚本
       UserParameter=mysql.status[*],/usr/local/zabbix/share/zabbix/alertscripts/chk_mysql.sh $1
       # 获取mysql运行状态
       UserParameter=mysql.ping,mysqladmin -uzabbix -pzabbix -P3306 -h127.0.0.1  ping | grep -c alive
  4.重启zabbix_agentd 服务
  
  5.添加MYSQL模板 查看监控数据
