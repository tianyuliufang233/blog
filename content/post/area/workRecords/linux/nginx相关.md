---
title: "Nginx相关"
date: 2025-06-05T14:18:30+08:00
categories:
- 收件箱
- 记录
tags:
- 阅读
- 记录
keywords:
- 记录
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
## 服务header存在下划线
解决方式：nginx http配置块中增加配置
```
underscores_in_headers on;
```

## traefik作为ingress时存在x-Forwarded-proto为https导致后端无法响应问题
解决方式：
```
apiVersion: traefik.containo.us/v1alpha1
kind: Middleware
metadata:
  name: forward-header
  namespace: default
  resourceVersion: '115158105'
spec:
  headers:
    customRequestHeaders:
      X-Forwarded-Proto: http
```