---
title: "Mongodb数据迁移"
date: 2023-10-24T10:31:20+08:00
categories:
- 工作
- 记录
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
```
sudo docker exec -i mongo-master mongodump -u root -p passwod --authenticationDatabase=admin --db=testdb -o /data/db/mongo-backup-dump_testdb
sudo docker exec -i mongo-master mongorestore --host 127.0.0.1:27017 -u mongouser -p password --authenticationDatabase=admin --db=testdb --dir=/data/db/mongo-backup-dump_testdbch
```
针对库建user
```
use('admin');
db.auth('root','xxxxxxx')
db.createUser({user:"piller_chatgpt",pwd:'bbbbbbbbb',
roles:[{role:"readWrite",db:"piller_chatgpt"}]
})
```