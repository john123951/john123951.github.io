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
netsh interface teredo set state server=teredo.remlab.net

#### 解决"客户端位于托管网络中"
netsh int ter set state enterpriseclient

#### 参考资料
> https://blog.felixc.at/2010/04/install-teredo-ipv6/
> http://test-ipv6.com/

公共Teredo服务器地址，若无法连通可替换为一下列表中的值：
* teredo.remlab.net / teredo-debian.remlab.net (法国) (Miredo 默认设置)
* teredo.autotrans.consulintel.com (西班牙)
* teredo.ipv6.microsoft.com (美国 雷蒙德) (Windows XP/2003/Vista/7/2008 系统默认设置)
* teredo.ngix.ne.kr (韩国)
* teredo.managemydedi.com (美国 芝加哥)