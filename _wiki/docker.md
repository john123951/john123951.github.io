---
layout: wiki
title: Docker
categories: docker
description: Docker 常用命令。
keywords: docker
---

### Tag

```
# 打标签
docker tag SOURCE_IMAGE[:TAG] IMAGE[:TAG]
docker tag demo:1 demo:latest

# 删除标签
docker rmi IMAGE[:TAG]
```