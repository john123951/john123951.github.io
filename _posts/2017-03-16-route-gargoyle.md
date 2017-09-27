---
layout: post
title: x86主机搭建家庭智能路由系统 ---- 安装软路由Gargoyle实现科学上网
categories: [软路由, 科学上网, 网络加速]
description: 
keywords: gargoyle, kuntcp, adbyby
---

## 目录
- [写在前面的话](#写在前面的话)
- [安装软路由系统](#安装软路由系统)
    - [物理环境](#物理环境)
    - [下载Gargoyle](#下载gargoyle)
    - [安装Gargoyle](#安装gargoyle)
    - [配置Gargoyle](#配置gargoyle)
    - [开启原路由的AP模式](#开启原路由的ap模式)
- [科学上网](#科学上网)
    - [DNS污染原理](#dns污染原理)
    - [解决方案](#解决方案)
    - [实现方式](#实现方式)
    - [使用adbyby过滤广告](#使用adbyby过滤广告)
    - [KMS服务器](#kms服务器)
    - [使用kuntcp进行双边加速](#使用kcptun进行双边加速)
- [总结](#总结)


### 写在前面的话
我在之前写的[设计篇](http://www.cnblogs.com/sweetWinne/p/6510261.html)中，提供的软路由方案是pfSense，但经过实践发现在pfSense上折腾ss和dnsmasq实在是费劲，还是使用成熟的openwrt平台来的方便一些。于是我尝试了[openwrt官方kvm版本](https://openwrt.org/)，发现其对虚拟化的支持并不是太好，只支持IDE硬盘和E1000虚拟化网卡，硬盘我倒不在乎，但网卡一定要强劲啊。后来又试了[石像鬼Gargoyle官方版本](https://www.gargoyle-router.com/)，也不尽如人意。
最后我终于找到了一个[石像鬼Gargoyle修改版本](http://koolshare.cn/thread-59568-1-1.html)，在这里感谢LEAN大神。


### 安装软路由系统

#### 物理环境
物理机拥有两块网卡，使用网卡eth0连接外网网线，负责拨号上网。网卡eth1则连接家中路由器，作AP使用。

![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170314135627932-776246185.png)

#### 下载Gargoyle
首先下载[石像鬼Gargoyle修改版本](http://koolshare.cn/thread-59568-1-1.html)，并解压出来img文件，使用Winscp或者scp命令将img文件上传到Proxmox主机。
`scp Gargoyle-1.9.0-R6-x64-Core-Edition-combined-squashfs.img root@pve:/tmp/`

#### 安装Gargoyle
1. 登录Proxmox创建一台KVM虚拟机，并分配两块虚拟网卡vtnet0、vtnet1，分别桥接物理网卡eth0、eth1。
2. 找到虚拟机磁盘文件，使用dd命令，将石像鬼Gargoyle镜像内容写入到虚拟磁盘中。
  `dd if=/tmp/Gargoyle-1.9.0-R6-x64-Core-Edition-combined-squashfs.img of=/dev/mapper/pve-vm--102--disk--1`
3. 启动虚拟机，这时Gargoyle就跑起来了。

![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170314203414776-45303100.png)

![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170314203615651-478658835.png)

#### 配置Gargoyle
登录路由进行管理：
- gargoyle管理后台  http://192.168.1.1
- openwrt 管理后台   http://192.168.1.1:8080

1. 点击上方导航中”网络“----“接口”选项，对”WAN“接口进行配置，将”上网方式“修改为”PPPoE“，填写宽带账号密码进行拨号。
2. 点击上方导航中”系统“----”启动项“选项，关闭一些不使用的服务，可以参考我的配置。

完成以上步骤后，软路由就配置好了。

![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170314204152901-522336090.png)

![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170314210604151-526778561.png)

#### 开启原路由的AP模式
软路由中没有无线网卡，使用原来的路由当作wifi发射器。鉴于不同品牌的路由配置方式也不同，我仅列出openwrt的配置方法。
1. 将“WAN接口”协议修改为“DHCP客户端”。
2. 关闭“LAN"接口的”DHCP服务器“选项。


### 科学上网
安装路由是件小事，科学上网才是我们的目的，某位伟人说过，想要打败你的敌人，首先要做的就是了解它。以下两种是目前最常见的封锁方式，解决它们就可以顺利地访问真正的互联网了。
- IP 封锁：主路由上限制访问指定IP。
- DNS 污染：黑名单的域名会被解析到错误的IP地址。

#### DNS污染原理
IP封锁就不解释了，这里主要说一下DNS污染的原理。
大家应该知道，浏览网页的第一步是请求DNS服务器，将域名解析成对应的IP地址，而后浏览器默认就会访问这个IP的80端口。
这里面有什么问题吗，当然，由于DNS查询请求是基于无状态的UDP协议，发出去包后就不管了，接收到第一个返回信息就进行处理，假设一次正常的查询请求需要经过三个路由节点，如果有人在第二个节点提前返回一个刻意伪造的查询结果，那么后返回的真实结果就会被忽略，造成DNS污染。

![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170314113316901-454224651.png)

#### 解决方案
知道了原理后，解决起来就很简单了，针对IP封锁，我们可以借助“加速服务器”来绕过gfw进行访问，同理，DNS查询请求也可以借助“加速服务器”来进行查询。

![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170314113337276-2066173721.png)

#### 实现方式
1. 使用ss-redir代理8.8.4.4的53端口（google的DNS解析服务器），作为UDP加速服务器。
2. 使用Dnsmasq做本地DNS服务器，负责判断哪些域名走国内的DNS服务器，哪些域名通过“UDP加速服务器”来查询，并将查询结果（IP地址）放入一个名叫“gfwlist”的ipset中。
3. 配置iptables，将所有目的地发往“gfwlist”中IP的流量转到shadowsocks，通过“加速服务器”来进行访问。

篇幅原因，这里就不展开了，想了解细节的朋友可以自行搜索”openwrt+ipset+ss“。因为lean大神的固件已经帮我们做了这些工作，只需要简单配置一下参数就可以了。

点击上方导航中”服务“----“ShadowsocksR设置”，配置好自己的服务器IP及密码，保存即可完成以上所有的工作(这里估计是有一个bug，脚本里主服务器的配置貌似没有使用，只需要配置备份服务器就可以了)。

![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170314213530166-588269965.png)

#### 使用adbyby过滤广告
adbyby默认是开启的，无需过多设置，如需要增加过滤的域名，只要在自定义列表中增加，并刷新缓存即可。

#### KMS服务器
KMS服务器默认开启，连上网，windows 10就自动激活了。

#### 使用kcptun进行双边加速
[kcptun](https://github.com/xtaci/kcptun)可以大幅提高ss客户端与ss服务端之间的连接速度，对我来说，这是流畅看视频的保证。
需要明白的是，使用kcptun，**必须同时安装”服务端“和”客户端“**，这里只介绍如何在路由器上安装客户端，首先下载[kcptun](https://github.com/xtaci/kcptun/releases)和[luci-app-kcptun](https://github.com/kuoruan/luci-app-kcptun/releases)，而后参照[文档](https://github.com/kuoruan/luci-app-kcptun)进行安装。
```
# 下载文件
cd /tmp
wget https://github.com/xtaci/kcptun/releases/download/v20170313/kcptun-linux-amd64-20170313.tar.gz --no-check-certificate
wget https://github.com/kuoruan/luci-app-kcptun/releases/download/v1.2.1/luci-app-kcptun_1.2.1-1_chaos-calmer_all.ipk --no-check-certificate
wget https://github.com/kuoruan/luci-app-kcptun/releases/download/v1.2.1/luci-i18n-kcptun-zh-cn_1.2.1-1_all.ipk --no-check-certificate

# 解压 kcptun
tar -zxvf kcptun-linux-amd64-20170313.tar.gz -C /usr/bin/

# 安装 luci-app-kcptun
opkg install luci-app-kcptun_1.2.1-1_chaos-calmer_all.ipk
opkg install luci-i18n-kcptun-zh-cn_1.2.1-1_all.ipk

# 卸载 luci-app-kcptun
#opkg remove luci-i18n-kcptun-zh-cn
#opkg remove luci-app-kcptun
```

安装完成后，就可以在Gargoyle中看到新的kcptun选项。
首先在“配置列表”中，将服务器信息配置好（由于环境不同配置也不尽相同，请参考官方文档），而后在“设置”中启用刚刚配置好的客户端，并将客户端程序设置为“/usr/bin/client_linux_amd64”。

![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170315221020198-320919922.png)
![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170315221032604-1600166772.png)

修改shadowsock设置，将服务器地址和端口设置为kcptun的加速通道。
```
{
    "server": "127.0.0.1",
    "server_port": 1443,
    "password": "xxxx",
    "method": "rc4-md5",
    "protocol": "origin",
    "obfs": "plain",
    "timeout": 120
}
```

最后重启路由，就可以享受到高速网络的快感了。

![](http://images2015.cnblogs.com/blog/600201/201703/600201-20170315221335979-184061066.png)


### 总结
自此，我们成功打造了一台自动趴強、过滤广告、性能优异的软路由。
本篇文章的内容涵盖较广，涉及到虚拟化、网络、openwrt操作系统，不熟悉的朋友可能会遇到很多问题，欢迎大家在留言区提出疑惑或更好的解决方案。下一篇中我会安装FreeNAS系统，配置一台带自动备份手机照片功能的NAS。