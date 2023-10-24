---
title: "Nacos_容器化快速部署"
date: 2023-10-24T10:34:44+08:00
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
需要nacos稳定高可用
# 部署环境
集群做高可用(3节点)，外部数据源  （若使用内部数据源需要开启embedded），目前有三种方案，一种是clb，一种是keepalive，一种nginx代理
|软件名称|版本|作用|插件|
|---|---|---|---|
|nacos|v2.0.3|注册中心，服务发现|暂无|
|centos|7.9|操作系统|暂无|
|docker|20.10.9|容器化提高安全及便捷性，方便各环境统一|暂无|
|mysql|8.0|持久化注册中心配置数据|暂无|

nacos端口及其作用：

|端口号|作用|
|---|---|
|7848|	用于节点选举来确定集群领袖（Leader）|
|8848|	对外暴露 API 与集群间数据同步（主端口）|
|9848|	客户端gRPC请求服务端端口，用于客户端向服务端发起连接和请求（此端口由主端口偏移1000计算得出）|
|9849|	服务端gRPC请求服务端端口，用于服务间同步等（此端口由主端口偏移1001计算得出）|

# 部署步骤
在云mysql新建nacos_conf_prod库，并导入数据如下。
```

# Copyright 1999-2018 Alibaba Group Holding Ltd.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
 

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info   */
/******************************************/
CREATE TABLE `config_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) DEFAULT NULL,
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  `c_desc` varchar(256) DEFAULT NULL,
  `c_use` varchar(64) DEFAULT NULL,
  `effect` varchar(64) DEFAULT NULL,
  `type` varchar(64) DEFAULT NULL,
  `c_schema` text,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfo_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_aggr   */
/******************************************/
CREATE TABLE `config_info_aggr` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(255) NOT NULL COMMENT 'group_id',
  `datum_id` varchar(255) NOT NULL COMMENT 'datum_id',
  `content` longtext NOT NULL COMMENT '内容',
  `gmt_modified` datetime NOT NULL COMMENT '修改时间',
  `app_name` varchar(128) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfoaggr_datagrouptenantdatum` (`data_id`,`group_id`,`tenant_id`,`datum_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='增加租户字段';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_beta   */
/******************************************/
CREATE TABLE `config_info_beta` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `beta_ips` varchar(1024) DEFAULT NULL COMMENT 'betaIps',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfobeta_datagrouptenant` (`data_id`,`group_id`,`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_beta';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_info_tag   */
/******************************************/
CREATE TABLE `config_info_tag` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `tag_id` varchar(128) NOT NULL COMMENT 'tag_id',
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL COMMENT 'content',
  `md5` varchar(32) DEFAULT NULL COMMENT 'md5',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  `src_user` text COMMENT 'source user',
  `src_ip` varchar(50) DEFAULT NULL COMMENT 'source ip',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_configinfotag_datagrouptenanttag` (`data_id`,`group_id`,`tenant_id`,`tag_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_info_tag';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = config_tags_relation   */
/******************************************/
CREATE TABLE `config_tags_relation` (
  `id` bigint(20) NOT NULL COMMENT 'id',
  `tag_name` varchar(128) NOT NULL COMMENT 'tag_name',
  `tag_type` varchar(64) DEFAULT NULL COMMENT 'tag_type',
  `data_id` varchar(255) NOT NULL COMMENT 'data_id',
  `group_id` varchar(128) NOT NULL COMMENT 'group_id',
  `tenant_id` varchar(128) DEFAULT '' COMMENT 'tenant_id',
  `nid` bigint(20) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`nid`),
  UNIQUE KEY `uk_configtagrelation_configidtag` (`id`,`tag_name`,`tag_type`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='config_tag_relation';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = group_capacity   */
/******************************************/
CREATE TABLE `group_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Group ID，空字符表示整个集群',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数，，0表示使用默认值',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='集群、各Group容量信息表';

/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = his_config_info   */
/******************************************/
CREATE TABLE `his_config_info` (
  `id` bigint(64) unsigned NOT NULL,
  `nid` bigint(20) unsigned NOT NULL AUTO_INCREMENT,
  `data_id` varchar(255) NOT NULL,
  `group_id` varchar(128) NOT NULL,
  `app_name` varchar(128) DEFAULT NULL COMMENT 'app_name',
  `content` longtext NOT NULL,
  `md5` varchar(32) DEFAULT NULL,
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `src_user` text,
  `src_ip` varchar(50) DEFAULT NULL,
  `op_type` char(10) DEFAULT NULL,
  `tenant_id` varchar(128) DEFAULT '' COMMENT '租户字段',
  PRIMARY KEY (`nid`),
  KEY `idx_gmt_create` (`gmt_create`),
  KEY `idx_gmt_modified` (`gmt_modified`),
  KEY `idx_did` (`data_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='多租户改造';


/******************************************/
/*   数据库全名 = nacos_config   */
/*   表名称 = tenant_capacity   */
/******************************************/
CREATE TABLE `tenant_capacity` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `tenant_id` varchar(128) NOT NULL DEFAULT '' COMMENT 'Tenant ID',
  `quota` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '配额，0表示使用默认值',
  `usage` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '使用量',
  `max_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个配置大小上限，单位为字节，0表示使用默认值',
  `max_aggr_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '聚合子配置最大个数',
  `max_aggr_size` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '单个聚合数据的子配置大小上限，单位为字节，0表示使用默认值',
  `max_history_count` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '最大变更历史数量',
  `gmt_create` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `gmt_modified` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='租户容量信息表';


CREATE TABLE `tenant_info` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'id',
  `kp` varchar(128) NOT NULL COMMENT 'kp',
  `tenant_id` varchar(128) default '' COMMENT 'tenant_id',
  `tenant_name` varchar(128) default '' COMMENT 'tenant_name',
  `tenant_desc` varchar(256) DEFAULT NULL COMMENT 'tenant_desc',
  `create_source` varchar(32) DEFAULT NULL COMMENT 'create_source',
  `gmt_create` bigint(20) NOT NULL COMMENT '创建时间',
  `gmt_modified` bigint(20) NOT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_tenant_info_kptenantid` (`kp`,`tenant_id`),
  KEY `idx_tenant_id` (`tenant_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='tenant_info';

CREATE TABLE `users` (
	`username` varchar(50) NOT NULL PRIMARY KEY,
	`password` varchar(500) NOT NULL,
	`enabled` boolean NOT NULL
);

CREATE TABLE `roles` (
	`username` varchar(50) NOT NULL,
	`role` varchar(50) NOT NULL,
	UNIQUE INDEX `idx_user_role` (`username` ASC, `role` ASC) USING BTREE
);

CREATE TABLE `permissions` (
    `role` varchar(50) NOT NULL,
    `resource` varchar(255) NOT NULL,
    `action` varchar(8) NOT NULL,
    UNIQUE INDEX `uk_role_permission` (`role`,`resource`,`action`) USING BTREE
);

INSERT INTO users (username, password, enabled) VALUES ('nacos', '$2a$10$EuWPZHzz32dJN7jexM34MOeYirDdFAZm2kuWj7VEOJhhZkDrxfvUu', TRUE);

INSERT INTO roles (username, role) VALUES ('nacos', 'ROLE_ADMIN');
```
分三台服务器，每个容器运行命令对应一台服务器，mysql使用云数据库实例
```
$ sudo docker run --name nacos1 -d -p 8849:8848 --restart=always -e JVM_XMS=256m -e JVM_XMX=256m -e MODE=cluster -e MEMBER_LIST={ip地址:ip端口} -e MYSQL_SERVICE_HOST={数据库地址} -e MYSQL_SERVICE_PORT={数据库端口} -e MYSQL_SERVICE_DB_NAME={数据库名} -e MYSQL_SERVICE_USER={数据库用户} -e MYSQL_SERVICE_PASSWORD={数据库密码} -e PREFER_HOST_MODE=node1 -e MEMBER_LIST=192.168.16.101:8847?raft_port=8807,192.168.16.101?raft_port=8808,192.168.16.101:8849?raft_port=8809 nacos/nacos-server:v2.0.3 
$ sudo docker run --name nacos2 -d -p 8849:8848 --restart=always -e JVM_XMS=256m -e JVM_XMX=256m -e MODE=cluster -e MEMBER_LIST={ip地址:ip端口} -e MYSQL_SERVICE_HOST={数据库地址} -e MYSQL_SERVICE_PORT={数据库端口} -e MYSQL_SERVICE_DB_NAME={数据库名} -e MYSQL_SERVICE_USER={数据库用户} -e MYSQL_SERVICE_PASSWORD={数据库密码} -e PREFER_HOST_MODE=node2 -e MEMBER_LIST=192.168.16.101:8847?raft_port=8807,192.168.16.101?raft_port=8808,192.168.16.101:8849?raft_port=8809 nacos/nacos-server:v2.0.3
$ sudo docker run --name nacos3 -d -p 8849:8848 --restart=always -e JVM_XMS=256m -e JVM_XMX=256m -e MODE=cluster -e MEMBER_LIST={ip地址:ip端口} -e MYSQL_SERVICE_HOST={数据库地址} -e MYSQL_SERVICE_PORT={数据库端口} -e MYSQL_SERVICE_DB_NAME={数据库名} -e MYSQL_SERVICE_USER={数据库用户} -e MYSQL_SERVICE_PASSWORD={数据库密码} -e PREFER_HOST_MODE=node3 -e MEMBER_LIST=192.168.16.101:8847?raft_port=8807,192.168.16.101?raft_port=8808,192.168.16.101:8849?raft_port=8809 nacos/nacos-server:v2.0.3
```
nacos1-docker-compose.yml内容如下
```
version: '3'
services:
  web:
    image: nacos/nacos-server:v2.0.3
    restart: always
    hostname: nacos1
    container_name: nacos-1
    ports:
      - "7848:7848"
      - "8848:8848"
      - "9848:9848"
      - "9849:9849"
    volumes:
      - /data/apps/nacos/conf.d/custom.properties:/home/nacos/init.d/custom.properties
      #- /data/data/nacos/logs:/home/nacos/logs
    environment:
      - MODE=cluster
      - PREFER_HOST_MODE=hostname
      - MYSQL_SERVICE_HOST=192.168.1.1
      - MYSQL_SERVICE_DB_NAME=nacos_conf_prod
      - MYSQL_SERVICE_USER=username
      - MYSQL_SERVICE_PASSWORD=123456
      - MYSQL_SERVICE_PORT=13306
      - NACOS_SERVERS=nacos1:8848 nacos2:8848 nacos3:8848
    extra_hosts:
      - "nacos2:192.168.1.2"
      - "nacos3:192.168.1.3"
    networks:
      - nacos
networks:
  nacos:
    driver: bridge
```



nacos2-docker-compose.yml内容如下
```
version: '3'
services:
  web:
    image: nacos/nacos-server:v2.0.3
    restart: always
    hostname: nacos2
    container_name: nacos-2
    ports:
      - "7848:7848"
      - "8848:8848"
      - "9848:9848"
      - "9849:9849"
    volumes:
      - /data/apps/nacos/conf.d/custom.properties:/home/nacos/init.d/custom.properties
      #- /data/data/nacos/logs:/home/nacos/logs
    environment:
      - MODE=cluster
      - PREFER_HOST_MODE=hostname
      - MYSQL_SERVICE_HOST=192.168.1.1
      - MYSQL_SERVICE_DB_NAME=nacos_conf_prod
      - MYSQL_SERVICE_USER=username
      - MYSQL_SERVICE_PASSWORD=123456
      - MYSQL_SERVICE_PORT=13306
      - NACOS_SERVERS=nacos1:8848 nacos2:8848 nacos3:8848
    extra_hosts:
      - "nacos1:172.29.2.10"
      - "nacos3:192.168.1.3"
    networks:
      - nacos
networks:
  nacos:
    driver: bridge
```
nacos3-docker-compose.yml内容如下
```
version: '3'
services:
  web:
    image: nacos/nacos-server:v2.0.3
    restart: always
    hostname: nacos3
    container_name: nacos-3
    ports:
      - "7848:7848"
      - "8848:8848"
      - "9848:9848"
      - "9849:9849"
    volumes:
      - /data/apps/nacos/conf.d/custom.properties:/home/nacos/init.d/custom.properties
      #- /data/data/nacos/logs:/home/nacos/logs
    environment:
      - MODE=cluster
      - PREFER_HOST_MODE=hostname
      - MYSQL_SERVICE_HOST=192.168.1.1
      - MYSQL_SERVICE_DB_NAME=nacos_conf_prod
      - MYSQL_SERVICE_USER=username
      - MYSQL_SERVICE_PASSWORD=123456
      - MYSQL_SERVICE_PORT=13306
      - NACOS_SERVERS=nacos1:8848 nacos2:8848 nacos3:8848
    extra_hosts:
      - "nacos1:172.29.2.10"
      - "nacos2:192.168.1.2"
    networks:
      - nacos
networks:
  nacos:
    driver: bridge
```                 

# 部署测试验证

打开nacos控制台
```
 $ curl -X GET "http://127.0.0.1:8848/nacos"
```



# 注意事项
如果存在防火墙或者nginx端口转发问题，需要进行相应的端口暴露配置。如在nginx中，在已经暴露8848(x)的基础上，需要额外暴露9848（x+1000)。
# 参考链接
nacos官方集群文档：[https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html](https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html)
dockerhub [nacos说明](https://registry.hub.docker.com/_/rabbitmq/)
nacos mysql初始化sql链接：https://github.com/alibaba/nacos/blob/master/distribution/conf/nacos-mysql.sql
启动后，调用openAPI 报错 [code:503,msg:server is DOWN now, please try again later!](https://nacos.io/zh-cn/docs/2.0.0-compatibility.html)
nacos集群[docker暴露端口参考](https://nacos.io/zh-cn/docs/2.0.0-compatibility.html)