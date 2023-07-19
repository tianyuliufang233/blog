---
title: "Kubernetes 常用命令案例"
date: 2023-07-28T10:16:40+08:00
categories:
- 收件箱
- kubernetes相关
tags:
- kubernetes
- 记录
keywords:
- 案例
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
## kubernetes 内存算法
```
#!/bin/bash
#!/usr/bin/env bash

# This script reproduces what the kubelet does
# to calculate memory.available relative to root cgroup.

# current memory usage
memory_capacity_in_kb=$(cat /proc/meminfo | grep MemTotal | awk '{print $2}')
memory_capacity_in_bytes=$((memory_capacity_in_kb * 1024))
memory_usage_in_bytes=$(cat /sys/fs/cgroup/memory/memory.usage_in_bytes)
memory_total_inactive_file=$(cat /sys/fs/cgroup/memory/memory.stat | grep total_inactive_file | awk '{print $2}')

memory_working_set=${memory_usage_in_bytes}
if [ "$memory_working_set" -lt "$memory_total_inactive_file" ];
then
    memory_working_set=0
else
    memory_working_set=$((memory_usage_in_bytes - memory_total_inactive_file))
fi

memory_available_in_bytes=$((memory_capacity_in_bytes - memory_working_set))
memory_available_in_kb=$((memory_available_in_bytes / 1024))
memory_available_in_mb=$((memory_available_in_kb / 1024))

echo "memory.capacity_in_bytes $memory_capacity_in_bytes"
echo "memory.usage_in_bytes $memory_usage_in_bytes"
echo "memory.total_inactive_file $memory_total_inactive_file"
echo "memory.working_set $memory_working_set"
echo "memory.available_in_bytes $memory_available_in_bytes"
echo "memory.available_in_kb $memory_available_in_kb"
echo "memory.available_in_mb $memory_available_in_mb"
```