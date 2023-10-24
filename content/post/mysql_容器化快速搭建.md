---
title: "Mysql_容器化快速搭建"
date: 2023-10-24T09:56:39+08:00
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
```
version: '3'
services:
  dev-mysql:
    image: mysql:8.0.23
    restart: always
    network_mode: bridge
    container_name: dev-mysql
    logging:
      driver: "json-file"
      options:
        max-size: "1g"
        max-file: "5"
    ulimits:
      nofile: 409600
      nproc: 409600
    environment:
      MYSQL_ROOT_PASSWORD: 'dFpUmyaaaa_bbbb'
    volumes:
      - "/data/apps/dev_mysql/conf/my.cnf:/etc/mysql/my.cnf"
      - "/data/data/dev_mysql/data:/var/lib/mysql"
    ports:
      - 13308:3306
```
my.cnf
```
# Copyright (c) 2017, Oracle and/or its affiliates. All rights reserved.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; version 2 of the License.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301 USA
 
#
# The MySQL  Server configuration file.
#
# For explanations see
# http://dev.mysql.com/doc/mysql/en/server-system-variables.html
 
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
secure-file-priv= /var/lib/mysql
ssl_ca=/var/lib/mysql/ca.pem
ssl_cert=/var/lib/mysql/server-cert.pem
ssl_key=/var/lib/mysql/server-key.pem
lower_case_table_names=1
 
# Custom config should go here
!includedir /etc/mysql/conf.d/
```