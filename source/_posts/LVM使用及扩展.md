---
title: LVM使用及扩展
date: 2018-11-10 21:11:12
tags: 

- lvm
---

## LVM介绍

LVM是 Logical Volume Manager（逻辑卷管理）的简写，它是Linux环境下对磁盘分区进行管理的一种机制，它由Heinz Mauelshagen在Linux 2.4内核上实现，目前最新版本为：稳定版1.0.5，开发版 1.1.0-rc2，以及LVM2开发版。Linux用户安装Linux操作系统时遇到的一个常见的难以决定的问题就是如何正确地评估各分区大小，以分配合适的硬盘空间。普通的磁盘分区管理方式在逻辑分区划分好之后就无法改变其大小，当一个逻辑分区存放不下某个文件时，这个文件因为受上层文件系统的限制，也不能跨越多个分区来存放，所以也不能同时放到别的磁盘上。而遇到出现某个分区空间耗尽时，解决的方法通常是使用符号链接，或者使用调整分区大小的工具，但这只是暂时解决办法，没有从根本上解决问题。随着Linux的逻辑卷管理功能的出现，这些问题都迎刃而解，用户在无需停机的情况下可以方便地调整各个分区大小。



## LVM使用

### 1. 查看可用磁盘信息

```shell
fdisk -l
```

### 2. 创建PV

```shell
pvcreate /dev/sdb

pvscan
pvs
pvdisplay
```

### 3. 创建VG，并将PV加入到VG

```shell
vgcreate vg_name /dev/sdb

vgscan
vgs
vgdisplay
```

### 4. 创建LV

```shell
lvcreate -L 5G -n lv_name vg_name

lvscan
lvs
lvdisplay
```

### 5. 给LV创建文件系统

```shell
# xfs文件系统
mkfs.xfs -f /dev/vg_name/lv_name

# ext4文件系统
mkfs.ext4 /dev/vg_name/lv_name
```

### 6. 将LV挂载到相应的目录

```shell
# 挂载
mount /dev/vg_name/lv_name /data

# 开机自动挂载
/dev/vg_name/lv_name /data xfs defaults 0 0
/dev/vg_name/lv_name /data exts defaults 0 0
# 使用UUID挂载，目录可能会发生变化
# blkid命令查看UUID
UUID=a613d4f4-2f10-4f74-aa23-1815faf8f2fa /data xfs defaults 0 0
# iscsi存储挂载，网卡必须先启动
/dev/vg_name/lv_name /data xfs defaults,_netdev 0 0

```



## LVM扩展

### 1. 查看磁盘信息

```shell
fdisk -l
```

### 2. VG扩容

```shell
# 一块新盘加入到卷组
vgextend vg_name /dev/sdc

# 创建新分区
fdisk /dev/sdc
# partprobe 将磁盘分区表变化信息通知内核，请求操作系统重新加载分区表，不需要重启系统
pvcreate /dev/sdc3
# 一个新分区加入到卷组
vgextend vg_name /dev/sdc3

```

### 3. LV扩容

```
lvextend -L +5G /dev/vg_name/lv_name
```

### 4. 扩展文件系统

```shell
# xfs扩展
xfs_growfs /dev/vg_name/lv_name

# ext4扩展
resize2fs -f /dev/vg_name/lv_name
```

