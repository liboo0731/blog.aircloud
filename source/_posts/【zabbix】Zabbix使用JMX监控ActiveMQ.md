---
title: 【zabbix】Zabbix使用JMX监控ActiveMQ
date: 2019-05-30 16:41:05
tags:
- zabbix
---

## Zabbix 使用 JMX 监控 ActiveMQ

### Zabbix Server 安装配置 Zabbix Java Gateway

```
安装 zabbix-java-gateway
# yum install zabbix-java-gateway -y

启动并设置开机自启
# systemctl start zabbix-java-gateway
# systemctl enable zabbix-java-gateway

修改zabbix_server.conf
# vi zabbix_server.conf

JavaGateway=172.16.2.224	//IP address (or hostname) of Zabbix Java gateway.
JavaGatewayPort=10052		//Port that Zabbix Java gateway listens on.
StartJavaPollers=5			//Number of pre-forked instances of Java pollers.

重启zabbix server
# systemctl restart zabbix-server
```



### ActiveMQ 配置

```
修改/etc/hosts 为对外地址，否则jmx远程无法使用
# cat /etc/hosts
172.16.2.229   localhost localhost.localdomain localhost4 localhost4.localdomain4
#127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

开启jmx
# vi conf/activemq.xml

<broker useJmx="true" xmlns="http://activemq.apache.org/schema/core" brokerName="localhost" dataDirectory="${activemq.data}">

        <managementContext>
            <managementContext createConnector="true" connectorPort="11099"/>
        </managementContext>
        ...

配置启动参数
# vi bin/env 

ACTIVEMQ_SUNJMX_START="$ACTIVEMQ_SUNJMX_START -Dcom.sun.management.jmxremote.port=11099"
ACTIVEMQ_SUNJMX_START="$ACTIVEMQ_SUNJMX_START -Dcom.sun.management.jmxremote.password.file=${ACTIVEMQ_CONF}/jmx.password"
ACTIVEMQ_SUNJMX_START="$ACTIVEMQ_SUNJMX_START -Dcom.sun.management.jmxremote.access.file=${ACTIVEMQ_CONF}/jmx.access"
ACTIVEMQ_SUNJMX_START="$ACTIVEMQ_SUNJMX_START -Dcom.sun.management.jmxremote.ssl=false"
ACTIVEMQ_SUNJMX_START="$ACTIVEMQ_SUNJMX_START -Dcom.sun.management.jmxremote=true"
ACTIVEMQ_SUNJMX_START="$ACTIVEMQ_SUNJMX_START -Dcom.sun.management.jmxremote.authenticate=false"	//不需要认证
ACTIVEMQ_SUNJMX_START="$ACTIVEMQ_SUNJMX_START -Djava.rmi.server.hostname=172.16.2.229"

设置权限
chmod 400 conf/jmx.*

重启服务
# systemctl restart activemq
```



### Web管理界面添加主机，使用 JMX 接口

```
Configuration -> Host --> Create host

Host name：host-172-16-2-229
Visible name：linux_host_233
Groups：Linux主机测试
# remove Agent interfaces
JMX interfaces：172.16.2.229

# Templates
Template App Generic Java JMX
```

