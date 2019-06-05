---
title: 【pssh】PSSH批量远程管理主机
date: 2018-10-21 20:30:51
tags:
- pssh
---

### pssh介绍

pssh是python写的可以并发在多台机器上批量执行命令的工具，它的用法可以媲美ansible的一些简单用法，执行起来速度比ansible快它支持文件并行复制，远程命令执行，杀掉远程主机上的进程等等。杀手锏是文件并行复制，当需要在远程主机批量上传下载的时候，最好使用它。

### pssh的使用

在使用pssh之前，必须要保证管理主机和本地主机进行过密钥的认证，或者是在进行批量时，没有做过密钥认证，但是必须保证被管理的多台主机的密码相同。关于如何做密钥认证，这里就不多说了，可以自行百度。

#### 安装

官网下载地址：[PSSH Download](https://code.google.com/archive/p/parallel-ssh/downloads)

```
wget https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/parallel-ssh/pssh-2.3.1.tar.gz
tar -zxvf pssh-2.3.1.tar.gz
cd pssh-2.3.1
python setup.py install
或者
yum install -y pssh
```

使用yum安装后，只能使用pssh，但是编译安装后会包括其他命令也安装了（pscp  prsync  pnuke  pslurp）

- pssh：在远程多台主机上并行运行命令
- pscp :把文件并行复制到多台远程主机上
- prsync:使用rsync协议本地文件同步到远程多台主机上。
- pnuke:在远程多台主机上并行killall某一进程
- pslurp:把文件从远程多台主机上复制到本地主机上

#### pssh常用参数

```
pssh  --help

Usage: pssh [OPTIONS] command [...]

Options:
  --version             show program's version number and exit
  --help                show this help message and exit
  -h HOST_FILE, --hosts=HOST_FILE
                        hosts file (each line "[user@]host[:port]")
  -H HOST_STRING, --host=HOST_STRING
                        additional host entries ("[user@]host[:port]")
  -l USER, --user=USER  username (OPTIONAL)
  -p PAR, --par=PAR     max number of parallel threads (OPTIONAL)
  -o OUTDIR, --outdir=OUTDIR
                        output directory for stdout files (OPTIONAL)
  -e ERRDIR, --errdir=ERRDIR
                        output directory for stderr files (OPTIONAL)
  -t TIMEOUT, --timeout=TIMEOUT
                        timeout (secs) (0 = no timeout) per host (OPTIONAL)
  -O OPTION, --option=OPTION
                        SSH option (OPTIONAL)
  -v, --verbose         turn on warning and diagnostic messages (OPTIONAL)
  -A, --askpass         Ask for a password (OPTIONAL)
  -x ARGS, --extra-args=ARGS
                        Extra command-line arguments, with processing for
                        spaces, quotes, and backslashes
  -X ARG, --extra-arg=ARG
                        Extra command-line argument
  -i, --inline          inline aggregated output and error for each server
  --inline-stdout       inline standard output for each server
  -I, --send-input      read from standard input and send as input to ssh
  -P, --print           print output as we get it

Example: pssh -h hosts.txt -l irb2 -o /tmp/foo uptime
```

具体常用介绍：

- -h   HOST_FILE   后边跟远程主机列表（ip）
- -H   HOST_STRING   后边跟远程主机名或者ip地址
- -l   USER  指定远程主机的用户名
- -p  PAR   指定pssh最大的并行线程数。
- -o  将输出的内容重定向到一个指定的文件中
- -O 指定ssh参数的具体配置
- -e  将执行错误重定向到一个指定的文件中
- -t  设定命令执行超时时间
- -x  传递ssh命令的一些参数
- -i  在远程主机上执行命令完成后显示标准输出和标准错误
- -P  在执行远程命令时，输出执行结果

 用法实例：

```
pssh -P  -i   -h  sz_vpc.txt  "w" 
```

### pscp拷贝文件到远程主机

```
pscp  -h  ip.txt /etc/wenjian.txt   /tmp/
```

### pnuke杀掉某一进程

这个命令类似yu  killall命令

```
pnuke  -h  iplist.txt   httpd
```

上边的意思是在远程主机上批量关闭httpd服务

能通过killall关闭的服务，都可以通过pnuke来批量完成

### pslurp 远程主机拷贝文件到本地主机

```
pslurp -h iplist.txt -L /home/ /hose/wenjian/yuanc.conf open.conf
```

上边是，将所有远程主机/hose/wenjian/yuanc.conf复制到本地主机/home/目录下，并且重新命名为open.conf  -L 来指定本地文件路径

下面这个是拷贝目录

```
 pslurp -h iplist.txt -r -L /home/ /hose/wenjian/ open
```



***PS：建议分发文件，执行命令，批量杀死进程，使用pssh,pscp，pnuke,速度很快的***