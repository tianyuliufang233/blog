---
title: "Podman添加国内源"
date: 2024-09-29T18:13:57+08:00
categories:
- 领域
- 容器相关
tags:
- 阅读
- 记录
keywords:
- 记录
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

## 修改方式
使用vi或者其他文本修改工具修改配置文件/etc/containers/registries.conf，修改前最好备份一下
```
cp /etc/containers/registries.conf /etc/containers/registries.conf.bak
vi /etc/containers/registries.conf
```
删除或者用#注释掉registries.conf文件的全部内容。  
把下面内容粘贴进去，保存文件
```
unqualified-search-registries = ["docker.io"]
[[registry]]
location = "docker.io"
[[registry.mirror]]
location = "docker.mirrors.ustc.edu.cn"
insecure = true
```
但前提是需要国内源可用。  