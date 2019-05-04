---
title: 【linux】Capabilities简介
date: 2018-09-29 14:20:11
tags:
- linux
---

Capabilities的主要思想在于分割root用户的特权，即将root的特权分割成不同的能力，每种能力代表一定的特权操作。例如：能力CAP_SYS_MODULE表示用户能够加载(或卸载)内核模块的特权操作，而CAP_SETUID表示用户能够修改进程用户身份的特权操作。在Capbilities中系统将根据进程拥有的能力来进行特权操作的访问控制。

在Capilities中，只有进程和可执行文件才具有能力，每个进程拥有三组能力集，分别称为cap_effective, cap_inheritable, cap_permitted(分别简记为:pE,pI,pP)，其中cap_permitted表示进程所拥有的最大能力集；cap_effective表示进程当前可用的能力集，可以看做是cap_permitted的一个子集；而cap_inheitable则表示进程可以传递给其子进程的能力集。系统根据进程的cap_effective能力集进行访问控制，cap_effective为cap_permitted的子集，进程可以通过取消cap_effective中的某些能力来放弃进程的一些特权。可执行文件也拥有三组能力集，对应于进程的三组能力集，分别称为cap_effective, cap_allowed 和 cap_forced（分别简记为fE,fI,fP），其中，cap_allowed表示程序运行时可从原进程的cap_inheritable中集成的能力集，cap_forced表示运行文件时必须拥有才能完成其服务的能力集；而cap_effective则表示文件开始运行时可以使用的能力。

 **Linux内核从2.2版本开始，就加进的Capabilities的概念与机制，并随着版本升高逐步得到改进。在linux中，root权限被分割成一下29中能力：**

- CAP_CHOWN:修改文件属主的权限

- CAP_DAC_OVERRIDE:忽略文件的DAC访问限制

- CAP_DAC_READ_SEARCH:忽略文件读及目录搜索的DAC访问限制

- CAP_FOWNER：忽略文件属主ID必须和进程用户ID相匹配的限制

- CAP_FSETID:允许设置文件的setuid位

- CAP_KILL:允许对不属于自己的进程发送信号

- CAP_SETGID:允许改变进程的组ID

- CAP_SETUID:允许改变进程的用户ID

- CAP_SETPCAP:允许向其他进程转移能力以及删除其他进程的能力

- CAP_LINUX_IMMUTABLE:允许修改文件的IMMUTABLE和APPEND属性标志

- CAP_NET_BIND_SERVICE:允许绑定到小于1024的端口

- CAP_NET_BROADCAST:允许网络广播和多播访问

- CAP_NET_ADMIN:允许执行网络管理任务

- CAP_NET_RAW:允许使用原始套接字

- CAP_IPC_LOCK:允许锁定共享内存片段

- CAP_IPC_OWNER:忽略IPC所有权检查

- CAP_SYS_MODULE:允许插入和删除内核模块

- CAP_SYS_RAWIO:允许直接访问/devport,/dev/mem,/dev/kmem及原始块设备

- CAP_SYS_CHROOT:允许使用chroot()系统调用

- CAP_SYS_PTRACE:允许跟踪任何进程

- CAP_SYS_PACCT:允许执行进程的BSD式审计

- CAP_SYS_ADMIN:允许执行系统管理任务，如加载或卸载文件系统、设置磁盘配额等

- CAP_SYS_BOOT:允许重新启动系统

- CAP_SYS_NICE:允许提升优先级及设置其他进程的优先级

- CAP_SYS_RESOURCE:忽略资源限制

- CAP_SYS_TIME:允许改变系统时钟

- CAP_SYS_TTY_CONFIG:允许配置TTY设备

- CAP_MKNOD:允许使用mknod()系统调用

- CAP_LEASE:允许修改文件锁的FL_LEASE标志



*参考链接：[Android的能力(capability)机制](https://blog.csdn.net/zhao_cancan/article/details/38366541)*

