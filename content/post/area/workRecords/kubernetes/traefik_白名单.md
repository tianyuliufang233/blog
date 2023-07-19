---
title: "Traefik_白名单"
date: 2023-10-24T09:59:52+08:00
categories:
- 收件箱
- kubernetes相关
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
```
traefik.ingress.kubernetes.io/whitelist-source-range: "1.2.3.0/24, fe80::/16"  #ingress 白名单
traefik.ingress.kubernetes.io/router.entrypoints: 'websecure, web'  #开启https及http
traefik.ingress.kubernetes.io/router.middlewares: default-https-redirect-scheme@kubernetescrd   #http重定向到https
```