---
layout: post
title: 全站启用HTTP2协议
categories: [server]
description: some word here
keywords: http2, certbot, nginx, ssl
---

### Http2协议
HTTP/2，超文本传输协议第2版，向上兼容HTTP/1.0、HTTP/1.1，于2015年5月正式发表。根据W3Techs的数据，在2017年5月，在排名前一千万的网站中，有13.7%支持了HTTP/2。

HTTP/2新特性包括多路复用，修复队头阻塞，允许设置设定请求优先级，头部压缩算法(HPACK)，服务器推送等。此外， HTTP/2 采用了二进制而非明文来打包、传输客户端—服务器间的数据。

简单来说，HTTP/2拥有以下优点：
- 提升网站访问速度
- 增加网站安全性
- 降低服务器压力


### Let's Encrypt 证书
[Let’s Encrypt](https://letsencrypt.org/)是一个于2015年三季度推出的数字证书认证机构，旨在以自动化流程消除手动创建和安装证书的复杂流程，并推广使万维网服务器的加密连接无所不在，为安全网站提供免费的SSL/TLS证书。([维基百科](https://zh.wikipedia.org/wiki/Let%27s_Encrypt))


### 配置Nginx启用Http2

#### 生成SSL证书
安装Let’s Encrypt提供的命令行工具certbot：
```
yum install certbot
```

创建并修改`/etc/nginx/default.d/certbot.conf`文件：
```
location ^~ /.well-known/ {
    alias    /usr/share/nginx/html/.well-known/;
}
```

生成证书
```
certbot certonly --webroot -w /usr/share/nginx/html/ -d lovewinne.com -d blog.lovewinne.com
```


#### Nginx配置
Nginx 1.9.0 以上的版本已经原生支持Http2，只需要在配置文件中打开即可。

修改`/etc/nginx/conf.d/blog.conf`文件：
```
server {
    listen       80;
    listen       [::]:80;
    server_name blog.lovewinne.com;
    return 301 https://$server_name$request_uri;
}
server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name  blog.lovewinne.com;
    root         /xxx/blog;

    # Load configuration files for the default server block.
    include /etc/nginx/default.d/*.conf;

    ssl_certificate /etc/letsencrypt/live/domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domain.com/privkey.pem;
    ssl_trusted_certificate /etc/letsencrypt/live/domain.com/chain.pem;

    location / {
        index index.html index.htm;
    }
}
```

#### 自动更新证书
Let's Encrypt的证书时效只有90天，需要自行更新证书，配置定时任务`crontab -e`每月1号自动更新。
```
#+------------ Minute            (range: 0-59)
#| +---------- Hour              (range: 0-23)
#| | +-------- Day of the Month  (range: 1-31)
#| | | +------ Month of the Year (range: 1-12)
#| | | | +---- Day of the Week   (range: 1-7, 1 standing for Monday)
#| | | | |
#* * * * *
10 2 1 * * /usr/bin/certbot renew  >> /var/log/le-renew.log

```

