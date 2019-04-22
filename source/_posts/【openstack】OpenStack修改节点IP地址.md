---
title: 【openstack】OpenStack修改节点IP地址
date:  2017-09-11 19:31:03
tags: 
    - openstack
---

### 1. 配置文件修改：

```bash
#!/bin/bash
#OpenStack node ip change 
echo '' 
echo '***********************************************' 
echo 'sed -i "s/oldIP/newIP/g" `grep -rl oldIP /etc/`' 
echo '***********************************************' 
echo '' 
echo 'eg: ./change_ip.sh 192.168.10.2 192.168.10.3' 
echo '' 
if [ -z "$1" ] 
then 
	echo "1st argument is empty!" 
	echo '' 
elif [ -z "$2" ]
then 
	grep -rl $1 /etc/ 
	grep -l $1 /root/keystonerc_admin 
	echo '' 
	echo "2nd argument is empty!" 
	echo '' 
else 
	sed -i "s/$1/$2/g" `grep -rl $1 /etc/` 
	sed -i "s/$1/$2/g" /root/keystonerc_admin 
	echo 'Successful' 
	echo '' 
fi

```

### 2. 数据库修改:

```mysql
mysql 
show databases; 
use keystone; 
show tables; 
select * from endpoint; 
upadte endpoint set url=replace(url,'$1','$2'); 
select * from endpoint; 
\q 

openstack-service restart 
openstack-status 

#echo "192.168.10.2 controller" >> /etc/hosts

```

