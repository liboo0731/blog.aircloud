---
title: vmware网络模式简介
date: 2017-10-10 17:11:12
tags: 

- vmware
- 网络
---

### vmnet0(桥接模式)

```
vmnet0(桥接模式),无实体虚拟网卡,其实就是一个协议而已，会在对应网卡上加入 VMware bridge protocol桥接协议，其作用相对于使用一个虚拟交换机连接虚拟机和物理机网卡。
```

### vmnet1(仅主机模式, hostory)

```
vmnet1(仅主机模式, hostory)，生成vmnet1虚拟网卡，并定义一个ip地址，虚拟机设置为仅主机模式，需要手动设置和vmnet1相同的子网网段，作用相同与vmnet0但由于vmnet1不接入其他网络，所以数据只在虚拟机和物理机间交换。
```

### vmnet8(NAT模式,网络地址转换)

```
vmnet8(NAT模式，网络地址转换)，生成vmnet8虚拟机网卡，并通过VMware Net service提供网关和地址转换服务，VMware DHCP service提供虚拟机ip地址自动分配服务，通过vmnet8与虚拟机通讯。
```

