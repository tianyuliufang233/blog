---
title: "Interface MTU"
date: 2023-08-25T11:31:55+08:00
categories:
- 收件箱
- linxu相关
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
# windows 查看interface 
```
netsh interface ipv4 show subinterfaces 
```
## 修改interface mtu
```
netsh interface ipv4 set subinterface " 连接名" mtu=1340 store=persistent 
```

