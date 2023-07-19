---
title: "基础镜像dockerfile"
date: 2023-10-24T10:02:15+08:00
categories:
- 项目
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
