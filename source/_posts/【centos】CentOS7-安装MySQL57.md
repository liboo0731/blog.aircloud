---
title: 【centos】CentOS7-安装MySQL5.7
date: 2018-03-21 21:14:19
tags:
- centos
- centos7
- mysql
---

### 配置 yum 源

在 <https://dev.mysql.com/downloads/repo/yum/> 找到 yum 源 rpm 安装包

### 安装 mysql 源

#### 下载

shell> wget <https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm>

#### 安装 mysql 源

shell> yum localinstall mysql57-community-release-el7-11.noarch.rpm

#### 用下面的命令检查 mysql 源是否安装成功

shell> yum repolist enabled | grep "mysql.*-community.*"



### 安装 MySQL

#### 使用 yum install 命令安装

shell> yum install mysql-community-server

#### 启动 MySQL 服务，在 CentOS 7 下，新的启动/关闭服务的命令是 systemctl start|stop

shell> systemctl start mysqld

#### 用 systemctl status 查看 MySQL 状态

shell> systemctl status mysqld

#### MySQL 启动状态，设置开机启动

shell> systemctl enable mysqld

shell> systemctl daemon-reload

#### 修改 root 本地账户密码，mysql 安装完成之后，生成的默认密码在 /var/log/mysqld.log 文件中。使用 grep 命令找到日志中的密码。

shell> grep 'temporary password' /var/log/mysqld.log

#### 查看临时密码，首次通过初始密码登录后，使用以下命令修改密码

shell> mysql -uroot -p mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'MyNewPass4!';

或者

mysql> set password for 'root'@'localhost'=password('MyNewPass4!');

#### 以后通过 update set 语句修改密码

mysql> use mysql; mysql> update user set password=PASSWORD('MyNewPass5!') where user='root';

注意：mysql 5.7 默认安装了密码安全检查插件（validate_password），默认密码检查策略要求密码必须包含：大小写字母、数字和特殊符号，并且长度不能少于8位。否则会提示 ERROR 1819 (HY000): Your password does not satisfy the current policy requirements 错误。查看 MySQL官网密码详细策略。



### 添加远程登录用户

默认只允许root帐户在本地登录，如果要在其它机器上连接mysql，必须添加一个允许远程连接的帐户。或者修改 root 为允许远程连接（不推荐）

### 添加一个允许远程连接的帐户

mysql> GRANT ALL PRIVILEGES ON *.* TO 'zhangsan'@'%' IDENTIFIED BY 'Zhangsan2018!' WITH GRANT OPTION;

### 修改 root 为允许远程连接（不推荐）

mysql> use mysql; mysql> UPDATE user SET Host='%' WHERE User='root'; mysql> flush privileges;

### 设置默认编码为 utf8

mysql 安装后默认不支持中文，需要修改编码。

修改 /etc/my.cnf 配置文件，在相关节点（没有则自行添加）下添加编码配置，如下：

```shell
[mysqld]
character-set-server=utf8

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8
```

重启mysql服务，查询编码。可以看到已经改过来了

shell> systemctl restart mysqld 

shell> mysql -uroot -p 

mysql> show variables like 'character%';

### 默认配置文件路径：

配置文件：/etc/my.cnf

日志文件：/var/log/var/log/mysqld.log

服务启动脚本：/usr/lib/systemd/system/mysqld.service

socket文件：/var/run/mysqld/mysqld.pid