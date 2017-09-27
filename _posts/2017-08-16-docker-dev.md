---
layout: post
title: docker搭建开发环境
categories: [virtualization, docker]
description: 
keywords: docker, portainer
---

## Docker 搭建开发环境
本文介绍如何将Docker集成到开发环境，自动构建应用，并使容器拥有独立的内网IP为开发人员提供服务。

术语解释
- Docker镜像：一个不可修改的"模板"，每个代码版本对应一个镜像版本，本身不可运行。
- Docker容器：镜像的"实例"，必须且只能指定一个"镜像"来创建容器，创建时可选择要暴露的内部接口或要挂载的目录等，本身可以启用、停止或删除，内部系统不应被修改，如需修改应创建一个新"镜像"，再运行此镜像的容器。

组件介绍
- [portainer](https://github.com/portainer/portainer)：轻量级Docker管理界面，支持Swarm集群。


### 自动构建流程
![](/images/blog/2017-08-16-docker-dev/infrastructure.png)


### Docker 配置

#### 打开Http方式 [Docker Engine API](https://docs.docker.com/engine/api/)
编辑文件`/usr/lib/systemd/system/docker.service`
找到`ExecStart=/usr/bin/dockerd -H fd:// `
修改为`ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -H fd://`
重加载 systemd daemon
`systemctl daemon-reload`

#### 加速器配置
为加快下载速度，配置daocloud的[加速镜像](https://www.daocloud.io/mirror#accelerator-doc)。
```
curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://7515421f.m.daocloud.io
```

#### 为容器分配独立IP
默认情况下，Docker创建的容器运行在私有的内部网络中。
想要赋予每个容器独立IP，可以使用Docker内置的[macvlan](https://docs.docker.com/engine/userguide/networking/get-started-macvlan/)驱动将容器桥接到宿主机的网卡上，这样宿主机网段内的其他主机便可通过IP地址访问到容器了。

首先在宿主机创建一个新网络，注意网关及子网掩码与宿主机网络一致。
```
docker network create -d macvlan \
    --subnet=10.16.10.0/21 \
    --gateway=10.16.8.254  \
    -o parent=eth0 sweet
```
之后创建的容器，如果需要独立IP，只需要配置`--network sweet`参数即可。


### 安装基本环境

#### 安装 Portainer
```
docker run -d -p 9000:9000 \
	-v /var/run/docker.sock:/var/run/docker.sock \
    --restart always \
    --name portainer portainer/portainer
```

#### 安装 Jenkins
```
docker run -p 8080:8080 \
	-v /usr/bin/docker:/usr/bin/docker \
	-v /var/jenkins_home:/var/jenkins_home \
    --restart always \
	--name jenkins jenkins/jenkins:lts-alpine
```

#### 配置 Portainer
访问 http://localhost:9000 。首次登录需设置管理员密码，并配置需要管理的Docker Server地址。

#### 配置 Jenkins
1. 访问 http://localhost:8080 。首次登录使用Docker控制台打印的一次性密码进行登录。
2. "系统设置"--"全局属性"中配置环境变量。  ![](/images/blog/2017-08-16-docker-dev/jenkins_env_settings.png)


### 自动构建

#### 项目构建任务
下面演示一个项目的构建计划如何配置，首先在项目根目录创建Dockerfile文件，声明如何构建docker镜像。
例如我创建的[项目](https://github.com/john123951/spring-boot-docker)中：
```
FROM java:8-alpine

RUN mkdir /app
WORKDIR /app
ADD target/docker-demo.jar /app

EXPOSE 9001
ENTRYPOINT ["java","-jar","docker-demo.jar"]
```

而后，Jenkins中新建一个构建任务，在"Post Steps"部分添加"Execute shell"，并做如下配置：
```
echo "================ Docker Build ================" >> /dev/null
docker -H $DOCKER_URL build --force-rm --tag $JOB_NAME:$BUILD_NUMBER $WORKSPACE

echo "================ Docker Remove Container ================" >> /dev/null
docker -H $DOCKER_URL rm --force $JOB_NAME

echo "================ Docker Run ================" >> /dev/null
docker -H $DOCKER_URL run -dt \
--network sweet \
--name $JOB_NAME $JOB_NAME:$BUILD_NUMBER
```

#### 清理镜像任务
虽然Docker采用了分层的方式存储镜像文件，但开发环境的高频率构建，会很快地填满硬盘，因此我们需要定期地删除无用镜像文件。

新建一个构建项目`clean`
配置"Build periodically"为`H 6 * * *`，每天早上6点执行一次清理任务。
配置"Execute shell"如下，删除48小时前没有被使用的镜像文件。
```
echo $DOCKER_URL
docker -H $DOCKER_URL images
docker -H $DOCKER_URL image prune -a -f --filter "until=48h"

echo "================ After Prune ================" >> /dev/null
docker -H $DOCKER_URL images
```


### 使用方式
自此，
Jenkins会按配置好的触发器执行构建，删除原有容器（原有的镜像不会删除），并启动一个新版本的容器。
Portainer中可以查看已经启动的容器，IP地址，Log日志，也可手动停用或创建上一版本的容器。
同网段的其他主机，可以直接访问容器的IP地址进行远程服务调用。
环境搭建完成。

![](/images/blog/2017-08-16-docker-dev/jenkins_dashboard.png)
![](/images/blog/2017-08-16-docker-dev/portainer_containers.png)
