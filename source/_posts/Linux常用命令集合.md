---
title: Linux 操作命令整理
date: 2018-10-16 19:41:32
tags: 
    - linux	
---

##### 删除文件/文件夹

```shell
#删除多个文件夹
rm -rf {META-INF,org,resources,WEB-INF}
#反向删除
rm -rf `ls | grep -v comps.xml$`
#查找删除
find iso/ -name boot.cat | xargs rm
find iso/ -name TRANS.TBL -exec rm {} \;
```

##### 查看文件/文件夹个数

```shell
#当前目录文件个数
ls -l | grep "^-" | wc -l 
find ./company -type f | wc -l 
#当前目录文件夹个数
ls -l | grep "^d" | wc -l 
#查看某文件夹下文件的个数，包括子文件夹里的。 
ls -lR | grep "^-" | wc -l 
#查看某文件夹下文件夹的个数，包括子文件夹里的。 
ls -lR | grep "^d" | wc -l 
```

##### 解压/压缩

```shell
#软件数据包解压
tar xzvf infoShare-web-3.0.4.tar.gz
#软件数据包压缩
tar zcvf infoShare-web-3.0.4.tar.gz infoShare-web-3.0.4/
```

##### 获取本地IP地址

```shell
ipaddr='172.0.0.1'
ipaddr=$(ip addr | awk '/^[0-9]+: / {}; /inet.*global/ {print gensub(/(.*)\/(.*)/, "\\1", "g", $2)}')
```

##### RPM定制打包

```shell
#RPM包生成
rpmbuild -ba infoShare-web.spec
```

##### ISO定制打包

```shell
#ISO定制化文件解包
unsquashfs -dest squashfs-root squashfs.img
#ISO定制化文件打包
mksquashfs squashfs-root squashfs.img -noappend -all-root

#ISO本地源仓库信息
mv ./*-comps.xml ./infoShare-x86_64-comps.xml && rm -rf `ls |grep -v comps.xml$`
#创建ISO 本地源
createrepo -g repodata/infoShare-x86_64-comps.xml ./

cp /usr/share/comps/i386/.discinfo iso/

#ISO镜像生成
mkisofs -r -V "CentOS 7 x86_64" -b isolinux/isolinux.bin -c isolinux/boot.cat -cache-inodes -J -l -no-emul-boot -boot-load-size 4 -boot-info-table -o /InfoShare-3.0.0.iso ./iso/
```

#### Linux基本操作

```shell
#查看当前登录用户信息
w 查看当前系统信息和用户登录信息
who 查看当前正在登录的用户
users 可用于打印输出登录服务器的用户名称。该命令除了有help和version选项外，再没有其他选项。如果某用户使用了多个终端，则相应的会显示多个重复的用户名。
whoami 查看你所使用的登录名称
id 显示真实有效的用户 ID(UID) 和组 ID(GID)。UID 是对一个用户的单一身份标识。组 ID（GID）则对应多个UID。 
last 用于显示特定用户登录系统的历史记录。如果没有指定任何参数，则显示所有用户的历史信息。在默认情况下，这些信息（所显示的信息）将来源于/var/log/wtmp文件。

#罗列所有用户
awk -F: '{print $1}' /etc/passwd
awk -F: '{print $1}' /etc/shadow

#过滤长度为三位的用户
awk -F: '{print $1}' /etc/passwd |grep ^...$

#cut是一个选取命令，就是将一段数据经过分析，取出我们想要的。一般来说，选取信息通常是针对“行”来进行分析的，并不是整篇信息分析的
cut -d: -f1 /etc/passwd

#compgen 是 bash 的内置命令，它将显示所有可用的命令，别名和函数。
compgen -u

#从系统数据库读数据。用于判定用户是否存在等，十分好用。支持的数据库：
ahosts ahostsv4 ahostsv6 aliases ethers group gshadow hosts netgroup
networks passwd protocols rpc services shadow
# 判定用户组test是否存在：如果存在就退出，不存在就创建
getent group test || groupadd test

netstat && ss

```

#### Linux用户、用户组

```shell
linux的权限系统主要是由用户、用户组和权限组成。
用户就是一个个的登录并使用linux的用户。linux内部用UID表示。
用户组就是用户的分组。linux内部用GID表示。
权限分为读、写、执行三种权限。

linux的用户信息保存在/etc/passwd文件中，另外，/etc/shadow文件存放的是用户密码相关信息。

/etc/passwd文件格式：
用户名:密码:UID:GID:用户信息:HOME目录路径:用户shell
其中UID为0则是用户root，1～499为系统用户，500以上为普通用户

/etc/shadow保存用户密码信息，包括加密后的密码，密码过期时间，密码过期提示天数等。

用户组信息保存在/etc/group文件中.
格式如下：
用户组名:组密码:GID:组内帐号（多个帐号用逗号分隔）

用户登录后，/etc/passwd文件里的GID为用户的初始用户组。
用户的初始用户组这一事实不会再/etc/group中体现。

-----------------------------------------------------------------

查看当前用户的用户组命令：
[root@local opt]#groups
root bin daemon sys adm disk wheel
输出的信息中，第一个用户组为当前用户的有效用户组（当前用户组）

切换有效用户组命令：
[root@local opt]#newgrp 用户组名
要离开新的有效用户组，则输入exit回车。

新建用户命令：
[root@local opt]#useradd 用户名 -g 初始用户组 -G 其他用户组（修改/etc/group） -c 用户说明 -u 指定UID

建完用户需要为用户设置密码：
[root@local opt]#passwd 用户名

用户要修改自己密码命令：
[root@local opt]#passwd

修改用户信息命令：
[root@local opt]#usermod 参数 用户名
参数:
 -c 说明
 -g 组名 初始用户组
 -e 过期日期 格式：YYYY-MM-DD
 -G 组名 其他用户组
 -l 修改用户名
 -L 锁定账号（在/etc/shadow文件中用户对应密码密码串的前面加上两个叹号(!!)）
 -U 解锁

删除用户命令：
[root@local opt]#userdel [-r] 用户名
其中，参数-r为删除用户的home目录。
其实，可能在系统其他地方也有该用户文件，要完整删除一个用户和其文件要先找到属于他的文件：
[root@local opt]#find / -user 用户名
然后删除，再运行userdel删除用户。

查看可用shell命令：
[root@local opt]#chsh -l
修改自己的shell命令：
[root@local opt]#chsh -s

查看自己或某人UID/GID信息：
[root@local opt]#id [用户名]
返回信息中groups为有效用户组

新增用户组命令：
[root@local opt]#groupadd 用户组名

修改用户组名命令：
[root@local opt]#groupmod -n 名称

删除用户组命令：
[root@local opt]#groupdel 用户组名

设置用户组密码命令：
[root@local opt]#gpasswd 用户组名

如果gpasswd加上参数则有其他功能

设置用户组管理员命令：
[root@local opt]#gpasswd -A 用户名 用户组名

添加某帐号到组命令：
[root@local opt]#gpasswd -M 用户名 用户组名

从组中删除某帐号命令：
[root@local opt]#gpasswd -d 用户名 用户组名

passwd相关参数操作：
-l 锁用户
-u 解锁用户
-n 天数  密码不可改天数
-x 天数  密码过期天数
-w 天数  警告天数
```

#### Linux文件权限

```shell
先看个实例：
[root@local opt]#ls -al
ls -al 命令是列出目录的所有文件，包括隐藏文件。隐藏文件的文件名第一个字符为'.'
-rw-r--r--  1 root root    81 08-02 14:54 gtkrc-1.2-gnome2
-rw-------  1 root root   189 08-02 14:54 ICEauthority
-rw-------  1 root root    35 08-05 10:02 .lesshst
drwx------  3 root root  4096 08-02 14:54 .metacity
drwxr-xr-x  3 root root  4096 08-02 14:54 nautilus

列表的列定义如下：
[权限属性信息] [连接数] [拥有者] [拥有者所属用户组] [大小] [最后修改时间] [文件名]

权限属性列表为10个字符：
第一个字符表示文件类型，d为目录 -为普通文件 l为连接 b为可存储的接口设备 c为键盘鼠标等输入设备
2、3、4个字符表示所有者权限，5、6、7个字符表示所有者同组用户权限，8、9、10为其他用户权限

修改文件所属组命令：
[root@local opt]#chgrp [-R] 组名 文件名
其中-R为递归设置

修改文件的所有者和组命令：
[root@local opt]#chown [-R] 用户[:用户组] 文件名

修改文件访问权限命令：
[root@local opt]#chmod [-R] 0777 文件名
```

#### Shell变量定义及使用

```shell
#有效的 Shell 变量名示例如下：
RUNOOB
LD_LIBRARY_PATH
_var
var2
#用语句给变量赋值
for file in `ls /etc`
或
for file in $(ls /etc)
#使用变量
your_name="qinjx"
echo $your_name
echo ${your_name}
```

#### 获取Linux所有磁盘

```shell
# 磁盘存在格式
# /dev/vda
# /dev/sda
fdisk -l 2>/dev/null | grep -o "Disk /dev/.d[a-z]"

#一般/dev/sda属于主分区，让用户选择此外的其他盘进行扩展
fdisk -l 2>/dev/null | grep -o "Disk /dev/.d[b-z]"
```

#### 获取系统内存大小

```shell
#打印内存大小，以kb为单位
awk '($1 == "MemTotal:"){print $2}' /proc/meminfo
```

#### 获取系统IP地址

```shell
ip addr | awk '/^[0-9]+: / {}; /inet.*global/ {print gensub(/(.*)\/(.*)/, "\\1", "g", $2)}'
```

#### 数据库权限修改

```shell
mysql -uroot -p'+w_:5sHO3r!S' -e "grant all privileges on *.* to 'infoShare'@'%' identified by 'PASSWORD';"
flush privileges;
```

#### 数据库密码修改

```shell
mysql -uroot -p'+w_:5sHO3r!S' -e "alter user 'infoShare'@'%' identified by 'PASSWORD';"
```

#### 防火墙端口修改

```shell
#罗列端口
firewall-cmd --list-ports
#添加端口
firewall-cmd --zone=public --permanent --add-port=80/tcp
#移除端口
firewall-cmd --zone=public --permanent --remove-port=80/tcp
#重启防火墙
systemctl restart firewalld
```

#### RPM包操作

```shell
#强制卸载rpm包
rpm -e --nodeps `rpm -qa|grep mariadb-libs`
#测试安装
rpm -ivh --test *.rpm

```

#### sed命令集合

```shell
#去除前后空格
sed -i s/[[:space:]]//g /etc/my.cnf
#最后一行追加内容
sed -i '$a\innodb_buffer_pool_size=8G' /etc/my.cnf

```

#### CentOS6服务相关

```shell
# /etc/init.d/
chkconfig --add fwdport
chkconfig --del fwdport
chkconfig fwdport on
chkconfig --list
```

#### 计算文件夹中所有文件md5值

```shell
find resources/ -type f -not \( -name '.*' \) -exec md5sum {} \;>resources.txt
```

#### ISO镜像生成

```shell
mkisofs -r -V "CentOS 7 x86_64" -b isolinux/isolinux.bin -c isolinux/boot.cat -cache-inodes -J -l -no-emul-boot -boot-load-size 4 -boot-info-table -o /Info.iso ./iso7.4/
```

