---
title: 【zabbix】Zabbix自定义邮件告警
date: 2019-05-29 15:53:16
tags:
- zabbix
---

## Zabbix自定义邮件告警

### 1. 安装配置mailx

```
# yum install -y mailx

如果存在Postfix，关闭并禁用
# systemctl stop postfix
# systemctl disable postfix

首先登陆163设置启用SMTP，追加如下配置
# vi /etc/mail.rc

set from=striveliboo@163.com
set smtp=smtp.163.com
set smtp-auth-user=striveliboo@163.com
set smtp-auth-password=<password>
set smtp-auth=login

```

### 2. 编写邮件发送脚本，并放置到相应位置

```
[root@host-172-16-2-224 alertscripts]# cd /usr/lib/zabbix/alertscripts/
[root@host-172-16-2-224 alertscripts]# cat mail.sh 
(1)邮件内容可能会作为附件传递，内容展示会乱码
#!/bin/bash

SENT_TO=$1
SENT_SUBJECT=$2
SENT_CONTENT=$3

echo "$SENT_CONTENT" |mailx -s "$SENT_SUBJECT" $SENT_TO

（2）解决第一种方法的问题，需要安装dos2unix
#!/bin/bash

SENT_TO=$1
SENT_SUBJECT=$2
SENT_CONTENT="/tmp/alert_$$.tmp"
echo "$3">$SENT_CONTENT

dos2unix $SENT_CONTENT

mailx -s "$SENT_SUBJECT" $SENT_TO<$SENT_CONTENT

[root@host-172-16-2-224 alertscripts]# chmod o+x mail.sh
[root@host-172-16-2-224 alertscripts]# sh mail.sh striveliboo@163.com subject content

```

### 3. 登陆界面，添加配置Media Types

```
Administration --> Media types --> create media type
# Name：
邮件告警发送
# Type：
Scripts
# Script name
mail.sh
# Script parameters
{ALERT.SENDTO}
{ALERT.SUBJECT}
{ALERT.MESSAGE}

```

### 4. 配置Actions

```
Configuration --> Actions --> create action
# Operations
故障{TRIGGER.STATUS}，服务器：{HOSTNAME1}发生：”{TRIGGER.NAME}“故障！

告警主机IP地址：{HOST.CONN}
告警主机：{HOSTNAME1}
告警时间：{EVENT.DATE} {EVENT.TIME}
告警等级：{TRIGGER.SEVERITY}
告警信息：{TRIGGER.NAME}
告警项目：{TRIGGER.KEY1}
问题详情：{ITEM.NAME}：{ITEM.VALUE}
当前状态：{TRIGGER.STATUS}：{ITEM.VALUE1}
事件ID：{EVENT.ID}

# Recovery operations
恢复{TRIGGER.STATUS}，服务器：{HOSTNAME1}: ”{TRIGGER.NAME}“已恢复！

告警主机IP地址：{HOST.CONN}
告警主机：{HOSTNAME1}
告警时间：{EVENT.DATE} {EVENT.TIME}
告警等级：{TRIGGER.SEVERITY}
告警信息：{TRIGGER.NAME}
告警项目：{TRIGGER.KEY1}
问题详情：{ITEM.NAME}：{ITEM.VALUE}
当前状态：{TRIGGER.STATUS}：{ITEM.VALUE1}
事件ID：{EVENT.ID}

```

