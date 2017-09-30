---
layout: post
title: 多种包管理器加速镜像
categories: [mirror, repository]
description: 各种包管理器的国内镜像站点
keywords: mirror, pip, gem
sticky: true
---

各种包管理器的国内镜像站点。


### python pip
```
# /etc/pip.conf
# ~/.pip/pip.conf    # unix
# %HOME%\pip\pip.ini # windows
[global]
trusted-host =  mirrors.aliyun.com
index-url = http://mirrors.aliyun.com/pypi/simple
```

### ruby
[doc](http://mirrors.aliyun.com/help/rubygems)

```
# 项目目录 Gemfile
source 'http://mirrors.aliyun.com/rubygems/'
```

配置镜像（本地环境有效）
```
bundle config 'mirror.https://rubygems.org' 'http://mirrors.aliyun.com/rubygems/'
```

### alpine
```
# /etc/apk/repositories
http://mirrors.ustc.edu.cn/alpine/v3.4/main/
```
