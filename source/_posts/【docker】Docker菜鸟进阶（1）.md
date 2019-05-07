---
title: 【docker】Docker菜鸟进阶（1）
date: 2019-05-07 18:35:54
tags:
- docker
---

### 文档

[DaoCloud Hub](https://hub.daocloud.io/)

[Docker Documentation](https://docs.docker.com/get-started/)

[DockerInfo](http://www.dockerinfo.net/document)

[Docker中文社区](http://www.docker.org.cn/)

[Docker源地址](https://download.docker.com/)

### 安装Docker

```shell
cd /etc/yum.repos.d/
#官方源
wget https://download.docker.com/linux/centos/docker-ce.repo
wget https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/centos/docker-ce.repo
#替换清华源
sed -i 's@https://download.docker.com@https://mirrors.tuna.tsinghua.edu.cn/docker-ce@g' docker-ce.repo
#安装docker-ce
yum install docker-ce -y
#镜像加速
cat >> /etc/docker/daemon.json <<EOF
{
	"registry-mirrors": ["https://guxaj7v7.mirror.aliyuncs.com","https://registry.docker-cn.com"]
}
EOF
#启动服务
systemctl enable docker
systemctl start docker
#查看版本信息
docker version
docker info

```

### 常用命令

```shell
docker search centos	#搜索镜像
docker pull centos:7.4.1708		#下载特定版本镜像
docker run hello-world /hello	#启动并执行命令
docker run -it centos:7.4.1708 /bin/bash	#交互模式启动镜像
docker run -d centos:7.4.1708 /bin/bash	#后台启动镜像
docker run -itd centos:7.4.1708 /bin/bash
#罗列容器
docker ps	#罗列启动的容器
docker ps -a	#罗列所有容器
docker start/stop/restart centos:7.4.1708	#启动/关闭/重启
#交互式进入容器
docker attach centos:7.4.1708	#多端同步，退出后关闭
docker exec -it centos:7.4.1708 /bin/bash	#独立运行
#退出
exit
CTRL+D
#查看镜像
docker images
docker images -a
#导出/导出镜像
docker save -o centos.7.4.1708.tar centos:7.4.1708
docker save -o centos.7.4.1708.tar e771603453ab centos:7.4.1708
docker load -i centos.7.4.1708.tar
#导出/导入容器
docker export -o centos.7.4.1708.tar centos:7.4.1708
docker export -o centos.7.4.1708.tar e771603453ab
docker import centos.7.4.1708.tar centos:7.4.1708
#删除镜像/容器
docker rm e771603453ab
for i in `docker ps -a|awk '{print $1}'|grep -v CONTAINER`;do docker rm $i;done
docker rmi e771603453ab
for i in `docker images -a|awk '{print $3}'|grep -v IMAGE`;do docker rmi $i;done

```

