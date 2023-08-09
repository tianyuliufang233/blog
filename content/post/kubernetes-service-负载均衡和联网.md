---
title: "Kubernetes Service 负载均衡和联网"
date: 2023-08-09T16:38:13+08:00
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
service 负载均衡和联网
<!--more-->
# kubernetes网络模型
从端口分分配、命名、服务发现、负载均衡、应用配置和迁移的角度看，Pod可以被视为虚拟机或者物理主机。
- Pod要能与其他节点上的Pod通信，且不需要网络地址转译
- 节点上的代理(系统守护进程、kubelet)可以和节点上的所有Pod通信。
kubernetes 网络解决四方面问题：
- 一个Pod中的容器之间通过本地回路(loopback)通信
- 集群网络在不同Pod之间提供通信。
- service api 运行你向外暴露Pod中运行的应用，以支持来自于集群外部的访问。
  - ingress提供专门用于暴露HTTP应用程序、网站和API的额外功能。
- 可以使用Service来发布仅供集群内部使用的服务
## service
kubernetes中Service是将运行在一个或一组Pod上的网络应用程序公开为网络服务的方法。service定义的抽象能够解耦关联。service由Pod定义的选择符来确定。也会有不带选择符的service。ingress将路由规则整合到单个资源。

云原生服务发现是通过kubernetes api进行服务发现，可以查询API服务器用于匹配EndpointSlices。只要Pod发生变化，Kubernetes会更新服务EndpointSlices。

定义service：
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app.kubernetes.io/name: proxy
spec:
  containers:
  - name: nginx
    image: nginx:stable
    ports:
      - containerPort: 80
        name: http-web-svc
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app.kubernetes.io/name: proxy
  ports:
  - name: name-of-service-port
    protocol: TCP
    port: 80
    targetPort: http-web-svc
```
### EndpointSlices
EndpointSlices对象表示针对服务的后端网络端点的子集(切片)。当EndpointSlice表示的端点数量达到阈值时，kubernetes会添加另一个空的EndpointSlice并在其中存储新的端点信息。默认100个。
### Endpoints
该资源定义了网络端点的列表，通常由service引用，以定义可以将流量发送到哪些Pod。
kubernetes限制单个Endpoints对象中可以容纳的端点数量。当一个service配置超过1000个后备端点时，kubernetes会截断EndpointSlice。appProtocol字段提供了一种为每个Service端口指定应用协议的方式。
### 多端口Service
对于某些服务，需要开多个端口
```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```
### 发布服务（服务类型）
可能希望将其暴露给kubernetes集群外部的IP地址。

## ingress
## ingress control
## EndpointSlice
## 网络策略
## service与Pod的DNS
## IPv4/IPv6双协议栈
## 拓扑感知路由
## windows网络
## Service ClusterIP分配
## 服务内部流量策略
