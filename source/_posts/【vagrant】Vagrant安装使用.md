---
title: 【vagrant】Vagrant安装使用
date: 2019-05-25 18:24:32
tags:
- vagrant
---

### 简介

Vagrant是一个基于Ruby的工具，用于创建和部署虚拟化开发环境。它 使用Oracle的开源 VirtualBox虚拟化系统，使用 Chef创建自动化虚拟环境。

### 下载

[Vagrant Download](https://www.vagrantup.com/downloads.html)

### 安装

#### 在CentOS 7上安装VirtualBox

```
首先安装VirtualBox依赖项。
# yum install gcc make perl -y

更新内核并重启生效
# yum -y install kernel kernel-headers kernel-tools kernel-devel
# rpm -e kernel-<old>
# reboot

接下来添加VirtualBox库。
# cd /etc/yum.repo.d/
# wget http://download.virtualbox.org/virtualbox/rpm/rhel/virtualbox.repo

现在安装和构建内核模块。
# yum install -y VirtualBox-6.0
或
# yum localinstall -y VirtualBox-6.0-6.0.8_130520_el7-1.x86_64.rpm
# /sbin/rcvboxdrv setup

```

#### 在CentOS 7上安装Vagrant

```
下载最新的vagrant，安装
# yum localinstall -y vagrant_2.2.4_x86_64.rpm
```

### 使用

```
mkdir -p ~/dev
cd ~/dev
vagrant init centos/7  # 用 centos/7 进行 box 初始化
vagrant up  # 启动环境

vagrant ssh  # SSH 登录
cd /vagrant  # 切换到开发目录，也就是宿主机上的 `~/dev`

vagrant package  # 打包开发环境

vagrant box add centos7 ~/box/package.box  # 添加 package.box 镜像并命名为 centos7
cd ~/dev  # 切换到项目目录
vagrant init centos7  # 用 centos7 镜像初始化。

# 常用命令
vagrant init  # 初始化
vagrant up  # 启动虚拟机
vagrant halt  # 关闭虚拟机
vagrant reload  # 重启虚拟机
vagrant ssh  # SSH 至虚拟机
vagrant status  # 查看虚拟机运行状态
vagrant destroy  # 销毁当前虚拟机
```

