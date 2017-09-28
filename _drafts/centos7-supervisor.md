---
layout: post
title: centos 7 supervisor
categories: [cate1, cate2]
description: some word here
keywords: keyword1, keyword2
---

### 安装
可以通过 yum 或 pip 方式安装
```
sudo yum install supervisor
# or
sudo pip install supervisor
```

### 基本配置
```
# 配置文件模板
echo_supervisord_conf > supervisord.conf
sudo cp supervisord.conf /etc/supervisord.conf
# 各个应用配置放在此处
sudo mkdir /etc/supervisord.d/
# 编辑配置
sudo vi /etc/supervisord.conf

# 在文件末端修改或添加
[include]
files = /etc/supervisord.d/*.ini
```

### 服务配置
```
#!/bin/sh
#
# /etc/rc.d/init.d/supervisord
#
# Supervisor is a client/server system that
# allows its users to monitor and control a
# number of processes on UNIX-like operating
# systems.
#
# chkconfig: - 64 36
# description: Supervisor Server
# processname: supervisord

# Source init functions
. /etc/rc.d/init.d/functions

prog="supervisord"

prefix="/usr/"
exec_prefix="${prefix}"
prog_bin="${exec_prefix}/bin/supervisord"
PIDFILE="/var/run/$prog.pid"

start()
{
       echo -n $"Starting $prog: "
       daemon $prog_bin --pidfile $PIDFILE
       [ -f $PIDFILE ] && success $"$prog startup" || failure $"$prog startup"
       echo
}

stop()
{
       echo -n $"Shutting down $prog: "
       [ -f $PIDFILE ] && killproc $prog || success $"$prog shutdown"
       echo
}

case "$1" in

 start)
   start
 ;;

 stop)
   stop
 ;;

 status)
       status $prog
 ;;

 restart)
   stop
   start
 ;;

 *)
   echo "Usage: $0 {start|stop|restart|status}"
 ;;

esac
```

添加自启动：
```
sudo chmod +x /etc/rc.d/init.d/supervisord
sudo chkconfig --add supervisord
sudo chkconfig supervisord on
sudo service supervisord start
```