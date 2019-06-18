---
title: 【zabbix】Zabbix自定义监控项
date: 2019-05-29 17:25:31
tags:
- zabbix
---

## 自定义监控项（Items）

监控项是从主机收集的数据信息。

配置主机后，你需要添加一些监控项以开始获取实际数据。

一个监控项是一个独立的指标。快速添加多个监控项的一种方法是将一个预定义的模板附加到主机。然而，为了优化系统性能，您可能需要对模板进行微调，使只有真正需要的监控项被频繁的监控到。

在单个监控项中，你可以指定从主机收集哪些数据。

为此，你可以使用监控项key。 从而，具有名称为system.cpu.load的监控项将收集处理器负载的数据，而名为net.if.in的监控项将收集传入的流量信息。

要用key指定更多的参数，请在key后添加方括号。 例如，system.cpu.load[avg5]将返回最近5分钟的处理器负载平均值，而net.if.in[eth0]将显示接口eth0中的流量。

```
# /etc/zabbix/zabbbix_agent.conf

UserParameter=key[*],[command|sh]
# <key[参数]>，<命令或者脚本>
# [*]：固定格式，表示server端是否传过来参数，在命令或者脚本中用$1,23...引用，shell脚本中的引用$$1,2,3..引用，
# 如果server端不传参数，[*]可以不写

Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

### 方法1：直接向 zabbix_agent 配置文件中添加

```
UnsafeUserParameters=0  #默认情况下，不允许用户在用户自定义参数中使用某些特殊符号。

定义了一个名为"os.mem.used"的key，然后这个key通过后面指定的命令来获取需要的值：
# UserParameter=os.mem.used,free -m | awk '/^Mem/ {print $3}'  #新增一个key叫作os.mem.used用于查看系统已用内存

UserParameter还可以通过脚本来取得多个key的值：
# UserParameter=mysql.status[*],/usr/local/zabbix/scripts/check_mysql.sh $1
# UserParameter=check_port[*],/usr/local/zabbix/scripts/check_port.sh -H 127.0.0.1 -p $1
[*]代表key接受任意个参数，这些参数就是在web端要明确写出来的参数，也是后面命令或脚本中接受的参数，用$1,$2来表示第几个参数。如果command中的命令本身带有$1等信息（如awk命令），需要写为$$1
```



### 方法2：编写独立的配置文件，并在 zabbix_agent 配置中 Include

```
打开zabbix_agent.conf 中的选项
Include=/etc/zabbix/zabbix_agentd.d/*.conf

为了更好区分和维护，将自定义的监控项写到自己的配置文件中
cat /etc/zabbix/zabbix_agentd.d/userparameter_mysql.conf

UserParameter=mysql.size[*],bash -c 'echo "select sum($(case "$3" in both|"") echo "data_length+index_length";; data|index) echo "$3_length";; free) echo "data_free";; esac)) from information_schema.tables$([[ "$1" = "all" || ! "$1" ]] || echo " where table_schema=\"$1\"")$([[ "$2" = "all" || ! "$2" ]] || echo "and table_name=\"$2\"");" | HOME=/var/lib/zabbix mysql -N'

UserParameter=mysql.ping,HOME=/var/lib/zabbix mysqladmin ping | grep -c alive
UserParameter=mysql.version,mysql -V

```

###  

### 用户自定义参数

#### 概述

用户定义参数可以用来帮助用户实现通过Zabbix agent执行非Zabbix原生的 agent check。

你可以编写一个命令来检索所需的数据，并将其包含在用户自定义参数[agent 配置文件](https://www.zabbix.com/documentation/4.0/zh/manual/appendix/config/zabbix_agentd)中 ('UserParameter' 参数配置)。 

一条用户自定义参数配置应当使用以下语法：

```
UserParameter=<key>,<command>
```

如你所见，一条用户自定义参数除了命令部分，还包括一个key。这个key将在配置监控项时使用。输入你选择的易于引用的key（key在一台主机中必须是唯一的）。重启agent。

接下来，在配置[配置监控项](https://www.zabbix.com/documentation/4.0/zh/manual/config/items/item)时，输入要执行的来自用户自定义参数中的，引用命令的key。

用户自定义参数是由Zabbix agent来执行命令的。在监控项预处理步骤前，最多可以返回512KB的数据。但是，请注意，最终可以存储在数据库中的文本值，在MySQL上的限制为64KB（其他数据库的信息请参阅[数据表](https://www.zabbix.com/documentation/4.0/zh/manual/config/notifications/action/operation/remote_command#overview)）。

**/bin/sh** 在UNIX操作系统中，作为命令行解释器使用。用户自定义参数参照agent check超时；如果超时时间到了，那么执行用户自定义参数的子进程将会被中止。

参见：

- [分布教程](https://www.zabbix.com/documentation/4.0/zh/manual/config/items/userparameters/extending_agent) 配置用户自定参数 parameters
- [命令执行](https://www.zabbix.com/documentation/4.0/zh/manual/appendix/command_execution)

##### 用户自定义参数用例

一个简单的命令：

```
UserParameter=ping,echo 1
```

agent 将始终为使用“ping”为key的监控项返回“1”。

一个复杂一些的例子：

```
UserParameter=mysql.ping,mysqladmin -uroot ping | grep -c alive
```

如果Mysql服务器是活动状态，agent将返回“1”，否则会返回“0”。

#### 灵活的用户自定义参数

灵活的用户自定义参数可以从key中接受参数。这是一种使用一个用户自定义参数创建多个监控项的方式。

灵活的用户自定义参数有以下语法：

```
UserParameter=key[*],command
```

| Parameter参数 | Description描述                                              |
| ------------- | ------------------------------------------------------------ |
| **Key**       | 唯一的监控项key。 [*] 用于定义该key接受括号内的参数。 参数需在配置监控项时给出 |
| **Command**   | 命令在执行时，引用key中指定的值 *只对灵活的用户参数有效*: 你可以在命令中使用位置引用$1 … $9来引用监控项Key中的相应参数。 Zabbix解析监控项Key的[]中包含的参数，并相应地替换$1，…，$9。 $0会替换为完整的原始命令（在对$0，…，$9执行替换之前的命令）运行。 不管位置参数（$0,…,$9)是用双引号( “ )还是单引号( ' )括起来，都会解析位置引用。 要使用位置引用解析，请指定双美元符号（$） - 例如， `awk '{print $$2}' `。 在这种情况下，执行命令时， `$$2` 实际上会变成 `$2` 。 |

仅对灵活的用户自定义参数进行搜索具有 `$ `符号的位置引用并由Zabbix agent解析替换。 对于简单的用户自定义参数，跳过此类参考处理，因此不需要任何$符号引用。

默认情况下，不允许用户在用户自定义参数中使用某些特殊符号。详情请移步 [UnsafeUserParameters](https://www.zabbix.com/documentation/4.0/zh/manual/appendix/config/zabbix_agentd) ，查询相关的符号列表

##### 示例一

先来一个简单的：

```
UserParameter=ping[*],echo $1
```

我们可以定义无数个监控项来监控所有形如ping[something]格式的东西。

- ping[0] - 将总是返回 ‘ 0 ’
- ping[aaa] - 将总是返回 ‘aaa’

##### 示例二

让我们更进一步！

```
UserParameter=mysql.ping[*],mysqladmin -u$1 -p$2 ping | grep -c alive
```

这个用户自定义参数可以用来监控 MySQL 数据库的状态。可以想下面的样式传入用户名和密码：

```
mysql.ping[zabbix,our_password]
```

##### 示例三

一个文件中有多少行匹配正则表达式？

```
UserParameter=wc[*],grep -c "$2" $1
```

这个用户自定义参数能用来计算一个文件中有多少行匹配相应的表达式。就像下面一样：

```
wc[/etc/passwd,root]
wc[/etc/services,zabbix]
```

#### 命令结果

命令的返回值是标准输出和标准错误。

标准错误情况下，不支持文本（字符、日志或是文本类型的信息）的监控项

返回文本的用户自定义参数（字符，日志，文本信息类型）可以返回空格。如果结果不可用，那么这个监控项会变为不支持状态。