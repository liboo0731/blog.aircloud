---
title: 【nagios】Nagios学习进阶（4）
date: 2019-05-23 09:47:50
tags:
- nagios
---

### 1. 连接被监控端出现Connection reset by peer 的问题

```
使用官方安装脚本（./fullinstall）在配置Allow host 时未输入监控端地址或输入错误导致的问题

# vi /etc/xinetd.d/nrpe
only_from       = 127.0.0.1 192.168.32.166

# systemctl restart xinetd

监控端测试：
/usr/local/nagios/libexec/check_nrpe -H 192.168.23.164
被监控端测试：
/usr/local/nagios/libexec/check_nrpe -H 127.0.0.1

```

### 2. 监控端只需要编译出check_nrpe 插件，并放到相应的位置即可

```
1. 直接从被监控端拷贝，并修改权限
2. 源码安装
./configure
make check_nrpe
make install-plugin

```

