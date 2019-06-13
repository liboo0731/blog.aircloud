---
title: 【lvs】LVS相关概念
date: 2019-04-23 09:50:58
tags: 
    - lvs
---

## 概念了解

### LVS

LVS是Linux Virtual Server的简写，意即Linux虚拟服务器，是一个虚拟的服务器集群系统。

针对可伸缩，高可用服务的需求，给予IP层和内容请求分发的负载均衡调度解决方法，并在Linux的内核中实现，将一组服务器构成一个实现可伸缩，高可用网络服务的虚拟服务器。

#### 负载调度器
一组服务器通过高速的局域网或者地理分布的广域网相互相连，在他们的前端有一个负载均衡调度器（Load Balancer），负载均衡调度器能无缝的将网络请求调度到真实的服务器上，从而使得服务器集群的结构对用户是透明的，用户通过访问集群系统提供的网络服务，就像访问一台高性能，高可用的服务器。

#### 三种IP负载均衡技术
1.VS/NAT（网络地址转换）
通过网络地址转换，调度器重写请求报文的目标地址，根据预设的调度算法，将请求分发给后端的真实服务器，真实服务器的响应报文通过调度器时，报文的源地址被重写，再返回到客户端，完成整个调度的过程。

2.VS/TUN（IP隧道模式）
调度器将请求的报文通过IP隧道转发至真实服务器，而真实的服务器直接将结果返回给用户，调度器只处理请求报文，由于一般网路服务的应答大于请求，采用IP隧道模式，集群系统的最大吞吐量可以提高10倍。

3.VS/DR（直接路由）
通过改写请求报文的MAC地址，将请求发送到真是服务器，真实服务器将响应直接返回给用户，之际额路由模式可以极大的提高集群系统的伸缩性，这种方法没有IP隧道的开销，集群中真实的服务器也没有必要必须支持IP隧道协议，只是需要调度器与真实服务器有一块网卡连在同一物理网段上。

*PS: DR和TUN模式都需要在真实服务器上对arp_ignore和arp_announce参数进行配置，主要是实现禁止响应对VIP的ARP请求。*

#### 参数配置

**arp_ignore**

- 0 - (默认值): 回应任何网络接口上对任何本地IP地址的arp查询请求 
- 1 - 只回答目标IP地址是来访网络接口本地地址的ARP查询请求 
- 2 -只回答目标IP地址是来访网络接口本地地址的ARP查询请求,且来访IP必须在该网络接口的子网段内 
- 3 - 不回应该网络界面的arp请求，而只对设置的唯一和连接地址做出回应 
- 4-7 - 保留未使用 
- 8 -不回应所有（本地地址）的arp查询

**arp_announce**

- 0 - (默认) 在任意网络接口（eth0,eth1，lo）上的任何本地地址 

- 1 - 尽量避免不在该网络接口子网段的本地地址做出arp回应. 当发起ARP请求的源IP地址是被设置应该经由路由达到此网络接口的时候很有用.此时会检查来访IP是否为所有接口上的子网段内ip之一.如果改来访IP不属于各个网络接口上的子网段内,那么将采用级别2的方式来进行处理. 
- 2 - 对查询目标使用最适当的本地地址.在此模式下将忽略这个IP数据包的源地址并尝试选择与能与该地址通信的本地地址.首要是选择所有的网络接口的子网中外出访问子网中包含该目标IP地址的本地地址. 如果没有合适的地址被发现,将选择当前的发送网络接口或其他的有可能接受到该ARP回应的网络接口来进行发送.

```shell
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce
```

### ARP

地址解析协议，即ARP（Address Resolution Protocol），是根据IP地址获取物理地址的一个TCP/IP协议。主机发送信息时将包含目标IP地址的ARP请求广播到网络上的所有主机，并接收返回消息，以此确定目标的物理地址；收到返回消息后将该IP地址和物理地址存入本机ARP缓存中并保留一定时间，下次请求时直接查询ARP缓存以节约资源。

## DR 模式Real Server 配置

### CentOS 6（CentOS 7 安装net-tools 包也可使用）

```shell
#!/bin/sh
# chkconfig: - 80 20
# description: RealServer

VIP=192.168.10.12
/etc/rc.d/init.d/functions
case "$1" in
    start)
        echo "Start LVS of RealServer"
        ifconfig lo:0 $VIP broadcast $VIP netmask 255.255.255.255 up
        route add -host $VIP dev lo:0
        echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
        echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
        echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
        echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
        ;;
    stop)
        ifconfig lo:0 down
        route del -host $VIP dev lo:0
        echo "Stop LVS of RealServer"
        echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore
        echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce
        echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore
        echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce
        ;;
    *)
        echo "Usage:$0{start|stop}"
        exit 1
esac
```

### CentOS 7

```shell
#!/bin/sh
# chkconfig: - 80 20
# description: RealServer

VIP=192.168.10.12
/etc/rc.d/init.d/functions
case "$1" in
start)
	ip address add $VIP/32 broadcast $VIP label lo:0 dev lo
	ip route add $VIP/32 dev lo:0
	echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
	echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
	echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
	echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
	sysctl -p >/dev/null 2>&1
	echo "RealServer Start OK"
	;;
stop)
	ip link set dev lo:0 down
	ip route del $VIP/32 dev lo:0 >/dev/null 2>&1
	echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore
	echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce
	echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore
	echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce
	echo "RealServer Stoped"
	;;
*)
	echo "Usage: $0 {start|stop}"
	exit 1
esac
```

