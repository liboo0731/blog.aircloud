---
title: 【nginx】nginx配置单台主机Web服务挂掉后自动进入维护页面
date: 2019-06-18 09:15:03
tags:
- nginx
---

线上业务如果挂掉后没有任何补救措施，直接无法访问会影响产品的好感度，针对单台业务环境增加临时维护页面，当业务中断后自动切换到维护页面，再及时处理故障，并使业务恢复正常。

### 安装Apache

用于模拟实际Web服务，修改监听端口为8080，创建一个简单的测试页面。

```
yum install epel-release -y
yum install httpd -y
```

### 安装Nginx

用作反向代理和负载均衡，并作为维护页面Web Server。

```
yum install nginx -y
```

### Nginx 配置

正常情况下，通过访问80端口会被转发到8080的业务界面，此时不会和维护页面有任何数据交换；当业务挂掉后会转发到8081的维护页面，业务恢复正常又会自动切换回来。

```
[root@localhost nginx]# egrep -v "^$|^#" nginx.conf
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        location / {
            proxy_pass http://websrv; 
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
    server {
        listen       8081 default_server;
        listen       [::]:8081 default_server;
        server_name  _;
        root         /usr/share/nginx/html;
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
        location / {
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
    upstream websrv {
        server 127.0.0.1:8080;
        server 127.0.0.1:8081 backup;
    }
}
```

