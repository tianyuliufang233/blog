---
title: "Elasticsearch_容器化快速搭建"
date: 2023-10-24T10:08:40+08:00
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

# 部署背景
待后续补充
# 部署环境
| 软件名称 | 版本 | 插件 | 版本 |
|---|---|---|---|
|elasticsearch|	7.15.2|	默认|暂无|
|centos|	7.9|	暂无|	暂无|
|docker|	20.10.9|	默认|	暂无|

# 部署步骤

1. es通过docker-compose.yaml部署，docker-compose.yaml位于/data/apps/elasticsearch目录下，内容如下：
```
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.2
    restart: always
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - xpack.security.enabled=true
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms2048m -Xmx2048m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - /data/data/elasticsearch:/usr/share/elasticsearch/data
    ports:
      - 19200:9200
    networks:
      - elastic

networks:
  elastic:
    driver: bridge
```
2. 启停命令为：
```
#  sudo docker-compose -f /data/apps/elasticsearch/docker-compose.yaml up -d  #启动
#  curl -XGET 127.0.0.1:19200/_cat/aliases?v  #验证集群状态
# 以下命令为需要停止容器的时候使用：
# sudo docker-compose -f /data/apps/elasticsearch/docker-compose.yaml stop #停止
# sudo docker-compose -f /data/apps/elasticsearch/docker-compose.yaml down -v 删除容器及卷
```
3. 配置内部账号elastic密码：
```
# sudo /usr/local/bin/docker-compose -f /data/apps/elasticsearch/docker-compose.yaml exec es01 /bin/bash  #进入容器bash终端
容器内执行如下命令配置密码：
# elasticsearch-setup-passwords interactive
# 退出容器
# exit 
```
4. 权限配置-添加role：
```
put 127.0.0.1:19202/_security/role/approle
{
        "cluster": [
            "all"
        ],
        "indices": [
            {
                "names": [
                    "*"
                ],
                "privileges": [
                    "all"
                ],
                "allow_restricted_indices": false
            }
        ]
}
```
5. 添加只读角色
```
put /_security/role/applogrole
{
        "cluster": [
            "monitor",
            "read_ccr"
        ],
        "indices": [
            {
                "names": [
                    "*"
                ],
                "privileges": [
                    "read",
                    "view_index_metadata",
                    "read_cross_cluster",
                    "monitor"
                ],
                "allow_restricted_indices": false
            }
        ]
}
```
6. 添加角色：
```
put 127.0.0.1:19200/_security/user/username
{
    "password": "123456",
    "roles":"approle",
    "full_name": "username"
}
```
7. 添加快照备份
```
#添加快照策略
PUT http://127.0.0.1:19200/_snapshot/es_master_cos_backup
{
    "type": "cos",
    "settings": {
        "access_key_id": "kid",
        "bucket": "test",
        "chunk_size": "500mb",
        "compress": "true",
        "access_key_secret": "kst",
        "base_path": "es_hospital_master_cos_backup",
        "region": "ap-chengdu"
    }
}
#添加周期快照策略
PUT http://127.0.0.1:19200/_slm/policy/daily-snapshots
{
  "schedule": "0 30 1 * * ?", 
  "name": "<snapshot-{now/d}>", 
  "repository": "es_master_cos_backup", 
  "retention": { 
    "expire_after": "7d", 
    "min_count": 7, 
    "max_count": 14
  }
}
#获取周期备份策略
GET http://127.0.0.1:19200/_slm/policy/daily-snapshots/
#获取快照策略
GET http://127.0.0.1:19200/_snapshot/_all
#执行周期备份策略
PUT http://127.0.0.1:19200/_slm/policy/daily-snapshots/_execute
{
  "snapshot_name": "daily-snap-2023.08.31-gwrqoo2xtea3q57vvg0uea"
}
```
# 部署测试验证
```
# 验证方式：
curl --user 'elastic:xxxx' -XGET '127.0.0.1:19200/_cat/aliases?v'
```
# 注意事项
java虚拟机内存最好设置为需求内存的一半，另一半留给系统做缓存使用以便于提高速度。
服务器内存配置较低时，务必开启bootstrap.memory_lock=true锁定内存，避免oom。
# 参考链接