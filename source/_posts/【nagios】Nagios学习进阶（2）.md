---
title: 【nagios】Nagios学习进阶（2）
date: 2019-05-21 14:49:51
tags:
- nagios
---

### Nagios安装目录

```
[root@localhost nagios]# pwd
/usr/local/nagios
[root@localhost nagios]# ll
total 12
drwxrwxr-x.  2 nagios nagios   38 May 20 10:46 bin
drwxrwxr-x.  3 nagios nagios   96 May 21 09:35 etc
drwxrwxr-x.  2 nagios nagios 4096 May 21 14:44 libexec
drwxrwxr-x.  2 nagios nagios 4096 May 20 10:46 sbin
drwxrwxr-x. 15 nagios nagios 4096 May 20 11:05 share
drwxrwxr-x.  5 nagios nagios  147 May 21 14:53 var
[root@localhost nagios]#

```

### Nagios安装目录详解

| 目录名       | 注解                                                         |
| ------------ | ------------------------------------------------------------ |
| bin          | Nagios 可执行程序所在目录                                    |
| etc          | Nagios 配置文件所在目录                                      |
| sbin         | Nagios CGI 文件所在目录，也就是执行外部命令所需文件所在的目录 |
| share        | Nagios网页文件所在的目录                                     |
| libexec      | Nagios 外部插件所在目录                                      |
| var          | Nagios 日志文件、lock 等文件所在的目录                       |
| var/archives | Nagios 日志自动归档目录                                      |
| var/rw       | 用来存放外部命令文件的目录                                   |

### Nagios配置目录详解

| **文件名或目录名**      | **用途**                                                     |
| ----------------------- | ------------------------------------------------------------ |
| cgi.cfg                 | 控制CGI访问的配置文件                                        |
| htpasswd.users          | Nagios账户信息                                               |
| nagios.cfg              | Nagios 主配置文件                                            |
| resource.cfg            | 变量定义文件，又称为资源文件，在些文件中定义变量，以便由其他配置文件引用，如$USER1$ |
| objects                 | objects 是一个目录，在此目录下有很多配置文件模板，用于定义Nagios 对象 |
| objects/commands.cfg    | 命令定义配置文件，其中定义的命令可以被其他配置文件引用       |
| objects/contacts.cfg    | 定义联系人和联系人组的配置文件                               |
| objects/localhost.cfg   | 定义监控本地主机的配置文件                                   |
| objects/printer.cfg     | 定义监控打印机的一个配置文件模板，默认没有启用此文件         |
| objects/switch.cfg      | 定义监控路由器的一个配置文件模板，默认没有启用此文件         |
| objects/templates.cfg   | 定义主机和服务的一个模板配置文件，可以在其他配置文件中引用   |
| objects/timeperiods.cfg | 定义Nagios 监控时间段的配置文件                              |
| objects/windows.cfg     | 监控Windows 主机的一个配置文件模板，默认没有启用此文件       |

*参考：*

*[Linux下Nagios的安装与配置](https://www.cnblogs.com/mchina/archive/2013/02/20/2883404.html)*



### 添加Linux 主机，监控对外开放的服务

1. 添加配置文件

```
touch /usr/local/nagios/etc/objects/192.168.23.123.cfg

vi /usr/local/nagios/etc/objects/192.168.23.123.cfg

define host {   
        use                     linux-server 
        host_name               host123
        alias                   Host 123
        address                 192.168.32.163
}     
define hostgroup {      
        hostgroup_name          host123-servers
        alias                   Host123 servers
        members                 host123     
}
define service {  
        use                     local-service
        host_name               host123
        service_description     check-host-alive
        check_command           check-host-alive
}

```

2. 添加配置项

```
vi /usr/local/nagios/etc/nagios.cfg

cfg_file=/usr/local/nagios/etc/objects/192.168.23.123.cfg
```

3. 验证配置

```
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

4. 重启服务

```
systemctl restart nagios
```



### Installing NRPE Using the Install Script

You must run the following commands as root.

```
cd /tmp
```

For CentOS & RHEL 5-7, Fedora 14+, SLES & OpenSUSE 11+, Ubuntu 12+, Debian 6+:

```
wget http://assets.nagios.com/downloads/nagiosxi/agents/linux-nrpe-agent.tar.gz
```

Unpack the installer, and run the install script:

```
tar xzf linux-nrpe-agent.tar.gz
cd linux-nrpe-agent
./fullinstall
```

The script takes care of the following setup:

- Installs prerequisite packages
- Creates required users and groups
- Defines services for xinetd
- Compiles and installs the NRPE agent and Nagios plugins
- Configures the firewall (except on SLES)
- Configures the agent

The script will stop to prompt you once to ask for the IP address(es) for your monitoring server(s).

You will need to type either a single address or multiple addresses separated by spaces. This will configure the xinetd superdaemon to allow connections from those addresses to the NRPE agent.

You now have NRPE installed. You may remove any installation files in the tmp directory.

 