---
title: "Coredns_二进制搭建"
date: 2023-10-24T10:26:33+08:00
categories:
- 项目
- kubernetes相关
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

# 背景:

# 环境：
|组件|	版本|	作用|
|centos|	7.9|	承载业务系统|
|coredns|	1.10.1|	域名解析|
安装包准备：  coredns
# 步骤:
配置/usr/lib/systemd/system/coredns.service  添加systemd管理
```
[Unit]
Description=CoreDNS
Documentation=https://coredns.io/manual/toc/
After=network.target
[Service]
# Type设置为notify时，服务会不断重启
# 关于type的设置，可以参考https://www.freedesktop.org/software/systemd/man/systemd.service.html#Options
Type=simple
User=root
# 指定运行端口和读取的配置文件
ExecStart=/usr/local/bin/coredns -conf /etc/coredns/coredns.conf
Restart=on-failure
[Install]
WantedBy=multi-user.target
```
配置/etc/coredns/coredns.conf
```
.:53 {
hosts {
  191.191.191.191 test.com # 本地解析
  fallthrough }
forward . 223.5.5.5   #失败转发解析
log
errors
cache
}
```
将coredns 加入守护进程
```
systemctl daemon-reload  
systemctl start coredns
systemctl stop coredns
```
防火墙开启udp53端口
```
firewall-cmd  --add-port=53/udp     #临时开启udp端口53
firewall-cmd  --add-port=53/udp    --permanent   #永久开启udp端口53
```
# 测试
```
nslookup   test.com   搭建的dns服务器ip
```
# 参考
CoreDNS篇1-CoreDNS简介和安装 - [知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/387806561)
coredns 添加hosts解析: [https://blog.csdn.net/u010533742/article/details/109641426](https://blog.csdn.net/u010533742/article/details/109641426)
coredns 配置[https://www.cnblogs.com/netonline/p/9953921.html](https://www.cnblogs.com/netonline/p/9953921.html)