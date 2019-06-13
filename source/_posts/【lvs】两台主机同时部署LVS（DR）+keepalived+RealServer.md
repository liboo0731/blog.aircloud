---
title: 【lvs】两台主机同时部署LVS（DR）+keepalived+RealServer
date: 2019-04-26 13:27:41
tags:
- lvs
---

## 两台主机同时部署LVS（DR）+keepalived+RealServer

### 问题原因

当 Director1（Master 主)，比如使用 rr 。会转发大约 50% 的包从 Director1 到  Director2 （Backup）。这时 Director2 因为 LVS-DR 的配置规则会接着给这些包在做一次 Load Balance 。又发回给 Director1 。这时会产生一个死的循环。随着时间的推移，不但不能正常的处理连接，服务器也会崩溃。



### RealServer 配置网卡和ARP 规则

```
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-lo:0

DEVICE=lo:0
IPADDR=192.168.23.253
NETMASK=255.255.255.255
ONBOOT=yes

[root@localhost ~]# vi /etc/sysctl.conf 

net.ipv4.conf.lo.arp_ignore = 1
net.ipv4.conf.lo.arp_announce = 2
#net.ipv4.conf.enp0s3.arp_ignore = 1
#net.ipv4.conf.enp0s3.arp_announce = 2
net.ipv4.conf.all.arp_ignore = 1
net.ipv4.conf.all.arp_announce = 2

[root@localhost ~]# sysctl -p
[root@localhost ~]# systemctl restart network
```



### Keepalived 配置，只能有一个LVS 存活

```
[root@localhost ~]# vi keepalived.conf

! Configuration File for keepalived
global_defs {
   #notification_email {
        #aaa@email.com
   #}
   #notification_email_from keepalived@email.com
   #smtp_server 127.0.0.1
   #smtp_connect_timeout 30
}

vrrp_instance web {
    state BACKUP 
    interface enp0s3 
    virtual_router_id 51
    virtual_ipaddress {
        192.168.23.253 
    }
    notify_backup "/root/notify_node.sh b"
    notify_master "/root/notify_node.sh m"
}

virtual_server 192.168.23.253 0 { 
    delay_loop 6
    lvs_sched wrr 
    lvs_method DR
    protocol TCP

    real_server 192.168.23.65 0 { 
        weight 3
        TCP_CHECK {
            connect_ip 192.168.23.65 
            connect_port 80 
        }
    }
    real_server 192.168.23.67 0 { 
        weight 3
        TCP_CHECK {
            connect_ip 192.168.23.67 
            connect_port 80 
        }
    }
}

---------------------------------------------------

#!/bin/bash
# /root/notify_node.sh

case $1 in
b)
/usr/sbin/ipvsadm -C
;;
m)
/usr/sbin/ipvsadm -R <<EOF
-A -t 192.168.23.253:0 -s wrr -p 360
-a -t 192.168.23.253:0 -r 192.168.23.65:0 -g -w 3
-a -t 192.168.23.253:0 -r 192.168.23.67:0 -g -w 3
EOF
;;
esac
/usr/sbin/ipvsadm -Ln
```



### 相关命令操作

```
# 安装ipvsadm 
yum install -y ipvsadm

# LVS规则保存及恢复
ipvsadm -Sn >/etc/sysconfig/ipvsadm
ipvsadm -R </etc/sysconfig/ipvsadm

# 关闭selinux
setenforce 0
sed -i 's/=enforcing/=disabled/g' /etc/sysconfig/selinux

# 抓包
tcpdump -i enp0s3 vrrp -n

# 防火墙配置
firewall-cmd --direct --permanent --add-rule ipv4 filter INPUT 0  --protocol vrrp -j ACCEPT
firewall-cmd --reload

# 测试访问
for i in {1..10}; do curl 192.168.23.253 ;done
for i in {1..10}; do curl -o /dev/null -s -w %{http_code} 192.168.23.253; echo ;done
```

