---
title: 【lvs】LVS学习实践（5）
date: 2019-04-27 08:58:27
tags:
- lvs
---

## LVS + Keepalived 双主配置

一般场景中实现LVS高可用及后端的LB，同时只有一台LVS提供服务，另一台作为Backup，显然没有做到最大利用。我们可以让两台LVS都作为Master，并又互为Backup，与master-backup架构相比，master-master架构需要多加一个vip。

### 服务器IP规划

```
lvs1+keepalived：192.168.1.2
lvs2+keepalived：192.168.1.3
web1：192.168.1.4
web2：192.168.1.5
vip：192.168.1.6，192.168.1.7
```

### Director（LVS+Keepalvied）主机配置

#### Master Host1 配置

```
[root@localhost ~]# cat /etc/keepalived/keepalived.conf
vrrp_instance bl_one {
    state MASTER             #指定Keepalived的角色，MASTER为主服务器，BACKUP为备用服务器
    interface eth0           #指定HA监测的接口
    lvs_sync_daemon_interface eth0
    virtual_router_id 38     #虚拟路由标识(1-255)，在一个VRRP实例中主备服务器ID必须一样
    priority 150             #优先级，数字越大越优先，主服务器优先级必须高于备服务器
    advert_int 3             #设置主备之间同步检查时间间隔，单位秒
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress { #定义虚拟IP地址
      192.168.1.6
    }
}
vrrp_instance bl_two {
    state BACKUP
    interface eth0
    lvs_sync_daemon_interface eth0
    virtual_router_id 48
    priority 120
    advert_int 3
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
      192.168.1.7
    }
}
virtual_server 192.168.1.6 80 {
    delay_loop 3                    #设置健康状态检查时间
    lb_algo rr                      #设置负载调度算法
    lb_kind DR                      #设置LVS实现负载均衡的机制
    persistence_timeout 50          #会话保持时间
    protocol TCP
    real_server 192.168.1.4 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 10       #设置响应超时时间
            nb_get_retry 3           #设置超时重试次数
            delay_before_retry 3     #设置超时重试间隔
            connect_port 80
        }
    }
    real_server 192.168.1.5 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}
virtual_server 192.168.1.7 80 {
    delay_loop 3
    lb_algo rr
    lb_kind DR
    persistence_timeout 50
    protocol TCP
    real_server 192.168.1.4 80 {
        weight 1
        HTTP_GET {
            url {
                 path /index.html
                 status_code 200
              }
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
    real_server 192.168.1.5 80 {
        weight 1
        HTTP_GET {
            url {
                 path /index.html
                 status_code 200
              }
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
} 
```

#### Master Host2 配置

```
[root@localhost ~]# cat /etc/keepalived/keepalived.conf
vrrp_instance bl_one {
    state BACKUP
    interface eth0
    lvs_sync_daemon_interface eth0
    virtual_router_id 38
    priority 120
    advert_int 3
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
      192.168.1.6
    }
}
vrrp_instance bl_two {
    state MASTER
    interface eth0
    lvs_sync_daemon_interface eth0
    virtual_router_id 48
    priority 150
    advert_int 3
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
      192.168.1.7
    }
}
virtual_server 192.168.1.6 80 {
    delay_loop 3
    lb_algo rr
    lb_kind DR
    persistence_timeout 50
    protocol TCP
    real_server 192.168.1.4 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
    real_server 192.168.1.5 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
}
virtual_server 192.168.1.7 80 {
    delay_loop 3
    lb_algo rr
    lb_kind DR
    persistence_timeout 50
    protocol TCP
    real_server 192.168.1.4 80 {
        weight 1
        HTTP_GET {
            url {
                 path /index.html
                 status_code 200
              }
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
    real_server 192.168.1.5 80 {
        weight 1
        HTTP_GET {
            url {
                 path /index.html
                 status_code 200
              }
            connect_timeout 10
            nb_get_retry 3
            delay_before_retry 3
            connect_port 80
        }
    }
} 
```

以上创建了两个实例lb_one、lb_two ，其中A主机做为VIP1地址的master，B主机做为VIP2地址的master，A、B主机互为backup 。另外还有一个技巧点，在master-backup架构中也可以配置多个VIP地址，只需要在实例的VIP配置中增加地址、并在后面指定对应的虚地址后面的realserver即可。在主备脚架中配置多个VIP的方法如下：

```
virtual_ipaddress {
  192.168.122.100
  192.168.122.110
  ......
}  
```

### RealServer主机配置

```
[root@localhost # cat dr_client.sh
#!/bin/bash
VIP1=192.168.1.6
VIP2=192.168.1.7
# vip's broadcast
BROADCAST=192.168.1.255
. /etc/rc.d/init.d/functions
case "$1" in
start)
  echo "reparing for Real Server"
    echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
    echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
    echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
    echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
    ifconfig lo:0 $VIP1 netmask 255.255.255.255 broadcast $BROADCAST up
    ifconfig lo:1 $VIP2 netmask 255.255.255.255 broadcast $BROADCAST up
     /sbin/route add -host $VIP1 dev lo:0
     /sbin/route add -host $VIP2 dev lo:1
     ;;
stop)
     ifconfig lo:0 down
     ifconfig lo:1 down
    echo "0" >/proc/sys/net/ipv4/conf/lo/arp_ignore
    echo "0" >/proc/sys/net/ipv4/conf/lo/arp_announce
    echo "0" >/proc/sys/net/ipv4/conf/all/arp_ignore
    echo "0" >/proc/sys/net/ipv4/conf/all/arp_announce
     ;;
*)
     echo "Usage: lvs {start|stop}"
     exit 1
esac 
```

