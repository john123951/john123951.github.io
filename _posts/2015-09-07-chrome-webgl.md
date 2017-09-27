---
layout: post
title: Linux下Chrome浏览器不支持WebGL的解决方式。
categories: [chrome, linux]
description: 
keywords: chrome, webgl
---

今天使用Chrome浏览器，总是报这样一个错误：  

<span style='color:red'>Uncaught TypeError: Cannot read property 'canvas' of null</span>。  

细看之下是无法获取WebGL上下文
```
canvas.getContext('experimental-webgl') == null
```

---
解决方式：打开Chrome的硬件加速功能

设置页面([chrome://settings/](chrome://settings/)) -- 高级设置 --勾选Use hardware acceleration when available -- 重启Chrome -- 解决
