---
title: 【centos】CentOS7-U盘安装
date: 2018-01-19 21:10:09
tags:
- centos
- centos7
---

### U盘安装CentOS7 系统尝试（1）：

1. 正常U盘启动，等待超时后在命令行查找并确定U盘设备是那一块盘

```shell
[root@packages x86_64]# blkid
/dev/sr0: UUID="2016-12-05-23-44-06-00" LABEL="CentOS 7 x86_64" TYPE="iso9660" PTTYPE="dos"
/dev/sda1: UUID="4b2c4a17-1150-494c-8201-904b5a252212" TYPE="xfs"
/dev/sda2: UUID="Z8Ty3N-kuAS-f9Sm-os3x-OtJq-xVzy-EKXgxm" TYPE="LVM2_member"
/dev/mapper/cl-root: UUID="ae156d1c-ad75-4ddf-9338-41d2b57f8048" TYPE="xfs"
/dev/mapper/cl-swap: UUID="104af556-91e3-45d1-8d0c-6dbc7ebeef03" TYPE="swap"
```

2. 重启服务器，从U盘启动

3. 在CentOS7 安装界面按“Tab”键修改为如下，其中“sdc4”为U盘设备

vmlinuz initrd=initrd.img inst.stage2=hd:/dev/sdc4 quiet

4. 按“Enter”键开始安装



### U盘安装CentOS7 系统尝试（2）：

1. 正常U盘启动到CentOS7 安装界面，选择第一项“Install CentOS 7”并按“Tab”键

2. 修改内容如下

vmlinuz initrd=initrd.img linux dd quiet

3. 按“Enter”键会列出你的设备列表，找到你的U盘设备

4. 重启服务器，从U盘启动

5. 在CentOS7 安装界面按“Tab”键修改为如下，其中“sdc4”为U盘设备

vmlinz initrd=initrd.img inst.stage2=hd:/dev/sdc4 quiet

6. 按“Enter”键开始安装