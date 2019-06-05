---
title: 【jenkins】Jenkins入门学习（1）
date: 2019-04-29 20:28:46
tags:
- jenkins
---

### Jenkins概念

Jenkins是一个开源的、可扩展的持续集成、交付、部署（软件/代码的编译、打包、部署）的基于web界面的平台。允许持续集成和持续交付项目，无论用的是什么平台，可以处理任何类型的构建或持续集成。

**官网：**[https://jenkins.io/zh/](https://jenkins.io/zh/)

**官方文档：**[https://jenkins.io/zh/doc/](https://jenkins.io/zh/doc/)

### Jenkins特性：

- 开源的java语言开发持续集成工具，支持CI，CD；

- 易于安装部署配置：可通过yum安装,或下载war包以及通过docker容器等快速实现安装部署，可方便web界面配置管理；

- 消息通知及测试报告：集成RSS/E-mail通过RSS发布构建结果或当构建完成时通过e-mail通知，生成JUnit/TestNG测试报告；

- 分布式构建：支持Jenkins能够让多台计算机一起构建/测试；

- 文件识别:Jenkins能够跟踪哪次构建生成哪些jar，哪次构建使用哪个版本的jar等； 

- 丰富的插件支持:支持扩展插件，你可以开发适合自己团队使用的工具，如git，svn，maven，docker等。

### Jenkins安装

安装最低配置：不少于256M内存，不低于1G磁盘，JDK版本>=8（openjdk也可以）。

**安装jenkins：**

```
[root@localhost ~]# yum install -y java-1.8.0-openjdk       //安装openjdk，因为jenkins基于java开发

[root@localhost ~]# wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo      //wget -O 下载文件并以指定的文件名保存
[root@localhost ~]# cat /etc/yum.repos.d/jenkins.repo 

[jenkins]
name=Jenkins
baseurl=http://pkg.jenkins.io/redhat
gpgcheck=1      //这里会检测key

[root@localhost ~]# rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key         //安装jenkins key
[root@localhost ~]# yum install -y jenkins
或
[root@localhost ~]# wget https://pkg.jenkins.io/redhat-stable/jenkins-2.164.3-1.1.noarch.rpm
[root@localhost ~]# yum localinstall -y jenkins-2.164.3-1.1.noarch.rpm
```

**启动jenkins：**

```
[root@lzx ~]# systemctl start jenkins
或
[root@localhost ~]# wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
[root@localhost ~]# java -jar jenkins.war --httpPort=8080

[root@lzx ~]# ps aux |grep jenkins
jenkins    1157 84.7 10.0 2320896 100884 ?      Ssl  23:28   0:08 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
```

```
[root@lzx ~]# less /var/log/jenkins/jenkins.log         //查看jenkins日志，截取下面部分
Jenkins initial setup is required. An admin user has been created and a password generated
.
Please use the following password to proceed to installation:

77faa20f2ad544f7bcb6593b1cf1436b        //admin密码，初始化安装时会用到

This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword        //admin密码也可以在这里查到
```

**访问安装：**

打开浏览器，访问 http://< ip : port > 进行安装



*参考链接：*

*[Jenkins入门（一）](https://blog.csdn.net/miss1181248983/article/details/82840006)*

