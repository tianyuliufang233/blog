---
title: "Docker部署"
date: 2025-06-05T14:09:29+08:00
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

# 部署背景
服务均基于docker运行
# 部署环境
|||||
|---|---|---|---|
|centos|	7.9	|暂无	|暂无|
|docker|	20.10.9|	默认|	暂无|
# 部署步骤
移除自带的docker：
```
$ sudo yum remove docker docker-client docker-client-latest docker-common docker-latest docker-latest-logrotate docker-logrotate docker-engine
```
添加docker官方软件源：
```
$ sudo yum install -y yum-utils
$ sudo yum-config-manager \
   --add-repo \
   https://download.docker.com/linux/centos/docker-ce.repo
```
安装docker：  
sudo yum install docker-ce docker-ce-cli containerd.io  
或使用  sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io #指定版本  

添加docker 默认配置到daemon.json：
```
{
    "data-root": "/data/data/docker",
    "storage-driver": "overlay2",
    "live-restore": true, #重启docker守护进程会影响网络和输入，但不会重启容器。可用于升级
    "log-driver": "json-file",  #docker 有fluentd日志驱动，有日志系统需求可以研究下。
    "log-opts": {
        "max-size": "1g", #单个日志文件限制
        "max-file": "10",  # 日志文件数量限制
    },
  "registry-mirrors": [
         "https://mirror.ccs.tencentyun.com"
    ],
     "default-address-pools" : [  # 配置桥接网络池
       {
            "base" : "172.18.0.0/16", #不能包含宿主机网段
            "size" : 24
       }
    ]   
    # "dns": ["10.20.1.2","10.20.1.3"] #dns视情况可添加
    # "registry-mirrors": ["https//registry.docker-cn.com"，"https://registry.docker-cn.com","http://hub-mirror.c.163.com","https://docker.mirrors.ustc.edu.cn"] #此项视情况设置image源
}
```
docker容器管理方式预计使用docker-compose
docker-compose 在未配置default-address-pools时，可以通过先创建network然后附加到现有bridge的方式预防创建的bridge与主机网络在同一个网段
```
$ docker network create --driver=bridge --subnet=172.18.15.0/16 --ip-range=172.18.15.0/24  prodnet  可
```
创建指定ip的网段
```
$ vim docker-compose.yaml
version: '3'
services:
  redis:
    image: redis
    container_name: docker_redis
    #networks:
    #  - test
    environment:
      - LANG=C.UTF-8
      - TZ=Asia/Shanghai
   ports:
      - 6379:6379
    network_mode: host
    logging:
      driver: "json-file"
      options:
        max-size: "1g"
        max-file: "3"
    ulimits:
      memlock:
        soft: -1
        hard: -1
      nofile: 409600
      nproc: 409600
    extra_hosts:
      mm-pay-rpc-service: 172.29.2.13
 #networks:
  #test:
    #external: true
    #name: prodnet
```
可在docker的daemon.json添加端口远程管理：
```
{
   "hosts": [
       "tcp://127.0.0.1:2375",
       "unix:///run/containerd/containerd.sock"
   ]
}
```
/etc/systemd/system/docker.service.d/docker.conf
```
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```
# 部署测试验证
验证方式：docker info 可以查看是否应用配置

# 注意事项
- 存储方面官方推荐使用卷，如果要挂载块设备，可以指定卷驱动程序，绑定挂载性能很高，绑定挂载可能存在在容器中修改宿主机的系统文件或目录的隐患。
- 非root模式与userns-remap模式相似，在容器中的root用户对应的是宿主机上运行dockerd的用户，降低整机风险，可考虑使用非root用户运行dockerd，注：官方推荐使用ubuntu内核或5.11版本。
- docker-compose最好设置下时区和编码。    environment:       - LANG=C.UTF-8     - TZ=Asia/Shanghai

# 参考链接
[非root模式运行dockerd限制](https://docs.docker.com/engine/security/rootless/#prerequisites)
[官方文档docker安装](https://docs.docker.com/engine/install/centos/)
[官方文档卷描述](https://docs.docker.com/storage/volumes/)
[官方文档生产环境安全描述](https://docs.docker.com/engine/security/)
[官方文档日志驱动相关](https://docs.docker.com/config/containers/logging/configure/)
[官方文档prometheus相关](https://docs.docker.com/config/daemon/prometheus/)
[官方文档网络概述](https://docs.docker.com/network/)

