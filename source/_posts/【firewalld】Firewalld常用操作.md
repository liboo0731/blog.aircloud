---
title: 【firewalld】Firewalld常用操作
date: 2019-03-12 14:40:07
tags:
- firewalld
---

### Firewalld引入 Zone 的概念

```shell
firewall-cmd --list-all-zones    #查看所有zone信息
```

- drop: 丢弃所有进入的包，而不给出任何响应

- block: 拒绝所有外部发起的连接，允许内部发起的连接

- public: 允许指定的进入连接

- external: 同上，对伪装的进入连接，一般用于路由转发

- dmz: 允许受限制的进入连接

- work: 允许受信任的计算机被限制的进入连接，类似 workgroup

- home: 同上，类似 homegroup

- internal: 同上，范围针对所有互联网用户
- trusted: 信任所有连接

*参考链接：[防火墙（firewalld）端口转发](http://www.itlnmp.com/373.html)*

### 端口开放与移除

```shell
# 添加端口
firewall-cmd --permanent --zone=<ZONE> --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --zone=<ZONE> --remove-port=80/tcp
# 重启生效
firewall-cmd --reload
# 罗列开放端口
firewall-cmd --list-ports
```

### IP伪装

```shell
firewall-cmd --permanent --zone=<ZONE> --add-masquerade

firewall-cmd --permanent --zone=<ZONE> --add-rich-rule='rule family=ipv4 source address=192.168.122.0/24 masquerade'
```

### 端口转发

```shell
firewall-cmd --permanent --zone=<ZONE> --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=192.168.122.7

firewall-cmd --permanent --zone=<ZONE> --add-rich-rule='rule family=ipv4 source address=192.168.122.0/24 forward-port port=80 to-port=8080 protocol=tcp accept'
```



### 一、firewalld 守护进程

firewall-cmd命令需要firewalld进程处于运行状态。我们可以使用systemctl status/start/stop/restart firewalld来控制这个守护进程。firewalld进程为防火墙提供服务。

当我们修改了某些配置之后（尤其是配置文件的修改），firewall并不会立即生效。可以通过两种方式来激活最新配置`systemctl restart firewalld`和`firewall-cmd --reload`两种方式，前一种是重启firewalld服务，建议使用后一种“重载配置文件”。重载配置文件之后不会断掉正在连接的tcp会话，而重启服务则会断开tcp会话。

### 二、控制端口/服务

可以通过两种方式控制端口的开放，一种是指定端口号另一种是指定服务名。虽然开放http服务就是开放了80端口，但是还是不能通过端口号来关闭，也就是说通过指定服务名开放的就要通过指定服务名关闭；通过指定端口号开放的就要通过指定端口号关闭。还有一个要注意的就是指定端口的时候一定要指定是什么协议，tcp还是udp。知道这个之后以后就不用每次先关防火墙了，可以让防火墙真正的生效。

```
`firewall-cmd --add-service=mysql ``# 开放mysql端口``firewall-cmd --remove-service=http ``# 阻止http端口``firewall-cmd --list-services  ``# 查看开放的服务``firewall-cmd --add-port=3306``/tcp` `# 开放通过tcp访问3306``firewall-cmd --remove-port=80tcp ``# 阻止通过tcp访问3306``firewall-cmd --add-port=233``/udp`  `# 开放通过udp访问233``firewall-cmd --list-ports  ``# 查看开放的端口`
```

### 三、伪装IP

防火墙可以实现伪装IP的功能，下面的端口转发就会用到这个功能。

```
`firewall-cmd --query-masquerade ``# 检查是否允许伪装IP``firewall-cmd --add-masquerade ``# 允许防火墙伪装IP``firewall-cmd --remove-masquerade``# 禁止防火墙伪装IP`
```

### 四、端口转发

端口转发可以将指定地址访问指定的端口时，将流量转发至指定地址的指定端口。转发的目的如果不指定ip的话就默认为本机，如果指定了ip却没指定端口，则默认使用来源端口。

如果配置好端口转发之后不能用，可以检查下面两个问题：

- 比如我将80端口转发至8080端口，首先检查本地的80端口和目标的8080端口是否开放监听了
- 其次检查是否允许伪装IP，没允许的话要开启伪装IP

```
`# 将80端口的流量转发至8080``firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080``# 将80端口的流量转发至``firewall-cmd --add-forward-port=port=80:proto=tcp:toaddr=192.168.1.0.1192.168.0.1``# 将80端口的流量转发至192.168.0.1的8080端口``firewall-cmd --add-forward-port=port=80:proto=tcp:toaddr=192.168.0.1:toport=8080`
```

- 当我们想把某个端口隐藏起来的时候，就可以在防火墙上阻止那个端口访问，然后再开一个不规则的端口，之后配置防火墙的端口转发，将流量转发过去。
- 端口转发还可以做流量分发，一个防火墙拖着好多台运行着不同服务的机器，然后用防火墙将不同端口的流量转发至不同机器。

*参考链接：[CentOS 7下用firewall-cmd控制端口与端口转发详解](https://www.jb51.net/article/112698.htm)*