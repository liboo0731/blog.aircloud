---
title: 【centos】CentOS免密登录
date: 2018-06-12 20:48:51
tags:
- centos
---

### CentOS免密登录

```shell
#!/bin/bash

host=$1

ssh-keygen -t rsa -P ''
ssh-copy-id -i ~/.ssh/id_rsa.pub ${host}
#scp /root/.ssh/id_rsa.pub root@${host}:/root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```

