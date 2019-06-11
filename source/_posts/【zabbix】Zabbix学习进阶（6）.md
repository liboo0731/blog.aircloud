---
title: 【zabbix】Zabbix学习进阶（6）
date: 2019-05-30 13:18:46
tags:
- zabbix
---

## Zabbix 配置SNMP 监控

简单网络管理协议（SNMP），由一组网络管理的标准组成，包含一个应用层协议（application layer protocol）、数据库模型（database schema）和一组资源对象。该协议能够支持网络管理系统，用以监测连接到网络上的设备是否有任何引起管理上关注的情况。现在服务器、交换机、路由器、打印机等设备都支持SNMP协议。

### Zabbix Server 安装配置SNMP

```
安装snmpd服务及工具
# yum install -y net-snmp*

备份配置文件
# cp /etc/snmp/snmpd.conf{,.bak}

开启如下配置项
# vi /etc/snmp/snmpd.conf

proc mountd
proc ntalkd 4
proc sendmail 10 1
disk / 10000
load 12 14 14

view    systemview    included   .1.3.6.1.2.1.1
view    systemview    included   .1.3.6.1.2.1.25.1.1
view    systemview    included   .1						//新增

防火墙开启端口161/UDP
firewall-cmd --permanent --add-port=161/udp
firewall-cmd --reload

# 启动并设置开机自启
systemctl start snmpd
systemctl enable snmpd
```



### Zabbix 被监控设备安装配置SNMP

```
安装snmpd服务及工具
# yum install -y net-snmp*

备份配置文件
# cp /etc/snmp/snmpd.conf{,.bak}

开启如下配置项
# vi /etc/snmp/snmpd.conf

view    systemview    included   .1.3.6.1.2.1.1
view    systemview    included   .1.3.6.1.2.1.25.1.1
view    systemview    included   .1						//新增
com2sec notConfigUser  default       public

防火墙开启端口161/UDP
firewall-cmd --permanent --add-port=161/udp
firewall-cmd --reload

# 启动并设置开机自启
systemctl start snmpd
systemctl enable snmpd
```



### 使用snmpwalk命令测试被监控设备计算机名

```
# snmpwalk -v 2c -c public 172.16.2.228 sysName
SNMPv2-MIB::sysName.0 = STRING: host-172-16-2-228

2c是指采用SNMP V2版本，172.16.2.228 是开启了SNMP服务的被监控设备ip，否则会获取失败，sysName是指获取被监控设备的计算机名。
```



### 管理界面添加主机

```
Configuration -> Host --> Create host

Host name：host-172-16-2-228
Visible name：linux_host_232
Groups：Linux主机测试
# remove Agent interfaces
SNMP interfaces：172.16.2.228

# Templates
Template OS Linux SNMPv2
```



