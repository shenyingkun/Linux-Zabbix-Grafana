
# Zabbix Agent 内网安装部署
## 首先关闭防火墙和seLinux
    service iptables stop
    chkconfig iptables off

## （1）安装依赖包

    yum install httpd libxml2-devel libcurl-devel
    yum install gcc*
    yum insall net-snmp*

## (2) 安装zabbix

    groupadd zabbix
    useradd -g zabbix -u 201 -m zabbix

安装包可在线下载，内网安装需要离线下载并上传服务器

    wget http://sourceforge.net/projects/zabbix/files/ZABBIX%20Latest%20Stable/3.0.3/zabbix-3.0.3.tar.gz
    tar zxvf zabbix-3.0.3.tar.gz
    cd zabbix-3.0.3
    ./configure --prefix=/usr/local/zabbix --sysconfdir=/etc/zabbix/  --enable-agent 
    make &&make install

报错1：configure: error: Unable to use libpcre (libpcre check failed)

    yum -y install pcre* 
 
 或安装 pcre库，

     tar –zxvf pcre-8.36.tar.gz
     cd pcre-8.36
     ./configure 
     make&make install

错误2：configure: error: Not found mysqlclient library

     yum install mysql-devel 

错误3：configure: error : Not found NET-SNMP library

     yum install net-snmp-devel 

报错4：Unable to use libevent (libevent check failed)

    rpm -ivh --nodeps libevent-devel-1.4.13-4.el6.x86_64.rpm
    rpm -ivh --nodeps libevent-headers-1.4.13-4.el6.noarch

## （3）配置zabbix_agentd

    vim /etc/zabbix/zabbix_agentd.conf
    
    Server=X.X.X.X
    
    ServerActive=X.X.X.X

创建zabbix所需要的脚本目录

    ln -s /usr/local/zabbix/sbin/* /usr/sbin/
    
    cp /zabbix-3.4.3/misc/init.d/fedora/core/zabbix_agentd /etc/init.d/
    
    chmod +x /etc/init.d/zabbix_*
    
    sed -i "s@BASEDIR=/usr/local@BASEDIR=/usr/local/zabbix@g" /etc/init.d/zabbix_agentd

## （4）启动agentd

    /etc/init.d/zabbix_agent start 
    
错误1

      Starting zabbix_server:  /usr/local/zabbix/sbin/zabbix_server: error while loading shared libraries: libpcre.so.1: cannot open shared    object file: No such file or directory
      
     vim /etc/ld.so.conf
     /usr/local/lib  ## 加入

    [root@zabbix_server sbin]# ldconfig
重试
