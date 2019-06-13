---
title: 【nagios】Nagios安装NDOUtils
date: 2019-05-23 09:47:50
tags:
- nagios
---

### 1. 连接被监控端出现Connection reset by peer 的问题

```
使用官方安装脚本（./fullinstall）在配置Allow host 时未输入监控端地址或输入错误导致的问题

# vi /etc/xinetd.d/nrpe
only_from       = 127.0.0.1 192.168.32.166

# systemctl restart xinetd

监控端测试：
/usr/local/nagios/libexec/check_nrpe -H 192.168.23.164
被监控端测试：
/usr/local/nagios/libexec/check_nrpe -H 127.0.0.1

```

### 2. 监控端只需要编译出check_nrpe 插件，并放到相应的位置即可

```
1. 直接从被监控端拷贝，并修改权限
2. 源码安装
./configure
make check_nrpe
make install-plugin

```



## CentOS 安装NDOUtils

### Prerequisites

Installing MySQL or MariaDB is required.

**CentOS 5.x / 6.x** 

```
yum install -y mysql mysql-server mysql-devel
```

**CentOS 7.x**

```
yum install -y mariadb mariadb-server mariadb-devel
```



### Start And Configure MySQL / MariaDB

Before configuring MySQL / MariaDB you must start the service and configure it to boot on startup.

**CentOS 5.x / 6.x**

```
service mysqld start
```

Check that it is running:

```
ps x | grep mysql | grep -v grep
```

Which should output something like:

```
 1969 pts/0    S      0:00 /bin/sh /usr/bin/mysqld_safe --datadir=/var/lib/mysql --socket=/var/lib/mysql/mysql.sock --pid-file=/var/run/mysqld/mysqld.pid --basedir=/usr --user=mysql
```

Configure it to start when the system boots:

```
chkconfig --add mysqld
chkconfig mysqld on
```

**CentOS 7.x**

```
systemctl start mariadb.service
```

Check that it is running:

```
ps ax | grep mysql | grep -v grep
```

Which should output something like:

```
2781 ?        Ss     0:00 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
2938 ?        Sl     0:00 /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock
```

Configure it to start when the system boots:

```
systemctl enable mariadb.service
```

 

### Define MySQL / MariaDB Root Password

Now to define the password for the root account in MySQL / MariaDB.

The password being defined is '**mypassword**' and will be used in future commands *(we suggest you use a more secure password)*.

The **'**single quotes**'** are used to define the boundaries of the password, this is extremely important when the password contains a space or special characters.

```
/usr/bin/mysqladmin -u root password 'mypassword'
```

 

### Password Note:

In future commands you will see the password is provided using the **-p** argument like follows:

```
mysql -u root -p'mypassword'
```

**NOTE:** It's very important to **NOT** put a space between the **-p** and the **'mypassword'**.

 

### Create Database

NDOUtils requires a database to be created which will be called **nagios**.

There will also be a dedicated user account called **ndoutils** with the password **ndoutils_password** *(we suggest you use a more secure password)*.

The storage location of the database will be the default location that MySQL / MariaDB uses, this can be changed however it is not covered in this guide.

This command will connect to the local MySQL / MariaDB database engine interface.

```
mysql -u root -p'mypassword'
```

Now execute these four commands *(press \**Enter** after each command)*:

```
CREATE DATABASE nagios DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
CREATE USER 'ndoutils'@'localhost' IDENTIFIED BY 'ndoutils_password';
GRANT USAGE ON *.* TO 'ndoutils'@'localhost' IDENTIFIED BY 'ndoutils_password' WITH MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ; 
GRANT ALL PRIVILEGES ON nagios.* TO 'ndoutils'@'localhost' WITH GRANT OPTION ; 
```

Now you can exit the local MySQL / MariaDB database engine interface.

```
\q
```

Run this command to ensure that the database has been created:

```
echo 'show databases;' | mysql -u ndoutils -p'ndoutils_password' -h localhost
```

The last command should output something like:

```
Database
information_schema
nagios
test
```

 

### Linux Kernel Settings

NDOUtils uses the kernel message queue for transferring the data from Nagios to NDOUtils. We are going to increase the default values the Kernel boots with to ensure it operates optimally.

First create a backup copy of the **/etc/sysctl.conf** file:

```
cp /etc/sysctl.conf /etc/sysctl.conf_backup
```

Now make the required changes:

```
sed -i '/msgmnb/d' /etc/sysctl.conf
sed -i '/msgmax/d' /etc/sysctl.conf
sed -i '/shmmax/d' /etc/sysctl.conf
sed -i '/shmall/d' /etc/sysctl.conf
printf "\n\nkernel.msgmnb = 131072000\n" >> /etc/sysctl.conf
printf "kernel.msgmax = 131072000\n" >> /etc/sysctl.conf
printf "kernel.shmmax = 4294967295\n" >> /etc/sysctl.conf
printf "kernel.shmall = 268435456\n" >> /etc/sysctl.conf
sysctl -e -p /etc/sysctl.conf
```

The last command should output something like:

```
net.ipv4.ip_forward = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.accept_source_route = 0
kernel.sysrq = 0
kernel.core_uses_pid = 1
net.ipv4.tcp_syncookies = 1
kernel.msgmnb = 131072000
kernel.msgmax = 131072000
kernel.shmmax = 4294967295
kernel.shmall = 268435456
```

There is no need to reboot the system, the last command ensured the new settings are active in the kernel.



### Downloading NDOUtils Source

```
cd /tmp
wget -O ndoutils.tar.gz https://github.com/NagiosEnterprises/ndoutils/releases/download/ndoutils-2.1.3/ndoutils-2.1.3.tar.gz
tar xzf ndoutils.tar.gz
```

 

### Compile NDOUtils

```
cd /tmp/ndoutils-2.1.3/
./configure
make all
```

 

### Install Binaries

This step installs the binary files.

```
make install
```

 

### Initialize Database

This prepares the database for NDOUtils.

```
cd db/
./installdb -u 'ndoutils' -p 'ndoutils_password' -h 'localhost' -d nagios
cd .. 
```

The command will produce output similar to the following:

```
DBD::mysql::db do failed: Table 'nagios.nagios_dbversion' doesn't exist at ./installdb line 52.
** Creating tables for version 2.0.1
     Using mysql.sql for installation...
** Updating table nagios_dbversion
Done!
```

That first line of output that says **failed: Table 'nagios.nagios_dbversion' doesn't exist** is expected, it's testing to make sure the database hasn't already been intiaialized.

 

### Install Configuration Files

This installs the config files as well as configuring the MySQL / MariaDB credentials so NDOUtils can connect to the database.

The two config files are:

**/usr/local/nagios/etc/ndo2db.cfg**

The following lines are defined in this file:

```
db_user=ndoutils
db_pass=ndoutils_password
```

You should make sure the username and password are correct.

**/usr/local/nagios/etc/ndomod.cfg**

No changes are required in this file.

Using the default username and password the following commands install the configuration files and make the required changes.

```
make install-config
mv /usr/local/nagios/etc/ndo2db.cfg-sample /usr/local/nagios/etc/ndo2db.cfg
sed -i 's/^db_user=.*/db_user=ndoutils/g' /usr/local/nagios/etc/ndo2db.cfg
sed -i 's/^db_pass=.*/db_pass=ndoutils_password/g' /usr/local/nagios/etc/ndo2db.cfg
mv /usr/local/nagios/etc/ndomod.cfg-sample /usr/local/nagios/etc/ndomod.cfg
```

 

### Install Service / Daemon

This installs the service or daemon files and configure them to start on boot.

**CentOS 5.x / 6.x**

```
make install-init
```

**CentOS 7.x**

```
make install-init
systemctl enable ndo2db.service
```

Information on starting and stopping services will be explained further on.

 

### Start Service / Daemon

Different Linux distributions have different methods of starting the ndo2db service.

**CentOS 5.x**

```
service ndo2db start
```

**CentOS 6.x**

```
start ndo2db
```

**CentOS 7.x**

```
systemctl start ndo2db.service
```

 

### Update Nagios To Use NDO Broker Module

Now you need to tell Nagios to use the NDO broker module. This is as simple as adding the following line to the **nagios.cfg** file:

```
broker_module=/usr/local/nagios/bin/ndomod.o config_file=/usr/local/nagios/etc/ndomod.cfg
```

The following commands will add that line as well as an extra line that explains what the module is for.

```
printf "\n\n# NDOUtils Broker Module\n" >> /usr/local/nagios/etc/nagios.cfg
printf "broker_module=/usr/local/nagios/bin/ndomod.o config_file=/usr/local/nagios/etc/ndomod.cfg\n" >> /usr/local/nagios/etc/nagios.cfg 
```

 

### Restart Nagios

Now you need to restart Nagios to use the NDO broker module. Different Linux distributions have different methods of restarting Nagios Core. We will also check the status of the Nagios service to ensure it's running after these changes.

**CentOS 5.x / 6.x**

```
service nagios restart
service nagios status
```

**CentOS 7.x**

```
systemctl restart nagios.service
systemctl status nagios.service
```

The last command should show Nagios running:

**CentOS 5.x / 6.x**

```
nagios (pid 5345) is running...
```

**CentOS 7.x**

```
● nagios.service - LSB: Starts and stops the Nagios monitoring server
   Loaded: loaded (/etc/rc.d/init.d/nagios)
   Active: active (running) since Tue 2016-10-04 12:31:00 AEDT; 22s ago
```

 

### Check NDOUtils Is Working

There are a couple of different ways to ensure NDO2DB is working.

This command will show Nagios successfully loaded the NDO module:

```
grep ndo /usr/local/nagios/var/nagios.log
```

The last command should output something like:

```
[1475544660] ndomod: NDOMOD 2.1.1 (09-06-2016) Copyright (c) 2009 Nagios Core Development Team and Community Contributors
[1475544660] ndomod: Successfully connected to data sink.  0 queued items to flush.
[1475544660] ndomod registered for process data
[1475544660] ndomod registered for timed event data
[1475544660] ndomod registered for log data'
[1475544660] ndomod registered for system command data'
[1475544660] ndomod registered for event handler data'
[1475544660] ndomod registered for notification data'
[1475544660] ndomod registered for service check data'
[1475544660] ndomod registered for host check data'
[1475544660] ndomod registered for comment data'
[1475544660] ndomod registered for downtime data'
[1475544660] ndomod registered for flapping data'
[1475544660] ndomod registered for program status data'
[1475544660] ndomod registered for host status data'
[1475544660] ndomod registered for service status data'
[1475544660] ndomod registered for adaptive program data'
[1475544660] ndomod registered for adaptive host data'
[1475544660] ndomod registered for adaptive service data'
[1475544660] ndomod registered for external command data'
[1475544660] ndomod registered for aggregated status data'
[1475544660] ndomod registered for retention data'
[1475544660] ndomod registered for contact data'
[1475544660] ndomod registered for contact notification data'
[1475544660] ndomod registered for acknowledgement data'
[1475544660] ndomod registered for state change data'
[1475544660] ndomod registered for contact status data'
[1475544660] ndomod registered for adaptive contact data'
[1475544660] Event broker module '/usr/local/nagios/bin/ndomod.o' initialized successfully.
```

This command will show you the database with populated data:

```
echo 'select * from nagios.nagios_logentries;' | mysql -u ndoutils -p'ndoutils_password'
```

The last command should output something like:

```
logentry_id    instance_id    logentry_time    entry_time    entry_time_usec    logentry_type    logentry_data    realtime_data    inferred_data_extracted
1    1    2016-10-04 12:31:00    2016-10-04 12:31:00    868450    262144    ndomod registered for log data'    1    1
2    1    2016-10-04 12:31:00    2016-10-04 12:31:00    868462    262144    ndomod registered for system command data'    1    1
3    1    2016-10-04 12:31:00    2016-10-04 12:31:00    868467    262144    ndomod registered for event handler data'    1    1
4    1    2016-10-04 12:31:00    2016-10-04 12:31:00    868472    262144    ndomod registered for notification data'    1    1
5    1    2016-10-04 12:31:00    2016-10-04 12:31:00    868479    262144    ndomod registered for service check data'    1    1
6    1    2016-10-04 12:31:00    2016-10-04 12:31:00    868486    262144    ndomod registered for host check data'    1    1
7    1    2016-10-04 12:31:00    2016-10-04 12:31:00    868491    262144    ndomod registered for comment data'    11
8    1    2016-10-04 12:31:00    2016-10-04 12:31:00    868496    262144    ndomod registered for downtime data'11
9    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869866    262144    ndomod registered for flapping data'11
10    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869878    262144    ndomod registered for program status data'    1    1
11    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869884    262144    ndomod registered for host status data'    1    1
12    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869888    262144    ndomod registered for service status data'    1    1
13    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869893    262144    ndomod registered for adaptive program data'    1    1
14    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869897    262144    ndomod registered for adaptive host data'    1    1
15    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869902    262144    ndomod registered for adaptive service data'    1    1
16    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869906    262144    ndomod registered for external command data'    1    1
17    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869911    262144    ndomod registered for aggregated status data'    1    1
18    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869915    262144    ndomod registered for retention data'1    1
19    1    2016-10-04 12:31:00    2016-10-04 12:31:00    869920    262144    ndomod registered for contact data'    11
20    1    2016-10-04 12:31:00    2016-10-04 12:31:00    871043    262144    ndomod registered for contact notification data'    1    1
21    1    2016-10-04 12:31:00    2016-10-04 12:31:00    871055    262144    ndomod registered for acknowledgement data'    1    1
22    1    2016-10-04 12:31:00    2016-10-04 12:31:00    871062    262144    ndomod registered for state change data'    1    1
23    1    2016-10-04 12:31:00    2016-10-04 12:31:00    871067    262144    ndomod registered for contact status data'    1    1
24    1    2016-10-04 12:31:00    2016-10-04 12:31:00    871072    262144    ndomod registered for adaptive contact data'    1    1
25    1    2016-10-04 12:31:00    2016-10-04 12:31:00    871077    262144    Event broker module '/usr/local/nagios/bin/ndomod.o' initialized successfully.    1    1
26    1    2016-10-04 12:31:00    2016-10-04 12:31:00    874858    262144    Successfully launched command file worker with pid 6026    1    1
```

 

### Service Commands

Different Linux distributions have different methods of starting / stopping / restarting / status ndo2db.

**CentOS 5.x**

```
service ndo2db start
service ndo2db stop
service ndo2db restart
service ndo2db status
```

**CentOS 6.x**

```
start ndo2db
stop ndo2db
restart ndo2db
status ndo2db
```

**CentOS 7.x**

```
systemctl start ndo2db.service
systemctl stop ndo2db.service
systemctl restart ndo2db.service
systemctl status ndo2db.service
```