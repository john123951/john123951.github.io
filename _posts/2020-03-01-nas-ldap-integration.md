---
layout: post
title: 统一认证，让NAS体验上升一个台阶
categories: [NAS, ldap, docker, 自建家用NAS计划]
description: 
keywords: ldap, docker
---

## 写在前面
今天是周末，隔离在家也没事干，继续来分享如何提升NAS体验。

本文的主要目的有两个【统一入口】和【统一认证】。

统一入口自不用多说，打开浏览器首页，就是各个服务的链接，这种体验用起来就回不去了，

统一认证，表示所有的软件都使用同样的认证源，形象点说就是你拿着支付宝，可以去几乎任何一家商场、超市、餐厅消费，因为花的都是支付宝的钱（是你支付宝的钱 ）。

认证中心也是如此，商场、超市、餐厅就是一个个服务，每次需要“登录”时都是归“认证中心”负责。



## 统一入口
![统一入口](/images/blog/2020-03-01-nas-ldap-integration/heimdall_dashboard.png)

上期的留言中，有位朋友问，如何给自己的NAS搭建一个统一的访问入口，就像上面图里的一样。下面我就详细说明一下如何搭建，动起手来其实也只需要5分钟。

统一入口需要使用到开源项目：[Heimdall](https://gitlab.com/BenjaminDobell/Heimdall)（[备用地址](https://github.com/linuxserver/Heimdall)），搭建方法使用docker，相信各位玩NAS的朋友应该不会陌生。

首先打开NAS的Docker功能，镜像名称输入 **linuxserver/heimdall** 。

第二步，映射 80端口和443端口。

第三步，启动起来，完成。 



​作者以自己使用的unRAID系统举例，在APP商店（参考其他文章）中找到“Heimdall”，点击安装即可。

![商店中可以直接安装](/images/blog/2020-03-01-nas-ldap-integration/app_heimdall.png)


![动画演示](/images/blog/2020-03-01-nas-ldap-integration/heimdall_install.gif)

添加快捷方式就点右下角的“三” ，输入应用名称“plex”，对应的图标就会出现，heimdall默认支持上百款应用，如果遇到没有的应用，也可以自己上传图标。

![heimdall](/images/blog/2020-03-01-nas-ldap-integration/heimdall_add.png)


## 搭建统一认证中心
想提升NAS体验，非常重要的一个要素，是使用统一认证中心。

> 认证中心，主要作用是统一保存用户相关的信息，像用户名、密码、头像、昵称。。。

nextcloud、jellyfin，这类需要登录后使用的软件，都应该将“认证”这件事情都交给“认证中心”处理，而优势也很明显，再也不用每个软件都去维护修改密码、修改头像、删除用户、新增用户，都去“认证中心”做就好了。

![大体逻辑是这样子的](/images/blog/2020-03-01-nas-ldap-integration/nas_docker_software_simple.jpg)

具体的服务，本文则选用openLDAP搭建认证中心。（我知道估计你没听过，但不重要） 

> LDAP属于一种通用通讯协议，可以简单理解为，保存用户的数据库

好了，说了这么多，现在就开始吧。



## 安装openldap
首先安装今天的主角“openldap”，“openldap”是保存所有用户的**数据库**，将来用户的新增、密码修改、头像修改等等都是在“openldap”中进行，这样就不用挨个服务改一遍了。

安装openldap的过程和heimdall是一样的，在“应用商店”中搜索“openldap”，然后安装。

允许配置的信息比较多 ，但**只需要修改下面几项，其余的都不用动**，

LDAP port： 认证服务的端口，不用修改，使用默认的389。

LDAP_ORGANISATION：组织名称，可以随意输入自己的名称。

LDAP_ADMIN_PASSWORD：管理员账号密码。

LDAP_CONFIG_PASSWORD：只读账号密码（用于在nextcloud里配置）。

LDAP_TLS：是否使用加密，这里**【一定要设置为“false”，否则无法启动】**。

![openldap配置](/images/blog/2020-03-01-nas-ldap-integration/ldap_config_01.png)
![openldap配置](/images/blog/2020-03-01-nas-ldap-integration/ldap_config_02.png)

完成后查看列表，如果显示绿色的“started”则表示安装成功。 

![安装成功](/images/blog/2020-03-01-nas-ldap-integration/openldap_complate.png)


## 增加用户
服务安装完成后，第一步先来配置我们平常的使用账号。

首先下载一个绿色版的配置工具， [ldapAdmin](http://www.ldapadmin.org/download/ldapadmin.html)

（如果遇到了那个原因，或者想将工具安装到NAS的用户，也可以在应用商店下载“phpldapadmin”）

打开软件后是空白的，点击左上角“建立连接”。

![主界面](/images/blog/2020-03-01-nas-ldap-integration/ldapadmin_main.png)

连接服务器，IP填写NAS的地址，端口389，然后账户“cn=admin,dc=oauth,dc=net”及对应的密码，

信息都正确后点击“Fetch DNs”，会自己读取到“Base”。（没看懂可以再看一遍 ）

![连接ldapAdmin](/images/blog/2020-03-01-nas-ldap-integration/ldapadmin_new_connection.png)

连接到服务器后，会看到这个树状的结构，使用起来和“win10的资源管理器”差不多，先创建“文件夹”，再在“文件夹”里创建用户。

![新建“文件夹”](/images/blog/2020-03-01-nas-ldap-integration/ldapadmin_create_ou.png)

![文件夹名称](/images/blog/2020-03-01-nas-ldap-integration/ldapadmin_create_ou_02.png)

然后创建平常使用的“用户”。

![创建新用户](/images/blog/2020-03-01-nas-ldap-integration/ldapadmin_create_user.png)

![用户基本信息](/images/blog/2020-03-01-nas-ldap-integration/ldapadmin_create_user_01.png)
![用户基本信息](/images/blog/2020-03-01-nas-ldap-integration/ldapadmin_create_user_02.png)
![右键“修改用户密码”](/images/blog/2020-03-01-nas-ldap-integration/ldapadmin_create_user_03.png)

经过上面的步骤，总算完成了一半的工作，现在“认证中心”中已经有了一个用户，如需要创建其他用户，重复操作即可。

下面来看看如何将nextcloud和jellyfin连接到“认证中心”，用刚刚新增的用户登录nextcloud。


## nextcloud集成认证中心（LDAP）
nextcloud默认LDAP集成功能并没有打开，需要手动打开，在【应用】中搜索“LDAP user and group backend”并启用，启用后【设置】中就可以修改LDAP配置了。

![打开LDAP功能](/images/blog/2020-03-01-nas-ldap-integration/nextcloud_ldap.png)
![管理LDAP](/images/blog/2020-03-01-nas-ldap-integration/nextcloud_ldap_00.png)

LDAP配置信息，填写NAS的IP和端口，账户密码使用“LDAP只读账号”，DN可以在ldapAdmin右键复制。

![复制“只读账号”DN](/images/blog/2020-03-01-nas-ldap-integration/ldapadmin_copy_dn.png)

![填写连接信息](/images/blog/2020-03-01-nas-ldap-integration/nextcloud_ldap_01.png)

后面的几页配置，保持默认即可。


![LDAP配置](/images/blog/2020-03-01-nas-ldap-integration/nextcloud_ldap_03.png)
![LDAP配置](/images/blog/2020-03-01-nas-ldap-integration/nextcloud_ldap_04.png)
![LDAP配置](/images/blog/2020-03-01-nas-ldap-integration/nextcloud_ldap_05.png)


经过以上的配置，“认证中心”中的用户就可以登录了。 

![登录](/images/blog/2020-03-01-nas-ldap-integration/nextcloud_login.png)



## jellyfin集成认证中心（LDAP）
jellyfin同样需要安装LDAP插件，在【插件】中找到“LDAP 认证”并安装。

![LDAP插件](/images/blog/2020-03-01-nas-ldap-integration/jellfin_plugin_ldap.png)
![LDAP插件](/images/blog/2020-03-01-nas-ldap-integration/jellfin_plugin_ldap_complate.png)

按要求重启后，再次找到【插件】中的“LDAP”进行设置。

![插件中找到安装好的LDAP](/images/blog/2020-03-01-nas-ldap-integration/jellfin_plugin_ldap_settings.png)

打开配置后，可以看到配置项也是好多 ，但其实都属于两类：LDAP服务器信息、可用登录用户的条件。

LDAP Server：填写NAS服务器IP

Secure LDAP：是否使用加密，**一定要取消勾选**。

Base DN：填写和nextcloud相同的配置。

LDAP User Filter：LDAP中的哪些用户可以登录，修改为(objectClass=inetOrgPerson)。

最下方填写LDAP的只读账号。

![LDAP配置](/images/blog/2020-03-01-nas-ldap-integration/jellyfin_ldap_settings.png)

全部配置完成后，重启Jellyfin，就可以使用新用户登录了。

![登录](/images/blog/2020-03-01-nas-ldap-integration/jellyfin_login.png)

## 尾言
到此为止，【统一入口】和【统一认证】就已经搭建好了。大家如果有其他的想法可以留言与我讨论。


   