# 隧道技术，随时随地接入家庭内网

## 写在前面

搭建NAS的朋友想必都做过外网映射，把内部的服务一个一个的映射到外网，从而可以随时访问自己的相册、文件等等。

所以这篇文章，我就反其道行之，通过VPN隧道技术，将自己反向接入内网，只需要映射一个端口，就可以通过这个端口随时置身于家庭内网之中，无限制地访问内网各个服务。





## 隧道技术简介

隧道技术，是通过互联网，建立一条虚拟链路，以传递数据的方式。



举个例子，小王和小刘说着不同语言的不同方言，单独来看，两个人虽然都能说话，但却没办法明白对方想表达的意思，如果这时候小王唱起一首童年的旋律，小刘也就秒懂了对方的情义。这里，歌曲就是依托于声音建立起的“隧道”。



理解了原理，下面要做的就简单多了。



建立一条“隧道”，这条隧道基于互联网，可以使我们获取家庭内部IP地址，随时访问内部服务。

实现的工具也有许多，OpenVPN、ZeroTier 等等，**WireGuard** 则是其中的佼佼者。

WireGuard 分为“服务端”与“客户端”，服务端部署在NAS，客户端可以是手机、平板或者公司的办公主机。



使用时，客户端通过“拨号”，接入服务端网络，而后就可以访问内部的服务了。





## 安装 WireGuard 服务端

作者使用的是unRAID平台，本篇文章就以此平台来进行说明，其他平台可以触类旁通，配置方法都是大同小异。



安装过程非常简单，“Community Applications” 中搜索 “WireGuard” 并进行安装。

**（注意：unRAID至少需要6.8.1，否则无法安装WireGuard，低版本的请自行百度升级）** **（注意：Community Applications 第三方库，安装教程请参看之前文章）**



![install.png](D:\codes\john123951.github.io\images\blog\2020-03-25-wireguard\install.png)





## 配置 WireGuard（创建用户）

![wireguard-setting.png](D:\codes\john123951.github.io\images\blog\2020-03-25-wireguard\wireguard-setting.png)



安装完成后，首先要进行配置，点击："Settings" -- "VPN Manager" 进入管理。

第一步，填写WireGuard名字。

第二步，点击右侧按钮生成服务端秘钥。

第三步，保持 "EndPoint" 使用默认的 51820 端口。

第四步，将服务设置为开机自动启动，并立即启用服务。



![wireguard-setting-genkey.png](D:\codes\john123951.github.io\images\blog\2020-03-25-wireguard\wireguard-setting-genkey.png)

![wireguard-setting-autorun.png](D:\codes\john123951.github.io\images\blog\2020-03-25-wireguard\wireguard-setting-autorun.png)



服务端配置好，开始给用户分配账号。

第一步，点击下方 "Add Peer"，设置Peer名字，

第二步，将 "Peer type of access" 设置为 "access to LAN"，这样才可以访问内网服务。

第三步，点击右侧生成客户端密钥对，并保存配置。



![wireguard-setting-peer.png](D:\codes\john123951.github.io\images\blog\2020-03-25-wireguard\wireguard-setting-peer.png)



经过以上几个步骤，服务端就已经设置好了，记得开启路由器的 51820 端口映射，这样外网才可以连接到 WireGuard 服务。





## 安装 WireGuard客户端





## 总结
