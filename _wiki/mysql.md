---
layout: wiki
title: crontab
categories: linux
description: crontab 格式。
keywords: linux, cron, crontab
---

### MySql 5.7

#### 新建用户 
创建用户 test，密码 123456。
```
mysql -u root -p

-- 创建用户
CREATE USER "test"@"localhost" IDENTIFIED BY "123456";
CREATE USER "test"@"%" IDENTIFIED BY "123456";

-- 删除账户及权限
drop user test@'localhost';
drop user test@'%';
```

#### 用户授权
授权格式：grant 权限 on 数据库.* to 用户名@登录主机 identified by "密码"
```
create database testDB default charset utf8 collate utf8_general_ci;

-- 所有权限
grant all privileges on testDB.* to "test"@"localhost" identified by "123456";
-- 部分权限
grant select,update on testDB.* to "test"@"localhost" identified by "123456";

-- 刷新系统权限表
flush privileges;
```

#### 修改用户密码
```
update mysql.user set authentication_string=password("新密码") where User="test" and Host="localhost"; 
flush privileges;
```