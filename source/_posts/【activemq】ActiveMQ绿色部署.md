---
title: 【activemq】ActiveMQ绿色部署
date: 2019-04-29 08:59:42
tags:
- activemq
---

### ActiveMQ下载地址

[Download](http://activemq.apache.org/download)

### ActiveMQ官方安装文档

[Getting Started](http://activemq.apache.org/getting-started)

### Linux安装jre

[How do I download and install 32-bit Java for Linux?](https://www.java.com/en/download/help/linux_install.xml)

### ActiveMQ绿色安装

1. 下载activemq和jre，并解压到同一目下
2. 配置activemq的Java环境

```shell
CURRENTPATH=$(dirname $(readlink -f $0))
sed -i '/^JAVACMD=/cJAVACMD=\"'$CURRENTPATH'/jre/bin/java\"' $CURRENTPATH/activemq/bin/env
sed -i 's#<transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>#<transportConnector name="stomp" uri="stomp://0.0.0.0:61613?transport.hbGracePeriodMultiplier=1.5"/>#' $CURRENTPATH/activemq/conf/activemq.xml
```

3. 编写systemd服务

```shell
# 服务管理使用伪用户
# useradd -r -s /bin/false activemq

[Unit]
Description=ActiveMQ

[Service]
User=activemq
Type=forking
ExecStart=/home/user/activemq/bin/activemq start
ExecStop=/home/user/activemq/bin/activemq stop
PIDFile=/home/user/activemq/data/activemq.pid

[Install]
WantedBy=multi-user.target

```

4. 开放防火墙端口

```shell
sudo firewall-cmd --permanent --add-port=61613/tcp
sudo firewall-cmd --permanent --add-port=61616/tcp
sudo firewall-cmd --reload
```

5. 设置伪用户权限

```shell
if ! id activemq &>/dev/null; then
	sudo useradd -r -s /bin/false activemq
	UPPERPATH=$CURRENTPATH
	while true
	do
		if [ "$HOME" == "$UPPERPATH" ]; then
			break
		fi
		UPPERPATH=$(dirname $UPPERPATH)
		setfacl -m u:activemq:rwx $UPPERPATH
		setfacl -d -m u:activemq:rwx $UPPERPATH
	done
	setfacl -R -m u:activemq:rwx $CURRENTPATH
	setfacl -dR -m u:activemq:rwx $CURRENTPATH
fi
```

6. 设置开机自启并启动服务

```
sudo systemctl enable activemq
sudo systemctl daemon-reload
sudo systemctl restart activemq
```

