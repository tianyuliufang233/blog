---
title: "Hyper V无法启动虚拟机"
date: 2023-08-16T18:26:13+08:00
categories:
- category
- subcategory
tags:
- tag1
- tag2
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
## 表现
首先为无法连接虚拟机管理服务，后在服务里面找到hv后台主机服务和虚拟机管理服务，启动后可以连接管理服务，但无法启动虚拟机。biso中已开启虚拟化，最后用下方命令解决。


## 修复
```
bcdedit /set hypervisorlaunchtype auto
```