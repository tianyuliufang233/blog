---
title: "基础镜像dockerfile"
date: 2023-10-24T10:02:15+08:00
categories:
- 领域
- 容器相关
tags:
- docker
- 案例
keywords:
- tech
#thumbnailImage: //example.com/image.jpg
---

<!--more-->
openjdk8
```
FROM openjdk:8-jdk-bullseye
ENV JAVA_PARAMS="-javaagent:/app/jmx_prometheus_javaagent-0.17.0.jar=30013:/app/tomcat.yml"
run echo 'deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free \n\
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free \n\
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free \n\
deb https://security.debian.org/debian-security bullseye-security main contrib non-free \n\
' > /etc/apt/sources.list
run apt update &&  cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&  echo Asia/Shanghai > /etc/timezone
run apt install -y --no-install-recommends  bzip2 unzip xz-utils ca-certificates curl wget iproute2 && rm -rf /var/lib/apt/lists/*

WORKDIR  /app
ADD ./jmx_prometheus_javaagent-0.17.0.jar .
ADD ./tomcat.yml ./
EXPOSE 30012
```
node:14
```
FROM node:14-alpine
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories && apk update &&  apk add curl tzdata && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&  echo Asia/Shanghai > /etc/timezone && apk del tzdata  && rm -rf /tmp/* /var/cache/apk/*
RUN npm install -g serve
WORKDIR  /app
```
node:16
```
FROM node:16-alpine
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories && apk update &&  apk add curl tzdata && cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&  echo Asia/Shanghai > /etc/timezone && apk del tzdata  && rm -rf /tmp/* /var/cache/apk/*
RUN npm install -g serve
WORKDIR  /app
```
ffmpeg
```
FROM openjdk:8-jdk-bullseye
#ENV JAVA_PARAMS="-javaagent:/app/jmx_prometheus_javaagent-0.17.0.jar=30013:/app/tomcat.yml"
run echo 'deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye main contrib non-free \n\
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-updates main contrib non-free \n\
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bullseye-backports main contrib non-free \n\
deb https://security.debian.org/debian-security bullseye-security main contrib non-free \n\
' > /etc/apt/sources.list
run apt update &&  cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&  echo Asia/Shanghai > /etc/timezone
run apt install -y --no-install-recommends  bzip2 unzip xz-utils ca-certificates curl wget iproute2 ttf-mscorefonts-installer && rm -rf /var/lib/apt/lists/*
 
WORKDIR  /app
ADD ./ffmpeg-6.0-amd64-static /usr/local/ffmpeg-6.0
ENV PATH=$PATH:/usr/local/ffmpeg-6.0
```
安装  ttf-mscorefonts-installer语言包
```
apt-get update && apt-get install --force-yes -y ttf-mscorefonts-installer && rm -rf /var/lib/apt/lists/*
```
world转图片，通过libreoffice转换为pdf，再将pdf转为图片
```
FROM python:3.10
WORKDIR /data/
RUN pwd && ls -al && sed -i 's|deb.debian.org|mirrors.aliyun.com|g' /etc/apt/sources.list.d/debian.sources && apt update -y && openssl version && apt-get install -y gcc fonts-dejavu-core libxinerama-dev libssl-dev libnss3 libdbus-1-3 libcups2  libx11-xcb1
ADD LibreOffice_24.2.5_Linux_x86-64_deb.tar.gz .
RUN cd LibreOffice_24.2.5.2_Linux_x86-64_deb/DEBS/ && find /usr -name libssl3.so && apt-get install -y ./*.deb && libreoffice24.2 --version && ln -s /usr/local/bin/libreoffice24
```


## 构建带jmx_exporter  agent的基础java环境镜像
```
From alpine
ENV JAVA_PARAMS="-javaagent:/app/jmx_prometheus_javaagent-0.17.0.jar=30013:/app/tomcat.yml" RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories && apk update &&  apk add openjdk8 curl busybox tzdata &&  cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&  echo Asia/Shanghai > /etc/timezone &&  apk del tzdata && rm -rf /tmp/* /var/cache/apk/*
WORKDIR  /app
ADD ./jmx_prometheus_javaagent-0.17.0.jar .
ADD ./tomcat.yml ./
EXPOSE 30013
```
构建命令：
```
docker build -f dockerfile  -t harbor.a.com/public/jmx_exporter:v1 .
```
为业务jar包构建镜像：
```
FROM harbor.a.com/public/jmx_exporter:v1
ENV JAVA_OPTS="-Xms512m -Xmx1g -Djava.security.egd=file:/dev/./urandom"
ADD ./a-datacenter-gateway-1.0.1-SNAPSHOT.jar ./
CMD java $JAVA_OPTS $JAVA_PARAMS -jar a-datacenter-gateway-1.0.1-SNAPSHOT.jar
EXPOSE 50000
```
构建命令：
```
docker build -f dockerfile -t name:tag .
```