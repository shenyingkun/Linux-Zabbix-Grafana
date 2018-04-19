## 1.Grafana 内网安装

   1）下载Grafana rpm包 
   
    https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-5.0.4-1.x86_64.rpm
    grafana-4.4.3-1.x86_64.rpm
    
   2）上传Grafana rpm包 到服务器
    
    rpm-ivh grafana-4.4.3-1.x86_64.rpm
    
   3）启动Grafana 服务
   
    /etc/init.d/grafana-server start
    
   4）打开web页面
   
    http://192.168.190.134:3000  admin/admin
    
 ## 2.添加zabbix 数据源
 
   1）下载zabbix 插件 alexanderzobnin-zabbix-app
   
    https://grafana.com/plugins/alexanderzobnin-zabbix-app
    
   2）上传插件到服务器
   
     /var/lib/grafana/plugins目录
     
   3）在web 插件管理页面开启zabbix 插件
   4）配置 Data Sources数据源
     
      Name          test #自定义
      Type          Zabbix
      Url           http://ZabbixServerIP/zabbix/api_jsonrpc.php
      Access        direct
      
   5）测试连接->Success
   
