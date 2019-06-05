---
title: 【redis】Redis绿色部署
date: 2019-03-22 09:01:52
tags:
- redis
---

## Linux 下安装

**下载地址：**<http://redis.io/download>，下载最新稳定版本。

```
$ wget http://download.redis.io/releases/redis-5.0.5.tar.gz
$ tar xzf redis-5.0.5.tar.gz
$ cd redis-5.0.5
$ make
```

make完后 redis-5.0.5目录下会出现编译后的redis服务程序redis-server，还有用于测试的客户端程序redis-cli，两个程序位于安装目录 src 目录下：

下面启动redis服务。

```
$ cd src
$ ./redis-server
```

注意这种方式启动redis 使用的是默认配置。也可以通过启动参数告诉redis使用指定配置文件使用下面命令启动。

```
$ cd src
$ ./redis-server ../redis.conf
```

**redis.conf** 是一个默认的配置文件。我们可以根据需要使用自己的配置文件。

启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了。 比如：

```
$ cd src
$ ./redis-cli
redis> set foo bar
OK
redis> get foo
"bar"
```



## redis绿色安装

### 安装到指定目录

```
# 安装到 /usr/local/redis

installpath=/usr/local/redis
mkdir -p $installpath
cd redis-5.0.5
make MALLOC=libc
make PREFIX=$installpath install
\cp -f redis.conf $installpath
```

### 功能测试（耗时较长，非必须执行）

```
在安装成功之后，可以运行测试，确认Redis的功能是否正常
[root@localhost ~]# make test

出现报错：
hadoop@stormspark:~/workspace/redis2.6.13/src$ make test
You need tcl 8.5 or newer in order to run the Redis test
make: *** [test] Error 1

解决方式
yum install -y tcl

重新执行测试
[root@localhost ~]# make test

......

\o/ All tests passed without errors!

Cleanup: may take some time... OK

```

### 修改配置文件

```
installpath=/usr/local/redis
sed -i '/^bind/c#bind 127.0.0.1' "$installpath/redis.conf"
sed -i '/^protected-mode/cprotected-mode no' "$installpath/redis.conf"
sed -i '/^logfile/clogfile "'$installpath'/redis.log"' "$installpath/redis.conf"
sed -i '/^pidfile/cpidfile "'$installpath'/redis.pid"' "$installpath/redis.conf"
```

### 编写Redis启停脚本

```
[root@localhost ~]# vi /usr/local/redis/redis.sh

#!/bin/sh
#Configurations injected by install_server below....

EXEC=/usr/local/redis/bin/redis-server
CLIEXEC=/usr/local/redis/bin/redis-cli
PIDFILE=/usr/local/redis/redis.pid
CONF="/usr/local/redis/redis.conf"
REDISPORT="6379"
###############
# SysV Init Information
# chkconfig: - 58 74
# description: redis_6379 is the redis daemon.
### BEGIN INIT INFO
# Provides: redis_6379
# Required-Start: $network $local_fs $remote_fs
# Required-Stop: $network $local_fs $remote_fs
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Should-Start: $syslog $named
# Should-Stop: $syslog $named
# Short-Description: start and stop redis_6379
# Description: Redis daemon
### END INIT INFO


case "$1" in
    start)
        if [ -f $PIDFILE ]
        then
            echo "$PIDFILE exists, process is already running or crashed"
        else
            echo "Starting Redis server..."
            $EXEC $CONF
        fi
        ;;
    stop)
        if [ ! -f $PIDFILE ]
        then
            echo "$PIDFILE does not exist, process is not running"
        else
            PID=$(cat $PIDFILE)
            echo "Stopping ..."
            $CLIEXEC -p $REDISPORT shutdown
            while [ -x /proc/${PID} ]
            do
                echo "Waiting for Redis to shutdown ..."
                sleep 1
            done
            echo "Redis stopped"
        fi
        ;;
    status)
        PID=$(cat $PIDFILE)
        if [ ! -x /proc/${PID} ]
        then
            echo 'Redis is not running'
        else
            echo "Redis is running ($PID)"
        fi
        ;;
    restart)
        $0 stop
        $0 start
        ;;
    *)
        echo "Please use start, stop, restart or status as first argument"
        ;;
esac

[root@localhost ~]# chmod u+x /usr/local/redis/redis.sh
```

### 使用redis 用户以systemd 服务启动

```
编写systemd服务
[root@localhost ~]# vi /usr/local/redis/redis.service

-----------------------------------------
[Unit]
Description=redis server

[Service]
User=redis
WorkingDirectory=/usr/local/redis
ExecStart=/usr/bin/sh redis.sh start
ExecStop=/usr/bin/sh redis.sh stop

[Install]
WantedBy=multi-user.target
-----------------------------------------
[Unit]
Description=redis server

[Service]
User=redis
WorkingDirectory=/usr/local/redis
ExecStart=/usr/local/redis/bin/redis-server redis.conf

[Install]
WantedBy=multi-user.target
-----------------------------------------

放置服务文件
[root@localhost ~]# \cp -f /usr/local/redis/redis.service /usr/lib/systemd/system/redis.service

创建redis用户
[root@localhost ~]# useradd -r -s /bin/false redis

修改文件所属
[root@localhost ~]# chown -R redis.redis /usr/local/redis

启动并设置开机自启
[root@localhost ~]# systemctl start redis
[root@localhost ~]# systemctl enable redis
```

### 防火墙配置

```
sudo firewall-cmd --permanent --add-port=6379/tcp
sudo firewall-cmd --reload
```