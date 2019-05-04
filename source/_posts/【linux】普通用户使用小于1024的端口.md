---
title: 【linux】普通用户使用小于1024的端口
date: 2019-04-02 13:20:48
tags:
- linux
---

普通用户启动服务需要绑定80和443端口，而在linux中，1024以下端口被称为privileged port，即特权端口，特权端口只能被由root启动的进程监听。

### 1. 权限绑定

#### 设置特权

为特定可执行文件赋予绑定特权端口的权限（永久）： 

```shell
setcap 'cap_net_bind_service=+ep' /path/to/program 

sudo setcap 'cap_net_bind_service=+ep' $(readlink -f $(which node))
```

#### 取消特权

```shell
setcap -r /path/to/program 
```

#### Now for the caveats:

1. You will need at least a 2.6.24 kernel
2. This won't work if your file is a script. (ie, uses a #! line to launch an interpreter). In this case, as far I as understand, you'd have to apply the capability to the interpreter executable itself, which of course is a security nightmare, since any program using that interpreter will have the capability. I wasn't able to find any clean, easy way to work around this problem.
3. Linux will disable LD_LIBRARY_PATH on any `program` that has elevated privileges like `setcap` or `suid`. So if your `program` uses its own `.../lib/`, you might have to look into another option like port forwarding.

#### 参考链接

[Is there a way for non-root processes to bind to “privileged” ports on Linux?](https://stackoverflow.com/questions/413807/is-there-a-way-for-non-root-processes-to-bind-to-privileged-ports-on-linux)

[Unset `setcap` additional capabilities on excutable](https://unix.stackexchange.com/questions/303423/unset-setcap-additional-capabilities-on-excutable)



### 2. 端口转发

 首先绑定1024以上的端口，检查IP FORWARD功能是否开启：

```shell
#修改文件
vi /etc/sysctl.conf
net.ipv4.ip_forward = 1
#重新加载
sysctl -p /etc/sysctl.conf
```

配置端口转发，root权限执行

```shell
iptables -A PREROUTING -t nat -p tcp --dport 80 -j REDIRECT --to-port 8080
```



### 3. setuid（不安全）

```shell
chown root:root nginx
chmod 4755 nginx
```



### 4. 反向代理（nginx）