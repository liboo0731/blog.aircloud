---
title: 【clamav】ClamAV开源杀毒软件配置
date: 2018-03-17 21:01:25
tags:
- clamav
---

Clam AntiVirus（ClamAV）是免费而且开放源代码的防毒软件，软件与病毒码的更新皆由社群免费发布。

目前ClamAV主要是使用在由Linux、FreeBSD等Unix-like系统架设的邮件服务器上，提供电子邮件的病毒扫描服务。

ClamAV 杀毒是Linux平台最受欢迎的杀毒软件，ClamAV属于免费开源产品，支持多种平台，如：Linux/Unix、MAC OS X、Windows、OpenVMS。

ClamAV是基于病毒扫描的命令行工具，但同时也有支持图形界面的ClamTK工具。

ClamAV主要用于邮件服务器扫描邮件。

它有多种接口从邮件服务器扫描邮件，支持文件格式有如：ZIP、RAR、TAR、GZIP、BZIP2、HTML、DOC、PDF,、SIS CHM、RTF等等。

ClamAV有自动的数据库更新器，还可以从共享库中运行。

命令行的界面让ClamAV运行流畅，你不必以后台进程的方式运行，当你想扫描时只需输入扫描命令指定文件或目录就行。

### Centos7安装杀毒软件ClamAV

#### 安装EPEL源

（<http://www.cyberciti.biz/faq/installing-rhel-epel-repo-on-centos-redhat-7-x/>）

Clamav可以通过EPEL源来安装，所以要首先安装EPEL，可以采用两种方法来安装：

**第一种，通过命令行安装**

[root@server_for_product ~]# yum install epel-release

**第二种，使用下载好的安装包进行安装**

[root@server_for_product ~]# cd /tmp

[root@server_for_product tmp]# wget <https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm>

[root@server_for_product tmp]# yum install epel-release-7.noarch.rpm

#### 安装ClamAV

在安装了EPEL源后，运行下面的命令安装ClamAV

[root@server_for_product ~]# yum install clamav-server clamav-data clamav-update clamav-filesystem clamav clamav-scanner-systemd clamav-devel clamav-lib clamav-server-systemd -y

#### ClamAV 配置使用

在两个配置文件/etc/freshclam.conf和/etc/clamd.d/scan.conf中移除“Example”字符

[root@server_for_product ~]#sed -i -e "s/^Example/#Example/" /etc/freshclam.conf

[root@server_for_product ~]#sed -i -e "s/^Example/#Example/" /etc/clamd.d/scan.conf

**手动更新病毒库**

[root@server_for_product ~]# freshclam

freshclam命令通过文件/etc/cron.d/clamav-update来自动运行

默认情况下是禁止了自动更新功能，需要移除文件/etc/sysconfig/freshclam最后一行的配置才能启用

移除后自动更新： // FRESHCLAM_DELAY=disabled-warn # REMOVE ME

定义服务器类型（本地或者TCP），在这里定义为使用本地socket，将文件/etc/clamd.d/scan.conf中的这一行前面的注释符号去掉：

\#LocalSocket /var/run/clamd.scan/clamd.sock

**配置开机启动**

[root@server_for_product ~]# systemctl enable [clamd@scan.serv](mailto:clamd@scan.serv)ice

[root@server_for_product ~]# ln -s '/usr/lib/systemd/system/clamd\@scan.service' '/etc/systemd/system/multi-user.target.wants/clamd\@scan.service'

**启动并检查服务状态**

[root@server_for_product ~]# systemctl start clamd@scan

[root@server_for_product ~]# systemctl status clamd@scan

#### 备注

如果在手动更新病毒库的时候遇到错误：Update failed. Your network may be down or none of the mirrors listed in freshclam.conf is working.

此时就要删除掉旧的镜像地址文件

[root@server_for_product ~]# rm -f /var/lib/clamav/mirrors.dat

再手动更新一次病毒库

[root@server_for_product ~]# freshclam