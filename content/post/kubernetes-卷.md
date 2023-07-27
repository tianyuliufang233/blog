---
title: "Kubernetes 卷"
date: 2023-07-27T11:58:09+08:00
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

## cephfs
允许将现有的fs挂载到pod中。在pod被删除时被保留，只是被卷卸载。
## configMap
提供了向pod注入配置数据的方法。configMap对象可以被挂载为卷。
```
apiVersion: v1
kind: Pod
metadata:
  name: configmap-pod
spec:
  containers:
    - name: test
      image: busybox:1.28
      volumeMounts:
        - name: config-vol
          mountPath: /etc/config
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level
```
## downloadAPI
在这类卷中，所公开的数据以纯文本格式的只读文件形式存在。
说明： 容器以 subPath 卷挂载方式使用 downward API 时，在字段值更改时将不能接收到它的更新。
## emptyDir
emptyDir会随pod创建和删除。容器崩溃不好导致pod被从节点上移除。
emptyDir的一些用途：
  - 缓存空间
  - 耗时较长的计算任务提供检查点，以便任务能方便地从崩溃前状态恢复执行。
  - 在web服务器容器服务数据时，保存内容管理器容器获取的文件。 

emptyDir.medium字段用来控制emptyDir卷的存储位置。



参考连接[kubernetes-volumes]("https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#configmap")