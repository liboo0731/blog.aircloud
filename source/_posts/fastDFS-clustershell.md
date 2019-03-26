---
title: 使用ClusterShell安装fastDFS
date: 2019-02-27 20:36:21
tags: 

- fastDFS
- clustershell
---

# clustershell 使用

### 1. 部署clush环境

```shell
# 安装clush（可以yum直接安装，也可以源码安装）
shell> yum install -y clustershell

# clush命令：
clush -a 全部 等于 clush -g all
clush -g 指定组
clush -w 操作主机名字，多个主机之间用逗号隔开
clush -g 组名 -c --dest 文件群发 （-c等于--copy）

# 注意：clush 是不支持环境变量的$PATH

# 常用的是下面几个参数：
-g 后面指定设置的组
-a 表示所有的组
-w 后面跟主机节点，多个主机中间用逗号隔开
-x 表示去掉某个节点进行操作。后面跟主机节点，多个主机中间用逗号隔开
-X 表示去掉某个组进行操作，多个组之间用逗号隔开
-b 相同输出结果合并

# 注意，clush操作远程机器，执行动作要放在双引号或单引号内进行
```

### 2. 配置 /etc/hosts

```
eg:
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.214 host-172-16-2-208
```

### 3. 配置 /etc/clustershell/groups

```shell
# 配置clush：
# 在/etc/clustershell目录下，手动创建groups文件
eg:
shell> touch /etc/clustershell/groups
shell> vim /etc/clustershell/groups
all: a1 host1 host2
name:host3 host4

# 需要注意的是all 是必须配置的，clush 有 -a 这个参数，主机间用空格分离。

eg:
shell> touch /etc/clustershell/groups
shell> vim /etc/clustershell/groups
all: host-172-16-2-208
```

### 4. 配置免密登录

```shell
# 如果没有事先生成ssh密匙的话，需要先生成：
shell> ssh-keygen
# 可选操作：为了方便，我们可以给需要登录的服务器起一个可读性更好的别名，如果你做了类似的操作，那么后面的<USER>@<SERVER>都可以换成对应的<HOST>：
shell> cat ~/.ssh/config
Host db_1
Hostname <SERVER>
User <USER>
Port <PORT>

Host db_2
Hostname <SERVER>
User <USER>
Port <PORT>

Host db_3
Hostname <SERVER>
User <USER>
Port <PORT>
# 然后把生成的公钥添加到需要登录的服务器指定位置：
shell> cat ~/.ssh/id_rsa.pub | ssh <USER>@<SERVER> "cat - >> ~/.ssh/authorized_keys"

# 如果你和我一样总记不清如何正确拼写authorized_keys，可以接着学一下ssh-copy-id的用法，这个命令可以让操作更简单点：
shell> ssh-copy-id -i ~/.ssh/id_rsa.pub "<USER>@<SERVER>"

# 注：每配置好一台免密码登录的服务器，最好手动实际操作一下，因为第一次连接会要求手动确认是否保存信息到~/.ssh/known_hosts文件。

eg:
[root@host-172-16-2-207 ~]# ssh-copy-id host-172-16-2-208
The authenticity of host 'host-172-16-2-208 (192.168.10.214)' can't be established.
ECDSA key fingerprint is 16:2a:df:68:97:e0:2a:e1:c2:3c:d7:ac:99:d7:76:d0.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@host-172-16-2-208's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'host-172-16-2-208'"
and check to make sure that only the key(s) you wanted were added.

```

### 5. 利用clush管理命令进行远程机器集群管理

```shell
# ssh首次登陆的时候，会提示输入"yes/no"，需要提前将这个执行。
shell> ssh 'host-172-16-2-208'

eg:
shell> clush -g db uptime
shell> clush -a hostname
shell> clush -b -a hostname
shell> clush -a "echo asdfsdf > /tmp/test"
```

# fastDFS 部署

## 1. 环境准备

### 1.1 使用的系统软件

| 名称                 | 说明                          |
| -------------------- | ----------------------------- |
| centos               | 7.x                           |
| libfatscommon        | FastDFS分离出的一些公用函数包 |
| FastDFS              | FastDFS本体                   |
| fastdfs-nginx-module | FastDFS和nginx的关联模块      |
| nginx                | nginx1.15.4                   |

### 1.2 编译环境

```
yum install git gcc gcc-c++ make automake autoconf libtool pcre pcre-devel zlib zlib-devel openssl-devel wget vim -y
```

### 1.3 磁盘目录

| 说明                                   | 位置           |
| -------------------------------------- | -------------- |
| 所有安装包                             | /usr/local/src |
| 数据存储位置                           | /home/dfs/     |
| #这里我为了方便把日志什么的都放到了dfs |                |

```
mkdir /home/dfs #创建数据存储目录
cd /usr/local/src #切换到安装目录准备下载安装包
```

### 1.4 安装libfatscommon

```
git clone https://github.com/happyfish100/libfastcommon.git --depth 1
cd libfastcommon/
./make.sh && ./make.sh install #编译安装
```

### 1.5 安装FastDFS

```
cd ../ #返回上一级目录
git clone https://github.com/happyfish100/fastdfs.git --depth 1
cd fastdfs/
./make.sh && ./make.sh install #编译安装
#配置文件准备
cp /etc/fdfs/tracker.conf.sample /etc/fdfs/tracker.conf
cp /etc/fdfs/storage.conf.sample /etc/fdfs/storage.conf
cp /etc/fdfs/client.conf.sample /etc/fdfs/client.conf #客户端文件，测试用
cp /usr/local/src/fastdfs/conf/http.conf /etc/fdfs/ #供nginx访问使用
cp /usr/local/src/fastdfs/conf/mime.types /etc/fdfs/ #供nginx访问使用
```

### 1.6 安装fastdfs-nginx-module

```
cd ../ #返回上一级目录
git clone https://github.com/happyfish100/fastdfs-nginx-module.git --depth 1
cp /usr/local/src/fastdfs-nginx-module/src/mod_fastdfs.conf /etc/fdfs
```

### 1.7 安装nginx

```
wget http://nginx.org/download/nginx-1.15.4.tar.gz #下载nginx压缩包
tar -zxvf nginx-1.15.4.tar.gz #解压
cd nginx-1.15.4/
#添加fastdfs-nginx-module模块
./configure --add-module=/usr/local/src/fastdfs-nginx-module/src/ 
make && make install #编译安装
```

## 2. rpm 安装

### 2.1 编译后的rpm 安装包 

```sh
[root@host-172-16-2-207 ~]# ll fdfs/
total 512
-rw-r--r--. 1 root root   1924 Feb 12 01:04 fastdfs-5.0.12-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root 170680 Feb 12 01:05 fastdfs-server-5.0.12-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root 133872 Feb 12 01:05 fastdfs-tool-5.0.12-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root 110888 Feb 12 01:05 libfastcommon-1.0.40-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root  40664 Feb 12 01:05 libfastcommon-devel-1.0.40-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root  36428 Feb 12 01:05 libfdfsclient-5.0.12-1.el7.centos.x86_64.rpm
-rw-r--r--. 1 root root  18284 Feb 12 01:05 libfdfsclient-devel-5.0.12-1.el7.centos.x86_64.rpm
[root@host-172-16-2-207 ~]# yum install -y ./fdfs/*.rpm

```

### 2.2 默认数据存储及日志位置

```shell
[root@host-172-16-2-207 ~]# mkdir -p /home/yuqing/fastdfs/
# 服务正常启动后
[root@host-172-16-2-207 ~]# ll /home/yuqing/fastdfs/
total 12
drwxr-xr-x. 259 root root 8192 Feb 12 03:17 data
drwxr-xr-x.   2 root root   46 Feb 12 01:13 logs
```

## 3. 单机部署

### 3.1 tracker配置

```
#服务器ip为 192.168.52.1
#我建议用ftp下载下来这些文件 本地修改
vim /etc/fdfs/tracker.conf
#需要修改的内容如下
port=22122  # tracker服务器端口（默认22122,一般不修改）
base_path=/home/dfs  # 存储日志和数据的根目录
```

### 3.2 storage配置

```
vim /etc/fdfs/storage.conf
#需要修改的内容如下
port=23000  # storage服务端口（默认23000,一般不修改）
base_path=/home/dfs  # 数据和日志文件存储根目录
store_path0=/home/dfs  # 第一个存储目录
tracker_server=192.168.52.1:22122  # tracker服务器IP和端口
http.server_port=8888  # http访问文件的端口(默认8888,看情况修改,和nginx中保持一致)
```

### 3.3 client测试

```
vim /etc/fdfs/client.conf
#需要修改的内容如下
base_path=/home/dfs
tracker_server=192.168.52.1:22122    #tracker服务器IP和端口
#保存后测试,返回ID表示成功 如：group1/M00/00/00/xx.tar.gz
fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/nginx-1.15.4.tar.gz
```

### 3.4 配置nginx访问

```
vim /etc/fdfs/mod_fastdfs.conf
#需要修改的内容如下
tracker_server=192.168.52.1:22122  #tracker服务器IP和端口
url_have_group_name=true
store_path0=/home/dfs
#配置nginx.config
vim /usr/local/nginx/conf/nginx.conf
#添加如下配置
server {
    listen       8888;    ## 该端口为storage.conf中的http.server_port相同
    server_name  localhost;
    location ~/group[0-9]/ {
        ngx_fastdfs_module;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    root   html;
    }
}
#测试下载，用外部浏览器访问刚才已传过的nginx安装包,引用返回的ID
http://192.168.52.1:8888/group1/M00/00/00/wKgAQ1pysxmAaqhAAA76tz-dVgg.tar.gz
#弹出下载单机部署全部跑通
```

## 4. 分布式部署

### 4.1 tracker配置

```
#服务器ip为 192.168.52.2,192.168.52.3,192.168.52.4
#我建议用ftp下载下来这些文件 本地修改
vim /etc/fdfs/tracker.conf
#需要修改的内容如下
port=22122  # tracker服务器端口（默认22122,一般不修改）
base_path=/home/dfs  # 存储日志和数据的根目录
```

### 4.2 storage配置

```
vim /etc/fdfs/storage.conf
#需要修改的内容如下
port=23000  # storage服务端口（默认23000,一般不修改）
base_path=/home/dfs  # 数据和日志文件存储根目录
store_path0=/home/dfs  # 第一个存储目录
tracker_server=192.168.52.2:22122  # 服务器1
tracker_server=192.168.52.3:22122  # 服务器2
tracker_server=192.168.52.4:22122  # 服务器3
http.server_port=8888  # http访问文件的端口(默认8888,看情况修改,和nginx中保持一致)
```

### 4.3 client测试

```
vim /etc/fdfs/client.conf
#需要修改的内容如下
base_path=/home/moe/dfs
tracker_server=192.168.52.2:22122  # 服务器1
tracker_server=192.168.52.3:22122  # 服务器2
tracker_server=192.168.52.4:22122  # 服务器3
#保存后测试,返回ID表示成功 如：group1/M00/00/00/xx.tar.gz
fdfs_upload_file /etc/fdfs/client.conf /usr/local/src/nginx-1.15.4.tar.gz
```

### 4.4 配置nginx访问

```
vim /etc/fdfs/mod_fastdfs.conf
#需要修改的内容如下
tracker_server=192.168.52.2:22122  # 服务器1
tracker_server=192.168.52.3:22122  # 服务器2
tracker_server=192.168.52.4:22122  # 服务器3
url_have_group_name=true
store_path0=/home/dfs
#配置nginx.config
vim /usr/local/nginx/conf/nginx.conf
#添加如下配置
server {
    listen       8888;    ## 该端口为storage.conf中的http.server_port相同
    server_name  localhost;
    location ~/group[0-9]/ {
        ngx_fastdfs_module;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
    root   html;
    }
}
```

## 5. 启动

### 5.1 防火墙

```
#不关闭防火墙的话无法使用
systemctl stop firewalld.service #关闭
systemctl restart firewalld.service #重启
```

### 5.2 tracker

```
/etc/init.d/fdfs_trackerd start #启动tracker服务
/etc/init.d/fdfs_trackerd restart #重启动tracker服务
/etc/init.d/fdfs_trackerd stop #停止tracker服务
chkconfig fdfs_trackerd on #自启动tracker服务
```

### 5.3 storage

```
/etc/init.d/fdfs_storaged start #启动storage服务
/etc/init.d/fdfs_storaged restart #重动storage服务
/etc/init.d/fdfs_storaged stop #停止动storage服务
chkconfig fdfs_storaged on #自启动storage服务
```

### 5.4 nginx

```
/usr/local/nginx/sbin/nginx #启动nginx
/usr/local/nginx/sbin/nginx -s reload #重启nginx
/usr/local/nginx/sbin/nginx -s stop #停止nginx
```

### 5.5 检测集群

```
/usr/bin/fdfs_monitor /etc/fdfs/storage.conf
# 会显示会有几台服务器 有3台就会 显示 Storage 1-Storage 3的详细信息
```

## 6. 说明

### 6.1 配置文件

```
tracker_server #有几台服务器写几个
group_name #地址的名称的命名
bind_addr #服务器ip绑定
store_path_count #store_path(数字)有几个写几个
store_path(数字) #设置几个储存地址写几个 从0开始
```

### 6.2 可能遇到的问题

```
如果不是root 用户 你必须在除了cd的命令之外 全部加sudo
如果不是root 用户 编译和安装分开进行 先编译再安装
如果上传成功 但是nginx报错404 先检查mod_fastdfs.conf文件中的store_path0是否一致
如果nginx无法访问 先检查防火墙 和 mod_fastdfs.conf文件tracker_server是否一致
如果不是在/usr/local/src文件夹下安装 可能会编译出错
```