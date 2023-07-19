---
title: "Rabbitmq镜像集群"
date: 2025-06-05T13:09:04+08:00
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
### 运行容器
docker run --restart=always -d --hostname prod-rabbitmq-node1 --name prod-rabbitmq-node1 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=a@2021 -p 52004:15672 -p 52003:5672 rabbitmq:management
docker run --restart=always -d --hostname prod-rabbitmq-node2 --name prod-rabbitmq-node2 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=a@2021 -p 52004:15672 -p 52003:5672 rabbitmq:management
docker run --restart=always -d --hostname prod-rabbitmq-node3 --name prod-rabbitmq-node3 -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=a@2021 -p 52004:15672 -p 52003:5672 rabbitmq:management
 
### 添加插件和激活延迟队列插件
docker cp /data/volumes/release-rabbitmq/rabbitmq_delayed_message_exchange-3.9.0.ez prod-rabbitmq-node1:/plugins
docker exec -it release-rabbitmq /bin/bash
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
docker restart release-rabbitmq
 
修改ulimit仅作参考：
docker daemon.json
{
  "default-ulimits": {
    "nofile": {
      "Name": "nofile",
      "Hard": 64000,
      "Soft": 64000
    }
  }
}
```
查看集群状态
```
# 分别在rabbit1、2、3执行如下命令查看集群状态
rabbitmqctl cluster_status
 
# 分别在 rabbit2、3上执行以下命令加入集群
rabbitmqctl stop_app
# => Stopping node rabbit@rabbit2 ...done.
 
rabbitmqctl reset
# => Resetting node rabbit@rabbit2 ...
 
rabbitmqctl join_cluster rabbit@rabbit1
# => Clustering node rabbit@rabbit2 with [rabbit@rabbit1] ...done.
 
rabbitmqctl start_app
# => Starting node rabbit@rabbit2 ...done.
```
rabbitmq-1-docker-compose.yaml
```
version: '3'
services:
  rabbitmq:
    image: rabbitmq-delayed-message:management
    restart: always
    hostname: rabbitmq1
    container_name: rabbitmq-1
    ports:
      - 4369:4369
      - 5671:5671
      - 15671:15671
      - 15672:15672
      - 5672:5672
      - 1883:1883
      - 25672:25672
    extra_hosts:
      - "rabbitmq2:172.29.2.11"
      - "rabbitmq3:172.29.2.6"
    networks:
      - rabbitmq
networks:
  rabbitmq:
    driver: bridge
```
部署测试验证
注意事项
注意插件权限，如果安装插件重启后任然为E状态，不是E*状态，很可能为插件ez文件缺少读取权限所致。
使用docker安装注意ulimits  
参考链接
[rabbitmq官方文档：集群对等发现](https://www.rabbitmq.com/cluster-formation.html)
[rabbitmq官方文档：仲裁队列描述](https://www.rabbitmq.com/quorum-queues.html#feature-comparison)
[rabbitmq官方文档：经典镜像队列描述](https://www.rabbitmq.com/pacemaker.html)
[rabbitmq-delayed-message-exchange插件说明](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange)
[rabbitmq官方文档：环境变量及注意](https://www.rabbitmq.com/configure.html#supported-environment-variables)
[rabbitmq官方文档：生产环境参考清单](https://www.rabbitmq.com/production-checklist.html)
[rabbitmq官方文档：网络优化参考](https://www.rabbitmq.com/networking.html)
