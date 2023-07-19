---
title: "Mysql8.0锁表问题排查"
date: 2024-01-16T16:15:27+08:00
categories:
- 收件箱
- linxu相关
tags:
- 工作
- 记录
keywords:
- 案例
#thumbnailImage: //example.com/image.jpg
---

<!--more-->

## 查看连接状态以及sql执行状态
show processlist;
## 查看所有事务
select * from information_schema.innodb_trx;
## 查看所有数据锁
select * from `performance_schema`.data_locks;
## 查看所有数据等待
select * from `performance_schema`.data_lock_waits;
## 查看最近死锁的日志
show engine innodb status

## 其他关于锁的状态
show status like 'Table%'
show OPEN TABLES where In_use > 0;
show status like 'innodb_row_lock_%';

## 最暴力的解除锁的方式
kill 线程id