---
title: 【lvs】LVS配置DR模式
date: 2019-04-24 10:13:06
tags:
- lvs
---

## LVS 使用DR 模式

#### 服务器IP 信息

```
lvs：192.168.1.2
web1：192.168.1.3
web2：192.168.1.4
vip：192.168.1.5
```

### LVS 服务器安装配置

```
# 安装 ipvsadm（lvs管理软件）
[root@localhost ~]# yum install -y ipvsadm

# 添加vip
[root@localhost ~]# ifconfig enp0s3:1 192.168.1.5 netmask 255.255.255.0
[root@localhost ~]# route add -host 192.168.1.5 dev eth1
或
[root@localhost ~]# ip address add 192.168.1.5/24 broadcast 192.168.1.255 label enp0s3:1 dev enp0s3
[root@localhost ~]# ip route add 192.168.1.5/24 dev enp0s3:1
或
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-enp0s3:1

DEVICE=enp0s3:1
IPADDR=192.168.1.5
NETMASK=255.255.255.0
ONBOOT=yes

# 创建虚拟服务并设置适合的调度算法
[root@localhost ~]# ipvsadm -A -t 192.168.1.5:80 -s wrr # Zero port specified for non-persistent service
# 添加RealServer组
[root@localhost ~]# ipvsadm -a -t 192.168.1.5:80 -r 192.168.1.3:80 -g -w 1
[root@localhost ~]# ipvsadm -a -t 192.168.1.5:80 -r 192.168.1.4:80 -g -w 2

# 保存调度规则
[root@localhost ~]# ipvsadm -Sn > /etc/sysconfig/ipvsadm

# 设置防火墙规则
[root@localhost ~]# firewalld-cmd --permanent --add-port=80/tcp
[root@localhost ~]# firewalld-cmd --reload

# 开启服务
[root@localhost ~]# systemctl start ipvsadm
[root@localhost ~]# systemctl enable ipvsadm
```

### Web 服务器（RealServer）配置

```
# 安装httpd
[root@localhost ~]# yum install -y httpd

# 配置回环网卡
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-lo:0

DEVICE=lo:0
BOOTPROTO=static
IPADDR=192.168.23.253
NETMASK=255.255.255.255
GATEWAY=192.168.23.1
ONBOOT=yes

# 配置ARP规则
[root@localhost ~]# vi /etc/sysctl.conf 

net.ipv4.conf.enp0s3.arp_ignore = 1
net.ipv4.conf.enp0s3.arp_announce = 2
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2

[root@localhost ~]# sysctl -p

# 配置Apache服务
[root@localhost ~]# vi /var/www/html/index.html

HOST 52 192.168.1.*

# 重启服务服务
[root@localhost ~]# systemctl restart network
[root@localhost ~]# systemctl restart httpd
[root@localhost ~]# systemctl enable httpd

# 设置防火墙规则
[root@localhost ~]# firewalld-cmd --permanent --add-port=80/tcp
[root@localhost ~]# firewalld-cmd --reload
```

### ipvsadm 配置

```
[root@localhost ~]# cat /etc/sysconfig/ipvsadm-config 
# Unload modules on restart and stop
#   Value: yes|no,  default: yes
# This option has to be 'yes' to get to a sane state for a ipvs
# restart or stop. Only set to 'no' if there are problems unloading ipvs
# modules.
IPVS_MODULES_UNLOAD="yes"

# Save current ipvs rules on stop.
#   Value: yes|no,  default: no
# Saves all ipvs rules to /etc/sysconfig/ipvsadm if ipvsadm gets stopped
# (e.g. on system shutdown).
IPVS_SAVE_ON_STOP="no"

# Save current ipvs rules on restart.
#   Value: yes|no,  default: no
# Saves all ipvs rules to /etc/sysconfig/ipvsadm if ipvsadm gets
# restarted.
IPVS_SAVE_ON_RESTART="no"

# Numeric status output
#   Value: yes|no,  default: yes
# Print IP addresses and port numbers in numeric format in the status output.
IPVS_STATUS_NUMERIC="yes"

```

### ipvsadm 用法

```
ipvsadm v1.27 2008/5/15 (compiled with popt and IPVS v1.2.1)
Usage:
  ipvsadm -A|E -t|u|f service-address [-s scheduler] [-p [timeout]] [-M netmask] [--pe persistence_engine] [-b sched-flags]
  ipvsadm -D -t|u|f service-address
  ipvsadm -C
  ipvsadm -R
  ipvsadm -S [-n]
  ipvsadm -a|e -t|u|f service-address -r server-address [options]
  ipvsadm -d -t|u|f service-address -r server-address
  ipvsadm -L|l [options]
  ipvsadm -Z [-t|u|f service-address]
  ipvsadm --set tcp tcpfin udp
  ipvsadm --start-daemon state [--mcast-interface interface] [--syncid sid]
  ipvsadm --stop-daemon state
  ipvsadm -h

Commands:
Either long or short options are allowed.
  --add-service     -A        add virtual service with options
  --edit-service    -E        edit virtual service with options
  --delete-service  -D        delete virtual service
  --clear           -C        clear the whole table
  --restore         -R        restore rules from stdin
  --save            -S        save rules to stdout
  --add-server      -a        add real server with options
  --edit-server     -e        edit real server with options
  --delete-server   -d        delete real server
  --list            -L|-l     list the table
  --zero            -Z        zero counters in a service or all services
  --set tcp tcpfin udp        set connection timeout values
  --start-daemon              start connection sync daemon
  --stop-daemon               stop connection sync daemon
  --help            -h        display this help message

Options:
  --tcp-service  -t service-address   service-address is host[:port]
  --udp-service  -u service-address   service-address is host[:port]
  --fwmark-service  -f fwmark         fwmark is an integer greater than zero
  --ipv6         -6                   fwmark entry uses IPv6
  --scheduler    -s scheduler         one of rr|wrr|lc|wlc|lblc|lblcr|dh|sh|sed|nq,
                                      the default scheduler is wlc.
  --pe            engine              alternate persistence engine may be sip,
                                      not set by default.
  --persistent   -p [timeout]         persistent service
  --netmask      -M netmask           persistent granularity mask
  --real-server  -r server-address    server-address is host (and port)
  --gatewaying   -g                   gatewaying (direct routing) (default)
  --ipip         -i                   ipip encapsulation (tunneling)
  --masquerading -m                   masquerading (NAT)
  --weight       -w weight            capacity of real server
  --u-threshold  -x uthreshold        upper threshold of connections
  --l-threshold  -y lthreshold        lower threshold of connections
  --mcast-interface interface         multicast interface for connection sync
  --syncid sid                        syncid for connection sync (default=255)
  --connection   -c                   output of current IPVS connections
  --timeout                           output of timeout (tcp tcpfin udp)
  --daemon                            output of daemon information
  --stats                             output of statistics information
  --rate                              output of rate information
  --exact                             expand numbers (display exact values)
  --thresholds                        output of thresholds information
  --persistent-conn                   output of persistent connection info
  --nosort                            disable sorting output of service/server entries
  --sort                              does nothing, for backwards compatibility
  --ops          -o                   one-packet scheduling
  --numeric      -n                   numeric output of addresses and ports
  --sched-flags  -b flags             scheduler flags (comma-separated)

```

