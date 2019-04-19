---
title: Shell常用脚本片段
date: 2019-04-19 09:14:25
tags: 
    - shell
---

### 禁止使用root账号运行

```shell
if [ $UID -eq 0 ]; then
	echo "禁止以root帐号运行此脚本"
	exit 1
fi
```

### 禁止使用普通账号运行

```shell
if [ $UID -ne 0 ]; then
	echo "请以root帐号运行此脚本"
	exit 1
fi
```

### 获取运行脚本的所在目录

```shell
SCRIPTPATH=$(dirname $(readlink -f $0))
```

### 多角色安装，交互询问

```shell
read -p "请选择该服务器的角色 (t:tracker, s:storage, b:both) [t/s/b]:" installRole
while ! echo $installScope | egrep -q '^[tsbTSB]$'
do
	read -p "输入错误，请重新输入 (t:tracker, s:storage, b:both) [t/s/b]:" installRole
done
```

### 解压缩（*.tar.gz）

```shell
#压缩
tar zcvf file.tar.gz file/
#解压到当前目录
tar xzvf file.tar.gz
#解压到指定目录
tar xzvf file.tar.gz -C /tmp
```

### RPM常见操作

```shell
rpm -ivh/e/U
#不考虑依赖
rpm -ivh/e/U --nodeps
#不考虑依赖，并强制
rpm -ivh/e/U --nodep --force

yum install
#本地安装，没有的包会用源安装
yum localinstall
#组安装
yum groupinstall
#会移除依赖
yum remove
#查询命令来源那个包
yum provides
```

### 常用内置变量

```shell

```

### 设置环境变量

```shell

```

### 常用文件操作

```shell

```

### 移除32位软件包

```shell
rpm -e --nodeps *.i?86
```

### 禁止yum安装32位的软件包

```shell
if ! egrep -q '^exclude=.*' /etc/yum.conf; then
	echo "exclude=*.i?86" >> /etc/yum.conf
fi
```

### 禁用usb存储

```shell
echo "blacklist usb-storage" > /etc/modprobe.d/blacklist-usbstorage
```

### 主机连通性测试

```shell
echo "Tracker Server配置示例：192.168.1.2:22122 192.168.1.3:22122 ..."
#REPLY=
while true
do
    msg='输入Tracker Server地址，多个值用空格隔开：'
    if [ $REPLY ]; then
        echo "正在检测网络连通性"
        failedHost=
        for host in $REPLY
        do
        #if ! echo "HEAD / HTTP/1.0" >/dev/tcp/${host/://}; then
        if ! echo "HEAD / HTTP/1.0" >/dev/tcp/${host%:*}/22; then
            failedHost=$host
            break
        fi
        done
        if [ $failedHost ]; then
            prompt="与$failedHost的网络不通，请重新$msg"
        else
            break
        fi
    else
    	prompt="请$msg"
    fi
    read -p "$prompt"
done
```

### 从变量中切割字符串

```shell
#定义变量
[admin@localhost ~]$ str=1.2.3.4:5
#获取长度
[admin@localhost ~]$ echo ${#str}
9
#替换字符
[admin@localhost ~]$ echo ${str/://}
1.2.3.4/5
#从左开始删除至第一个匹配字符
[admin@localhost ~]$ echo ${str#*.}
2.3.4:5
[admin@localhost ~]$ echo ${str##*.}
4:5
#从右开始删除至匹配到的第一个字符
[admin@localhost ~]$ echo ${str%:*}
1.2.3.4
[admin@localhost ~]$ echo ${str%.*}
1.2.3
[admin@localhost ~]$ echo ${str%%.*}
1
[root@localhost ~]# echo ${str:3}
.3.4:5
[root@localhost ~]# echo ${str:0:3}
1.2
[root@localhost ~]# echo ${str:0-3}
4:5
[root@localhost ~]# echo ${str:0-3:1}
4

```

### 远程和控制台登录添加登录失败超时控制

```shell
for file in "/etc/pam.d/sshd" "/etc/pam.d/login"
do
	if ! egrep -q '^auth\s*required\s*pam_tally2.*' $file; then
		sed -i "1 aauth       required     pam_tally2.so deny=3 unlock_time=28800 even_deny_root root_unlock_time=3600" $file
	fi
done
```

### 新建普通用户并设置随机密码

```shell
if ! id admin &>/dev/null;then
	useradd admin
	echo "为admin设置随机8位密码"
	MATRIX='0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz~!@#$%^&*()_+=-'
	LENGTH=12
	while [ "${n:=1}" -le "$LENGTH" ]
	do
		PASS="$PASS${MATRIX:$(($RANDOM%${#MATRIX})):1}"
		let n+=1
	done
	echo $PASS | passwd --stdin admin
	echo $PASS >/root/adminPassword.txt
fi
```

### ACL规则设置、查看、取消

```shell
#文件或文件夹设置ACL规则
setfacl -m u:admin:rwx file
setfacl -m u:admin:rwx fodler/
#设置默认ACL规则，只对文件夹有效
setfacl -d -m u:admin:rwx fodler/
#递归设置，文件和文件夹都有效
setfacl -R -m u:admin:rwx fodler/

#查看ACL规则
getfacl file
getfacl fodler/

#删除文件的acl规则
setfacl -x u:admin file
#删除所有扩展的acl规则，基本的acl规则(所有者，群组，其他）将被保留.
setfacl -b fodler/
```

### 防火墙常用设置

```shell
#禁止外部连接，只开放ssh
firewall-cmd --set-default-zone=drop &>/dev/null
firewall-cmd --load-zone-defaults=drop --permanent &>/dev/null
firewall-cmd --permanent --zone=drop --add-port=22/tcp &>/dev/null
firewall-cmd --reload &>/dev/null

```

