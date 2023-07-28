---
title: "Kubernetes 常用命令案例"
date: 2023-07-28T10:16:40+08:00
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
常用命令案例
<!--more-->
## kubernetes 进行替换 configMap
```
kubectl create configmap --namespace default --kubeconfig=config test.config  -o yaml 
--dry-run=client --from-file=test.yaml |kubectl replace --kubeconfig=config -f -
# 思路为调试生成yaml，然后将yaml替换kubernetes configmap对象
```
## kubernetes deployment 滚动更新
```
kubectl rollout restart deployment appname -n default --kubeconfig /test_kubeconfig
```
## kubernetes 只修改镜像
```
kubectl set image deployment/containerName deploymentName=Repository/imageName:xxxxx -n namespace --kubeconfig kubeconfig
```
