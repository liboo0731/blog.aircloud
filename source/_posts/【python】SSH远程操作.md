---
title: 【python】SSH远程操作
date: 2018-03-02 09:58:55
tags:
- python
- ssh
---

paramiko模块提供了ssh及sft进行远程登录服务器执行命令和上传下载文件的功能。这是一个第三方的软件包，使用之前需要安装。

### 1. 基于用户名和密码的 sshclient 方式登录

```python
# 建立一个sshclient对象
ssh = paramiko.SSHClient()
# 允许将信任的主机自动加入到host_allow 列表，此方法必须放在connect方法的前面
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
# 调用connect方法连接服务器
ssh.connect(hostname='192.168.2.129', port=22, username='super', password='super')
# 执行命令
stdin, stdout, stderr = ssh.exec_command('df -hl')
# 结果放到stdout中，如果有错误将放到stderr中
print(stdout.read().decode())
# 关闭连接
ssh.close()
```

### 2. 基于用户名和密码的 transport 方式登录

方法1是传统的连接服务器、执行命令、关闭的一个操作，有时候需要登录上服务器执行多个操作，比如执行命令、上传/下载文件，方法1则无法实现，可以通过如下方式来操作

```python
# 实例化一个transport对象
trans = paramiko.Transport(('192.168.2.129', 22))
# 建立连接
trans.connect(username='super', password='super')
# 将sshclient的对象的transport指定为以上的trans
ssh = paramiko.SSHClient()
ssh._transport = trans
# 执行命令，和传统方法一样
stdin, stdout, stderr = ssh.exec_command('df -hl')
print(stdout.read().decode())
# 关闭连接
trans.close()
```

### 3 基于公钥密钥的 SSHClient 方式登录

```python
# 指定本地的RSA私钥文件,如果建立密钥对时设置的有密码，password为设定的密码，如无不用指定password参数
pkey = paramiko.RSAKey.from_private_key_file('/home/super/.ssh/id_rsa', password='12345')
# 建立连接
ssh = paramiko.SSHClient()
ssh.connect(hostname='192.168.2.129',
            port=22,
            username='super',
            pkey=pkey)

# 执行命令
stdin, stdout, stderr = ssh.exec_command('df -hl')
# 结果放到stdout中，如果有错误将放到stderr中
print(stdout.read().decode())
# 关闭连接
ssh.close()
```

以上需要确保被访问的服务器对应用户.ssh目录下有authorized_keys文件，也就是将服务器上生成的公钥文件保存为authorized_keys。并将私钥文件作为paramiko的登陆密钥

### 4. 基于密钥的 Transport 方式登录

指定本地的RSA私钥文件,如果建立密钥对时设置的有密码，password为设定的密码，如无不用指定password参数

```python
pkey = paramiko.RSAKey.from_private_key_file('/home/super/.ssh/id_rsa', password='12345')
# 建立连接
trans = paramiko.Transport(('192.168.2.129', 22))
trans.connect(username='super', pkey=pkey)
# 将sshclient的对象的transport指定为以上的trans
ssh = paramiko.SSHClient()
ssh._transport = trans
# 执行命令，和传统方法一样
stdin, stdout, stderr = ssh.exec_command('df -hl')
print(stdout.read().decode())
# 关闭连接
trans.close()
##### 传文件 SFTP ###########
# 实例化一个trans对象# 实例化一个transport对象
trans = paramiko.Transport(('192.168.2.129', 22))
# 建立连接
trans.connect(username='super', password='super')
# 实例化一个 sftp对象,指定连接的通道
sftp = paramiko.SFTPClient.from_transport(trans)
# 发送文件
sftp.put(localpath='/tmp/11.txt', remotepath='/tmp/22.txt')
# 下载文件
# sftp.get(remotepath, localpath)
trans.close()
```

### 5. 实现输入命令立马返回结果的功能

以上操作都是基本的连接，如果我们想实现一个类似xshell工具的功能，登录以后可以输入命令回车后就返回结果：

```python
import paramiko
import os
import select
import sys

# 建立一个socket
trans = paramiko.Transport(('192.168.2.129', 22))
# 启动一个客户端
trans.start_client()

# 如果使用rsa密钥登录的话

'''
default_key_file = os.path.join(os.environ['HOME'], '.ssh', 'id_rsa')
prikey = paramiko.RSAKey.from_private_key_file(default_key_file)
trans.auth_publickey(username='super', key=prikey)
'''

# 如果使用用户名和密码登录
trans.auth_password(username='super', password='super')
# 打开一个通道
channel = trans.open_session()
# 获取终端
channel.get_pty()
# 激活终端，这样就可以登录到终端了，就和我们用类似于xshell登录系统一样
channel.invoke_shell()
# 下面就可以执行你所有的操作，用select实现
# 对输入终端sys.stdin和 通道进行监控,
# 当用户在终端输入命令后，将命令交给channel通道，这个时候sys.stdin就发生变化，select就可以感知
# channel的发送命令、获取结果过程其实就是一个socket的发送和接受信息的过程

while True:
    readlist, writelist, errlist = select.select([channel, sys.stdin,], [], [])
    # 如果是用户输入命令了,sys.stdin发生变化
    if sys.stdin in readlist:
        # 获取输入的内容
        input_cmd = sys.stdin.read(1)
        # 将命令发送给服务器
        channel.sendall(input_cmd)

    # 服务器返回了结果,channel通道接受到结果,发生变化 select感知到
    if channel in readlist:
        # 获取结果
        result = channel.recv(1024)
        # 断开连接后退出
        if len(result) == 0:
            print("\r\n**** EOF **** \r\n")
            break
        # 输出到屏幕
        sys.stdout.write(result.decode())
        sys.stdout.flush()

# 关闭通道
channel.close()
# 关闭链接
trans.close()
```

### 6. 支持tab自动补全

```python
import paramiko
import os
import select
import sys
import tty
import termios

'''
实现一个xshell登录系统的效果，登录到系统就不断输入命令同时返回结果
支持自动补全，直接调用服务器终端
'''

# 建立一个socket
trans = paramiko.Transport(('192.168.2.129', 22))
# 启动一个客户端
trans.start_client()

# 如果使用rsa密钥登录的话
'''
default_key_file = os.path.join(os.environ['HOME'], '.ssh', 'id_rsa')
prikey = paramiko.RSAKey.from_private_key_file(default_key_file)
trans.auth_publickey(username='super', key=prikey)
'''

# 如果使用用户名和密码登录
trans.auth_password(username='super', password='super')
# 打开一个通道
channel = trans.open_session()
# 获取终端
channel.get_pty()
# 激活终端，这样就可以登录到终端了，就和我们用类似于xshell登录系统一样
channel.invoke_shell()

# 获取原操作终端属性
oldtty = termios.tcgetattr(sys.stdin)
try:
    # 将现在的操作终端属性设置为服务器上的原生终端属性,可以支持tab了
    tty.setraw(sys.stdin)
    channel.settimeout(0)
    while True:
        readlist, writelist, errlist = select.select([channel, sys.stdin,], [], [])
        # 如果是用户输入命令了,sys.stdin发生变化
        if sys.stdin in readlist:
            # 获取输入的内容，输入一个字符发送1个字符
            input_cmd = sys.stdin.read(1)
            # 将命令发送给服务器
            channel.sendall(input_cmd)
        # 服务器返回了结果,channel通道接受到结果,发生变化 select感知到
        if channel in readlist:
            # 获取结果
            result = channel.recv(1024)
            # 断开连接后退出
            if len(result) == 0:
                print("\r\n**** EOF **** \r\n")
                break
            # 输出到屏幕
            sys.stdout.write(result.decode())
            sys.stdout.flush()
finally:
    # 执行完后将现在的终端属性恢复为原操作终端属性
    termios.tcsetattr(sys.stdin, termios.TCSADRAIN, oldtty)

# 关闭通道
channel.close()
# 关闭链接
trans.close()
```

### 7. SSH批量登陆执行命令（1）

```python
#-*- coding: utf-8 -*-
#!/usr/bin/python
import paramiko
import threading

def ssh2(ip,username,passwd,cmd):
    try:
        ssh = paramiko.SSHClient()
        ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
        ssh.connect(ip,22,username,passwd,timeout=5)
        for m in cmd:
            stdin, stdout, stderr = ssh.exec_command(m)
#           stdin.write("Y")   #简单交互，输入 ‘Y’
            out = stdout.readlines()
            #屏幕输出
            for o in out:
                print o,
        print '%s\tOK\n'%(ip)
        ssh.close()
    except :
        print '%s\tError\n'%(ip)

if __name__=='__main__':
    cmd = ['cal','echo hello!']#你要执行的命令列表
    username = ""  #用户名
    passwd = ""    #密码
    threads = []   #多线程
    print "Begin......"
    for i in range(1,254):
        ip = '192.168.1.'+str(i)
        a=threading.Thread(target=ssh2,args=(ip,username,passwd,cmd))
        a.start()
```

### 8. SSH批量登陆执行命令（2）

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import pexpect

def ssh_cmd(ip, passwd, cmd):
    ret = -1
    ssh = pexpect.spawn('ssh root@%s "%s"' % (ip, cmd))
    try:
        i = ssh.expect(['password:', 'continue connecting (yes/no)?'], timeout=5)
        if i == 0 :
            ssh.sendline(passwd)
        elif i == 1:
            ssh.sendline('yes\n')
            ssh.expect('password: ')
            ssh.sendline(passwd)
        ssh.sendline(cmd)
        r = ssh.read()
        print r
        ret = 0
    except pexpect.EOF:
        print "EOF"
        ssh.close()
        ret = -1
    except pexpect.TIMEOUT:
        print "TIMEOUT"
        ssh.close()
        ret = -2
    return ret
```

