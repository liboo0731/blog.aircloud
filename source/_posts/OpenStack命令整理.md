---
title: OpenStack命令整理
date: 2017-09-11 17:47:49
tags: 
    - openstack
---

### 文件批量复制

```bash
cat allrpm.txt |while read name ;do cp -a /mnt/Packages/$name.rpm /root/orgin/ ;done
```

### 批量删除云主机

```bash
source ~/keystonerc_admin
a=`nova list |grep '[a-zA-Z0-9]\{8\}-[a-zA-Z0-9]\{4\}-[a-zA-Z0-9]\{4\}-[a-zA-Z0-9]\{4\}-[a-zA-Z0-9]\{12\}' -o`
for b in $a
do
    echo $b
    nova force-delete $b
    #nova delete $b
done
```

### 设置安全组规则（Nova-network）

```bash
source ~/keystonerc_admin
nova secgroup-add-rule default tcp 1 65535 0.0.0.0/0
nova secgroup-add-rule default udp 1 65535 0.0.0.0/0
nova secgroup-add-rule default icmp -1 -1 0.0.0.0/0
```

###  设置安全组规则（Neutron）

```bash
source ~/keystonerc_admin
a=`neutron security-group-rule-list |grep '[a-zA-Z0-9]\{8\}-[a-zA-Z0-9]\{4\}-[a-zA-Z0-9]\{4\}-[a-zA-Z0-9]\{4\}-[a-zA-Z0-9]\{12\}' -o`
for b in $a
do
	neutron security-group-rule-delete $b
done
a1=`neutron security-group-list |grep '[a-zA-Z0-9]\{8\}-[a-zA-Z0-9]\{4\}-[a-zA-Z0-9]\{4\}-[a-zA-Z0-9]\{4\}-[a-zA-Z0-9]\{12\}' -o`
for b1 in $a1
do
	neutron security-group-rule-create --protocol tcp --port-range-min 1 --port-range-max 65535 --direction ingress $b1
	neutron security-group-rule-create --protocol udp --port-range-min 1 --port-range-max 65535 --direction ingress $b1
	neutron security-group-rule-create --protocol icmp --direction ingress $b1
	neutron security-group-rule-create --protocol icmp --direction egress $b1
done
```

### 上传镜像（Icehouse）

```bash
source /root/keystonerc_admin
glance image-create --name cirros --file cirros-0.3.1-x86_64-disk.img --disk-format qcow2 --container-format bare --is-public True --progress
```

### 上传镜像（Mitaka）

```bash
source /root/keystonerc_admin
glance image-create --name cirros --file cirros-0.3.1-x86_64-disk.img --disk-format qcow2 --container-format bare --progress
```

### 创建网络（Neutron）

```bash
source ~/keystonerc_admin
neutron net-create private
neutron subnet-create private 172.16.1.0/24 --name private_subnet --dns-nameserver 223.5.5.5
neutron router-create router1
neutron router-interface-add router1 private_subnet
neutron net-create public --router:external
neutron subnet-create public 10.0.0.0/24 --name public_subnet --enable_dhcp=False --allocation_pool start=10.0.0.100,end=10.0.0.200 --gateway 10.0.0.1
neutron router-gateway-set router1 public
neutron floatingip-create public
```

