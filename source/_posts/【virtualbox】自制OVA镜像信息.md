---
title: 【virtualbox】自制OVA镜像信息
date: 2019-06-06 15:10:12
tags: 
- virtualbox
- ova
---

## OVA镜像信息

### CentOS-7-x86_64-Minimal-1708.ova
- 登录：root/q




### Jenkins.ova

- 登录：root/q

- Web：liboo/liboo
- Web地址：http://< ip : 8080 >



### ZabbixServer.ova

- 登录：root/q

- MySQL：root/liboo，zabbix/password
- Web地址：http://< ip >/zabbix



### ZabbixProxy.ova

- 登录：root/q

- MySQL：root/liboo，zabbix/password
- 修改：/etc/zabbix/zabbix_proxy.conf

```
Hostname=<hostname>
Server=<zabbix_server>
DBPassword=<zabbix_proxy_user_password>
```




### ZabbixAgent.ova
- 登录：root/q

- 修改：/etc/zabbix/zabbix_agentd.conf

```
Server=<zabbix_server_or_zabbix_proxy_ip>
ServerActive=<zabbix_server_or_zabbix_proxy_ip>
Hostname=<hostname>
```




### Nagios.ova
- 登录：root/q

- Web：nagiosadmin/liboo
- Web地址：http://< ip >/nagios
- MySQL：root/liboo，ndoutils/ndoutils_password



### LVS1+Keepalived1+Web1.ova

- 登录：root/q

- Web地址：http://< ip >
- 修改：

```
$ cat /etc/sysconfig/network-scripts/ifcfg-lo:0

DEVICE=lo:0
IPADDR=192.168.23.253	# 需要修改
NETMASK=255.255.255.255
ONBOOT=yes

$ cat /etc/keepalived/keepalived.conf

! Configuration File for keepalived

vrrp_instance VI_1 {
    state MASTER
    interface enp0s3	# 需要修改
    virtual_router_id 91
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.23.253	# 需要修改
    }
    notify_backup "/root/notify_node.sh b"
    notify_master "/root/notify_node.sh m"
}

virtual_server 192.168.23.253 0 {	# 需要修改
    delay_loop 6
    lb_algo wrr
    lb_kind DR
    persistence_timeout 50
    protocol TCP

    real_server 192.168.23.56 0 {	# 需要修改
        weight 3
        TCP_CHECK {
            connect_ip 192.168.23.56	# 需要修改
            connect_port 80
        }
    }
    real_server 192.168.23.58 0 {	# 需要修改
        weight 3
        TCP_CHECK {
            connect_ip 192.168.23.58	# 需要修改
            connect_port 80
        }
    }
}
```



### LVS2+Keepalived2+Web2.ova

- 登录：root/q
- Web地址：http://< ip >
- 修改：


```
$ cat /etc/sysconfig/network-scripts/ifcfg-lo:0

DEVICE=lo:0
IPADDR=192.168.23.253	# 需要修改
NETMASK=255.255.255.255
ONBOOT=yes

$ cat /etc/keepalived/keepalived.conf

! Configuration File for keepalived

vrrp_instance VI_1 {
    state BACKUP
    interface enp0s3	# 需要修改
    virtual_router_id 91
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.23.253	# 需要修改
    }
    notify_backup "/root/notify_node.sh b"
    notify_master "/root/notify_node.sh m"
}

virtual_server 192.168.23.253 0 {	# 需要修改
    delay_loop 6
    lb_algo wrr
    lb_kind DR
    persistence_timeout 50
    protocol TCP

    real_server 192.168.23.56 0 {	# 需要修改
        weight 3
        TCP_CHECK {
            connect_ip 192.168.23.56	# 需要修改
            connect_port 80
        }
    }
    real_server 192.168.23.58 0 {	# 需要修改
        weight 3
        TCP_CHECK {
            connect_ip 192.168.23.58	# 需要修改
            connect_port 80
        }
    }
}
```

