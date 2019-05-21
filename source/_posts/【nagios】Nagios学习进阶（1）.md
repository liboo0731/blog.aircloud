---
title: 【nagios】Nagios学习进阶（1）
date: 2019-05-20 14:02:09
tags:
- nagios
---

## What Nagios Provides

Designed with **scalability** and **flexibility** in mind, Nagios gives you the peace of mind that comes from knowing your organization’s business processes won’t be affected by unknown outages.

Nagios is a powerful tool that provides you with instant awareness of your organization’s mission-critical IT infrastructure. Nagios allows you to detect and repair problems and mitigate future issues before they affect end-users and customers.

**By using Nagios, you can:**

- Plan for infrastructure upgrades before outdated systems cause failures
- Respond to issues at the first sign of a problem
- Automatically fix problems when they are detected
- Coordinate technical team responses
- Ensure your organization’s SLAs are being met
- Ensure IT infrastructure outages have a minimal effect on your organization’s bottom line
- Monitor your entire infrastructure and business processes



## How It Works

### Monitoring

IT staff configure Nagios to monitor critical IT infrastructure components, including system metrics, network protocols, applications, services, servers, and network infrastructure.

### Alerting

Nagios sends alerts when critical infrastructure components fail and recover, providing administrators with notice of important events. Alerts can be delivered via email, SMS, or custom script.

### Response

IT staff can acknowledge alerts and begin resolving outages and investigating security alerts immediately. Alerts can be escalated to different groups if alerts are not acknowledged in a timely manner.

### Reporting

Reports provide a historical record of outages, events, notifications, and alert response for later review. Availability reports help ensure your SLAs are being met.

### Maintenance

Scheduled downtime prevents alerts during scheduled maintenance and upgrade windows.

### Planning

Trending and capacity planning graphs and reports allow you to identify necessary infrastructure upgrades before failures occur.



## Features

### Comprehensive

MonitoringCore provides monitoring of all mission-critical infrastructure components in your IT infrastructure.

### Visibility

Gain a centralized view of your entire IT operations and review detailed status information through the web interface.

### Awareness

Alerts with escalation capabilities are delivered to IT staff via email and SMS to ensure fast detection of outages.

### Problem Remediation

Event handlers can automatically restart failed applications, servers, devices, and services when problems are found.

### Proactive Planning

Trending and capacity planning extensions allow you to proactively plan for upgrades.

### Reporting

Availability reports ensure SLAs are being met and historical reports provide record of critical information.

### Multi-Tenant

CapabilitiesMulti-user access and user-specific views can be configured to ensure clients see specific information.

### Extendable Architecture

Multiple API’s provide integration with in-house / third-party applications, and community-developed add-ons.



## Favorite Nagios Projects
### NCPA

NCPA is a cross-platform monitoring agent that runs on Windows, Linux/Unix, and Mac OS/X machines. Its features include both active and passive checks, remote management, and a local monitoring interface.

### DNX

DNX is a modular extension of Nagios that offloads a significant portion of the work normally done by Nagios to a distributed network of remote hosts. The DNX module ensures that work is distributed fairly and evenly among the registered DNX client hosts.

### NRPE

NRPE allows you to remotely execute Nagios plugins on other Linux/Unix machines. This allows you to monitor remote machine metrics (disk usage, CPU load, etc.). NRPE can also communicate with Windows agent addons like NSClient++, so you can check metrics on remote Windows machines as well.

### NRDP

NRDP is a flexible data transport mechanism and processor for Nagios. It is designed with a simple and powerful architecture that allows for it to be easily extended and customized to fit individual users’ needs. It uses standard ports protocols (HTTP(S) and XML) and can be implemented as a replacement for NSCA.

### NSCA

NSCA allows you to integrate passive alerts and checks from remote machines and applications with Nagios. Useful for processing security alerts, as well as deploying redundant and distributed Nagios setups.

### NSClient++

NSClient++ is a monitoring agent/daemon for Microsoft Windows systems that works with Nagios. Make monitoring Windows machines easy.

### Nagiosgraph

Nagiosgraph is graphing tool that parses output and performance data from Nagios plugins and stores the data in RRD files. nagiosgraph displays data in Nagios trends, as popups for hosts and services, or in separate reports. Easy to set up and eminently customizable.

### NSTI

NSTI is a Nagios addon that makes managing SNMP traps easier. NSTI provide a PHP frontend for the SNMPTT database backend and allows you to filter SNMP results quickly and effectively to get a comprehensive overview of the information you want to see.

### NConf

NConf is a PHP frontend for configuring Nagios. It differs from other tools by offering enterprise-class features like templates, dependencies and the ability to configure a large-scale, distributed Nagios server topology.

### NDOUtils

NDOUtils allows you to export current and historical data from one or more Nagios instances to a MySQL database. Several community addons use this as one of their data sources. NDOUtils consists of a standalone daemon, a Nagios event broker, and several helper utilities.

### BPI

Nagios Business Process Intelligence (BPI) is an advanced grouping tool that allows you to set more complex dependencies to determine groups states. Nagios BPI provides an interface to effectively view the ‘real’ state of the network. Rules for group states can be determined by the user, and parent-child relationships are easily identified when you need to ‘drill down’ on a problem. This tool can also be used in conjunction with a check plugin to allow for notifications through Nagios.

### NagVis

Nagvis is a visualization addon for Nagios. It can be used to visualize Nagios data, e.g. to display IT processes like a mail system or a network infrastructur



## CentOS Easy Setup

### Reference

*[Nagios](https://www.nagios.org/)*

*[Nagios Core Document](https://assets.nagios.com/downloads/nagioscore/docs/)*

### Security-Enhanced Linux

This guide is based on SELinux being disabled or in permissive mode. Steps to do this are as follows.

```
sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
setenforce 0
```

### Prerequisites

Perform these steps to install the pre-requisite packages.

```
yum install -y gcc glibc glibc-common wget unzip httpd php gd gd-devel perl postfix
```

### Downloading the Source

```
cd /tmp
wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.3.tar.gz
tar xzf nagioscore.tar.gz
```

### Compile

```
cd /tmp/nagioscore-nagios-4.4.3/
./configure
make all
```

### Create User And Group

This creates the nagios user and group. The apache user is also added to the nagios group.

```
make install-groups-users
usermod -a -G nagios apache
```

### Install Binaries

This step installs the binary files, CGIs, and HTML files.

```
make install
```

### Install Service / Daemon

This installs the service or daemon files and also configures them to start on boot. The Apache httpd service is also configured at this point.

**CentOS 5.x / 6.x**

```
make install-daemoninit
chkconfig --level 2345 httpd on
```

**CentOS 7.x**

```
make install-daemoninit
systemctl enable httpd.service
```

Information on starting and stopping services will be explained further on.

### Install Command Mode

This installs and configures the external command file.

```
make install-commandmode
```

### Install Configuration Files

This installs the *SAMPLE* configuration files. These are required as Nagios needs some configuration files to allow it to start.

```
make install-config
```

### Install Apache Config Files

This installs the Apache web server configuration files. Also configure Apache settings if required.

```
make install-webconf
```

### Configure Firewall

You need to allow port 80 inbound traffic on the local firewall so you can reach the Nagios Core web interface.

**CentOS 5.x / 6.x**

```
iptables -I INPUT -p tcp --destination-port 80 -j ACCEPT
service iptables save
ip6tables -I INPUT -p tcp --destination-port 80 -j ACCEPT
service ip6tables save
```

**CentOS 7.x**

```
firewall-cmd --zone=public --add-port=80/tcp
firewall-cmd --zone=public --add-port=80/tcp --permanent
```

### Create nagiosadmin User Account

You'll need to create an Apache user account to be able to log into Nagios.

The following command will create a user account called nagiosadmin and you will be prompted to provide a password for the account.

```
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```

When adding additional users in the future, you need to remove **-c** from the above command otherwise it will replace the existing nagiosadmin user *(and any other users you may have added)*.

### Start Apache Web Server

**CentOS 5.x / 6.x**

```
service httpd start
```

**CentOS 7.x**

```
systemctl start httpd.service
```

### Start Service / Daemon

This command starts Nagios Core.

**CentOS 5.x / 6.x**

```
service nagios start
```

 **CentOS 7.x**

```
systemctl start nagios.service
```

### Test Nagios

Nagios is now running, to confirm this you need to log into the Nagios Web Interface.

Point your web browser to the ip address or FQDN of your Nagios Core server, for example:

http://10.25.5.143/nagios

http://core-013.domain.local/nagios

You will be prompted for a username and password. The username is nagiosadmin (you created it in a previous step) and the password is what you provided earlier.

Once you have logged in you are presented with the Nagios interface. Congratulations you have installed Nagios Core.

### BUT WAIT ...

Currently you have only installed the Nagios Core engine. You'll notice some errors under the hosts and services along the lines of:

```
(No output on stdout) stderr: execvp(/usr/local/nagios/libexec/check_load, ...) failed. errno is 2: No such file or directory 
```

These errors will be resolved once you install the Nagios Plugins, which is covered in the next step.

### Installing The Nagios Plugins

Nagios Core needs plugins to operate properly. The following steps will walk you through installing Nagios Plugins.

These steps install nagios-plugins 2.2.1. Newer versions will become available in the future and you can use those in the following installation steps. Please see the [releases page on GitHub](https://github.com/nagios-plugins/nagios-plugins/releases) for all available versions.

Please note that the following steps install most of the plugins that come in the Nagios Plugins package. However there are some plugins that require other libraries which are not included in those instructions. Please refer to the following KB article for detailed installation instructions:

[Documentation - Installing Nagios Plugins From Source](https://support.nagios.com/kb/article.php?id=569)

### Prerequisites

Make sure that you have the following packages installed.

**CentOS 5.x**

```
yum install -y gcc glibc glibc-common make gettext automake wget openssl-devel net-snmp net-snmp-utils epel-release
yum install -y perl-Net-SNMP
cd /tmp
wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.60.tar.gz
tar xzf autoconf-2.60.tar.gz 
cd /tmp/autoconf-2.60
./configure 
make
make install
```

**CentOS 6.x / 7.x**

```
yum install -y gcc glibc glibc-common make gettext automake autoconf wget openssl-devel net-snmp net-snmp-utils epel-release
yum install -y perl-Net-SNMP
```

### Downloading The Source

```
cd /tmp
wget --no-check-certificate -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/archive/release-2.2.1.tar.gz
tar zxf nagios-plugins.tar.gz
```

### Compile + Install

```
cd /tmp/nagios-plugins-release-2.2.1/
./tools/setup
./configure
make
make install
```

### Test Plugins

Point your web browser to the ip address or FQDN of your Nagios Core server, for example:

http://10.25.5.143/nagios

http://core-013.domain.local/nagios

Go to a host or service object and "Re-schedule the next check" under the Commands menu. The error you previously saw should now disappear and the correct output will be shown on the screen.

### Service / Daemon Commands

Different Linux distributions have different methods of starting / stopping / restarting / status Nagios.

**CentOS 5.x / 6.x**

```
service nagios start
service nagios stop
service nagios restart
service nagios status
```

**CentOS 7.x**

```
systemctl start nagios.service
systemctl stop nagios.service
systemctl restart nagios.service
systemctl status nagios.service
```



## Centos7.x-nagios-script 

```shell
#!/bin/bash

#CentOS 7.x

#Security-Enhanced Linux
sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
setenforce 0

#Prerequisites
yum install -y gcc glibc glibc-common wget unzip httpd php gd gd-devel perl postfix

#Downloading the Source
cd /tmp
wget -O nagioscore.tar.gz https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.3.tar.gz
tar xzf nagioscore.tar.gz

#Compile
cd /tmp/nagioscore-nagios-4.4.3/
./configure
make all

#Create User And Group
make install-groups-users
usermod -a -G nagios apache

#Install Binaries
make install

#Install Service / Daemon
make install-daemoninit
systemctl enable httpd.service

#Install Command Mode
make install-commandmode

#Install Configuration Files
make install-config

#Install Apache Config Files
make install-webconf

#Configure Firewall
firewall-cmd --zone=public --add-port=80/tcp --permanent
firewall-cmd --reload

#Create nagiosadmin User Account
htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin

#Start Apache Web Server
systemctl start httpd.service

#Start Service / Daemon
systemctl start nagios.service


#Installing The Nagios Plugins

#Prerequisites
yum install -y gcc glibc glibc-common make gettext automake autoconf wget openssl-devel net-snmp net-snmp-utils epel-release
yum install -y perl-Net-SNMP

#Downloading The Source
cd /tmp
wget --no-check-certificate -O nagios-plugins.tar.gz https://github.com/nagios-plugins/nagios-plugins/archive/release-2.2.1.tar.gz
tar zxf nagios-plugins.tar.gz

#Compile + Install
cd /tmp/nagios-plugins-release-2.2.1/
./tools/setup
./configure
make
make install

#Service / Daemon Commands
systemctl restart nagios.service
systemctl status nagios.service
systemctl restart httpd.service
systemctl status httpd.service

```

