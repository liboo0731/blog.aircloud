---
title: 【lvs】LVS学习实践（3）
date: 2019-04-25 10:21:58
tags:
- lvs
---

## 使用keepalived 实现LVS 的高可用

#### 服务器IP 信息

```
lvs1+keepalived：192.168.1.2
lvs2+keepalived：192.168.1.3
web1：192.168.1.4
web2：192.168.1.5
vip：192.168.1.6
```

### LVS 服务器安配置

```
# 安装相关软件
[root@localhost ~]# yum install -y ipvsadm keepalived

# 配置keepalived.conf
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

vrrp_instance lvs_web {
    state MASTER	#另一个设置BACKUP
    interface ens160
    virtual_router_id 51
    virtual_ipaddress {
        192.168.1.6 
    }
}

virtual_server 192.168.1.6 0 {
    delay_loop 6
    lvs_sched wrr 
    lvs_method DR
    protocol TCP

    real_server 192.168.1.4 0 { 
        weight 3
        TCP_CHECK {
            connect_ip 192.168.1.4 
            connect_port 80
        }
    }
    real_server 192.168.1.5 0 { 
        weight 3
        TCP_CHECK {
            connect_ip 192.168.1.5 
            connect_port 80
        }
    }
}

# 启动服务
[root@localhost ~]# systemctl restart keepalived
[root@localhost ~]# systemctl enable keepalived

# 查看lvs状态
[root@localhost ~]# ipvsadm -Ln
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

