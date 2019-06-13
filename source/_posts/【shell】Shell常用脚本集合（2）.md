---
title: 【shell】Shell常用脚本集合（2）
date: 2019-03-19 09:14:25
tags: 
    - shell
---

### 批量修改文件后缀名

```shell
#!/bin/bash

read -p "old extension:" oldext
read -p "new extension:" newext
read -p "The directory:" dir
cd $dir

for file in $(ls $dir | grep .$oldext)
        do
        name=$(ls $file | cut -d. -f1)
        mv $file ${name}.$newext
        echo "$name.$oldext ====> $name.$newext"
        done
echo "all files has been modified."
```

```shell
#!/bin/sh
 
echo "input what suffix will be replaced :"
read SUFFIX_SRC
echo "input what suffix of file to rename to:"
read SUFFIX_DST
 
for i in *.$SUFFIX_SRC
do
    if [ -e $i ]; then
        #echo "mv $i to ${i%.*}.$SUFFIX_DST"
        #mv $i ${i%.*}.$SUFFIX_DST
        echo "mv $i to `basename $i .$SUFFIX_SRC`.$SUFFIX_DST"
        mv $i `basename $i .$SUFFIX_SRC`.$SUFFIX_DST
    else
        echo "file does not exist."
        exit -1
    fi
done
echo "all files has been modified."
```

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
while ! echo $installRole | egrep -q '^[tsbTSB]$'
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

### 10个环境变量

```shell
变量名称			作用
HOME			用户的家目录（即主目录）
SHELL			用户在使用的shell解释器名称
HISTSIZE		输出的历史命令记录条数
HISTFILESIZE	保存的历史命令记录条数
MAIL			邮件保存路径
LANG			系统语言、语系名称
RANDOM			生成一个随机数字
PS1				bash解释器的提示符
PATH			定义解释器搜索用户执行命令的路径
EDITOR			用户默认的文本编辑器

PS: 
1.可以用 env 命令来查看当前linux系统中所有的环境变量； 
2.用export命令可以把linux的一般变量转换成全局变量。
```

### 设置环境变量

```shell
#指明头文件的搜索路径
export C_INCLUDE_PATH=$CURRENTPATH/include
#指明库搜索路径
export LD_LIBRARY_PATH=/home/admin/fdfs/lib64
```

### 常用文件操作

```shell
#替换
sed -i '/^WorkingDirectory=/cWorkingDirectory='$CURRENTPATH'/realserver' $CURRENTPATH/realserver.service
#删除
sed -i '/^tracker_server=/d' $TARGET_CONF_PATH/storage.conf
#追加
sed -i '$a tracker_server='$ip $TARGET_CONF_PATH/storage.conf
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
REPLY=
while true
do
    msg='输入Tracker Server地址，多个值用空格隔开：'
    if [ "$REPLY" ]; then
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
[root@localhost ~]$ echo ${str:3}
.3.4:5
[root@localhost ~]$ echo ${str:0:3}
1.2
[root@localhost ~]$ echo ${str:0-3}
4:5
[root@localhost ~]$ echo ${str:0-3:1}
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

- 除特权用户，所属某用户文件/夹不能被其他用户删除
- 暂时总结：删除是从右到左询问权限，新建是从左到右询问权限

- 使用 -d 参数将默认权限延伸到新产生的文件夹，权限延伸必须是连续的，也就是父目录必须有默认权限设置

- 如果没有设置任何ACL权限，使用 -d 参数设置的默认权限最大为基础权限
- 如果设置过ACL权限，使用 -d 参数设置的默认权限最大为已设置的ACL权限

```shell
#先使用 -m 设置权限，在使用 -d 将设置的权限延伸到新产生的文件或文件夹
#文件或文件夹设置ACL规则
setfacl -m u:admin:rwx file
setfacl -m u:admin:rwx fodler/
#设置默认ACL规则，只对文件夹有效，可延伸到新建的子文件夹
setfacl -d -m u:admin:rwx fodler/
#递归设置，只对已存在的文件和文件夹有效
setfacl -R -m u:admin:rwx fodler/

#查看ACL规则
getfacl file
getfacl fodler/

#删除文件的acl规则
setfacl -x u:admin file
#删除所有扩展的acl规则，基本的acl规则(所有者，群组，其他）将被保留.
setfacl -b fodler/
```

### 向上递归设置ACL

```shell
#脚本必须位于用户HOME目录下
#!/bin/bash

if [ $UID -eq 0 ]; then
        echo "禁止以root帐号运行此脚本"
        exit 1
fi

CURRENTPATH=$(dirname $(readlink -f $0))

if ! id activemq &>/dev/null; then
	sudo useradd -r -s /bin/false activemq
	UPPERPATH=$CURRENTPATH
	while true
	do
		if [ "$HOME" == "$UPPERPATH" ]; then
			break
		fi
		UPPERPATH=$(dirname $UPPERPATH)
		setfacl -m u:admin:rwx $UPPERPATH
		setfacl -d -m u:admin:rwx $UPPERPATH
		setfacl -m u:activemq:rwx $UPPERPATH
		setfacl -d -m u:activemq:rwx $UPPERPATH
	done
	setfacl -R -m u:admin:rwx $CURRENTPATH
	setfacl -dR -m u:admin:rwx $CURRENTPATH
	setfacl -R -m u:activemq:rwx $CURRENTPATH
	setfacl -dR -m u:activemq:rwx $CURRENTPATH
fi
```

### 防火墙常用设置

```shell
#禁止外部连接，只开放ssh
firewall-cmd --set-default-zone=drop
firewall-cmd --load-zone-defaults=drop --permanent
firewall-cmd --permanent --zone=drop --add-port=22/tcp
firewall-cmd --reload

#firewall-offline-cmd is an offline command line client of the firewalld daemon. It should be used only if the firewalld service is not running. For example to migrate from system-config-firewall/lokkit or in the install environment to configure firewall settings with kickstart.
firewall-offline-cmd --set-default-zone=drop
firewall-offline-cmd --load-zone-defaults=drop
firewall-offline-cmd --add-port=22/tcp

#keepalived配置规则开放
firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0 --in-interface ens33 --destination 224.0.0.18 --protocol vrrp -j ACCEPT
```

### 自动交互

```shell
[root@localhost ~]# echo -e '123456\n123456'|passwd tom
Changing password for user tom.
New password: BAD PASSWORD: The password is shorter than 8 characters
Retype new password: passwd: all authentication tokens updated successfully.
[root@localhost ~]# echo 123456|passwd --stdin tom
Changing password for user tom.
passwd: all authentication tokens updated successfully.
[root@localhost ~]# passwd tom <<EOF
123456
123456
EOF
Changing password for user tom.
New password: BAD PASSWORD: The password is shorter than 8 characters
Retype new password: passwd: all authentication tokens updated successfully.
[root@localhost ~]#
```

### 模板写入

```shell
#!/bin/sh

a=1
b=2

cat > t1.txt <<EOF
a=$a
b=$b
EOF

cat <<EOF > t2.txt
a=$a
b=$b
EOF

```

### Sed常用操作

```shell
#第一行前面添加（1i）
sed -i '1istr' file
#第一行后面添加（1a）
sed -i '1astr' file
#替换第一行（1c）
sed -i '1cstr' file
#删除第一行（1d）
sed -i '1d' file
#末尾追加（$a）
sed -i '$astr' file
#匹配替换开头是a的字符为str
sed -i 's/^a/str/' file
#匹配全局替换
sed -i 's/^a/str/g' file
#匹配行前面添加（1i）
sed -i '/^a/istr' file
#匹配行后面添加（1a）
sed -i '/^a/astr' file
#替换匹配行（1c）
sed -i '/^a/cstr' file
#删除匹配行（1d）
sed -i '/^a/d' file
#删除文件中行首的空格
sed -i 's/^[ \t]*//' result.data 
#删除文件中行末空格：
sed -i 's/[ \t]*$//g' result.data
#删除文件中所有的空格：
sed -i s/[[:space:]]//g result.data
#把文件中空格变为‘，’：
sed -i 's/[ \t]/,/g' result.data
```

