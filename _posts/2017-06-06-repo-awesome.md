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
https://mirrors.aliyun.com/alpine/v3.4/main/
```

### CentOS 7
```
# 备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

# 安装阿里云镜像
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

# EPEL
mv /etc/yum.repos.d/epel.repo /etc/yum.repos.d/epel.repo.backup
mv /etc/yum.repos.d/epel-testing.repo /etc/yum.repos.d/epel-testing.repo.backup
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo

# Nux Dextop
# http://li.nux.ro/repos.html
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm
```


