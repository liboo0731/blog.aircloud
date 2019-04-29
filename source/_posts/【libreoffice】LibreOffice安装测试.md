---
title: 【libreoffice】LibreOffice安装测试
date: 2019-04-29 09:03:04
tags:
- libreoffice
---

### LibreOffice下载地址

[libreoffice-6.2.3](http://download.documentfoundation.org/libreoffice/stable/6.2.3/rpm/x86_64/)

### JODConverter下载地址

[jodconverter-2.2.2](https://sourceforge.net/projects/jodconverter/files/JODConverter/2.2.2/)

### LibreOffice安装

```shell
#下载
wget http://download.documentfoundation.org/libreoffice/stable/6.2.3/rpm/x86_64/LibreOffice_6.2.3_Linux_x86-64_rpm.tar.gz
wget http://download.documentfoundation.org/libreoffice/stable/6.2.3/rpm/x86_64/LibreOffice_6.2.3_Linux_x86-64_rpm_sdk.tar.gz
#解压
tar xzvf LibreOffice_6.2.3_Linux_x86-64_rpm.tar.gz
tar xzvf LibreOffice_6.2.3_Linux_x86-64_rpm_sdk.tar.gz
#安装
yum localinstall LibreOffice_6.2.3_Linux_x86-64_rpm/RPMS/*.rpm
yum localinstall LibreOffice_6.2.3_Linux_x86-64_rpm_sdk/RPMS/*.rpm
#启动
/opt/libreoffice6.2/program/soffice --headless --accept="socket,host=127.0.0.1,port=8100;urp;" --nofirststartwizard &
```

### LibreOffice测试

1. 下载jodconverter，解压
2. 将test.doc拷贝到lib目录下
3. 执行 java -jar jodconverter-cli-2.2.2.jar  test.doc test.pdf 命令，查看是否生成pdf文件

### LibreOffice安装排错

```shell
#执行命令：
/opt/libreoffice6.0/program/soffice -help

#错误信息：
/opt/libreoffice6.0/program/soffice.bin: error while loading shared libraries: libcairo.so.2: cannot open shared object file: No such file or directory
#解决方案：
yum install cairo

#错误信息：
/opt/libreoffice6.0/program/soffice.bin: error while loading shared libraries: libcups.so.2: cannot open shared object file: No such file or directory
#解决方案：
yum install cups-libs

#错误信息：
/opt/libreoffice6.0/program/soffice.bin: error while loading shared libraries: libSM.so.6: cannot open shared object file: No such file or directory
#解决方案：
yum install libSM

```

