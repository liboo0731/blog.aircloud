---
title: OpenStack平台qcow2镜像制作
date:  2017-09-08 09:57:42
tags: 
    - openstack
    - qcow2
---

## 一、qcow2镜像制作流程：

### 1. 预先准备：

- 虚拟机物理机均可，虚拟机需要开启硬件虚拟化 
- 操作系统： centos6/centos7 桌面版 
- 待制作镜像系统（iso）： win7/8/10 
- virtio驱动： win默认iso镜像不包含virtio驱动，而通过openstack管理虚拟机需要两个virtio驱动，一个是硬盘的，一个是网卡的。

```bash
#下载地址：https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.140-1/virtio-win-0.1.140.iso 
#64位：https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.140-1/virtio-win-0.1.140_amd64.vfd  
#32位：https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/virtio-win-0.1.140-1/virtio-win-0.1.140_x86.vfd 
```

> *以下操作均在root权限下执行，将下载的win7iso镜像和virtio驱动均放在 /root 目录下*



### 2. 安装相关RPM软件包：

```
yum install tigervnc  vnc*   -y
yum install qemu-kvm qemu-img –y
```

### 3. 创建连接：

```bash
ln -s /usr/libexec/qemu-kvm /usr/bin/kvm
ln -s /usr/bin/qemu-img /usr/bin/kvm-img
```

### 4. 正式安装：

#### 4.1 创建：

```bash
qemu-img create -f qcow2 win7.qcow2 10G
```

#### 4.2 vnc启动安装，关机：

```bash
kvm -m 1024 -cdrom win7.iso -drive file=win7.qcow2,if=virtio,boot=on -fda virtio-win-0.1.102_amd64.vfd -boot d -nographic -vnc :3
```

#### 4.3 vnc启动，更改网卡驱动为Redhat：

```bash
kvm -m 1024 -drive file=win7.qcow2,if=virtio,boot=on -cdrom virtio-win-0.1.102.iso -net nic,model=virtio -net user -boot c -nographic -vnc :3
```

#### 4.4 关机拷贝：

```bash
vncviewer
```



## 二、基于OpenStack平台的qcow2镜像直接获取

> 待制作操作系统：Windows/linux
>
> 处理：使用刻录工具将硬盘及网卡驱动放进iso包中，并另存为新的iso镜像文件（只针对Windows镜像，linux本身就有相关驱动文件）
>

### OpenStack-icehouse平台

- 新建一个“云主机类型”（flavor），启用第二块存储盘（Ephemeral disk）设置其大小足够安装iso镜像，将默认的第一块存储盘（root disk）大小设置为0。

- 选择此“云主机类型”启动一个云主机实例，此时第一块存储盘就被当成cdrom，数据将被安装到第二块盘中，利用VNC窗口安装操作系统，手动选择硬盘驱动。

- 系统安装完成后使用命令弹出此“光驱设备”（root disk），执行云主机快照（create snapshot）功能，获取你所需要的qcow2镜像

```bash
#其中 “1” 为云主机的ID，可通过virsh list命令查询
virsh attach-disk --type cdrom --mode readonly 1 "" hda --live
```

> *注意：*
>
> *在OpenStack-icehouse版本代码里默认只有一块Ephemeral disk，数据会被默认存储到一个名为 disk.local 文件中，在把root disk作为cdrom弹出以后，创建快照时会直接对Ephemeral disk进行快照。在高版本的OpenStack中，这部分代码进行了优化，Ephemeral disk可能不只一个，所以返回值不能确定，进行上述操作时会报错，或者你可能对root disk的iso镜像创建了一个快照。*
>
> *既然我们已经知道创建的云主机会存储到什么地方，那就可以直接对存储的数据进行操作。使用高版本的OpenStack按照上述方式安装iso镜像，直接对数据盘（qcow2格式）进行镜像上传，至此你就获得了一台初始的qcow2模板虚拟机。*