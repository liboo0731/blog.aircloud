---
title: 【python】通过检测tcp端口来判断服务是否正常
date: 2019-07-04 18:47:03
tags:
- python
---

### 通过检测tcp端口来判断服务是否正常

1. 使用socket模块来连接尝试来确定端口是否开放
2. 如果输入域名，先获取并返回与之对应的IP
3. 定义常用的端口与服务，以便于一次性检测
4. 实现本机常用端口的统一检测
5. 实现远程主机的常用端口统一检测
6. 开放自定义主机与端口的检测方法

```python
#!/usr/bin/env python

import socket
import sys

red='\033[31m'
green='\033[32m'
end='\033[0m'

Ports = [22, 8080, 8443, 8088, 3306, 6379, 61613, 61616, 22122, 23000]
Services = ['SSH', 'HTTP', 'HTTPS', 'Office', 'MySQL', 'Redis', 'ActiveMQ', 'ActiveMQ', 'Tracker', 'Storage']

def get_host_by_name(domain):
    try:
        return socket.gethostbyname(domain)
    except socket.error, e:
        print red + 'error: ' + end + '%s - %s' %(domain, e)
        return 0 

def tcp_port_check(ip, port, timeout=1.0):
    s = socket.socket()
    s.settimeout(timeout)
    s.connect((ip, port))
    s.close()

def ifs_tcp_port_check(domain):
    ip = get_host_by_name(domain)
    if ip:
        print('*** Host: ' + ip + ' ***')
        for port,service in zip(Ports,Services):
            try:
                tcp_port_check(ip, port)
                print(green + '[pass] ' + end + service + '(' + str(port) + ')' + ' is up')
            except:
                print(red + '[failed] ' + end  + service + '(' + str(port) + ')' + ' is down')

def sg_tcp_port_check(domain,port):
    ip = get_host_by_name(domain)
    if ip:
        print('*** Host: ' + ip + ' ***')
        try:
            tcp_port_check(ip, int(port))
            print(green + '[pass] ' + end + 'Port ' + port + ' is open')
        except:
            print(red + '[failed] ' + end  + 'Port ' + port + ' is down')

def useage():
    print(' -l     Check all local services')
    print(' -r     Check all remote services')
    print(' -s     Check if the port is open')

def main():
    if len(sys.argv) == 2 and sys.argv[1] == '-l':
        ifs_tcp_port_check('127.0.0.1')
    elif len(sys.argv) == 3 and sys.argv[1] == '-r':
        ifs_tcp_port_check(sys.argv[2])
    elif len(sys.argv) == 4 and sys.argv[1] == '-s':
        sg_tcp_port_check(sys.argv[2], sys.argv[3])
    else:
        useage()

if __name__=='__main__':
    main()

```

