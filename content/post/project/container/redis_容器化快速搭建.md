---
title: "Redis_容器化快速搭建"
date: 2023-10-24T09:47:01+08:00
categories:
- 项目
- 容器相关
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
```
version: '3.1'
 
services:
  redis-test-2:
    image: redis
    restart: always
    hostname: redis-test-2
    ports:
      - 52002:6379
    ulimits:
      nproc: 65535
      nofile:
        soft: 20480
        hard: 40960
    logging:
      driver: "json-file"
      options:
        max-size: "1g"
        max-file: "5"
    volumes:
      - /data/apps/redis-test-2/conf/redis.conf:/etc/redis/redis.conf
      - /data/data/redis-test-2/data:/data
    network_mode: bridge
    command: redis-server /etc/redis/redis.conf --aof-use-rdb-preamble yes
    #networks:
    # - test
#networks:
#  test:
#    driver: host
```
redis.conf
```
#daemonize yes 否则无法启动容器
#密码
requirepass password-string
#开启aof持久化
appendonly yes
appendfsync always
auto-aof-rewrite-min-size 64mb
```