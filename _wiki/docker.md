---
layout: wiki
title: Docker
categories: docker
description: Docker 常用命令。
keywords: docker
---

### 容器操作
```
# 启动所有容器
docker ps --all --format "{{.Names}}" |xargs docker start

```


### 镜像操作
```
```


### 删除&清理
```
# 删除旧容器
docker rm $(docker images celery-scheduler --filter "before=celery-scheduler:latest")

# 删除旧镜像
docker rmi $(docker images celery-scheduler --filter "before=celery-scheduler:latest" -q)
```


### Tag
```
# 打标签
docker tag SOURCE_IMAGE[:TAG] IMAGE[:TAG]
docker tag demo:1 demo:latest

# 删除标签
docker rmi IMAGE[:TAG]

```
