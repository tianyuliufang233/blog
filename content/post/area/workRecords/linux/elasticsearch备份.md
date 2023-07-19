---
title: "Elasticsearch备份"
date: 2024-06-06T17:33:15+08:00
categories:
- 收件箱
- linxu相关
tags:
- 阅读
- 记录
keywords:
- 记录
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

```
GET http://127.0.0.1:9200/system_message/_search
content-type: application/json
Authorization: Basic ***************
 
{
  "query": {
    "match_all": {}
  },
  "sort":[
    {
        "createTime": {
            "order": "desc"
        }
    }
  ]
}
 
GET http://127.0.0.1:9200/message_template/_search
content-type: application/json
Authorization: Basic ***************
 
{
  "query": {
    "match_all": {}
  }
}
 
GET http://127.0.0.1:9200/_cat/indices
content-type: application/json
Authorization: Basic ***************

GET http://127.0.0.1:9200/_cat/indices
content-type: application/json
Authorization: Basic ***************
 
GET http://127.0.0.1:9200/_slm/policy
content-type: application/json
Authorization: Basic ***************
 
GET http://127.0.0.1:9200/_snapshot
content-type: application/json
Authorization: Basic ***************
 
GET http://127.0.0.1:9200/_slm/stats
content-type: application/json
Authorization: Basic ***************
 
GET http://127.0.0.1:9200/_snapshot/sync_backup/*
content-type: application/json
Authorization: Basic ***************
 
GET http://127.0.0.1:9200/_snapshot/es_master_cos_backup/*
content-type: application/json
Authorization: Basic ***************
 
PUT http://127.0.0.1:9200/_snapshot/sync_backup
content-type: application/json
Authorization: Basic ***************
 
{
        "type": "cos",
        "settings": {
            "access_key_id": "********************************************",
            "bucket": "harbor",
            "chunk_size": "500mb",
            "compress": "true",
            "access_key_secret": "***********************************",
            "base_path": "sync_backup",
            "region": "ap-nanjin"
        }
    }
 
PUT http://127.0.0.1:9200/_snapshot/sync_backup/snapshot_9
content-type: application/json
Authorization: Basic ***************
 
{
    "indices": "system,message"
}
 
PUT http://127.0.0.1:9200/_snapshot/es_master_cos_backup/snapshot_9
content-type: application/json
Authorization: Basic ***************
 
{
    "indices": "system,message"
}
 
DELETE http://127.0.0.1:9200/_snapshot/sync_backup/snapshot_6
content-type: application/json
Authorization: Basic ***************
 
DELETE http://127.0.0.1:9200/_snapshot/sync_backup
content-type: application/json
Authorization: Basic ***************
 
 
 
#########################################################
 
 
#########################################################
 
 
GET http://127.0.0.1:9200/_snapshot
content-type: application/json
Authorization: Basic ***************
 
PUT http://127.0.0.1:9200/_snapshot/sync_backup
content-type: application/json
Authorization: Basic ***************
 
{
        "type": "cos",
        "settings": {
            "access_key_id": "*********************************",
            "bucket": "harbor",
            "chunk_size": "500mb",
            "compress": "true",
            "access_key_secret": "******************************************",
            "base_path": "sync_backup",
            "region": "ap-nanjin"
        }
    }
 
GET http://127.0.0.1:9200/_snapshot/sync_backup/*
content-type: application/json
Authorization: Basic ***************
 
PUT http://127.0.0.1:9200/_snapshot/sync_backup/snapshot_6
content-type: application/json
Authorization: Basic ***************
 
{
    "indices": "system,message"
}
 
PUT http://127.0.0.1:9200/_snapshot/es_master_cos_backup/snapshot_6
content-type: application/json
Authorization: Basic ***************
 
{
    "indices": "system_message,message_template"
}
 
GET http://127.0.0.1:9200/_cat/indices
content-type: application/json
Authorization: Basic ***************
 
DELETE http://127.0.0.1:9200/_snapshot/sync_backup/snapshot_6
content-type: application/json
Authorization: Basic ***************
 
DELETE http://127.0.0.1:9200/_snapshot/sync_backup
content-type: application/json
Authorization: Basic ***************
 
GET http://127.0.0.1:9200/system_message/_search
content-type: application/json
Authorization: Basic ***************
 
{
  "query": {
    "match_all": {}
  },
  "sort":[
    {
        "updateTime": {
            "order": "asc"
        }
    }
  ]
}
 
PUT http://127.0.0.1:9200/_snapshot/sync_backup
content-type: application/json
Authorization: Basic ***************
 
{
        "type": "cos",
        "settings": {
            "access_key_id": "**************************************************",
            "bucket": "harbor",
            "chunk_size": "500mb",
            "compress": "true",
            "access_key_secret": "******************************",
            "base_path": "sync_backup",
            "region": "ap-nanjin"
        }
    }
 
GET http://127.0.0.1:9200/_cat/indices
content-type: application/json
Authorization: Basic ***************
 
DELETE http://127.0.0.1:9200/system_message
content-type: application/json
Authorization: Basic ***************
 
DELETE http://127.0.0.1:9200/message_template
content-type: application/json
Authorization: Basic ***************
 
GET http://127.0.0.1:9200/_snapshot
content-type: application/json
Authorization: Basic ***************
 
GET http://127.0.0.1:9200/_snapshot/sync_backup/*
content-type: application/json
Authorization: Basic ***************
 
POST http://127.0.0.1:9200/_snapshot/sync_backup/snapshot_6/_restore
content-type: application/json
Authorization: Basic ***************
 
{
    "indices": "system,message"
}
 
GET http://127.0.0.1:9200/_cat/indices
content-type: application/json
Authorization: Basic ***************
```