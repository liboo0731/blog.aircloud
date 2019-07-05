---
title: 【web】Sinopia 构建Windows 本地NPM 仓库
date: 2019-07-05 09:49:55
tags:
- web
- sinopia
- nssm
---

## Sinopia 构建Windows 本地NPM 仓库

### 安装Node 环境

[Downloads](https://nodejs.org/en/download/)

![1.png](1.png)



### 下载NSSM

[Download](https://www.nssm.cc/download/)

![2.png](2.png)



### 如何使用NSSM

服务安装命令： `nssm install <servicename>`

参考：

[使用NSSM将exe封装为服务](https://www.cnblogs.com/TianFang/p/7912648.html)

![3.png](3.png)



### Installing As a Windows Service

Petrik van der Velde edited this page Oct 31, 2017 · [10 revisions](https://github.com/rlidwka/sinopia/wiki/Installing-As-a-Windows-Service/_history)

Loosely based upon the instructions found [here](http://asysadmin.tumblr.com/post/32941224574/running-nginx-on-windows-as-a-service). I crafted the following and it provided me with a fully working sinopia service installation:

1. Create a directory for sinopia

   - `mkdir c:\sinopia`
   - `cd c:\sinopia`

2. Install sinopia locally (I ran into npm issues with global installs)

   - `npm install sinopia`

3. Create your config.yaml file in this location (`c:\sinopia\config.yaml`)

4. Windows Service Setup

   #### Using NSSM:

   - Download [NSSM](https://www.nssm.cc/download/) and extract

   - Open an administrative command

   - Browse to the nssm extract location

   - Run `nssm install sinopia` At a minimum you must fill in the *Application* tab Path, Startup directory and Arguments fields. Assuming an install with node in the system path and a location of c:\sinopia the below values will work:

     - Path: `node`
     - Startup directory: `c:\sinopia`
     - Arguments: `c:\sinopia\node_modules\sinopia\lib\cli.js -c c:\sinopia\config.yaml`

     You can adjust other service settings under other tabs as desired. When you are done, click *Install service* button

   - Start the service `sc start sinopia`

   #### Using WinSW

   - Download [WinSW](http://repo.jenkins-ci.org/releases/com/sun/winsw/winsw/)
     - Place the executable (e.g. `winsw-1.19-bin.exe`) into this folder (`c:\sinopia`) and rename it to `sinopia-winsw.exe`
   - Create a configuration file in `c:\sinopia`, named `sinopia-winsw.xml` with the following configuration

```
<service>
  <id>sinopia</id>
  <name>sinopia</name>
  <description>sinopia</description>
  <executable>node</executable>
  <arguments>c:\sinopia\node_modules\sinopia\lib\cli.js -c c:\sinopia\config.yaml</arguments>
  <logmode>roll</logmode>
  <workingdirectory>c:\sinopia\</workingdirectory>
</service>
```

* Install your service
    * `cd c:\sinopia`
    * `sinopia-winsw.exe install`
* Start your service
    * `sinopia-winsw.exe start`

Some of the above config is more verbose than I had expected, it appears as though 'workingdirectory' is ignored, but other than that, this works for me and allows my sinopia instance to persist between restarts of the server, and also restart itself should there be any crashes of the sinopia process.



### Sinopia 目录结构

![4.png](4.png)



### Sinopia 配置文件（config.yaml）

```
#
# This is the default config file. It allows all users to do anything,
# so don't use it on production systems.
#
# Look here for more config file examples:
# https://github.com/rlidwka/sinopia/tree/master/conf
#

# path to a directory with all packages
storage: ./storage

auth:
  htpasswd:
    file: ./htpasswd
    # Maximum amount of users allowed to register, defaults to "+inf".
    # You can set this to -1 to disable registration.
    max_users: 3

# a list of other known repositories we can talk to
uplinks:
  npmjs:
    url: http://registry.npm.taobao.org/

packages:
  '@*/*':
    # scoped packages
    access: $all
    publish: $authenticated

  '*':
    # allow all users (including non-authenticated users) to read and
    # publish all packages
    #
    # you can specify usernames/groupnames (depending on your auth plugin)
    # and three keywords: "$all", "$anonymous", "$authenticated"
    access: $all

    # allow all known users to publish packages
    # (anyone can register by default, remember?)
    publish: $authenticated

    # if package is not available locally, proxy requests to 'npmjs' registry
    proxy: npmjs

# log settings
logs:
  - {type: stdout, format: pretty, level: http}
  #- {type: file, path: sinopia.log, level: info}

listen: 0.0.0.0:4873
```

