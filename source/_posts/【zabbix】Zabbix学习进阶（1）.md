---
title: 【zabbix】Zabbix学习进阶（1）
date: 2019-05-22 13:32:44
tags:
- zabbix
---

## CentOS 7 安装Zabbix 4.0

### 1. 安装LAMP

```
 yum install -y httpd mariadb-server mariadb php php-mysql php-gd libjpeg* php-ldap php-odbc php-pear php-xml php-xmlrpc php-mhash php-mbstring php-bcmath
```

### 2. 修改Apache 配置

```
# vi /etc/httpd/conf/httpd.conf

ServerName zabbixserver
DirectoryIndex index.html index.php
```

### 3. 修改PHP 配置

```
# vi /etc/php.ini

date.timezone = PRC	//修改为中国时区
```

### 4. 修改Firewalld 和 Selinux 

```
systemctl stop firewalld

sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
setenforce 0

# 在 enforcing 模式下启用 SELinux 状态后，您需要执行以下命令以启用 Zabbix 前端和 Zabbix server 之间的通信：
setsebool -P httpd_can_connect_zabbix on
setsebool -P httpd_can_network_connect_db on
systemctl restart httpd
```

### 5. 启动服务

```
systemctl enable httpd mariadb
systemctl start httpd mariadb
```

### 6. 初始化数据库，设置root 用户密码

```
mysql_secure_installation
```

### 7. 测试LAMP 是否搭建成功

```
# vi /var/www/html/index.php

<?php
phpinfo();
?>

访问：http://server_ip_or_name
```



### 8. 添加zabbix 软件仓库

```
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
yum clean all
```

### 9. 修改软件仓库，使用国内源（清华源）

```
# vi /etc/yum.repos.d/zabbix.repo

[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/$basearch/
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/zabbix/non-supported/rhel/7/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=0
```

### 10. 安装Zabbix server，Web前端，agent

```
yum -y install zabbix-server-mysql zabbix-web-mysql zabbix-agent
```

### 11.  创建初始数据库

```
# mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> grant all privileges on zabbix.* to zabbix@localhost identified by 'password';
mysql> quit;
```

### 12. 导入初始架构和数据，系统将提示您输入新创建的密码

```
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

### 13. 为Zabbix server 配置数据库

```
# egrep -v "^$|^#" /etc/zabbix/zabbix_server.conf 

LogFile=/var/log/zabbix/zabbix_server.log
LogFileSize=0
PidFile=/var/run/zabbix/zabbix_server.pid
SocketDir=/var/run/zabbix
DBName=zabbix
DBUser=zabbix
DBPassword=<zabbix_user_password>
SNMPTrapperFile=/var/log/snmptrap/snmptrap.log
Timeout=4
AlertScriptsPath=/usr/lib/zabbix/alertscripts
ExternalScripts=/usr/lib/zabbix/externalscripts
LogSlowQueries=3000
```

### 14. 为Zabbix 前端配置PHP

```
# vi /etc/httpd/conf.d/zabbix.conf

php_value date.timezone Asia/Shanghai
```

### 15. 启动Zabbix server 和agent 进程

```
systemctl enable zabbix-server zabbix-agent
systemctl restart zabbix-server zabbix-agent httpd
```

### 16. 配置Zabbix 前端

```
访问：http://server_ip_or_name/zabbix 按提示输入相关信息
```

### 17. 开始使用Zabbix

```
用户名：Admin
密码：zabbix
```



## 被控端配置

### 1.  添加zabbix 软件仓库

```
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/x86_64/zabbix-release-4.0-1.el7.noarch.rpm
yum clean all
```

### 2. 修改软件仓库，使用国内源（清华源）

```
# vi /etc/yum.repos.d/zabbix.repo

[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/4.0/rhel/7/$basearch/
enabled=1
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=https://mirrors.tuna.tsinghua.edu.cn/zabbix/non-supported/rhel/7/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=0
```

### 3. 安装Zabbix Agent

```
yum install -y zabbix-agent
```

### 4. 配置 Zabbix Agent

```
# egrep -v "^$|^#" /etc/zabbix/zabbix_agentd.conf 

PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=<zabbix_server_ip>
ServerActive=<zabbix_server_ip>
Hostname=<hostname>
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

### 5. 修改Firewalld 和 Selinux 

```
systemctl stop firewalld

sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
```

### 6. 启动服务

```
systemctl enable zabbix-agent
systemctl restart zabbix-agent
```

### 7. 在WEB平台上增加被控主机

