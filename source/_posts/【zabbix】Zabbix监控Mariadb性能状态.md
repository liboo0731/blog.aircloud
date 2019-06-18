---
title: 【zabbix】Zabbix监控Mariadb性能状态
date: 2019-05-29 16:29:34
tags:
- zabbix
---

# Zabbix 监控Mariadb 性能状态

Zabbix 内置Mysql 的监控模版，因为Mariadb 和Mysql 两者的相关性，所以这个模版也能用在Mariadb Services上

### Mariadb 

首先要在mariadb新建一个账户，这个账户不需要有任何权限。这个账户只是用来登入mariadb获取服务状态。

```
GRANT USAGE ON *.* TO 'zabbix'@'localhost' IDENTIFIED BY 'passwd';
FLUSH PRIVILEGES;
```

### Zabbix Agent

完成mariadb的用户添加后，还要在mariadb服务器安装Zabbix Agent。通过以下文件可以得知还需要新建一个文件，并在这个文件内填入mariadb的信息：

```
/etc/zabbix/zabbix_agentd.d/userparameter_mysql.conf 

# For all the following commands HOME should be set to the directory that has .my.cnf file with password information.

# Flexible parameter to grab global variables. On the frontend side, use keys like mysql.status[Com_insert].
# Key syntax is mysql.status[variable].
UserParameter=mysql.status[*],echo "show global status where Variable_name='$1';" | HOME=/var/lib/zabbix mysql -N | awk '{print $$2}'

# Flexible parameter to determine database or table size. On the frontend side, use keys like mysql.size[zabbix,history,data].
# Key syntax is mysql.size[<database>,<table>,<type>].
# Database may be a database name or "all". Default is "all".
# Table may be a table name or "all". Default is "all".
# Type may be "data", "index", "free" or "both". Both is a sum of data and index. Default is "both".
# Database is mandatory if a table is specified. Type may be specified always.
# Returns value in bytes.
# 'sum' on data_length or index_length alone needed when we are getting this information for whole database instead of a single table
UserParameter=mysql.size[*],bash -c 'echo "select sum($(case "$3" in both|"") echo "data_length+index_length";; data|index) echo "$3_length";; free) echo "data_free";; esac)) from information_schema.tables$([[ "$1" = "all" || ! "$1" ]] || echo " where table_schema=\"$1\"")$([[ "$2" = "all" || ! "$2" ]] || echo "and table_name=\"$2\"");" | HOME=/var/lib/zabbix mysql -N'

UserParameter=mysql.ping,HOME=/var/lib/zabbix mysqladmin ping | grep -c alive
UserParameter=mysql.version,mysql -V
```

然后新建以下文件并进行修改，要注意的是 /var/lib/zabbix 这个路径可能并不存在，需要手动新建：

```
# mkdir -p /var/lib/zabbix

# vi /var/lib/zabbix/.my.cnf

[mysql]
user=zabbix
password=zabbix
host=127.0.0.1

[mysqladmin]
user=zabbix
password=zabbix
host=127.0.0.1
```

完成后需要通过以下命令手动重启mariadb services和zabbix agent services：

```
重启mariadb
# systemctl restart mariadb

重启zabbix agent
# systemctl restart zabbix-agent
```