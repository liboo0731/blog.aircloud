---
title: 【centos】YUM源创建及使用
date: 2017-08-08 21:30:05
tags:
- centos
- yum
---

### YUM源创建前目录结构：

```shell
[root@packages ~]# tree tmp-packages/

tmp-packages/
├── Packages
│   └── xxx.rpm
└── repodata
    └── comps.xml

2 directories, 2 files
```

### YUM源创建后目录结构：

```shell
[root@packages ~]# tree tmp-packages/

tmp-packages/
├── Packages
│   └── xxx.rpm
└── repodata
    ├── comps.xml
    ├── comps.xml.gz
    ├── filelists.sqlite.bz2
    ├── filelists.xml.gz
    ├── other.sqlite.bz2
    ├── other.xml.gz
    ├── primary.sqlite.bz2
    └── primary.xml.gz

2 directories, 9 files
```

### YUM源创建及使用流程如下：

1. 将所有RPM安装包拷贝到“Packages”目录下
2. 编辑“comps.xml”文件，并移至“repodata”目录下
3. 执行“creatrtepo -g ./repodata/comps.xml ./”命令
4. 编辑“xxx.repo”软件仓库文件，移至“/etc/yum.repo.d/”目录下
5. 使用yum 命令安装RPM 包

### “xxx.repo”文件内容参考如下：

- [....]     代表这个库的名字，必须是唯一的，不可以重复。
- name＝     是这个库的说明，没有太大的实际意义，只是一个字段说明。
- baseurl＝  说明采取什么方式传输，具体路径在哪里，可以使用的方式有,file://，ftp://，http://等。
- enabled=1  说明启用这个更新库，0表示不启用。
- gpgcheck=1 表示使用gpg文件来检查软件包的签名。
- gpgkey=    表示gpg文件所存放的位置，此处也可以有http方式的位置。



