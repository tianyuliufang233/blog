---
title: "Npm_registry_verdaccio_容器化快速搭建"
date: 2023-10-24T09:49:24+08:00
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
docker-compose.yaml
```
version: '3.1'
services:
  verdaccio:
    image: verdaccio/verdaccio
    container_name: "verdaccio"
    network_mode: host
    environment:
      - VERDACCIO_PORT=4873               #配置端口
      - VERDACCIO_PUBLIC_URL='https://npm.aaa.com'  #设置访问域名
    volumes:
      - "/data/data/verdaccio/storage:/verdaccio/storage"
      - "/data/apps/verdaccio/config:/verdaccio/conf"
      - "/data/data/verdaccio/plugins:/verdaccio/plugins"
```
config.yaml配置文件如下
```
storage: ./storage
auth:
  htpasswd:
    file: ./htpasswd
uplinks:
  npmjs:
    url: https://registry.npmmirror.com/
packages:
  "@*/*":
    access: $all
    publish: $authenticated
    proxy: npmjs
  "**":
    proxy: npmjs
i18n:
  web: zh-CN
web:
  enable: true
  title: 前端npm
  logo: http://static-img-barrel-1306477242.cos.ap-chengdu.myqcloud.com/icon/logo.png
  scope: "@piller"
  favicon: http://doman.com/browser_favicon.ico
  primary_color: "#409EFF"
  showInfo: false
  showFooter: false
listen: 0.0.0.0:4000
log: { type: stdout, format: pretty, level: http }
```
nginx代理配置如下：
```
server {
    listen       80;
    server_name  npm.aaa.com;
    access_log  /data/logs/nginx/npm.aaa.com.access.log;
    error_log  /data/logs/nginx/npm.aaa.com.error.log;
 
    listen              443 ssl;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ssl_certificate     /etc/nginx/ssl/aaa.com_bundle.crt;
    ssl_certificate_key /etc/nginx/ssl/aaa.com.key;
 
    location / {
        proxy_buffer_size 64k;
        proxy_buffering on;
        proxy_buffers   4 32k;
        proxy_busy_buffers_size 64k;
        proxy_max_temp_file_size 10m;
        client_max_body_size 100m;
        proxy_set_header    Host $host;
        proxy_set_header    X-Real-IP $remote_addr;
        proxy_set_header    X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header    X-Forwarded-Proto $scheme;
        proxy_read_timeout  600;
        proxy_redirect off;
        proxy_pass http://127.0.0.1:80;
         
    }
}
```
具体参数参考官方文档：[Configuration File | Verdaccio](https://verdaccio.org/docs/configuration/)