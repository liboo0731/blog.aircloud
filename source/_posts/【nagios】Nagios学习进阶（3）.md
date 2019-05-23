---
title: 【nagios】Nagios学习进阶（3）
date: 2019-05-22 13:31:54
tags:
- nagios
---

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

 

### Nagios监控Linux主机内部信息

1. 监控端安装软件

```
# nagioscore
wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.3.tar.gz

# nagios-plugins
wget --no-check-certificate -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/archive/release-2.2.1.tar.gz

# nrpe（需要check_nrpe 插件）
wget http://assets.nagios.com/downloads/nagiosxi/agents/linux-nrpe-agent.tar.gz
```

2. 被监控端安装软件

```
# nrpe
wget http://assets.nagios.com/downloads/nagiosxi/agents/linux-nrpe-agent.tar.gz
```

3. 被监控端配置

```
# egrep -v "^#|^$" /usr/local/nagios/etc/nrpe.cfg

log_facility=daemon
pid_file=/var/run/nrpe.pid
server_port=5666
nrpe_user=nagios
nrpe_group=nagios
allowed_hosts=127.0.0.1
 
dont_blame_nrpe=1
debug=0
command_timeout=60
connection_timeout=300
include_dir=/usr/local/nagios/etc/nrpe
command[check_users]=/usr/local/nagios/libexec/check_users -w 5 -c 10
command[check_load]=/usr/local/nagios/libexec/check_load -w 15,10,5 -c 30,25,20
command[check_hda1]=/usr/local/nagios/libexec/check_disk -w 20% -c 10% -p /dev/sda2
command[check_zombie_procs]=/usr/local/nagios/libexec/check_procs -w 5 -c 10 -s Z
command[check_total_procs]=/usr/local/nagios/libexec/check_procs -w 150 -c 200

```

4. 监控端配置

```
# vi /usr/local/naigos/etc/objects/command.cfg

define command {

    command_name check_nrpe
    command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}

# vi /usr/local/nagios/etc/objects/192.168.23.123.cfg

define host {
    use     linux-server   
    host_name   host123
    alias       My Linux Server  
    address     192.168.23.123  
}
define service {
    use         generic-service
    host_name       host123
    service_description check host alive
    check_command       check-host-alive
}
define service {
    use         generic-service
    host_name       host123
    service_description check users
    check_command       check_nrpe!check_users
}
define service {
    use         generic-service
    host_name       host123
    service_description Load
    check_command       check_nrpe!check_load
}
define service {
    use         generic-service
    host_name       host123
    service_description check sda2
    check_command       check_nrpe!check_hda1
}
define service {
    use         generic-service
    host_name       host123
    service_description check zombie procs
    check_command       check_nrpe!check_zombie_procs
}
define service {
    use         generic-service
    host_name       host123
    service_description check total procs
    check_command       check_nrpe!check_total_procs
}

# vi /usr/local/nagios/etc/nagios.cfg

cfg_file=/usr/local/nagios/etc/objects/192.168.23.123.cfg
```

5. 验证配置

```
/usr/local/nagios/bin/nagios -v /usr/local/nagios/etc/nagios.cfg
```

6. 重启服务

```
systemctl restart nagios
```

