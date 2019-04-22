---
title: 【fastdfs】使用普通用户安装fastDFS
date: 2019-02-26 19:56:31
tags: 
    - fastdfs
---

# 普通用户安装fastDFS

## 一、安装依赖

```shell
yum install -y zlib zlib-devel pcre pcre-devel gcc gcc-c++ openssl openssl-devel libevent libevent-devel perl unzip
```



## 二、创建fastdfs用户并设置密码

```shell
useradd fastdfs
passwd fastdfs
```



## 三、下载libfastcommon源码进行安装

```shell
cd /tmp
wget -c https://github.com/happyfish100/libfastcommon/archive/V1.0.7.tar.gz
tar -xvf V1.0.7.tar.gz
cd libfastcommon-1.0.7
./make.sh
./make.sh install
```



## 四、设置软连接

```shell
 # 此时 libfastcommon.so默认安装到了/usr/lib64/libfastcommon.so，而FastDFS主程序设置的lib目录是/usr/local/lib，所以设置软连接
 ln -s /usr/lib64/libfastcommon.so /usr/local/lib/libfastcommon.so
 ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so
 ln -s /usr/lib64/libfdfsclient.so /usr/local/lib/libfdfsclient.so
 ln -s /usr/lib64/libfdfsclient.so /usr/lib/libfdfsclient.so
```



## 五、使用fastdfs登录系统，在/home/fastdfs创建目录

```shell
mkdir src
mkdir tracker
mkdir storage
```



## 六、下载fastdfs5.05源码并解压

```shell
cd src
wget -c https://github.com/happyfish100/fastdfs/archive/V5.05.tar.gz
tar -zxvf V5.05.tar.gz
cd fastdfs-5.05
```



## 七、修改make.sh并进行安装

```shell
修改make.sh中的TARGET_PREFIX、TARGET_CONF_PATH和TARGET_INIT_PATH：
TARGET_PREFIX=/home/fastdfs
TARGET_CONF_PATH=/home/fastdfs/conf
TARGET_INIT_PATH=/home/fastdfs/init.d

然后执行
./make.sh
./make.sh install
注：安装时报如下的错误不用理会，因为配置文件在/home/fastdfs/conf里

mkdir: cannot create directory `/etc/fdfs': Permission denied
```



## 八、修改配置文件名，去掉.sample后缀

```shell
cd /home/fastdfs/conf
mv client.conf.sample client.conf
mv storage.conf.sample storage.conf
mv storage_ids.conf.sample  storage_ids.conf
mv tracker.conf.sample tracker.conf
```



## 九、修改storage.conf

```shell
group_name=group1
bind_addr=121.42.155.178 #改为实际的地址
base_path=/home/fastdfs/storage
store_path0=/home/fastdfs/storage
tracker_server=121.42.155.178:22122 #改为实际的地址
run_by_group=fastdfs
run_by_user=fastdfs
```



## 十、修改tracker.conf

```shell
bind_addr=121.42.155.178 #改为实际的地址
base_path=/home/fastdfs/tracker
run_by_group=fastdfs
run_by_user=fastdfs
use_storage_id = true
storage_ids_filename = storage_ids.conf
id_type_in_filename = id
```



## 十一、修改storage_ids.conf

```shell
100001   group1  121.42.155.178 #改为实际的地址
```



## 十二、修改client.conf

```shell
base_path=/home/fastdfs
tracker_server=121.42.155.178:22122 #改为实际的地址
```



## 十三、修改 /home/fastdfs/init.d/fdfs_storaged

```shell
PRG=/home/fastdfs/bin/fdfs_storaged
CONF=/home/fastdfs/conf/storage.conf

将fdfs_storaged中所有的usr/local替换为home/fastdfs：
sed  -i  's/usr\/local/home\/fastdfs/g'  fdfs_storaged 
```



## 十四、修改 /home/fastdfs/init.d/fdfs_trackerd

```shell
PRG=/home/fastdfs/bin/fdfs_trackerd
CONF=/home/fastdfs/conf/tracker.conf

将fdfs_trackerd中所有的usr/local替换为home/fastdfs：
sed  -i  's:usr/local:home/fastdfs:g'  fdfs_trackerd
```



## 十五、启动

```shell
/home/fastdfs/init.d/fdfs_trackerd start
/home/fastdfs/init.d/fdfs_storaged start
```



## 十六、常见问题

```shell
1.如何让server进程退出运行？

直接kill即可让server进程正常退出，可以使用killall命令，例如：

killall fdfs_trackerd

killall fdfs_storaged

也可以使用如下命令：

/home/fastdfs/bin/fdfs_trackerd /home/fastdfs/conf/tracker.conf stop

/home/fastdfs/bin/fdfs_storaged /home/fastdfs/conf/storage.conf stop

千万不要使用-9参数强杀，否则可能会导致binlog数据丢失的问题。

 

2.如何查看storage状态？

fdfs_monitor  /home/fastdfs/conf/storage.conf
```



# 普通用户开机启动服务

```shell
# 1. 使用root账户登录，修改/etc/rc.local 文件
[root@host-172-16-2-210 ~]# cat /etc/rc.local 
#!/bin/bash
# THIS FILE IS ADDED FOR COMPATIBILITY PURPOSES
#
# It is highly advisable to create own systemd services or udev rules
# to run scripts during boot instead of using this file.
#
# In contrast to previous versions due to parallel execution during boot
# this script will NOT be run after all other services.
#
# Please note that you must run 'chmod +x /etc/rc.d/rc.local' to ensure
# that this script will be executed during boot.

touch /var/lock/subsys/local

su - test -c "/bin/sh /home/test/init.d/fdfs_trackerd start"
su - test -c "/bin/sh /home/test/init.d/fdfs_storaged start"

# 2. 给/etc/rc.local 文件赋予执行权限
chmod +x /etc/rc.d/rc.local

# 3. 禁用防火墙
systemctl disable firewalld
systemctl stop firewalld

# 4. 重启测试

```



