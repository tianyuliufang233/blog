---
title: "Kubernetes Pod Grpc测试"
date: 2025-06-05T12:51:24+08:00
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
```
apiVersion: v1
kind: Pod
metadata:
  name: test-grpc
spec:
  containers:
    - name: agnhost
      # 镜像自发布以来已更改（以前使用的仓库为 "k8s.gcr.io"）
      image: registry.k8s.io/e2e-test-images/agnhost:2.35
      command: ["/agnhost", "grpc-health-checking"]
      ports:
        - containerPort: 5000
        - containerPort: 8080
      readinessProbe:
        grpc:
          port: 5000
```
