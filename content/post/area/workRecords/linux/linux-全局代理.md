---
title: "Linux 全局代理"
date: 2024-01-05T11:20:56+08:00
categories:
- 收件箱
- linxu相关
tags:
- shell
- 案例
keywords:
- 说明
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
## linux 代理

|环境变量| 描述| 值示例|
|---|---|---|
|http_proxy|为http变量设置代理；默认不填开头以http协议传输|http://10.0.0.51:8080  http://user:pass@10.0.0.10:8080    socks4://10.0.0.51:1080  socks5://192.168.1.1:1080|
|https_proxy|为https变量设置代理；|同上|
|ftp_proxy|为ftp变量设置代理；|同上|
|all_proxy|全部变量设置代理，设置了这个时候上面的不用设置|同上|
|no_proxy|无需代理的主机或域名； 可以使用通配符； 多个时使用“,”号分隔；|.aiezu.com,10...,192.168.., *.local,localhost,127.0.0.1|



