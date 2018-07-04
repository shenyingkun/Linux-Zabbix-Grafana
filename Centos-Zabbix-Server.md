# Zabbix内网安装部署

## （1）安装apache

    yum install httpd libxml2-devel net-snmp-devel libcurl-devel

## （2）安装php

Zabbix 对PHP的要求最低为5.4，故需要将PHP升级到5.4以上

可以在线安装升级

    rpm -ivh http://repo.webtatic.com/yum/el6/latest.rpm
    yum install php56w php56w-gd php56w-mysql php56w-bcmath php56w-mbstring php56w-xml php56w-ldap

或直接下载RPM包安装，前提需要先将老版本PHP卸载干净

卸载

    rpm -e --nodeps php-pgsql-5.3.3-49.el6.x86_64
    rpm -e --nodeps php-odbc-5.3.3-49.el6.x86_64
    rpm -e --nodeps php-xmlrpc-5.3.3-49.el6.x86_64 
    rpm -e --nodeps php-common-5.3.3-49.el6.x86_64
    rpm -e --nodeps php-pdo-5.3.3-49.el6.x86_64
    rpm -e --nodeps php-pear-1.9.4-5.el6.noarch
    rpm -e --nodeps php-mysql-5.3.3-49.el6.x86_64 
    rpm -e --nodeps php-soap-5.3.3-49.el6.x86_64
    rpm -e --nodeps php-gd-5.3.3-49.el6.x86_64
    rpm -e --nodeps php-pecl-memcache-3.0.5-4.el6.x86_64
    rpm -e --nodeps php-5.3.3-49.el6.x86_64
    rpm -e --nodeps php-cli-5.3.3-49.el6.x86_64
    rpm -e --nodeps php-ldap-5.3.3-49.el6.x86_64
    rpm -e --nodeps php-xml-5.3.3-49.el6.x86_64

安装 PHP rpm包（注意安装顺序）

    rpm -ivh libXpm-3.5.10-2.el6.x86_64.rpm
    yum install -y  gcc*
    yum install -y httpd
    rpm -ivh t1lib-5.1.2-6.el6_2.1.x86_64.rpm 
    rpm -ivh php56w-common-5.6.33-1.w6.x86_64.rpm 
    rpm -ivh php56w-cli-5.6.33-1.w6.x86_64.rpm 
    rpm -ivh php56w-5.6.33-1.w6.x86_64.rpm 
    rpm -ivh php56w-bcmath-5.6.33-1.w6.x86_64.rpm
    rpm -ivh php56w-gd-5.6.33-1.w6.x86_64.rpm 
    rpm -ivh php56w-ldap-5.6.33-1.w6.x86_64.rpm 
    rpm -ivh php56w-mbstring-5.6.33-1.w6.x86_64.rpm
    rpm -ivh php56w-pdo-5.6.33-1.w6.x86_64.rpm
    rpm -ivh php56w-mysql-5.6.33-1.w6.x86_64.rpm  
    rpm -ivh php56w-xml-5.6.33-1.w6.x86_64.rpm 

查看PHP版本 

    php -v
    5.6.33

修改PHP参数，不修改参数会在安装界面报参数错误（注意：前面；号要去掉）

    vim /etc/php.ini
    date.timezone = Asia/Shanghai #去掉分号
    post_max_size = 32M
    max_execution_time = 300
    max_input_time = 300
    always_populate_raw_post_data = -1 #去掉分号
    
    sed -i "s/;date.timezone =/date.timezone = Asia\/\Shanghai/g" /etc/php.ini 
    sed -i "s/post_max_size = 8M/post_max_size = 32M/g" /etc/php.ini
    sed -i "s/max_execution_time = 30/max_execution_time = 300/g" /etc/php.ini
    sed -i "s/max_input_time = 60/max_input_time = 300/g" /etc/php.ini
    sed -i "s/;always_populate_raw_post_data = -1/always_populate_raw_post_data = -1/g" /etc/php.ini
    
    /usr/bin/php &  #启动服务

## （3）安装mysql 系统安装的时候可以提前把MySQL安装上，或者制作安装源在线安装

    yum -y install mysql-5 mysql-connector-odbc mysql-libs mysql-server libdbi-dbd-mysql mysql-connector-java mysql-bench mysql-test mysql-devel
    
    service mysqld start  #启动mysql
    
    chkconfig mysqld on

## (4) 在mysql中创建zabbix所需要的库和用户

    mysql -uroot -p
    mysql> CREATE DATABASE zabbix CHARACTER SET utf8 COLLATE utf8_bin;
    mysql> GRANT ALL PRIVILEGES ON zabbix.* TO zabbix@localhost IDENTIFIED BY 'zabbix';
    mysql> flush privileges; 
    mysql> show databases; 
    +--------------------+   
    | Database          |   
    +--------------------+   
    | information_schema |   
    | mysql              |   
    | performance_schema |   
    | zabbix            |   
    +--------------------+

## (5) 安装zabbix

    groupadd zabbix
    useradd -g zabbix -u 201 -m zabbix

安装包可在线下载，内网安装需要离线下载并上传服务器

    wget http://sourceforge.net/projects/zabbix/files/ZABBIX%20Latest%20Stable/3.0.3/zabbix-3.0.3.tar.gz
    tar zxvf zabbix-3.0.3.tar.gz
    cd zabbix-3.0.3
    ./configure --prefix=/usr/local/zabbix --sysconfdir=/etc/zabbix/ --enable-server --enable-agent --with-net-snmp --with-libcurl --with-mysql --with-libxml2
    make &&make install

报错1：configure: error: Unable to use libpcre (libpcre check failed)

    yum -y install pcre* 
 
 或安装 pcre库，

     tar –zxvf pcre-8.36.tar.gz
     cd pcre-8.36
     ./configure 
     make&&make install

错误2：configure: error: Not found mysqlclient library

     yum install mysql-devel 

错误3：configure: error : Not found NET-SNMP library

     yum install net-snmp-devel 

报错4：Unable to use libevent (libevent check failed)

    rpm -ivh --nodeps libevent-devel-1.4.13-4.el6.x86_64.rpm
    rpm -ivh --nodeps libevent-headers-1.4.13-4.el6.noarch
    rpm -ivh --nodeps libevent-doc-1.4.13-4.el6.noarch.rpm

## (6)按顺序导入zabbix库

    cd  /root/zabbix-3.0.3/database/mysql
    mysql -uzabbix -pzabbix zabbix < database/mysql/schema.sql
    mysql -uzabbix -pzabbix zabbix < database/mysql/images.sql
    mysql -uzabbix -pzabbix zabbix < database/mysql/data.sql

## （7）配置zabbix_server

    vi /etc/zabbix/zabbix_server.conf
    DBHost=localhost  #//数据库ip地址
    DBName=zabbix #//
    DBUser=zabbix #//
    DBPassword=zabbix #//
    ListenIP=192.168.10.10  #// zabbix server ip地址
    StartIPMIPollers=10
    StartPollersUnreachable=10
    StartTrappers=10
    StartPingers=10
    StartDiscoverers=10
    CacheSize=256M
    StartDBSyncers=40
    HistoryCacheSize=128M
    TrendCacheSize=128M
    HistoryTextCacheSize=128M
    ValueCacheSize=128M
    Timeout=30
    AlertScriptsPath=/etc/zabbix/alertscripts      #//修改
    ExternalScripts=/etc/zabbix/externalscripts    #//修改
    LogSlowQueries=10000
    StartProxyPollers=50

创建zabbix所需要的脚本目录

    mkdir /etc/zabbix/alertscripts
    mkdir /etc/zabbix/externalscripts
    ln -s /usr/local/zabbix/sbin/* /usr/sbin/
    cp /zabbix-3.0.3/misc/init.d/fedora/core/zabbix_* /etc/init.d/  #复制服务启动脚本
    chmod +x /etc/init.d/zabbix_*
    sed -i "s@BASEDIR=/usr/local@BASEDIR=/usr/local/zabbix@g" /etc/init.d/zabbix_server
    sed -i "s@BASEDIR=/usr/local@BASEDIR=/usr/local/zabbix@g" /etc/init.d/zabbix_agentd
## （8）配置web

    vi /etc/httpd/conf/httpd.conf
    添加
    ServerName 127.0.0.1
    DocumentRoot  "/var/www/html"
    
    创建目录
    mkdir -p /var/www/html/zabbix
    cp -r /zabbix-3.0.3/frontends/php/* /var/www/html/zabbix/
    chown -R apache.apache /var/www/html/zabbix/
    启动
    /etc/init.d/zabbix_server start
    /etc/init.d/zabbix_agentd start
    service httpd restart
    chkconfig httpd on
    chkconfig zabbix_server on
    chkconfig zabbix_agentd on
    
    错误1
     Starting zabbix_server:  /usr/local/zabbix/sbin/zabbix_server: error while loading shared libraries: libpcre.so.1: cannot open shared    object file: No such file or directory
     vim /etc/ld.so.conf
     加入
    /usr/local/lib
    [root@zabbix_server sbin]# ldconfig
    重试
## (9)在web页面配置zabbixserver

用浏览器访问 http://192.168.190.133/zabbix/

问题1

WEB 页面报错 At least one of MySQL, SQL, Oracle or IBM DB2 should be supported.

由RPM包 php56w-mysql-5.6.33-1.w6.x86_64 导致，需要重新安装

问题2

zabbix在安装后，在主页面一直提示：zabbix server is not running the information displayed may not be current.

将文件/etc/zabbix/zabbix_server.conf中的ListenIP=127.0.0.1注释掉即可！

问题3

安装后提示：Get value from agent failed: cannot connect to [[127.0.0.1]:10050]: [111] Connection refused

主机配置选项 agent 配置错误

问题4

Received empty response from Zabbix Agent at [192.168.1.2]. Assuming that agent dropped connection because of access permission

大概意思是说没有权限访问agent端口10050，解决方法如下：

cat zabbix_agentd.conf| grep Server=
Server=X.X.X.X  # zabbix server ip地址配置无误
