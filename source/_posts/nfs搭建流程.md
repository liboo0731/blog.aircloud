---
title: NFS服务器搭建
date: 2018-9-3 16:13:12
tags: 

- nfs
---

# NFS 服务器搭建

## 1.下载
```
yum install nfs-utils nfs-utils-lib
yum install portreserve            
```

## 2.开启服务
```
/etc/init.d/portreserve start
```
注意yum install portreserve实际下载的包名，这里要启动这个服务。

```
/etc/init.d/nfs start
chkconfig --level 35 portreserve on
chkconfig --level 35 nfs on
```

## 3.服务端设置
```
vi /etc/exports
```
文件中写入相应的信息如

```
/nfsshare 192.168.0.101(rw,sync,no_root_squash) 192.168.1.1(rw)
```
`/nfsshare` 是可以访问目录的绝对路径  
`192.168.0.101`是访问者的ip  
`rw`访问这可读可写 `ro` read only 
`sync` 同步模式    
`async` 非同步  
`no_root_squash` root 用户登录有最高权限  
`root_squash`root用户没有权限，匿名登录  
`all_squash`所有用户匿名登录

同一目录对多个主机开放，可以连续写下去。可以用通配符，但是不能用在IP网段上。  
修改文件后用`exportfs -ra`重新导出


## 4.客户端设置
显示服务端有哪些贡献目录
```
showmount -e <服务端ip>
```
客户端挂载那个目录

```
mount -r nfs 192.168.11.124:/nfsshare /mnt/nfsshare
```

永久挂载
```
vi /etc/fstab

192.168.11.124：/nfsshare /mnt nfs defaults 0 0
```

## 5.一些指令

showmount -e 显示本地可贡献目录  
showmount -e <server ip> 显示远程可共享目录  
exportfs -v 显示本地共享目录和选项  
exportfs -a 显示所有共享目录


参考<https://www.tecmint.com/how-to-setup-nfs-server-in-linux/>  
需要更多信息请阅读<http://cn.linux.vbird.org/linux_server/0330nfs.php#nfsserver_need>
