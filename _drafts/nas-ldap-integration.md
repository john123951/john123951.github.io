---
layout: post
title: 自建家用NAS计划 ---- 基于Docker的常用软件
categories: [NAS, unRAID, docker, 自建家用NAS计划]
description: some word here
keywords: keyword1, keyword2
---



Docker打造完美的NAS体验，媲美群晖

谈谈如何使用docker，搭建一台“群晖”

使用开源软件，一台得心应手的NAS

整合NAS，你需要有【统一认证中心】
NAS也能用上【统一认证中心】

# 大纲
- 

----
## 搭建统一认证中心
提升体验非常重要的一个要素是，使用统一认证中心。

认证中心，主要作用是统一保存用户相关的信息，像用户名、密码、头像、昵称。。。

而nextcloud、jellyfin这类需要登录才可以使用的软件，可以将“认证”的权限都交给“认证中心”处理，
优势也很明显，再也不用每个软件都去维护修改密码、修改头像、删除用户、新增用户，都去“认证中心”做就好了。


说了这么多，现在就开始吧。
本文使用openLDAP搭建认证中心，（LDAP属于一种通用通讯协议，可以简单理解为，保存用户的数据库）

### openldap


### nextcloud配置LDAP

### jellyfin配置LDAP


## 尾言


> [Github]()