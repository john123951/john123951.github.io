---
layout: post
title: 使用teredo访问IPV6网络
categories: [ipv6]
description: some word here
keywords: ipv6, teredo
---


### 启动服务
iphlpsvc (IP Helper)

### 查看状态
netsh int teredo show state

### 更改服务器地址
netsh interface teredo set state server=teredo2.remlab.net

#### 解决"客户端位于托管网络中"
netsh int ter set state enterpriseclient

#### 参考资料
> https://blog.felixc.at/2010/04/install-teredo-ipv6/
> http://test-ipv6.com/
> [[IPV6] 开启 Teredo 之微软官方教程](https://github.com/XX-net/XX-Net/issues/7156)
> https://github.com/XX-net/XX-Net/issues/6918

```
# 安装依赖
sudo pacman -S cmake libperconaserverclient
cd /usr/lib
sudo ln -s libperconaserverclient.so libperconaserverclient_r.so

```

