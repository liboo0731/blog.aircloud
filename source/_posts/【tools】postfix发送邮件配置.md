---
title: 【tools】postfix发送邮件配置
date: 2019-04-28 20:06:12
tags: 
    - tools
    - postfix
---

### 简单配置及测试脚本

```shell
#!/bin/sh

yum install cyrus-sasl-* -y

cat >/etc/postfix/sasl_passwd <<EOF
[smtp.abc.cn]:25   abc@163.cn:password
EOF
postmap /etc/postfix/sasl_passwd
rm -f /etc/postfix/sasl_passwd 

cat >/etc/postfix/sender_canonical <<EOF
root abc@163.cn
EOF
postmap /etc/postfix/sender_canonical
rm -f postmap /etc/postfix/sender_canonical

cat >>/etc/postfix/main.cf <<EOF
#指定默认的邮件发送服务器
relayhost = [smtp.abc.cn]:25
#激活sasl认证
smtp_sasl_auth_enable = yes
#指定sasl密码配置文件
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
#非匿名登录
smtp_sasl_security_options = noanonymous
#指定认证类型 （提示：需要yum安装cyrus-sasl-*组件，否则发邮件时会报错：no mechanism available）
smtp_sasl_type = cyrus
#linux用户与发件人的对应关系配置文件
sender_canonical_maps = hash:/etc/postfix/sender_canonical
EOF


echo "test mail" | sendmail abc@163.cn
#yum install mailx -y
echo "邮件正文" | mail -s 邮件主题 test@126.com 

```

